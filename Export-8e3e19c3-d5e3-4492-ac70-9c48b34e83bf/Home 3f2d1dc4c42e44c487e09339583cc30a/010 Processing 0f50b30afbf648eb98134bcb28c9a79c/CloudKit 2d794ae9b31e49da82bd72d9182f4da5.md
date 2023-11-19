# CloudKit

iCloud  Container Identifiers / Ubiquity Container Identifiers  二者似乎是一致

![Untitled](CloudKit%202d794ae9b31e49da82bd72d9182f4da5/Untitled.png)

![Screenshot 2023-02-11 at 21.23.44.png](CloudKit%202d794ae9b31e49da82bd72d9182f4da5/Screenshot_2023-02-11_at_21.23.44.png)

从下面这个方法的说明可以了解到关于identifier的部分知识

[Apple Developer Documentation](https://developer.apple.com/documentation/foundation/filemanager/1411653-url)

## **iCloud开发**

- 使用iCloud的开发的前提是要有开发者账号，个人或企业均可。

---

[iCloud开发: key-value Storage,CloudKit,iCloud Documents - struggle_time - 博客园](https://www.cnblogs.com/songliquan/p/16009342.html)

### **iCloud三种类型的存储方式**

| 类型 | 说明 |
| --- | --- |
| key-value storage | 键值对的存储服务，用于一些简单的数据存储 |
| iCloud Documents | 文档存储服务，用于将文件保存到iCloud中 |
| CloudKit | 云端数据库服务 |

如何确定用哪种形式的存储方式？

****Determining If CloudKit Is Right for Your App****

Explore various ways of storing your app’s data in iCloud.

[Apple Developer Documentation](https://developer.apple.com/documentation/cloudkit/determining_if_cloudkit_is_right_for_your_app)

---

# iCloud Documents

[iCloud-Documents存储 - 掘金](https://juejin.cn/post/6971996422149406756)

### 这篇很好

[iCloud Drive, Document-Based App](https://swiftdict.com/icloud-drive-and-document-based-app.swift)

# Cloudkit

## 基本概念介绍

### **CloudKit 知识扫盲**

- CKContainer 容器，或者沙盒，每个应用只能访问自己的容器。
- CKDatabase 顾名思义，数据库了，包含私有数据库和公有数据库，用户只能访问自己的私有数据库，一些不敏感的数据也可以存储在公有数据库中。
- CKRecord 数据记录，keyvalue形式存储的，存储一些基本类型（NSString,NSNumber,NSData,NSDate,CLLocation,CKAsset,CKReference等）
- CKRecordZone 类似分区，是用来保存Record的。所有的Record都是保存在这里，应用有一个默认的zone，也可以自定义zone。
- CKAsset 文件存储记录
- CKQuery 数据库查询对象，指定查询条件进行数据查询

### 来自官方文档

[Designing with CloudKit - iCloud - Apple Developer](https://developer.apple.com/icloud/cloudkit/designing/)

[Designing with CloudKit - iCloud - Apple Developer](CloudKit%202d794ae9b31e49da82bd72d9182f4da5/Designing%20with%20CloudKit%20-%20iCloud%20-%20Apple%20Developer%20bf8b0496665142c2897ad5e1842263b6.md)

## 官方关于cloudkit的开发文档汇总

[Apple Developer Documentation](https://developer.apple.com/documentation/cloudkit)

### 一篇关于cloudkit的介绍

看起来cloudkit能做的事情很多，远不止是iCloud document存储那么简单。

我是在搜索iCloud document何cloudkit之间区别找到这篇文章

[CloudKit - NSHipster](CloudKit%202d794ae9b31e49da82bd72d9182f4da5/CloudKit%20-%20NSHipster%206aa5457c49c1468a9706a84c7ae7eb81.md)

[CloudKit](https://nshipster.cn/cloudkit/)

# 范例

[Getting Started with Core Data and CloudKit](https://www.kodeco.com/13219461-getting-started-with-core-data-and-cloudkit)

- Apps that are already using CloudKit can’t use Core Data and CloudKit with their existing CloudKit containers. To fully manage all aspects of data mirroring, Core Data owns the CloudKit schema created from the Core Data model. Existing CloudKit containers aren’t compatible with this schema.
**With that in mind, you need to create a new container.**

### 这个范例后半部份没有细看

[Sharing Core Data With CloudKit in SwiftUI](https://www.kodeco.com/29934862-sharing-core-data-with-cloudkit-in-swiftui#toc-anchor-001)

## 肘子关于Core Data with CloudKit中大文件的回答

CoreData 可以保存任意尺寸的二进制文件。在开启 allows external storage 后，Sqlite 会将尺寸大于 100KB 的二进制数据保存在数据库所在目录的一个隐藏子目录中，从而不影响数据库的操作效率。
这种方式的优势为，Core Data with CloudKit 会像对待其他数据类型一样自动完成数据的同步工作。但不足之处是，由于二进制数据都是保存在数据库中，对于只能接受 url 的音频、视频 API ，需要在播放前将数据转换成文件保存在本地后才能播放。
如果内容仅为图片的话，这种方法应该没有什么问题，关键是工作量很小。

另一种方式是只在 Core Data 中保存少量数据（例如图片的缩率图、音视频的特征数据等）以及特定的 ID （ 该 ID 可对应本地的一个 url 以及 CloudKit 上的一个 CKRecord ）。
这样二进制数据将以文件的形式保存在本地（ 与 ID 对应）。在一台设备上添加新内容后，需要另外自行编写代码（ 使用 CloudKit API）将数据保存在服务器上。当另一台设备接收到 Core Data 同步的数据后，自动或手动（ 用户按需 ）从 CloudKit 上下载数据。
这种方法最大的优势时可以让用户按需下载数据。但需要编写更多的代码。