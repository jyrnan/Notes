# 基于DLNA实现iOS，Android投屏：SSDP发现设备 | Eliyar's Blog（补充全）

[https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/](https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/)

SSDP能够在局域网能简单地发现设备提供的服务。SSDP有两种发现方式：主动通知和搜索响应方式。

UPnP 技术是架构在 IP 网络之上。因此拥有一个网络中唯一的 IP 地址是 UPnP 设备正常工作的基础。UPnP 设备首先查看网络中是否有 DHCP 服务器，如果有，那么使用 DHCP 分配的 IP 即可；如果没有，则需要使用LLA技术来为自己找适合的IP地址。

另外，在 UPnP 运行过程中，UPnP 设备都需要周期性检测网络中是否有 DHCP 服务器存在，一旦发现有 DHCP 服务器，就必须终止使用 LLA 技术获取的 IP 地址，改用 DHCP 分配的 IP 地址。

## SSDP

SSDP:Simple Sever Discovery Protocol，简单服务发现协议，此协议为网络客户提供一种无需任何配置、管理和维护网络设备服务的机制。此协议采用基于通知和发现路由的多播发现方式实现。协议客户端在保留的多播地址：239.255.255.250：1900（IPV4）发现服务，（IPv6 是：FF0x::C）同时每个设备服务也在此地址上上监听服务发现请求。如果服务监听到的发现请求与此服务相匹配，此服务会使用单播方式响应。

常见的协议请求消息有两种类型，第一种是服务通知，设备和服务使用此类通知消息声明自己存在；第二种是查询请求，协议客户端用此请求查询某种类型的设备和服务。

**iOS中使用GCDAsyncUdpSocket发送和接受SSDP请求、响应及通知，安卓也需要用类此框架来完成**

所以我们发现设备也有两种方法

1. 主动通知方式：当设备加入到网络中，向网络上所有控制点通知它所提供的服务，通知消息采用多播方式。
2. 搜索——响应方式：当一个控制点加入到网络中，在网络搜索它感兴趣的所有设备和服务，搜索消息采用多播方式发送，而设备针对搜索的响应则是使用单播方式发送。

## SSDP 设备类型及服务类型

| 设备类型 | 表示文字 |
| --- | --- |
| UPnP_RootDevice | upnp:rootdevice |
| UPnP_InternetGatewayDevice1 | urn:schemas-upnp-org:device:InternetGatewayDevice:1 |
| UPnP_WANConnectionDevice1 | urn:schemas-upnp-org:device:WANConnectionDevice:1 |
| UPnP_WANDevice1 | urn:schemas-upnp-org:device:WANConnectionDevice:1 |
| UPnP_WANCommonInterfaceConfig1 | urn:schemas-upnp-org:service:WANCommonInterfaceConfig:1 |
| UPnP_WANIPConnection1 | urn:schemas-upnp-org:device:WANConnectionDevice:1 |
| UPnP_Layer3Forwarding1 | urn:schemas-upnp-org:service:WANIPConnection:1 |
| UPnP_WANConnectionDevice1 | urn:schemas-upnp-org:service:Layer3Forwarding:1 |

## 主动通知方式

当设备添加到网络后，定期向（239.255.255.250:1900）发送SSDP通知消息宣告自己的设备和服务。

宣告消息分为 `ssdp:alive(设备可用)` 和 `ssdp:byebye(设备不可用)`

### ssdp:alive 消息

```xml
NOTIFY * HTTP/1.1           // 消息头
NT:                         // 在此消息中，NT头必须为服务的服务类型。（如：upnp:rootdevice）
HOST:                       // 设置为协议保留多播地址和端口，必须是：239.255.255.250:1900（IPv4）或FF0x::C(IPv6
NTS:                        // 表示通知消息的子类型，必须为ssdp:alive
LOCATION:                   // 包含根设备描述得URL地址  device 的webservice路径（如：[http://127.0.0.1:2351/1.xml](http://127.0.0.1:2351/1.xml))
CACHE-CONTROL:              // max-age指定通知消息存活时间，如果超过此时间间隔，控制点可以认为设备不存在 （如：max-age=1800）
SERVER:                     // 包含操作系统名，版本，产品名和产品版本信息( 如：Windows NT/5.0, UPnP/1.0)
USN:                        // 表示不同服务的统一服务名，它提供了一种标识出相同类型服务的能力。如：
// 根/启动设备 uuid:f7001351-cf4f-4edd-b3df-4b04792d0e8a::upnp:rootdevice
// 连接管理器  uuid:f7001351-cf4f-4edd-b3df-4b04792d0e8a::urn:schemas-upnp-org:service:ConnectionManager:1
// 内容管理器 uuid:f7001351-cf4f-4edd-b3df-4b04792d0e8a::urn:schemas-upnp-org:service:ContentDirectory:1
```

### ssdp:byebye 消息

当设备即将从网络中退出时，设备需要对每一个未超期的 `ssdp:alive` 消息多播形式发送 `ssdp:byebye` 消息，其格式如下：

```xml
NOTIFY * HTTP/1.1       // 消息头
HOST:                   // 设置为协议保留多播地址和端口，必须是：239.255.255.250:1900（IPv4）或FF0x::C(IPv6
NTS:                    // 表示通知消息的子类型，必须为ssdp:byebye
USN:                    // 同上
```

## 搜索——响应方式

当控制点，如手机客户端，加入到网络中，可以通过多播搜索消息来寻找网络上感兴趣的设备。我写DLNA模块时候也用主动搜索方式来发现设备。主动搜索可以使用多播方式在整个网络上搜索设备和服务，也可以使用单播方式搜索特定主机上的设备和服务。

### 多播搜索消息

一般情况我们使用多播搜索消息来搜索所有设备即可。多播搜索消息如下：

如果需要实现投屏，则设备类型 `ST` 为 `urn:schemas-upnp-org:service:AVTransport:1`

### 多播搜索响应

多播搜索 `M-SEARCH` 响应与通知消息很类此，只是将NT字段作为ST字段。响应必须以一下格式发送：

```xml
HTTP/1.1 200 OK             // * 消息头
LOCATION:                   // * 包含根设备描述得URL地址  device 的webservice路径（如：[http://127.0.0.1:2351/1.xml](http://127.0.0.1:2351/1.xml))
CACHE-CONTROL:              // * max-age指定通知消息存活时间，如果超过此时间间隔，控制点可以认为设备不存在 （如：max-age=1800）
SERVER:                     // 包含操作系统名，版本，产品名和产品版本信息( 如：Windows NT/5.0, UPnP/1.0)
EXT:                        // 为了符合HTTP协议要求，并未使用。
[BOOTID.UPNP.ORG](http://bootid.upnp.org/):            // 可以不存在，初始值为时间戳，每当设备重启并加入到网络时+1，用于判断设备是否重启。也可以用于区分多宿主设备。
[CONFIGID.UPNP.ORG](http://configid.upnp.org/):          // 可以不存在，由两部分组成的非负十六进制整数，由两部分组成，第一部分代表跟设备和其上的嵌入式设备，第二部分代表这些设备上的服务。
USN:                        // * 表示不同服务的统一服务名
ST:                         // * 服务的服务类型
DATE:                       // 响应生成时间
```

其中主要关注带有 `*` 的部分即可。这里还有一个大坑，**有些设备返回来的字段名称可能包含有小写，如LOCATION和Location，需要做处理。**

此外还需根据LOCATION保存设备的IP和端口地址。

响应例子如下：

```xml
HTTP/1.1 200 OK
Cache-control: max-age=1800
**Usn**: uuid:88024158-a0e8-2dd5-ffff-ffffc7831a22::urn:schemas-upnp-org:service:AVTransport:1
**Location**: [http://192.168.1.243:46201/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/desc.xml](http://192.168.1.243:46201/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/desc.xml)
Server: Linux/3.10.33 UPnP/1.0 Teleal-Cling/1.0
Date: Tue, 01 Mar 2016 08:47:42 GMT+00:00
Ext:
**St**: urn:schemas-upnp-org:service:AVTransport:1
```

控制点发现设备之后仍然对设备知之甚少，仅能知道UPnP类型，UUID和设备描述URL。为了进一步了解设备和服务，需要获取并解析XML描述文件。

描述文件有两种类型：`设备描述文档(DDD)`和`服务描述文档(SDD)`

## 设备描述文档

设备描述文档是对设备的基本信息描述，包括厂商制造商信息、设备信息、设备所包含服务基本信息等。

设备描述采用XML格式，可以通过HTTP GET请求获取。其链接为设备发现消息中的Location。如上述设备的描述文件获取请求为

```xml
GET [http://192.168.1.243:46201/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/desc.xml](http://192.168.1.243:46201/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/desc.xml) HTTP/1.1
HOST: 192.168.1.243:46201
```

设备响应如下

```xml
HTTP/1.1 200 OK
Content-Length    : 3612
Content-type      : text/xml
Date              : Tue, 01 Mar 2016 10:00:36 GMT+00:00

<?xml version="1.0" encoding="UTF-8"?>
<root xmlns="urn:schemas-upnp-org:device-1-0" xmlns:qq="http://www.tencent.com">
    <specVersion>
        <major>1</major>
        <minor>0</minor>
    </specVersion>
    <device>
        <deviceType>urn:schemas-upnp-org:device:MediaRenderer:1</deviceType>
        <UDN>uuid:88024158-a0e8-2dd5-ffff-ffffc7831a22</UDN>
        <friendlyName>客厅的小米盒子</friendlyName>
        <qq:X_QPlay_SoftwareCapability>QPlay:1</qq:X_QPlay_SoftwareCapability>
        <manufacturer>Xiaomi</manufacturer>
        <manufacturerURL>http://www.xiaomi.com/</manufacturerURL>
        <modelDescription>Xiaomi MediaRenderer</modelDescription>
        <modelName>Xiaomi MediaRenderer</modelName>
        <modelNumber>1</modelNumber>
        <modelURL>http://www.xiaomi.com/hezi</modelURL>
        <serialNumber>11262/180303452</serialNumber>
        <presentationURL>device_presentation_page.html</presentationURL>
        <UPC>123456789012</UPC>
        <dlna:X_DLNADOC xmlns:dlna="urn:schemas-dlna-org:device-1-0">DMR-1.50</dlna:X_DLNADOC>
        <dlna:X_DLNACAP xmlns:dlna="urn:schemas-dlna-org:device-1-0">,</dlna:X_DLNACAP>
        <iconList>
            <icon>
                <mimetype>image/png</mimetype>
                <width>128</width>
                <height>128</height>
                <depth>8</depth>
                <url>icon/icon128x128.png</url>
            </icon>
        </iconList>
        <serviceList>
            <service>
                <serviceType>urn:schemas-upnp-org:service:AVTransport:1</serviceType>
                <serviceId>urn:upnp-org:serviceId:AVTransport</serviceId>
                <controlURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/action</controlURL>
                <eventSubURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event</eventSubURL>
                <SCPDURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/desc.xml</SCPDURL>
            </service>
            <service>
                <serviceType>urn:schemas-upnp-org:service:RenderingControl:1</serviceType>
                <serviceId>urn:upnp-org:serviceId:RenderingControl</serviceId>
                <controlURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/RenderingControl/action</controlURL>
                <eventSubURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/RenderingControl/event</eventSubURL>
                <SCPDURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/RenderingControl/desc.xml</SCPDURL>
            </service>
            <service>
                <serviceType>urn:schemas-upnp-org:service:ConnectionManager:1</serviceType>
                <serviceId>urn:upnp-org:serviceId:ConnectionManager</serviceId>
                <controlURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/ConnectionManager/action</controlURL>
                <eventSubURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/ConnectionManager/event</eventSubURL>
                <SCPDURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/ConnectionManager/desc.xml</SCPDURL>
            </service>
            <service>
                <serviceType>urn:mi-com:service:RController:1</serviceType>
                <serviceId>urn:upnp-org:serviceId:RController</serviceId>
                <controlURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/RController/action</controlURL>
                <eventSubURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/RController/event</eventSubURL>
                <SCPDURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/RController/desc.xml</SCPDURL>
            </service>
        </serviceList>
        <av:X_RController_DeviceInfo xmlns:av="urn:mi-com:av">
            <av:X_RController_Version>1.0</av:X_RController_Version>
            <av:X_RController_ServiceList>
                <av:X_RController_Service>
                    <av:X_RController_ServiceType>controller</av:X_RController_ServiceType>
                    <av:X_RController_ActionList_URL>http://192.168.1.243:6095/</av:X_RController_ActionList_URL>
                </av:X_RController_Service>
                <av:X_RController_Service>
                    <av:X_RController_ServiceType>data</av:X_RController_ServiceType>
                    <av:X_RController_ActionList_URL>http://api.tv.duokanbox.com/bolt/3party/</av:X_RController_ActionList_URL>
                </av:X_RController_Service>
            </av:X_RController_ServiceList>
        </av:X_RController_DeviceInfo>
    </device>
</root>
```

其中响应消息体为XML格式的设备描述内容。信息结构比较明确，就不一一介绍了。解析该XML，保存设备的一些基本信息如 `deviceType` 、 `friendlyName` 、 `iconList` 等。之后我们关注该设备提供的服务列表，投屏最关注的服务为 `urn:schemas-upnp-org:service:AVTransport:1`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root xmlns="urn:schemas-upnp-org:device-1-0" xmlns:qq="http://www.tencent.com">
    <device>
        <serviceList>
            <service>
                <serviceType>urn:schemas-upnp-org:service:AVTransport:1</serviceType>
                <serviceId>urn:upnp-org:serviceId:AVTransport</serviceId>
                <controlURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/action</controlURL>
                <eventSubURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/event</eventSubURL>
                <SCPDURL>/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/desc.xml</SCPDURL>
            </service>
        </serviceList>
    </device>
</root>
```

- serviceId : 必有字段。服务表示符，是服务实例的唯一标识。
- serviceType : 必有字段。UPnP服务类型。格式定义与deviceType类此。详看文章开头表格。
- SCPDURL : 必有字段。Service Control Protocol Description URL，获取设备描述文档URL。
- controlURL : 必有字段。向服务发出控制消息的URL，详见 [基于DLNA实现iOS，Android投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)
- eventSubURL : 必有字段。订阅该服务时间的URL，详见 [基于DLNA实现iOS，Android投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)

如只需要实现简单的投屏，则保存`urn:schemas-upnp-org:service:AVTransport:1`服务的上述信息即可。如需要进一步了解该服务，则需要获取并解析服务描述文档。

**坑点1：有些设备 `SCPDURL` 、 `controlURL` 、 `eventSubURL` 开头包含 `/` ，有些设备不包含，拼接URL时需要注意。**

## 服务描述文档

**为了实现简单的投屏和控制（播放、暂停、停止、快进）操作并不需要解析服务描述文件。所有动作均为UPnP规范动作，具体动作请求参见[基于DLNA实现iOS，Android投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)**

服务描述文档是对服务功能的基本说明，包括服务上的动作及参数，还有状态变量和其数据类型、取值范围等。

和设备描述文档一致，服务描述文档也是采用XML语法，并遵守标准UPnP服务schema文件格式要求。获取上述服务SDD语法如下：

```xml
GET [http://192.168.1.243:46201/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/desc.xml](http://192.168.1.243:46201/dev/88024158-a0e8-2dd5-ffff-ffffc7831a22/svc/upnp-org/AVTransport/desc.xml)
HOST: 192.168.1.243:46201
```

设备响应如下：

```xml
HTTP/1.1 200 OK
Content-Length    : 3612
Content-type      : text/xml
Date              : Tue, 01 Mar 2016 10:00:36 GMT+00:00
<!-- 省略了部分动作和状态变量 -->
<?xml version="1.0" encoding="UTF-8"?>
<scpd xmlns="urn:schemas-upnp-org:service-1-0">
    <specVersion>
        <major>1</major>
        <minor>0</minor>
    </specVersion>
    <actionList>
        <action>
            <name>Pause</name>
            <argumentList>
                <argument>
                    <name>InstanceID</name>
                    <direction>in</direction>
                    <relatedStateVariable>A_ARG_TYPE_InstanceID</relatedStateVariable>
                </argument>
            </argumentList>
        </action>
        <action>
            <name>Play</name>
            <argumentList>
                <argument>
                    <name>InstanceID</name>
                    <direction>in</direction>
                    <relatedStateVariable>A_ARG_TYPE_InstanceID</relatedStateVariable>
                </argument>
                <argument>
                    <name>Speed</name>
                    <direction>in</direction>
                    <relatedStateVariable>TransportPlaySpeed</relatedStateVariable>
                </argument>
            </argumentList>
        </action>
        <action>
            <name>Previous</name>
            <argumentList>
                <argument>
                    <name>InstanceID</name>
                    <direction>in</direction>
                    <relatedStateVariable>A_ARG_TYPE_InstanceID</relatedStateVariable>
                </argument>
            </argumentList>
        </action>
        <action>
            <name>SetAVTransportURI</name>
            <argumentList>
                <argument>
                    <name>InstanceID</name>
                    <direction>in</direction>
                    <relatedStateVariable>A_ARG_TYPE_InstanceID</relatedStateVariable>
                </argument>
                <argument>
                    <name>CurrentURI</name>
                    <direction>in</direction>
                    <relatedStateVariable>AVTransportURI</relatedStateVariable>
                </argument>
                <argument>
                    <name>CurrentURIMetaData</name>
                    <direction>in</direction>
                    <relatedStateVariable>AVTransportURIMetaData</relatedStateVariable>
                </argument>
            </argumentList>
        </action>
        ...
    </actionList>
    <serviceStateTable>
        <stateVariable sendEvents="no">
            <name>CurrentTrackURI</name>
            <dataType>string</dataType>
        </stateVariable>
        <stateVariable sendEvents="no">
            <name>CurrentMediaDuration</name>
            <dataType>string</dataType>
        </stateVariable>
        <stateVariable sendEvents="no">
            <name>AbsoluteCounterPosition</name>
            <dataType>i4</dataType>
        </stateVariable>
        <stateVariable sendEvents="no">
            <name>RelativeCounterPosition</name>
            <dataType>i4</dataType>
        </stateVariable>
        <stateVariable sendEvents="no">
            <name>A_ARG_TYPE_InstanceID</name>
            <dataType>ui4</dataType>
        </stateVariable>
        ...
    </serviceStateTable>
</scpd>
```

- actionList 目前服务上所包含的动作列表。
- actionList 目前服务上所包含的状态变量。

以Pause动作为例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<scpd xmlns="urn:schemas-upnp-org:service-1-0">
    <actionList>
        <action>
            <!-- 动作名称 -->
            <name>Pause</name>
            <!-- 参数列表 -->
            <argumentList>
                <argument>
                    <!-- 参数名称 -->
                    <name>InstanceID</name>
                    <!-- 输出或输出-->
                    <direction>in</direction>
                    <!-- 声明参数有关的状态变量 -->
                    <relatedStateVariable>A_ARG_TYPE_InstanceID</relatedStateVariable>
                </argument>
            </argumentList>
        </action>
        ...
    </actionList>
    <serviceStateTable>
    <!-- 状态变量 -->
        <stateVariable>
            <!-- 是否发送事件消息，如果为yes则该状态变量发生变化时生成事件消息。 -->
            <stateVariable sendEvents="no">
            <!-- 状态变量名称 -->
            <name>A_ARG_TYPE_InstanceID</name>
            <!-- 状态数据类型 -->
            <dataType>ui4</dataType>
        </stateVariable>
        ...
    </serviceStateTable>
</scpd>
```

**为了实现简单的投屏和控制（播放、暂停、停止、快进）操作并不需要解析服务描述文件。所有动作均为UPnP规范动作，具体动作请求参见[基于DLNA实现iOS，Android投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)**