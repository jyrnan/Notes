# 基于DLNA实现iOS，Android投屏：订阅事件通知 | Eliyar's Blog

[https://eliyar.biz/DLNA_with_iOS_Android_Part_3_Subscribe_Event/index.html](https://eliyar.biz/DLNA_with_iOS_Android_Part_3_Subscribe_Event/index.html)

![DLNA_sunscribe_event.png](%E5%9F%BA%E4%BA%8EDLNA%E5%AE%9E%E7%8E%B0iOS%EF%BC%8CAndroid%E6%8A%95%E5%B1%8F%EF%BC%9A%E8%AE%A2%E9%98%85%E4%BA%8B%E4%BB%B6%E9%80%9A%E7%9F%A5%20Eliyar's%20Blog%205a8cb74902a948388da9b73f2f1c181b/DLNA_sunscribe_event.png)

服务运行时，可能改变有些状态信息变量的值，这是需要及时地更新给控制点。因此控制点可以通过订阅操作，让服务通过发送事件消息来发布更新。

事件消息包括一个或多个状态变量以及他们的当前数值。这些消息也是采用 XML 格式，遵循通用事件通知体系 GENA 规定。

服务运行过程中，该服务的 `服务描述文件SDD` 中 `状态变量 <stateVariable>` 发生了变化并且该变量的 `<sendEvents>` 属性为 `yes` 时，将会产生一个事件（Event）消息。如该状态变量的 `<multicast>` 属性为 `yes` ，则该服务把这个事件消息向整个网进行多播（Multicast）。如果为 `no` 或者不存在这个属性，则通过单播（Unicast）给订阅者发送消息。

单播事件消息的订阅及推送是遵循通用事件通知结构（General Event Notification Architecture，GENA）协议。协议中，控制点通常是个订阅者（Subscriber），它向服务提供者（通常是某个设备上的服务）发送订阅消息（SUBSCRIBE），建立订阅关系，然后可以继续更新订阅消息（Renewal），或者最后退订消息（Cancel）。另外，UPnP对GENA进行了一些扩展，如在事件消息中增加了一个key，来表示事件的顺序。

事件订阅和通知过程如下。

# 订阅

事件订阅说白了就是给某个服务的 `订阅 URL<eventSubURL>` 发送一条包含 `回调 URL<Callback URL>` 和 `订阅期限 <duration>` 的订阅请求。

以 `设备描述文档 DDD` 中描述 `AVTransport` 服务的片段例，默认其 `HOST: 192.168.1.243:46201`

| 1234567 | <service>    <serviceType>urn:schemas-upnp-org:service:AVTransport:1</serviceType>    <serviceId>urn:upnp-org:serviceId:AVTransport</serviceId>    <controlURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/action</controlURL>    <eventSubURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event</eventSubURL>    <SCPDURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/desc.xml</SCPDURL></service> |
| --- | --- |

## 订阅请求

上述服务的订阅请求如下，其中注意点就是 `回调URL CALLBACK` 必须带有 `<>` 否则回调不成功。为了接受回调还需要手机上运行一个 `HTTP Server`，具体实现请看下一部分。

| 123456 | SUBSCRIBE /dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event HTTP/1.1HOST: 192.168.1.243:46201USER-AGENT: iOS/9.2.1 UPnP/1.1 SCDLNA/1.0CALLBACK: <http://192.168.1.100:5000/dlna/callback>NT: upnp:eventTIMEOUT: Second-3600    // 订阅期限 |
| --- | --- |

## 订阅响应

### 成功响应

如果订阅成功，则服务 30s 内返回如下的响应。其中 `SID` 为订阅标识符，必须以uuid开头。订阅成功后需要保存，后续续订和取消订阅均需要提供该标识符。此外还需要保存订阅期限 `TIMEOUT: Second-3600`

| 123456 | HTTP/1.1 200 OKServer: Linux/3.10.33 UPnP/1.0 IQIYIDLNA/iqiyidlna/NewDLNA/1.0SID: uuid:f392-a153-571c-e10bContent-Type: text/html; charset="utf-8"TIMEOUT: Second-3600Date: Thu, 03 Mar 2016 19:01:42 GMT |
| --- | --- |

### 订阅失败

若订阅失败，发布者必须返回一个订阅失败响应。格式如下：

| 12345 | HTTP/1.1 error code errordescrioptionServer: OS/Version UPnP/1.1 product/versionSID: uuid:subscibe-UUIDContent-Length: 0Date: Thu, 03 Mar 2016 19:01:42 GMT |
| --- | --- |

## iOS实现

用Swift实现的订阅请求如下

| 12345678910111213141516171819202122232425262728 | func subscribe() {    let url =  "192.168.1.243:46201" +  "/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event"    let request = NSMutableURLRequest(URL: NSURL(string: url)!)    request.HTTPMethod = "SUBSCRIBE"    request.addValue("iOS/9.2.1 UPnP/1.1 SCDLNA/1.0", forHTTPHeaderField: "User-Agent")    // 必须加上<>，不要问我为什么，不然没法订阅成功    request.addValue("<http://192.168.1.100:5000/dlna/callback>", forHTTPHeaderField: "CALLBACK")    request.addValue("upnp:event", forHTTPHeaderField: "NT")    request.addValue("Second-3600", forHTTPHeaderField: "TIMEOUT")    let task = NSURLSession.sharedSession().dataTaskWithRequest(request) { data, response, error in        guard error == nil && data != nil else {            print("error=\(error)")            return        }        // 检查订阅是否失败        if let httpStatus = response as? NSHTTPURLResponse where httpStatus.statusCode != 200 {            print("Subscribe Filed With Error Code:\(httpStatus.statusCode)")            print("response = \(response)")            return        }        // 若订阅成功，则保存SID        if let response = response as? NSHTTPURLResponse {            self.lastSubscribeSID = response.allHeaderFields["SID"] as? String ?? ""        }    }    task.resume()} |
| --- | --- |

# 续订

如果需要续订某个服务，则必须在订阅期限过期前，将续订消息发往服务器进行续订。

## 续订请求

| 1234 | SUBSCRIBE /dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event HTTP/1.1HOST: 192.168.1.243:46201SID: uuid:subscibe-UUIDTIMEOUT: Second-3600    // 订阅期限 |
| --- | --- |

# 取消订阅

不需要在关注特定服务的事件时，需要向服务器发送取消订阅消息。

## 取消订阅请求

| 123 | UNSUBSCRIBE /dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event HTTP/1.1HOST: 192.168.1.243:46201SID: uuid:subscibe-UUID |
| --- | --- |

# 单播事件消息

当服务器上的状态变量发生变数时，通过单播给订阅者发送通知。单播通过 HTTP 协议发送。需要在本地运行一个 `HTTP Server` 来接受请求。接收事件消息成功后，只需要简单返回一个 `HTTP/1.1 200 OK` 作为回应即刻。

**坑：有些设备返回的xml中 `<` `>` 被转义，导致解析时候出错。所以需要先反转义，然后再解析。**

单播消息格式如下

| 123456789101112131415161718 | NOTIFY /dlna/callback HTTP/1.0Host: 192.168.1.100:5000Content-Length: 325Content-Type: text/xml; charset="utf-8"User-Agent: Neptune/1.1.3, 6SID: uuid:ac6dce5a-6047-7862-fd41-e5596960f57a  // 订阅标识符NTS: upnp:propchange                            // GENA规定，必须是 upnp:propchangeNT: upnp:event                                  // GENA规定，必须是 upnp:eventSEQ: 4                                          // 事件编号，初始值为0。<?xml version="1.0" encoding="UTF-8"?><e:propertyset xmlns:e="urn:schemas-upnp-org:event-1-0">    <e:property>        <!-- 消息内容 -->        <variableName>new values</variableName>    </e:property></e:propertyset> |
| --- | --- |

## 播放消息

忽略头部的停止播放消息

| 123456789101112 | <?xml version="1.0" encoding="UTF-8"?><e:propertyset xmlns:e="urn:schemas-upnp-org:event-1-0">    <e:property>        <LastChange>            <Event xmlns="urn:schemas-upnp-org:metadata-1-0/AVT/">                <InstanceID val="0">                    <TransportState val="PLAYING"/>                </InstanceID>            </Event>        </LastChange>    </e:property></e:propertyset> |
| --- | --- |

## 停止播放消息

忽略头部的停止播放消息

| 123456789101112 | <?xml version="1.0" encoding="UTF-8"?><e:propertyset xmlns:e="urn:schemas-upnp-org:event-1-0">    <e:property>        <LastChange>            <Event xmlns="urn:schemas-upnp-org:metadata-1-0/AVT/">                <InstanceID val="0">                    <TransportState val="STOPPED"/>                </InstanceID>            </Event>        </LastChange>    </e:property></e:propertyset> |
| --- | --- |

# iOS实现

iOS实现我用到了一下开源库

1. [GCDWebServer](https://github.com/swisspol/GCDWebServer) - 轻量 iOS/OSX GCD的服务器框架
2. [AEXML](https://github.com/tadija/AEXML) - 轻量 XML 解析库

## 创建 HTTP Server

首先需要利用 [GCDWebServer](https://github.com/swisspol/GCDWebServer) 创建一个 HTTP server 接受事件消息回调。具体代码如下

| 12345678910111213141516171819 | private func startWebServer() {    let webServer = GCDWebServer()      // 为回调消息添加处理回调事件    webServer.addHandlerForMethod("NOTIFY", pathRegex: "/dlna/callback", requestClass: GCDWebServerDataRequest.self) {        (request) -> GCDWebServerResponse! in        // 转换 request 类型为 GCDWebServerDataRequest，然后读取请求 body        if let re = request as? GCDWebServerDataRequest {            if re.hasBody() {                // 如果请求有 body 部分，则开始解析。                self.parseNotifMassage(re.data)            }        }        return GCDWebServerDataResponse(HTML:"<html><body><p>Hello World</p></body></html>")    }    webServer.startWithPort(8899, bonjourName: nil)} |
| --- | --- |

创建 webServer 后，可以通过 `webServer.serverURL` 获取 `serverURL` 。 这时把 `"<\(webServer.serverURL)dlna/callback>"` 作为回调 URL 。按照前文给出代码进行订阅就可以收到事件消息了。

## 解析消息

接收到通知消息后，利用 [GCDWebServer](https://github.com/swisspol/GCDWebServer) 解析 XML，获取具体的动作。目前只对播放状态做了处理。

| 1234567891011121314151617181920212223242526272829303132333435363738394041 | private func parseNotifMassage(data:NSData) {    do {        // 这里有个坑，有些设备返回的xml中<>被转义，导致解析时候出错。所以需要先反转义，然后再解析。        // reTransfer()是我写的简单的 String 扩展，具体看最后        let string = (NSString(data: data, encoding: NSUTF8StringEncoding) as! String).reTransfer()        let xmlData = string.dataUsingEncoding(NSUTF8StringEncoding)!        // 把 XML 转换成        let xml = try AEXMLDocument(xmlData: xmlData)        let status = xml.root["e:property"]["LastChange"]["Event"]["InstanceID"]["TransportState"].attributes        if !status.isEmpty {            switch status.first!.1.uppercaseString {            case "TRANSITIONING":                print("正在传输")            case "PLAYING":                print("播放")            case "PAUSED_PLAYBACK":                print("暂停播放")            case "STOPPED":                print("停止播放")            default :                print("未定义动作 - \(status.first!.1)")            }        } else {            print("未定义XML - \(xml.xmlString)")        }    }    catch {        print(error)        return    }}extension String {    func reTransfer() -> String {        let re1 = self.stringByReplacingOccurrencesOfString("&gt;", withString: ">")        let re2 = re1.stringByReplacingOccurrencesOfString("&lt;", withString: "<")        return re2    }} |
| --- | --- |