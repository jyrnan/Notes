# 通过Workspace用CocoaPods管理依赖进行SDK开发

# 安装CocoaPods

安装步骤略…

[CocoaPods的安装与使用 - 简书](CocoaPods%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8%20-%20%E7%AE%80%E4%B9%A6%20c92cc3dd34b94f8c943943a2d984148c.md)

# 创建文件

可以利用workspace来进行项目管理，一般将SDK项目和Demo项目放在一个workspace里面

## 项目

- 创建workspace文件夹`DemoWorkspacePodsSDK`
- 通过Xcode中`File/New/Workspace`创建workspace文件`WSPodsSDK.xcworkspace`保存到`DemoWorkspacePodsSDK`
- 通过Xcode中`File/New/Project`在workspace中添加两个项目，分别命名为`SDK`和`App`

![Untitled](%E9%80%9A%E8%BF%87Workspace%E7%94%A8CocoaPods%E7%AE%A1%E7%90%86%E4%BE%9D%E8%B5%96%E8%BF%9B%E8%A1%8CSDK%E5%BC%80%E5%8F%91%20bcfccc9a426a44d9903ab7fa7b82db81/Untitled.png)

## 创建Podfile

- 在workspace的根文件夹`DemoWorkspacePodsSDK` 执行 `pod ini SDK/SDK.xcodeproj`
    
    > `pod ini` 会在当前文件夹下创建一个默认Podfile，该命令必须指定一个项目，所以这里先指定了`SDK/SDK.xcodeproj`这个项目，当然也可以指定另外的项目，但其实这里选择那个项目并不重要，到后面会通过修改Podfile对各个项目分别设置依赖关系
    > 
- 在Podfile中添加相关内容，特别是workspace名称等，具体内容参考Podfile的语法

```bash
workspace 'WSPodsSDK.xcworkspace' //如果不添加这一行cocoapods会自己创建新的workspace文件
project 'SDK/SDK.xcodeproj' // pod ini自动添加内容

# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'
use_frameworks! //需要，否则会生成的是.a文件来供调用

target 'SDK' do
  # Comment the next line if you don't want to use dynamic frameworks
  project 'SDK/SDK.xcodeproj'

  # Pods for SDK
  pod 'CocoaAsyncSocket', '~> 7.6.3
end

target 'SDKTests' do
  # Pods for testing
  project 'SDK/SDK.xcodeproj'
  pod 'CocoaAsyncSocket', '~> 7.6.3'
end

target 'App' do
  project 'App/App.xcodeproj'
  pod 'CocoaAsyncSocket', '~> 7.6.3'
end
```

- 在`Podfile` 所在目录执行`pod install`，此时CocoaPods会根据当前的Podfile来安装依赖关系，并会生成Pods项目，最终文件夹如图

![Untitled](%E9%80%9A%E8%BF%87Workspace%E7%94%A8CocoaPods%E7%AE%A1%E7%90%86%E4%BE%9D%E8%B5%96%E8%BF%9B%E8%A1%8CSDK%E5%BC%80%E5%8F%91%20bcfccc9a426a44d9903ab7fa7b82db81/Untitled%201.png)

Xcode中项目文件如下图

![Untitled](%E9%80%9A%E8%BF%87Workspace%E7%94%A8CocoaPods%E7%AE%A1%E7%90%86%E4%BE%9D%E8%B5%96%E8%BF%9B%E8%A1%8CSDK%E5%BC%80%E5%8F%91%20bcfccc9a426a44d9903ab7fa7b82db81/Untitled%202.png)

## 代码部分

- 在SDK内部应该可以**`import** CocoaAsyncSocket`

```bash
import Foundation
import CocoaAsyncSocket
import Pods_SDK

public class SDKClass: NSObject {
    @objc public var name: String = "SDKClass"
    
    @objc public func sayHello() {print(#line, #function, self, name)}
    
    public let socket: GCDAsyncUdpSocket = GCDAsyncUdpSocket()
}
```

- 用来验证SDK的App，可以添加这个SDK framework

![Untitled](%E9%80%9A%E8%BF%87Workspace%E7%94%A8CocoaPods%E7%AE%A1%E7%90%86%E4%BE%9D%E8%B5%96%E8%BF%9B%E8%A1%8CSDK%E5%BC%80%E5%8F%91%20bcfccc9a426a44d9903ab7fa7b82db81/Untitled%203.png)

![Untitled](%E9%80%9A%E8%BF%87Workspace%E7%94%A8CocoaPods%E7%AE%A1%E7%90%86%E4%BE%9D%E8%B5%96%E8%BF%9B%E8%A1%8CSDK%E5%BC%80%E5%8F%91%20bcfccc9a426a44d9903ab7fa7b82db81/Untitled%204.png)

- App添加了SDK后，就可以使用SDK内部的类型了

```swift
import SwiftUI
import SDK

struct ContentView: View {
    let vm: SDKClass = SDKClass()
    var body: some View {
        VStack {
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundColor(.accentColor)
            Text("Hello, world! \(vm.name)")
                .onTapGesture {
                    vm.sayHello()
                    try? vm.socket.beginReceiving()
                }
        }
        .padding()
    }
}
```

<aside>
🤔 需要注意的是：在这里SDK通过CocaoPods引入了`CocoaAsyncSocket` 第三方库， 在App调用SDK的时候，App也需要引入`CocoaAsyncSocket`， 否则会出现找不到该库的错误。可以理解成SDK有这个依赖需求，但是需要在调用该SDK的宿主App来实现依赖需求

</aside>