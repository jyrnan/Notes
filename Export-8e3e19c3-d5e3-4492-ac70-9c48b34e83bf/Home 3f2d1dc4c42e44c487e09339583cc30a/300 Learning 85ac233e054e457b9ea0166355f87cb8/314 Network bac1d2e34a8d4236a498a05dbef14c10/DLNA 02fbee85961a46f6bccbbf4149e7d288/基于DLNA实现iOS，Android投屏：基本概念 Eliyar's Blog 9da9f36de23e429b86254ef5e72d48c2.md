# 基于DLNA实现iOS，Android投屏：基本概念 | Eliyar's Blog

[https://eliyar.biz/DLNA_with_iOS_Android/](https://eliyar.biz/DLNA_with_iOS_Android/)

![UPnP1.jpg](%E5%9F%BA%E4%BA%8EDLNA%E5%AE%9E%E7%8E%B0iOS%EF%BC%8CAndroid%E6%8A%95%E5%B1%8F%EF%BC%9A%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%20Eliyar's%20Blog%209da9f36de23e429b86254ef5e72d48c2/UPnP1.jpg)

由于我司需求，需要在iOS和安卓客户端实现DLNA投屏和控制。经过一番折腾，决定由我来研究DLNA。说起来又兴奋又紧张，兴奋希望自己能够弄出来然后跟安卓组讲解原理，紧张是因为怕自己能力不足做不出来。

DLNA网上的资料比较笼统不好入门，官方资料直接是每个1000多页的10几个PDF文档，根本无从下手。相关开源项目有名的有[Platinum UPnP](http://www.plutinosoft.com/platinum/)，但是由于它是基于C++实现的，相关文档并不全面。iOS相关开源项目都三四年没更新的，找来找去只好自己去啃自己去实现了。还好买到一本不错的书《智能家庭网络：技术、标准与应用实现》。通过近俩星期的研究，搞懂了DLNA核心协议UPnP基本逻辑，实现了投屏和控制功能的Demo。

下面就整理一下实现基本概念，实现过程和一些坑。

如果要直接看实现过程，请看以下三篇文章

- [基于DLNA实现iOS，Android投屏：SSDP发现设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/)
- [基于DLNA实现iOS，Android投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)
- [基于DLNA实现iOS，Android投屏：订阅事件通知](https://eliyar.biz/DLNA_with_iOS_Android_Part_3_Subscribe_Event/)

# 基础概念

## DLNA

DLNA的全称是DIGITAL LIVING NETWORK ALLIANCE(数字生活网络联盟)， 其宗旨是Enjoy your music, photos and videos, anywhere anytime， DLNA(Digital Living Network Alliance) 由索尼、英特尔、微软等发起成立、旨在解决个人PC，消费电器，移动设备在内的无线网络和有线网络的互联互通，使得数字媒体和内容服务的无限制的共享和增长成为可能，目前成员公司已达280多家。

DLNA标准包括多项协议及标准，其中最重要的部分是UPnP。对于我们目前的需求UPnP就能满足全部要求。

## UPnP

通用即插即用（英语：Universal Plug and Play，简称UPnP）是由“通用即插即用论坛”（UPnP™ Forum）推广的一套网络协议。该协议的目标是使家庭网络（数据共享、通信和娱乐）和公司网络中的各种设备能够相互无缝连接，并简化相关网络的实现。UPnP通过定义和发布基于开放、因特网通讯网协议标准的UPnP设备控制协议来实现这一目标。

UPnP这个概念是从即插即用（Plug-and-play）派生而来的，即插即用是一种热拔插技术。

### 协议栈

UPnP设备体系结构包含了设备之间、控制点之间、设备和控制点之间的通信。完整的UPnP由设备寻址、设备发现、设备描述、设备控制、事件通知和基于Html的描述界面几部分构成。

1. UPnP是一个多层协议构成的框架体系，每一层都以相邻的下层为基础，同时又是相邻上层的基础。直至达到应用层为止。该图中的最下面是就是IP和TCP，共两层，负责设备的IP地址。
2. 三层是HTTP、HTTPU、HTTPMU，这一层，属于传送协议层。传送的是内容都经过“封装”后，存放在特定的XML文件中的。对应的SSDP、GENA、SOAP指的是保存在XML文件中的数据格式。到这一层，已经解决了UPnP设备的IP地址和传送信息问题。
3. 第四层是UPnP设备体系定义，仅仅是一个抽象的、公用的设备模型。任何UPnP设备都必须使用这一层。
4. 第五层是UPnP论坛的各个专业委员会的设备定义层，在这个论坛中，不同电器设备由不同的专业委员会定义，例如：电视委员会只负责定义网络电视设备部分，空调器委员会只负责定义网络空调设备部分，依此类推。所有的不同类型的设备都被定义成一个专门的架构或者模板，供建立设备的时候使用。可以推知，进入这一层，设备已经被指定了明确用途。当然，这些都必须遵守标准化的规范。从目前看，UPnP已经可以支持大部分的设备：从电脑、电脑外设，移动设备和家用消费类电子设备等等，无所不包，随着这个体系的普及，将可能有更多的厂家承认这一标准，最终，可能演化为公认的行业标准。
5. 最上层，也就是应用层，由UPnP设备制造厂商定义的部分。这一层的信息是由设备制造厂商来“填充” 的，这部分一般有设备厂商提供的、对设备控制和操作的底层代码，然后，就是名称序列号呀，厂商信息之类的东西。

### 设备

设备是提供服务的网路实体，是一个逻辑概念，一个屋里设备可以包含一个或者多个逻辑设备。例如一台PC可以有两个逻辑设备———视频播放器和图片浏览器。

### 服务

服务是UPnP中最小的可控单元，它包括一系列可控制而动作和一组记录该服务目前情况的状态。服务是依赖于设备存在的。

### 控制点

控制UPnP设备工作的网络终端，主要功能包括获取设备描述和相关服务列表；获取感兴趣的服务描述；发出控制消息控制设备动作；向感兴趣的服务发出订阅消息，以便当服务状态改变时，自动获得时间通知。

### 一些术语

### UUID

UUID含义是通用唯一识别码（Universally Unique Identifier），其目的是让分布式系统中的所有元素，都有唯一的辨识资讯，而不需要透过中央控制端来做辨识资讯的指定。其格式为xxxxxxxx-xxxx-xxxx-xxxxxxxxxxxxxxxx(8-4-4-16),分别为当前日期和时间，时钟序列，全局唯一的IEEE机器识别号，如果有网卡，从网卡mac地址获得，没有网卡以其他方式获得。

### UDN

单一设备名（Unique Device Name），基于UUID，表示一个设备。在不同的时间，对于同一个设备此值应该是唯一的。

### URI

Web上可用的每种资源 - HTML文档、图像、视频片段、程序等 - 由一个通用资源标志符（Universal Resource Identifier,简称”URI”）进行定位。 URI一般由三部分组成：访问资源的命名机制；存放资源的主机名；资源自身的名称，由路径表示。考虑下面的URI，它表示了当前的HTML 4.0规范：`http://www.webmonkey.com.cn/html/html40/`它表示一个可通过HTTP协议访问的资源，位于主机`www.webmonkey.com.cn`上，通过路径`/html/html40`访问。

### URL

URL是URI命名机制的一个子集，URL是Uniform Resource Location的缩写，译为“统一资源定位符”。通俗地说，URL是Internet上用来描述信息资源的字符串，主要用在各种www客户程序和服务器程序上。采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服务器的地址和目录等。

### URN

URN：URL的一种更新形式，统一资源名称(URN,Uniform Resource Name)。唯一标识一个实体的标识符，但是不能给出实体的位置。标识持久性Internet资源。URN可以提供一种机制，用于查找和检索定义特定命名空间的架构文件。尽管普通的URL可以提供类似的功能，但是在这方面，URN 更加强大并且更容易管理，因为 URN 可以引用多个 URL。

# 实现

## 工作机制

UPnP设备的发现和控制分为6个步骤：寻址、发现、描述、控制、事件及展现。

这三点分别在以下三篇文章中介绍

- [基于DLNA实现iOS，Android投屏：SSDP发现设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/)
- [基于DLNA实现iOS，Android投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)
- [基于DLNA实现iOS，Android投屏：订阅事件通知](https://eliyar.biz/DLNA_with_iOS_Android_Part_3_Subscribe_Event/)

## 整体流程

整体工作流程如下：

# 参考

- [upnp协议简介（一）](http://blog.csdn.net/braddoris/article/details/41646789)