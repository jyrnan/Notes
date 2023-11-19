# Swift制作静态库（.framework) - 掘金

### 动态库和静态库的区别

**库**：一段二进制文件+加头文件，使用场景一个是供别人使用，一个是在代码改动较小，减少编译时间，因为只是一段二进制文件，Link一下，即可使用.

**Framework**: 一种打包方式，简单将二进制文件、头文件和其他一些信息聚合在一起。

**iOS中的Framework分类**:

- 系统级别: Dynamic Framework, 系统提供的framework都是动态库，比如UIKit.framework，系统的framwork不需要拷贝目标程序。
- 用户级别：Static Framework(静态库)和Dynamic Framework(动态库)（Embeded Framework)

Static Framework： 二进制文件+头文件+资源文件

Embeded Framework： 用户可以制作的动态库，受到iOS平台的沙盒机制和签名机制限制，具有部分动态特性，可以在Extension可执行文件和APP可执行文件中执行，不能在不同app进程中共享，而且需要拷贝到目标程序。

| 静态库 | 一般以.a或者.framework结尾,在编译时候，只拷贝一份之后就不会再改变了 | 目标程序没有外部依赖，可以直接运行 | 目标程序体积大 |
| --- | --- | --- | --- |
| 动态库 | 一般以.dylib或者.framework结尾, 并且不会别拷贝到目标程序， 目标程序只会存储指向动态库的指引，等到程序运行，才会被加载 | 同一个份库可供多个程序使用, 运行载入特性使得目标程序体积小 | 1.动态载入会带来一部分的性能损失2.依赖外部环境，如果动态库缺失或者版本错误，就会导致程序无法运行 |

### iOS设备CPU架构

模拟器：i386, x86_64

真机: arm7, arm7s, arm64,最新的设备基本都是arm64

### 静态库制作

### 新建Static Framework

Xcode -> File -> new -> Project-> 选中 iOS ->Static Framework

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d36064449tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d36064449tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

这里创建LLStaticSDK的工程

### 新建swift文件

注意这里的关键词public，也可以用open。

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3a04a047tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3a04a047tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 工程配置

### 1. Mach-O Type设置(Static Library)

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3b61feb3tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3b61feb3tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 2.version设置

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3d7ae4bbtplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3d7ae4bbtplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 3.Build Active Architecture Only(NO,适配所有设备)

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3bd1ef09tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d3bd1ef09tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 4. Dead Code Stripping(NO, 可选)

主要是目的是瘦包

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d69feb7a7tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d69feb7a7tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 调试

### 方式一: 终端操作

分别选择在真机和模拟器command+b编译文件，然后选择Product->这里创建LLStaticSDK的工程.framework showInFinder显示如图俩个文件夹，分别是真机和模拟器, 分别执行

```bash
$ lipo -info /Users/wujian/Library/Developer/Xcode/DerivedData/LLStaticSDK-bnsiapbkckebfyefumdziltevphd/Build/Products/Debug-iphoneos/LLStaticSDK.framework/LLStaticSDK
Non-fat file: /Users/wujian/Library/Developer/Xcode/DerivedData/LLStaticSDK-bnsiapbkckebfyefumdziltevphd/Build/Products/Debug-iphoneos/LLStaticSDK.framework/LLStaticSDK is architecture: arm64

```

```bash
lipo -info /Users/wujian/Library/Developer/Xcode/DerivedData/LLStaticSDK-bnsiapbkckebfyefumdziltevphd/Build/Products/Debug-iphonesimulator/LLStaticSDK.framework/LLStaticSDK
Non-fat file: /Users/wujian/Library/Developer/Xcode/DerivedData/LLStaticSDK-bnsiapbkckebfyefumdziltevphd/Build/Products/Debug-iphonesimulator/LLStaticSDK.framework/LLStaticSDK is architecture: x86_64

```

可以看出LLStaticSDK文件分别描述设备的CPU架构分别是arm64， x86_64

接下来就要对这俩部分进行合并处理, $ lipo -create [目标文件1] [目标文件2] -output [输出文件]

```bash
lipo -create /Users/wujian/Library/Developer/Xcode/DerivedData/LLStaticSDK-bnsiapbkckebfyefumdziltevphd/Build/Products/Debug-iphoneos/LLStaticSDK.framework/LLStaticSDK
/Users/wujian/Library/Developer/Xcode/DerivedData/LLStaticSDK-bnsiapbkckebfyefumdziltevphd/Build/Products/Debug-iphonesimulator/LLStaticSDK.framework/LLStaticSDK -output ~/Desktop/LLStaticSDK.lipo

```

这里将LLStaticSDK替换真机的LLStaticSDK的文件

接着把模拟器文件夹中的LLStaticSDK swiftmodule的文件和真机的文件合并

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d68f783d3tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d68f783d3tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

最后将真机的这里将LLStaticSDK.framework托人Demo中，进行测试

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d6b474fe8tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d6b474fe8tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 方式二：工程依赖

方式一调试起来很麻烦，每次都要进行重新操作才能重新调试。方式二比较推荐

### 1.首先将SDK工程放入Demo如图目录

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d6bb590a1tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d6bb590a1tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 2.创建依赖

Target -> Framwork, Library and embeded content -> + -> addOther -> LLStaticSDK.xcodeproj, 添加流程和显示结果如下

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d79e584catplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d79e584catplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d94c672e0tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d94c672e0tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d97ee6a21tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d97ee6a21tplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

### 3.在SDK工程下新建target

[Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d9a967b0ftplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/1739a18d9a967b0ftplv-t2oaga2asx-zoom-in-crop-mark3024000.awebp)

![Untitled](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/Untitled.png)

### 4.添加脚本

将[脚本内容](https://link.juejin.cn/?target=https%3A%2F%2Fgist.github.com%2Fcromandini%2F1a9c4aeab27ca84f5d79)copy到如下图位置

![Untitled](Swift%E5%88%B6%E4%BD%9C%E9%9D%99%E6%80%81%E5%BA%93%EF%BC%88%20framework)%20-%20%E6%8E%98%E9%87%91%2027d0d10adba4415abf9af32ccf441910/Untitled%201.png)

将脚本中的${PROJECT_NAME}替换成自身的framework名 编译一下，lipo -info 会发现支持的CPU架构为i386 armv7 x86_64 arm64。 然后项目就在集成在一起，修改起来就很方便了