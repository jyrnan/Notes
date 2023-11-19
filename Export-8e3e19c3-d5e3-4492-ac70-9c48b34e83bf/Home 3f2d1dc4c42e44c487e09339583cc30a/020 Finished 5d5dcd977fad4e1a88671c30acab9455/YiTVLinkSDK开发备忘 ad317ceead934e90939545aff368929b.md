# YiTVLinkSDK开发备忘

## 测试部分

- YiTVLinkSDKTests这个测试Target的Host Application应该选择None。因为如果选择了宿主应用，例如YiMultiLink_iOS，会因为宿主应用也会编译一次，导致以下错误
    - 类型重复定义错误，原因估计是宿主应用和单元测试都引入了SDK
    - 宿主应用中添加了SDK服务，会在本地生产相应socket对象，导致测试中的socket初始化冲突

![Untitled](YiTVLinkSDK%E5%BC%80%E5%8F%91%E5%A4%87%E5%BF%98%20ad317ceead934e90939545aff368929b/Untitled.png)

[真机测试发送UDP广播失败](YiTVLinkSDK%E5%BC%80%E5%8F%91%E5%A4%87%E5%BF%98%20ad317ceead934e90939545aff368929b/%E7%9C%9F%E6%9C%BA%E6%B5%8B%E8%AF%95%E5%8F%91%E9%80%81UDP%E5%B9%BF%E6%92%AD%E5%A4%B1%E8%B4%A5%200d9a7cd389504b6e86d74dc95b2d86c3.md)

[KryoCocoa使用备忘](../300%20Learning%2085ac233e054e457b9ea0166355f87cb8/312%20Swift%20f3f7435fa56948829718e9373255e683/KryoCocoa%E4%BD%BF%E7%94%A8%E5%A4%87%E5%BF%98%207afa6055a28c462992e31149157bdfe9.md)

# JSON转换

```swift
let json = "{\"tcpPort\":8604,\"udpPort\":8605}".data(using: .utf8)!
if let dic = try? JSONDecoder().decode([String:UInt16].self, from: json) {
    print(dic["tcpPort"])
    print(dic["udpPort"])
}
```