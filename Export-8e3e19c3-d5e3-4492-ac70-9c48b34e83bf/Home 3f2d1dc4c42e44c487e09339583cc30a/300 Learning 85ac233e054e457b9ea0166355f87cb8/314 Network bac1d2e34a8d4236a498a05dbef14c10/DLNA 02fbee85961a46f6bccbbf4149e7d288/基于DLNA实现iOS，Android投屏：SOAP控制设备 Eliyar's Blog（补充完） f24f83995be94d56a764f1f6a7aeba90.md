# 基于DLNA实现iOS，Android投屏：SOAP控制设备 | Eliyar's Blog（补充完）

[https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)

UPdP网络中，控制点和服务之间使用简单对象访问协议（Simple Object Access Protocol，SOAP）

根据[基于DLNA实现iOS，Android投屏：SSDP发现设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/)收到设备描述文档（DDD）和服务描述文档（SDD），通过解析DDD获取 `<controlURL>` 控制点可以知道该设备上某个服务的控制点地址。再通过解析 DDD 中 `<action>` 中的 `<name>` 和 `<argumentList>` 获取该服务动作的动作名称，参数要求。控制点向 `controlURL` 发出服务调用信息，表明动作名称和相应参数来调用相应的服务。

# ****SOAP简单对象访问协议****

**控制点和服务之间使用简单对象访问协议（Simple Object Access Protocol，SOAP）的格式。**SOAP 的底层协议一般也是HTTP。在 UPnP 中，把 SOAP 控制/响应信息分成 3 种： 

- UPnP Action Request、
- UPnP Action Response-Success
- UPnP Action Response-Error。

SOAP 和 SSDP 不一样，所使用的 HTTP 消息是有 Body 内容，Body 部分可以写想要调用的动作，叫做 Action invocation，可能还要传递参数，如想播放一个网络上的视频，就要把视频的URL传过去；服务收到后要 response ，回答能不能执行调用，如果出错则返回一个错误代码。

参考：

[SOAP 教程 | 菜鸟教程](https://www.runoob.com/soap/soap-tutorial.html)

## 动作调用（UPnP Action Request）

使用POST方法发送控制消息的格式如下

```xml
POST <control URL> HTTP/1.0
Host: hostname:portNumber
Content-Lenght: byte in body
Content-Type: text/xml; charset="utf-8"
SOAPACTION: "urn:schemas-upnp-org:service:serviceType:v#actionName"

<!--必有字段-->
<?xml version="1.0" encoding="utf-8"?>
<!--SOAP必有字段-->
<s:Envelope s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
    <s:Body>
        <!--Body内部分根据不同动作不同-->
        <!--动作名称-->
        <u:actionName xmlns:u="urn:schemas-upnp-org:service:serviceType:v">
            <!--输入参数名称和值-->
            <argumentName>in arg values</argumentName>
             <!--若有多个参数则需要提供-->
        </u:actionName>
    </s:Body>
</s:Envelope>
```

- control URL: [基于DLNA实现iOS，Android投屏：SSDP发现设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/) 中提到的 `设备描述文件` 中 `urn:upnp-org:serviceId:AVTransport` 服务的 `<controlURL>`
- HOST： 上述服务器的根地址和端口号。
- actionName： 需要调用动作的名称，对应相应服务的 `服务描述文件<SCPDURL>` 中的 `<action>` 的 `<name>` 字段。
- argumentName： 输入参数名称，对应相应服务的 `服务描述文件<SCPDURL>` 中的 `<action>` `<argument>` `<name>` 字段。
- in arg values： 输入参数值，具体的可以通过 ，可以通过 `服务描述文件<SCPDURL>` `<action>` `<relatedStateVariable>` 提到的状态变量来得知值得类型。
- urn:schemas-upnp-org:service:serviceType:v：对应该 `设备描述文件` 相应服务的 `<serviceType` 字段。

## 动作响应（UPnP Action Response-Succes）

收到控制点发来的动作调用请求后，设备上的服务必须执行动作调用。，并在 30s 内响应。如果需要超过 30s 才能完成执行的动作，则可以先返回一个应答消息，等动作执行完成再利用事件机制返回动作响应。

```xml
HTTP/1.0 200 OK                             // 响应成功响应头
Content-Type: text/xml; charset="utf-8"
Date: Tue, 01 Mar 2016 10:00:36 GMT+00:00
Content-Length: byte in body

<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <!--之前部分为固定字段-->
        <!--之前部分为固定字段-->
        <u:actionNameResponse xmlns:u="urn:schemas-upnp-org:service:serviceType:v">
            <!--输出变量名称和值-->
            <arugumentName>out arg value</arugumentName>
            <!--若有多个输出变量则继续写，没有可以不存在输出变量-->
        </u:actionNameResponse>
    </s:Body>
</s:Envelope>
```

- actionNameResponse： 响应的动作名称
- arugumentName： 当动作带有输出变量时必选，输出变量名称
- out arg values： 输出变量名称值

## 动作错误响应（UPnP Action Response-Succes）

如果处理动作过程中出现错误，则返回一个一下格式的错误响应。

```xml
HTTP/1.0 500 Internal Server Error          // 响应成功响应头
Content-Type: text/xml; charset="utf-8"
Date: Tue, 01 Mar 2016 10:00:36 GMT+00:00
Content-Length: byte in body

<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:Fault>
            <!--之前部分为固定字段-->
            <faultcode>s:Client</faultcode>
            <faultstring>UPnPError</faultstring>
            <detail>
                <UPnPError xmlns="urn:schemas-upnp-org:control-1-0">
                    <errorCode>402</errorCode>
                    <errorDescription>Invalid or Missing Args</errorDescription>
                </UPnPError>
            </detail>
        </u:actionNameResponse>
    </s:Body>
</s:Envelope>
```

- faultcode： SOAP规定使用元素，调用动作遇到的错误类型，一般为s:Client。
- faultstring： SOAP规定使用元素，值必须为 UPnPError。
- detail： SOAP规定使用元素，错误的详细描述信息。
- UPnPError： UPnP规定元素。
- errorCode： UPnP规定元素，整数。详见下表。
- errorDescription： UPnP规定元素，简短错误描述。
- 

| errorCode | errorDescription | 描述 |
| --- | --- | --- |
| 401 | Invalid Action | 这个服务中没有该名称的动作 |
| 402 | Invalid Args | 参数数据错误 not enough in args, too many in arg, no in arg by that name, one or more in args 之一 |
| 403 | Out of Sycs | 不同步 |
| 501 | Action Failed | 可能在当前服务状态下返回，以避免调用此动作 |
| 600 ~ 699 | TBD | 一般动作错误，由 UPnP 论坛技术委员会定义 |
| 700 ~ 799 | TBD | 面向标准动作的特定错误，由 UPnP 论坛工作委员会定义 |
| 800 ~ 899 | TBD | 面向非标准动作的特定错误，由 UPnP 厂商会定义 |

# ****投屏基本命令及其响应****

所有命令以发向 [基于DLNA实现iOS，Android投屏：SSDP发现设备](https://eliyar.biz/code/iOS/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/) 发现的设备。除了网址以外，其余部分均不需要修改。

所有动作请求使用 `POST` 请求发送，并且请求Header均如下所示，其中：

- control URL: [基于DLNA实现iOS，Android投屏：SSDP发现设备](https://eliyar.biz/code/iOS/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/) 中提到的 `设备描述文件` 中 `urn:upnp-org:serviceId:AVTransport` 服务的 `<controlURL>`。
- HOST： 上述服务器的根地址和端口号。
- urn:schemas-upnp-org:service:serviceType:v：对应相应设备的 `设备描述文件` 相应服务的 `<serviceType` 字段。
- actionName： 需要调用动作的名称，对应相应服务的 `服务描述文件<SCPDURL>` 中的 `<action>` 的 `<name>` 字段。
- 

```html
POST /dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/action HTTP/1.0
Host: 192.168.1.243:46201
Content-Length: byte in body
Content-Type: text/xml; charset="utf-8"
SOAPACTION: "urn:schemas-upnp-org:service:serviceType:v#actionName"
```

**下面请求和响应均忽略Header，参数列表中列出Header的SOAPACTION值**

## 设置播放资源URI

### 动作请求

设置当前播放视频动作统一名称为 `SetAVTransportURI` 。 需要传递参数有

- InstanceID：设置当前播放时期时为 0 即可。
- CurrentURI： 播放资源URI
- CurrentURIMetaData： 媒体meta数据，可以为空
    
    <aside>
    💡 这个媒体数据是可以添加的，用来区分投屏的是视频还是图片，或者更多。可以拖过添加<TIDL-Lite>标签来区分。
    所以在下面这个项目中作了区分。
    
    [DLNA(一)](https://www.jianshu.com/p/afecbcf2c496)
    
    还能碰到这样的提示：通过添加TIDL标签解决报错
    
    [DLNA投屏失败时添加<DIDL-Lite>标签](https://www.jianshu.com/p/6771f2852991)
    
    </aside>
    
- Header_SOAPACTION： “urn:upnp-org:serviceId:AVTransport#SetAVTransportURI”

**有些设备传递播放URI后就能直接播放，有些设备设置URI后需要发送播放命令，可以在接收到 `SetAVTransportURIResponse` 响应后调用播放动作来解决。**

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
    <s:Body>
        <u:SetAVTransportURI xmlns:u="urn:schemas-upnp-org:service:AVTransport:1">
            <InstanceID>0</InstanceID>
            <CurrentURI>http://125.39.35.130/mp4files/4100000003406F25/clips.vorwaerts-gmbh.de/big_buck_bunny.mp4</CurrentURI>
            <CurrentURIMetaData />
        </u:SetAVTransportURI>
    </s:Body>
</s:Envelope>
```

### 响应

```xml
<?xml version="1.0" encoding="UTF-8"?>
<s:Envelope s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
    <s:Body>
        <u:SetAVTransportURIResponse xmlns:u="urn:schemas-upnp-org:service:AVTransport:1"/>
    </s:Body>
</s:Envelope>
```

## 播放

### 动作请求

播放视频动作统一名称为 `Play` 。 需要传递参数有

- InstanceID：设置当前播放时期时为 0 即可。
- Speed：播放速度，默认传 1 。
- Header_SOAPACTION： “urn:upnp-org:serviceId:AVTransport#Pause”

```xml
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:Play xmlns:u="urn:schemas-upnp-org:service:AVTransport:1">
            <InstanceID>0</InstanceID>
            <Speed>1</Speed>
        </u:Play>
    </s:Body>
</s:Envelope>
```

### 响应

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:PlayResponse xmlns:u="urn:schemas-upnp-org:service:AVTransport:1" />
    </s:Body>
</s:Envelope>
```

## 暂停

### 动作请求

暂停视频动作统一名称为 `Pause` 。 需要传递参数有

- InstanceID：设置当前播放时期时为 0 即可。
- Header_SOAPACTION： “urn:upnp-org:serviceId:AVTransport#Pause”

```xml
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:Play xmlns:u="urn:schemas-upnp-org:service:AVTransport:1">
            <InstanceID>0</InstanceID>
            <Speed>1</Speed>
        </u:Play>
    </s:Body>
</s:Envelope>
```

### 响应

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:PlayResponse xmlns:u="urn:schemas-upnp-org:service:AVTransport:1" />
    </s:Body>
</s:Envelope>
```

## 获取播放进度

### 动作请求

获取播放进度动作统一名称为 `GetPositionInfo` 。 需要传递参数有

- InstanceID：设置当前播放时期时为 0 即可。
- MediaDuration： 可以为空。
- Header_SOAPACTION： “urn:upnp-org:serviceId:AVTransport#MediaDuration”

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:GetPositionInfo xmlns:u="urn:schemas-upnp-org:service:AVTransport:1">
            <InstanceID>0</InstanceID>
            <MediaDuration />
        </u:GetPositionInfo>
    </s:Body>
</s:Envelope>
```

### 响应

获取播放进度响应中包含了比较多的信息，其中我们主要关心的有一下三个：

- TrackDuration： 目前播放视频时长
- RelTime： 真实播放时长
- AbsTime： 相对播放时长

注：目前为止还没发现 `RelTime` `AbsTime` 和不一样的情况，选用 `RelTime` 就ok。

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:GetPositionInfoResponse xmlns:u="urn:schemas-upnp-org:service:AVTransport:1">
            <Track>0</Track>
            <TrackDuration>00:04:32</TrackDuration>
            <TrackMetaData />
            <TrackURI />
            <RelTime>00:00:07</RelTime>
            <AbsTime>00:00:07</AbsTime>
            <RelCount>2147483647</RelCount>
            <AbsCount>2147483647</AbsCount>
        </u:GetPositionInfoResponse>
    </s:Body>
</s:Envelope>
```

## 跳转至特定进度或视频

### 动作请求

跳转到特定的进度或者特定的视频（多个视频播放情况），需要调用 `Seek` 动作，传递参数有：

- InstanceID: 一般为 0 。
- Unit：REL_TIME（跳转到某个进度）或 TRACK_NR（跳转到某个视频）。
- Target： 目标值，可以是 00:02:21 格式的进度或者整数的 TRACK_NR。
- Header_SOAPACTION： “urn:upnp-org:serviceId:AVTransport#Seek”

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:Seek xmlns:u="urn:schemas-upnp-org:service:AVTransport:1">
            <InstanceID>0</InstanceID>
            <Unit>REL_TIME</Unit>
            <Target>00:02:21</Target>
        </u:Seek>
    </s:Body>
</s:Envelope>
```

### 响应

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <s:Body>
        <u:SeekResponse xmlns:u="urn:schemas-upnp-org:service:AVTransport:1" />
    </s:Body>
</s:Envelope>
```

需要用到库

1. [AEXML](https://github.com/tadija/AEXML) - 轻量 XML 库，用于构造和解析XML

## 构造动作XML

首先利用 [AEXML](https://github.com/tadija/AEXML) 构造动作 XML 部分。由于所有动作结构相似，写了个构造方法

```swift
private func prepareXMLFileWithCommand(command:AEXMLElement) -> String {
    // 创建 AEXMLDocument 实例
    let soapRequest = AEXMLDocument()
    // 设置XML外层
    let attributes = [
        "xmlns:s" : "http://schemas.xmlsoap.org/soap/envelope/","s:encodingStyle" : "http://schemas.xmlsoap.org/soap/encoding/"]
    let envelope = soapRequest.addChild(name: "s:Envelope", attributes: attributes)
    let body = envelope.addChild(name: "s:Body")

    // 把 command 添加到 XML 中间
    body.addChild(command)
    return soapRequest.xmlString
}
```

根据不同动作构造 XML ，比如 `传递URI` 和 `播放动作`

```swift
/**
投屏

- parameter URI: 视频URL
*/
func SetAVTransportURI(URI:String) {
    let command = AEXMLElement("u:SetAVTransportURI",attributes: ["xmlns:u" : "urn:schemas-upnp-org:service:AVTransport:1"])
    command.addChild(name: "InstanceID", value: "0")
    command.addChild(name: "CurrentURI", value: URI)
    command.addChild(name: "CurrentURIMetaData")
    let xml = self.prepareXMLFileWithCommand(command)

    self.sendRequestWithData(xml,action: "SetAVTransportURI")
}

/**
播放视频
*/
func Play() {
    let command = AEXMLElement("u:Play",attributes: ["xmlns:u" : "urn:schemas-upnp-org:service:AVTransport:1"])
    command.addChild(name: "InstanceID", value: "0")
    command.addChild(name: "Speed", value: "1")
    let xml = self.prepareXMLFileWithCommand(command)

    self.sendRequestWithData(xml,action: "Play")
}
```

## 发送动作请求

```swift
private func sendRequestWithData(xml:String, action:String) {
    let request = NSMutableURLRequest(URL: NSURL(string: controlURL)!)
    // 使用 POST 请求发送动作
    request.HTTPMethod = "POST"
    request.addValue("text/xml", forHTTPHeaderField: "Content-Type")
    // 添加SOAPAction动作名称
    request.addValue("\(service.serviceId)#\(action)", forHTTPHeaderField: "SOAPAction")
    request.HTTPBody = xml.dataUsingEncoding(NSUTF8StringEncoding)

    let task = NSURLSession.sharedSession().dataTaskWithRequest(request) { data, response, error in
        guard error == nil && data != nil else {
            print("error=\(error)")
            return
        }

        // 检查是否正确响应
        if let httpStatus = response as? NSHTTPURLResponse where httpStatus.statusCode != 200 {
            print("statusCode should be 200, but is \(httpStatus.statusCode)")
            print("response = \(NSString(data: data!, encoding: NSUTF8StringEncoding)))")
        }

        // 解析响应
        self.parseRequestResponseData(data!)

    }
    task.resume()
}
```

## 解析响应

解析请求响应

```swift
private func parseRequestResponseData(data:NSData) {
    do {
        let xmlDoc = try AEXMLDocument(xmlData: data)

        if let response = xmlDoc.root["s:Body"].first?.children.first {
            switch response.name {
            case "u:SetAVTransportURIResponse":
                print("设置URI成功")
                //获取播放长度
            case "u:GetPositionInfoResponse":
                // 进度需要进一步解析。如realTime = response["RelTime"].value
                print("已获取播放进度")
            case "u:PlayResponse":
                print("已播放")
            case "u:PauseResponse":
                print("已暂停")
            case "u:StopResponse":
                print("已停止")
            default :
                print("未定义响应  - \(xmlDoc.xmlString)")
            }
        } else {
            print("返回不符合规范 - XML:\(xmlDoc.xmlString)")
        }
    }
    catch {
        return
    }
}
```