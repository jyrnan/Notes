# SPM

### 创建一个SPM的方法：

- 新建项目文件，运行代码

```swift
swift package init --type=excutable
```

### 给Package添加依赖：

- 在Package文件里面添加package相关信息，在Xcode里面会自动下载。

```bash
dependencies: [
        // 💧 A server-side Swift web framework.
        .package(url: "https://github.com/vapor/vapor.git", from: "4.0.0"),
        .package(url: "https://github.com/vapor/leaf.git", from: "4.0.0-rc.1"),
    ],
```