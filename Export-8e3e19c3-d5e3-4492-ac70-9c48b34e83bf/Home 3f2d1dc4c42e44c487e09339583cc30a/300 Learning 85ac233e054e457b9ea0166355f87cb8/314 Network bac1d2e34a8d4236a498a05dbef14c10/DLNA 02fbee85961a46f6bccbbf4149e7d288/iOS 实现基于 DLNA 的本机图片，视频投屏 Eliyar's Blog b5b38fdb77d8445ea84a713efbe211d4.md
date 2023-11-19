# iOS 实现基于 DLNA 的本机图片，视频投屏 | Eliyar's Blog

[https://eliyar.biz/iOS_DLNA_with_local_image_and_video/index.html](https://eliyar.biz/iOS_DLNA_with_local_image_and_video/index.html)

![184EBD1C-9943-4762-B552-282FC4ACF773.png](iOS%20%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E%20DLNA%20%E7%9A%84%E6%9C%AC%E6%9C%BA%E5%9B%BE%E7%89%87%EF%BC%8C%E8%A7%86%E9%A2%91%E6%8A%95%E5%B1%8F%20Eliyar's%20Blog%20b5b38fdb77d8445ea84a713efbe211d4/184EBD1C-9943-4762-B552-282FC4ACF773.png)

DLNA 投网络上的媒体文件已经在前几篇实现过了。现在记录一下把本地图片和视频文件投到 DLNA 设备。

# 基础知识

## DLNA

关于 DLNA 的基础知识请看一下四篇文章：

- [基于 DLNA 实现 iOS，Android 投屏：基本概念](https://eliyar.biz/DLNA_with_iOS_Android/)
- [基于 DLNA 实现 iOS，Android 投屏：SSDP发现设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_1_Find_Device_Using_SSDP/)
- [基于 DLNA 实现 iOS，Android 投屏：SOAP控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)
- [基于 DLNA 实现 iOS，Android 投屏：订阅事件通知](https://eliyar.biz/DLNA_with_iOS_Android_Part_3_Subscribe_Event/)

## GCDWebServer

[GCDWebServer](https://github.com/swisspol/GCDWebServer) 是一个现代化的轻量级的基 于HTTP 1.1 的 GCD server，它主要用于嵌入 OS X & iOS apps。GCDWebServer 在我们的实现中扮演 HTTP Server 的作用。使用前确保你已阅读一下两篇：

- [(译)GCDWebServer概述](https://github.com/coderyi/blog/blob/master/articles/2015/1112_GCDWebServer_README.md)
- [GCDWebServer Readme](https://github.com/swisspol/GCDWebServer)

# 实现思路

目前我有两种思路

1. 实现完整的 DLNA Media Server，提供媒体目录和存储。
2. 跑一个 HTTP Server，产生文件 URL，然后把 URL 投到 DLNA 设备，相当于投网络视频。

其中方案1是 DLNA 标准的做法，方案2相当于是简化版的 DMS( DLNA Media Server)。考虑到我的 DLNA 实现全都是自己写的，我选择了方案2。

注：

- 相册 - iOS 系统相册
- DLNAMediaHelper - 我写的一个 Helper 单例，用于保存随机产生的 url 和对应的 PHAsset 资源、获取某个具体 URL 对应的媒体文件数据或者路径。
- DLNAManager - DLNA 投屏部分是实现类，用于处理 SSDP发现设备，发送 SOAP 命令以及接受订阅。
- DLNA 设备 - 目标 DLNA 设备
- Webserver - 本机（iphone）HTTP Server

# 具体实现过程

## 相册获取 PHAsset

从相册获取媒体需要熟悉 PhotoKit，具体不阐述了。此处我选择了能够获取图片和视频的第三方开源框架 [CTAssetsPickerController](https://github.com/chiunam/CTAssetsPickerController)

## 产生 URL，保存字典

其实这一步没什么难度，首先根据时间戳产生一个 URL，并把这个 URL 作为 key, asset 作为 value 存到一个字典内，用于后期处理请求即可。具体实现看下面。

## 投 URL 到 DLNA 设备

详见 [基于 DLNA 实现 iOS，Android 投屏：SOAP 控制设备](https://eliyar.biz/DLNA_with_iOS_Android_Part_2_Control_Using_SOAP/)

## 获取 PHAsset 对应的资源文件

PHAsset 代表照片库中的一个资源，不是真正的原始数据。所以我们还需要使用 PHAsset 获取对应资源。具体的获取方法我均放在 `DLNAMediaManager` 类中。

DLNAMediaManager 实现

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293 | class DLNAMediaManager {    static let shared = DLNAMediaManager()    /// url:asset关系字典    var mediaList: [String:PHAsset] = [:]    /// 处理媒体资源请求的 webserver 的 URL，我这是直接用的是我的 DLNAManager 里面的 Server    var serverURL: String {        get {            return DLNAManager.sharedManager.webServer.serverURL.URLString        }    }    /**     产生 url 方法     - parameter forAsset: 目标 asset     - returns: 产生的响应 url     */    func generateURL(forAsset :PHAsset) -> String {        var url = ""        if forAsset.mediaType ==  .Video {            url = serverURL + "videos/" + UIUTil.gettimestampForNow().md5() + ".mov"        } else {            url = serverURL + "images/" + UIUTil.gettimestampForNow().md5() + ".jpg"        }        mediaList[url] = forAsset        return url    }    /**     获取 Image 文件的 NSData，只需要在响应时候去获取再返回即可     - parameter url:      资源对应的请求url     - parameter callBack: 资源获取完成回调     */    func fetchImageData(url:String, callBack:((data:NSData?)->Void)) {        if let asset = imageList[url] {            let options = PHImageRequestOptions()            options.synchronous = false            options.deliveryMode = .HighQualityFormat            options.networkAccessAllowed = true            PHImageManager.defaultManager().requestImageForAsset(asset,                                                                 targetSize: PHImageManagerMaximumSize,                                                                 contentMode: .Default,                                                                 options: options)            { (result, info) -> Void in                if let image = result, data = UIImageJPEGRepresentation(image, 1.0) {                    callBack(data:data)                } else {                    callBack(data: nil)                }            }        } else {            callBack(data: nil)        }    }    /**     获取视频文件文件路径     - parameter url:      资源对应请求url     - parameter callBack: 资源获取完成回调     */    func fetchVideoFile(url:String, callBack:((fileURL:String?)->Void)) {        if let asset = imageList[url] {            let imageManager = PHImageManager.defaultManager()            let videoRequestOptions = PHVideoRequestOptions()            videoRequestOptions.deliveryMode = .Automatic            videoRequestOptions.version = .Current            videoRequestOptions.networkAccessAllowed = true            imageManager.requestAVAssetForVideo(asset,                                                options: videoRequestOptions,                                                resultHandler:                { (avAsset, avAudioMix, info) -> Void in                if let nextURLAsset = avAsset as? AVURLAsset,                    filepath = nextURLAsset.URL.path {                    callBack(fileURL: filepath)                } else {                    callBack(fileURL: nil)                    }            })        } else {            return callBack(fileURL: nil)        }    }} |
| --- | --- |

## Webserver 接受请求

使用 GCD，具体处理方法如下，请注意其中图片和视频文件处理方法略有不同。关于 GCDWebServer 具体细节请见 [GCDWebServer Readme](https://github.com/swisspol/GCDWebServer)

| 1234567891011121314151617181920212223242526272829303132333435363738394041 | func startWebServer() {   /**    *  若不符合其中两个请求方式，则返回404    */   let MediaNotFoundResponse = GCDWebServerResponse(statusCode:404)   /**    *  图片文件响应，图片文件返回 Data Response    */   webServer.addHandlerForMethod("GET", pathRegex: "/images/", requestClass: GCDWebServerRequest.self) { (request, completionBlock) in       let url = request.URL.URLString       DLNAImageHelper.fetchImageData(url, callBack: { (data) in           if let data = data {               let response = GCDWebServerDataResponse(data: data, contentType: "image/jpeg")               completionBlock(response)           } else {               completionBlock(MediaNotFoundResponse)           }       })   }   /**    *  Video 文件响应，Video 文件数据很大，必须使用FileResonse    *    *  注意 `byteRange: request.byteRange` 这里，如果不这么处理 DLNA 设备无法播放媒体文件。    *    */   webServer.addHandlerForMethod("GET", pathRegex: "/videos/", requestClass: GCDWebServerRequest.self) { (request, completionBlock) in       let url = request.URL.URLString       DLNAImageHelper.fetchVideoFile(url, callBack: { (fileURL) in           if let file = fileURL {               let response = GCDWebServerFileResponse(file: file, byteRange: request.byteRange)               completionBlock(response)           } else {               completionBlock(MediaNotFoundResponse)           }       })   }   webServer.startWithPort(8899, bonjourName: nil)} |
| --- | --- |

# 参考

- [iOS 开发之照片框架详解](http://kayosite.com/ios-development-and-detail-of-photo-framework-part-two.html)
- [How to get URL for a PHAsset?](http://stackoverflow.com/questions/38183613/how-to-get-url-for-a-phasset)
- [How to implement Video Seek support with an embedded HTTP server on iOS?](http://stackoverflow.com/questions/36651451/how-to-implement-video-seek-support-with-an-embedded-http-server-on-ios)
- [照片框架](https://objccn.io/issue-21-4/)

[# iOS](https://eliyar.biz/tags/iOS/) [# Swift](https://eliyar.biz/tags/Swift/) [# DLNA](https://eliyar.biz/tags/DLNA/)