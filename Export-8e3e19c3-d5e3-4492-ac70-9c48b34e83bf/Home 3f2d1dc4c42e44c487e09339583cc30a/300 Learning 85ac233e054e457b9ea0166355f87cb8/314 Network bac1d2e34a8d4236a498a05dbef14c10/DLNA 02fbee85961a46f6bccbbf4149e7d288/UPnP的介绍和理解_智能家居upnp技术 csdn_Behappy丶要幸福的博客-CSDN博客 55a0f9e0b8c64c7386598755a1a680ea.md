# UPnP的介绍和理解_智能家居upnp技术 csdn_Behappy丶要幸福的博客-CSDN博客

[https://blog.csdn.net/be_happy_mr_li/article/details/52919759](https://blog.csdn.net/be_happy_mr_li/article/details/52919759)

## 前言

做android智能硬件开发一年，蓝牙接触多的就是spp模拟串口通信，而更多的是upnp，因为大部分的项目都是基于cling库的wifi方案的项目。设备的wifi方案相对于蓝牙方案，传输速度快，覆盖范围广，能够脱离设备独立联网，协议规范简单明了，但价格相对要高一些。

```
cling库地址：
http://4thline.org/projects/cling/12
```

## UPnP简介

upnp是 universal plug and play，即：即插即用设备，可以当作是一个相对复杂的网络协议，毕竟它包含了很多其他的网络协议，如：ip（设备寻址），tcp、udp（数据打包发送）、http（数据传递格式）等。

upnp可以扩展，也就是说你还可以在启动加入其他的协议，比如：传递数据时，http协议再包一层json协议，或者数据传递使用xml协议来传递等等。

upnp之所以强大，感觉很大一个原因是基于互联网，这样对等设备可以通过互联网自由交互，也就是说任何可以联网的设备都可以使用upnp协议。

## UPnP工作步骤

有兴趣的可以去看看这个upnp文档：

```
英文版ppt介绍：
http://101.96.10.63/trinea.github.io/doc/upnp/UPnP_UDA_tutorial_July2014.pdf

中文版文档介绍：
http://read.pudn.com/downloads37/doc/comm/125876/UDA1.0-ChinesePDF.pdf12345
```

工作步骤其实只是一个工作的逻辑，大概的按功能划分为6个步骤，如下图1所示：

[UPnP%E7%9A%84%E4%BB%8B%E7%BB%8D%E5%92%8C%E7%90%86%E8%A7%A3_%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85upnp%E6%8A%80%E6%9C%AF%20csdn_Behappy%E4%B8%B6%E8%A6%81%E5%B9%B8%E7%A6%8F%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%2055a0f9e0b8c64c7386598755a1a680ea/20161107154604158](UPnP%E7%9A%84%E4%BB%8B%E7%BB%8D%E5%92%8C%E7%90%86%E8%A7%A3_%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85upnp%E6%8A%80%E6%9C%AF%20csdn_Behappy%E4%B8%B6%E8%A6%81%E5%B9%B8%E7%A6%8F%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%2055a0f9e0b8c64c7386598755a1a680ea/20161107154604158)

工作流程如下图2所示：

[UPnP%E7%9A%84%E4%BB%8B%E7%BB%8D%E5%92%8C%E7%90%86%E8%A7%A3_%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85upnp%E6%8A%80%E6%9C%AF%20csdn_Behappy%E4%B8%B6%E8%A6%81%E5%B9%B8%E7%A6%8F%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%2055a0f9e0b8c64c7386598755a1a680ea/20161108113327404](UPnP%E7%9A%84%E4%BB%8B%E7%BB%8D%E5%92%8C%E7%90%86%E8%A7%A3_%E6%99%BA%E8%83%BD%E5%AE%B6%E5%B1%85upnp%E6%8A%80%E6%9C%AF%20csdn_Behappy%E4%B8%B6%E8%A6%81%E5%B9%B8%E7%A6%8F%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%2055a0f9e0b8c64c7386598755a1a680ea/20161108113327404)

### 0、寻址-Addressing

既然基于互联网，那么基本的p2p协议需要满足，所以寻址算是upnp的基础。

所以就必须要有寻址，寻址的过程就是设备获取ip地址的过程，这个里面一般都使用DHCP（Dynamic Host Configuration Protocol，动态主机配置协议），即路由动态的分配一个没有使用的ip地址给设备。

### 1、发现-Discovery

既然来到了互联网的大世界，通过寻址有了身份标识ip，闲的无聊那么必然就要找个小伙伴交流沟通，当然最好是妹子，所以这时候就有了一个发现的概念。

首先你要先自我介绍，也就是网络中的控制点介绍自己（是什么、能干什么），因为不知道当前世界有多少个控制点，所以一般通过喊话的形式来介绍自己。也就是说新加入的设备以多播的形式广播自己的信息。

当然如果作为一个控制点，也有权限搜索自己感兴趣的设备。搜的时候控制点自己不知道有没有自己感兴趣的设备。同样这也是通过多播的形式，当世界的另一端乖妹子听到了这个信息，那么就会以单独的形式应答，这就是单播响应。`就相当于面试一样，公司要找一个美工妹子，发布一个广告，添加要求福利待遇，然后留个邮箱，感兴趣的美工妹子们就会发简历到这个邮箱`。人可以喊，公司可以登招聘广告，但是设备不可以。那么设备是怎么做到的，设备是依托于ssdp（Simple Service Discovery Protocol）简单服务发现协议，这个协议类似定义了怎么喊，喊的格式，怎么登招聘广告，广告的格式是怎么样的。如下所示：

- 设备介绍自己（喊话介绍自己格式）

```
NOTIFY * HTTP/1.1 (标准的http1.1协议)
HOST: 239.255.255.250:1900（路由固定的多播地址和端口 互联网编号分配组织很强势的定义好了的）
CACHE-CONTROL: max-age = seconds until advertisement expires 指代广播有效持续时间
LOCATION: URL for UPnP description for root device
NT: search target 设备类型
NTS: ssdp:alive
USN: advertisement UUID 复合标识符1234567
```

其中特别说明的是LOCATION，LOCATION是关于设备更多信息的 URL 地址（或有关服务的内含设备），这也就是描叙设备信息的一个文档地址，后面会用到。

- 控制点搜索（公司发布招聘广告格式）

```
M-SEARCH * HTTP/1.1
HOST: 239.255.255.250:1900（路由固定的多播地址和端口 互联网编号分配组织很强势的定义好了的）
MAN: "ssdp:discover"
MX: seconds to delay response
ST: search target12345
```

`这里需要注意的是ssdp其实包含两个简单协议，即：HTTPMU、HTTPU。HTTPMU（Http Multicast UDP）是http协议的变种，即通过udp的方式组播发送http格式的内容，HTTPU即通过udp的方式发送单播给host。`

### 2、描述-Description

既然有了寻找小伙伴的方式，那么怎么挑选小伙伴呢？发现直接能得到的只有一些简单的标识信息，如设备（或服务）的 UPnP 类型、设备的全球唯一标识符和设备 UPnP 描述的 URL 地址。而要了解一个小伙伴当然是不够的，就相当于`招聘光靠简历个人简介是完全不够了，还需要了解他的长相身材，工作能力以及性格价值观等`。

所以就需要获取设备描述，那么怎么获取呢？对的，怎么获取，其实在设备发现的时候就已经把设备描述带出来了，只是封装在一个url中，也就是LOCATION中的描述url，通过location中的url能获取一个xml，一般设备的描述都是通过xml来标识的。

设备描述一般包含两个部分：描述所包含的物理与逻辑设备、一个或多个服务描述（描述设备对外暴露的能力）。也就相当于`物体除了基本的外观名称以外，还包括他能干什么`。设备描述包括特定厂商、制造商信息，如模块名称和编号、序列号、制造商名称、特定厂商网站 URL 等（详细信息如下）。对于设备中的每种服务，设备描述都包含服务类型、名称、服务描述 URL、控制 URL 以及事件 URL。设备描述还包含所有嵌入式设备描述与 URL 地址集。

如下所示：