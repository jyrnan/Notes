# Swift Package Manager的一些Tips – 92IT

### 概览

Swift Package Manager（SPM）是 Xcode 内置的包管理工具，支持远程公/私有库和本地库。

### 创建 Package Manager

**创建方法**

两种方法：

- 在 Xcode 菜单栏依次选中 **File > New > Package Manager**
- 在目标文件夹中使用命令：`Swift package init`

创建完成后，在 **Sources** 文件下添加代码，然后按 cmd + B 编译。如果发现编译器报错，是因为测试代码有误。如果我们不需要编写测试代码，注释即可。

![http://123.57.164.21/wp-content/uploads/2022/02/图片-9.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-9.png)

**目录结构**

如下是一个 Package 的目录结构：

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled.png)

**Package.swift** 包含如下内容：

需要特别注意在**Package.swift**中指定 platforms，也就是这个Package能够支持的平台。

```swift
// swift-tools-version:5.3
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    // 包名
    name: "Biu",
    // 需要注意Package运行的平台, 否则代码可能编译不过去！！！！！
    platforms: [
            .iOS(.v14),
    ],
    // 包对外提供的 products（库、可执行文件）
    products: [
        .library(
            name: "Biu",
            targets: ["Biu"]),
    ],
    // 依赖的其它包
    dependencies: [
        // .package(url: /* package url */, from: "1.0.0"),
    ],
   // 包含的 targets
    targets: [
       // 每个 target 所需依赖
        .target(
            name: "Biu",
            dependencies: []),
        .testTarget(
            name: "BiuTests",
            dependencies: ["Biu"]),
    ]
)
```

```swift
name: 包的项目名称
    platforms: 支持的平台及对应平台的最低版本
    targets: 包含多个target的集合，我们指定target的名字为ZZPackage，xcode会自动把Sources/ZZPackage目录下的所有文件添加到package中。如果你想再新建一个target, 需要在Sources/目录下新建一个文件夹，然后再targets数组中添加新的target。
    .target是PackageDescription.Target实例类，参数说明：
        name: target名字
        dependencies: target的依赖，主要指定Package添加的依赖module的名字
        path: target的路径，如果自定义文件夹需要设置此参数
        exclude: target path中不希望被包含的path
        sources: 资源文件路径
        publicHeadersPath: 公共header文件路径
    products: 对外公开导出target产物，使得其他target能够使用它们。如果不写会编译报错
        .library(
        name: "ZZPackage",
        type: .static,
        targets: ["ZZPackage"]) : 可指定静态库或动态库，默认静态库
    dependencies: 添加包所依赖的其他第三方package包的集合
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.0.0"),
        .package(url: "https://github.com/SnapKit/SnapKit.git", from: .init(5, 0, 1)),
        .package(url: "https://github.com/SnapKit/SnapKit.git", from: .init(stringLiteral: "5.0.1")),
        .package(url: "https://github.com/SnapKit/SnapKit.git", Package.Dependency.Requirement.branch("master")), : 指定分支，如果第三方不支持spm的话可以使用这种方式
        .package(path: "../ZZPackage") : 关联本地的SPM库
    swiftLanguageVersions: 支持的swift版本
```

### **本地添加和测试 Package**

- 新建一个 Biu_test 工程，直接将 Biu 目录拖入工程
- **General -> Frameworks, Libraries, and Embedded Content** 下添加 Package
- 导入 Package（`import Biu`）即可

![http://123.57.164.21/wp-content/uploads/2022/02/图片-11.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-11.png)

![http://123.57.164.21/wp-content/uploads/2022/02/图片-12.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-12.png)

注：如遇到无法导入或找不到 Package 的问题，可尝试退出工程或重启 Xcode 解决

### 发布与更新 Package

可以使用 Git 命令，也可以使用 Xcode 内置的版本控制工具。

初次发布时，初始化 Package 为 Git 仓库，打上 Tag，再推送至远程仓库。

更新 Package 后，打上新的 Tag，Push 即可。

**使用 Package**

**添加**

我们先移除上文中的工程 Biu_test 引入的本地 Package（Biu），然后在 Xcode 中选择 **File > Swift Packages > Add Package Dependency…**，输入 Package 地址（[https://github.com/keisme/Biu](https://github.com/keisme/Biu)），点击 Next 安装。

安装完成后，重新运行项目，结果与预期一致。

**更新**

如果需要更新 Package，选择 **File > Swift Packages > Update to Latest Package Versions**。

**移除**

在工程中找到 **Swift Package Manager**，移除相应的 Package。

### Package的例子

**File > New > Package Manager** 创建一个TestPackage

![http://123.57.164.21/wp-content/uploads/2022/02/图片-15-2048x1363.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-15-2048x1363.png)

9创建一个SwiftUIView类

![http://123.57.164.21/wp-content/uploads/2022/02/图片-29.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-29.png)

我们把TestPackage上传到Github上去。

![http://123.57.164.21/wp-content/uploads/2022/02/图片-19-2048x1480.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-19-2048x1480.png)

新建一个Xcode工程–>点击**File**–> **Add Packages**

![http://123.57.164.21/wp-content/uploads/2022/02/图片-21-2048x1098.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-21-2048x1098.png)

右上角输入github的url -> 选择**Add Github Enterprise account**。

输入Account 和 Token。

![http://123.57.164.21/wp-content/uploads/2022/02/图片-23.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-23.png)

Token的获取方法，Setting -> Developer settings

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%201.png)

Personal access tokens –> Generate new token

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%202.png)

用Token登录后，可以看到这个TestPackage 点击Add Package添加到工程

![http://123.57.164.21/wp-content/uploads/2022/02/图片-26-2048x1124.png](http://123.57.164.21/wp-content/uploads/2022/02/图片-26-2048x1124.png)

在DemoApp中我们可以看到可以引用到 TestPackage中的 SwiftUIView。

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%203.png)

# Swift Package Manager 添加资源文件

从更新到现在，SwiftPM 令人诟病的一个问题就是无法在包里添加资源文件。这对于已经习惯于使用 CocoaPods 的开发者造成了很大的麻烦，当然目前 SwiftPM 差于 Cocoapods 不止这一点。SwiftPM 也意识到了这一点，从去年就可以看到 github 的 SwiftPM 对应仓库的有 `resource` 等 API 相关提交。

此次的 SwiftPM 更新中除了上面说的可以添加资源文件，还添加了本地化等功能。

SwiftPM 的资源文件管理功能在 swift-tool-version 5.3，即 Swift 5.3，对应 Xcode 12。所以 `package.swift` 配置中需要声明 swift 5.3 以上(这行并不是注释，而是文件解析中必须的配置)：

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%204.png)

## **添加和配置资源文件**

对于一些使用目的明确的文件类型，比如下面图中的这些。开发者不需要在 `package.swift` 文件中配置任何东西，因为 Xcode 知道这些类型的文件是代表什么，比如 `.xcassets` 文件代表图片、颜色资源， `xib` 代表用户界面文件等

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%205.png)

而对于一些使用目的不太明确的文件类型（如下图中的一些文件类型），则需要在 `package.swift` 文件中配置。例如纯文本文件，这种文件中的数据可能是需要在运行时被加载而计算或者展示，也可能只是一个开发者文档。

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%206.png)

对于上面这种意义不明的文件，就需要在 `package.swift` 清单中根据规则配置，下面以这个 GameLogin 作为例子：

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%207.png)

- 对于 `Media.xcasset` 和 `main.storyboard` 文件，Xcode 能明确知道它代表什么，所以不需要在这个配置文件中标记
- `internal Note.txt` 文件和 `Artwork Creation` 文件夹是模块内部文件，所以写在 `target` 的 `exclude` 属性中，这样 Xcode 就不会把它编译到包里
- 其他不能自动识别的类型并且需要被加载到 package 里的文件则配置在 `resource` 属性中。

上面就是配置资源文件的一些规则，其中我们可以看到对于 `resource` 属性，有两个静态方法： `process()` 和 `copy()` 。根据 session 中的介绍， `process()` 是推荐的方式，它所配置的文件会根据具体使用的平台和内置规则进行适当的优化。比如在运行时将 `storyboard` 或者 `asset catalog` 转换成适当的形式，也包括压缩图片等。如果文件类型无法识别，或者不能根据平台做任何优化，就只会被简单的拷贝，也就是 `copy()` 。

## **构建过程**

当一个 App 使用 package 时，这个 package 包括源文件和资源文件。在编译时首先会将 Package 中每个 target 的源文件编译成 module 链接到 App 中，然后这些 target 中的资源文件则会被加工成 bundle 放到这些 module 中。

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%208.png)

在 Apple 平台中，App 和 App extension 都是 bundle 集合，这些 package 的 bundle 就是 App 的一部分，所以不需要做其他处理，就能在运行时获取这些 bundle。 当被编译到一个 unbundle 产物时，比如脚本工具，则需要在脚本启动的同时加载资源 bundle(这一步的具体步骤还不太理解)

## **访问资源文件**

在编译有资源文件的 Package 中，会自动创建并添加到 module 中一个文件：`resource_bundle_accessor.swift` ，里面的内容大概等价于下面这样：

```swift
import Foundation
extension Bundle {
 static let module = Bundle(path: "\(Bundle.main.bundlePath)/path/to/this/targets/resource/bundle")
}
```

对于 Swift 和 OC 分别可以使用下面这种方式，当然也可以使用 UIImage 自己的带有 `Bundle` 参数的 Api、

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%209.png)

<aside>
💡 这里是添加资源文件的关键部分，具体步骤

[ SPM中添加媒体资源文件](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/SPM%E4%B8%AD%E6%B7%BB%E5%8A%A0%E5%AA%92%E4%BD%93%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6%209cffbbb920f64e78a9af601dc7428848.md)

</aside>

对于SwiftUI来说，可以直接通过**Bundle.module** 来访问，图片放到Target下面的的Media.xcassets即可

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%2010.png)

```swift
struct HeaderBackgroundView: View {
    var body: some View {
        GeometryReader { gr in
            ZStack {
                Image("imageName", bundle: Bundle.module).resizable().frame(width: gr.size.width, height: gr.size.height * 0.26)
            }.frame(width: gr.size.width, height: gr.size.height * 0.26)
        }
    }
}
```

题外话：在SwiftUI中读取指定Bundle中图片的例子

```swift
import SwiftUI
struct ContentView: View {
    
    var body: some View {
        Group{
            if let resBundlePath = Bundle.main.path(forResource: "Resources", ofType: "bundle"),
               let resBundle = Bundle(path: resBundlePath),
               let uiImage = UIImage(named: "imagename", in: resBundle, with: nil){
                    Image(uiImage: uiImage)
                        .resizable()
                        .scaledToFit()
            }else{
                Color.red
            }
        }
    }
}
```

# Package中开发SwiftUI画面

在Package中开发SwiftUI画面有些尴尬，始终没找到运行的方法，只能先通过 SwiftUIView_Previews来调试了。

![Untitled](Swift%20Package%20Manager%E7%9A%84%E4%B8%80%E4%BA%9BTips%20%E2%80%93%2092IT%2059af0298be884b1e93d3a632f1ed207d/Untitled%2011.png)