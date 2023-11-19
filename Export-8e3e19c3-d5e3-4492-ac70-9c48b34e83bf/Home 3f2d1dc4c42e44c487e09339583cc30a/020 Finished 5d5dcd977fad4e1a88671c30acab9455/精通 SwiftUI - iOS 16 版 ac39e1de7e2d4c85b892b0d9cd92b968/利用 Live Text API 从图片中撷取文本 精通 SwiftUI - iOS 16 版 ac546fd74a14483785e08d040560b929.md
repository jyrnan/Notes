# 利用 Live Text API 从图片中撷取文本 | 精通 SwiftUI - iOS 16 版

去年，iOS 15 新增了一个非常有用的功能 ── Live Text。你可能也有听过 OCR（光学字元辨识的缩写）一词，这是一个将文本图像转换为机器可读的文本格式的过程；这就是 Live Text 所做的事。

Live Text 内置在相机 App 和相片 App 中。如果你还没有尝试过这个功能，现在就来打开相机 App 试试吧！当我们把电话的镜头对准文本图像，右下角就会出现一个 Live Text 按钮。点击按钮后，iOS 就会自动撷取文本。然后，你就可以将其复制，并粘贴到其他 App（例如备忘录 App）中。

对于使用者来说，这是一个非常强大又方便的功能。而对于开发者来说，我们都会希望把这个 Live Text 功能加到自己的 App 中。在 iOS 16，Apple 发布了 Live Text API，让开发者可以在自己的 App 中加入 Live Text 功能。在这个章节中，让我们一起来看看如何在 SwiftUI 中使用 Live Text API。

### 利用 DataScannerViewController 启用 Live Text

在 WWDC 的 *[Capturing Machine-readable Codes and Text with VisionKit](https://developer.apple.com/videos/play/wwdc2022-10025)* Session 中，Apple 工程师展示了以下图片：

![%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-avfoundation.png](%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-avfoundation.png)

图 39.1. AVfoundation 和 Vision 框架提供的 API

文本辨识 (Text Recognization) 不是 iOS 16 的新功能。在旧版本的 iOS 上，我们已经可以使用 AVFoundation 和 Vision 框架中的 API，来侦测和辨识文本。但是，实现起来十分复杂，尤其是对于那些刚接触 iOS 开发的人来说。

在新版本的 iOS 中，以上所有的步骤都被简化为 VisionKit 中的一个新类别 `DataScannerViewController`。我们在 App 内使用这个视图控制器，就可以自动显示有 Live Text 功能的相机 UI。

要使用这个类别，让我们先汇入 `VisionKit` 框架，并检查设备是否支持数据扫描器功能：

```
DataScannerViewController.isSupported

```

Live Text API 只支持 2018 年后推出、有 Neural Engine 的设备。另外，我们也要检查 availability，来确保使用者批准 App 使用数据扫描器：

```
DataScannerViewController.isAvailable

```

通过两项检查后，我们就可以开始扫描了。让我们看看以下示例代码，来启用有 Live Text 功能的相机：

```
let dataScanner = DataScannerViewController(
                     recognizedDataTypes: [.text()],
                     qualityLevel: .balanced,
                     isHighlightingEnabled: true
                  )

present(dataScanner, animated: true) {
    try? dataScanner.startScanning()
}

```

我们只需要创建一个 `DataScannerViewController` 实例，并指定要辨识的数据类型即可。在示例中，我们要辨识的是文本，因此我们会把数据类型指定为 `.text()`。准备好实例之后，我们就可以调用 `startScanning()` 方法来开始扫描。

### 在 SwiftUI 使用 DataScannerViewController

`DataScannerViewController` 类别现时只支持 UIKit，因此在 SwiftUI 我们需要多做几个步骤。我们需要采用 `UIViewControllerRepresentable` 协议，才可以在 SwiftUI 项目中使用这个类别。在这个情况下，让我们创建一个新的 `DataScanner`结构：

```
import SwiftUI
import VisionKit

struct DataScanner: UIViewControllerRepresentable {
    .
  .
  .
}

```

这个结构会接受两个 binding 变量 (variable)：一个用于触发数据扫描，另一个则用来储存扫描到的文本的 binding。

```
@Binding var startScanning: Bool
@Binding var scanText: String

```

让我们实现以下方法，来采用 `UIViewControllerRepresentable` 协议：

```
func makeUIViewController(context: Context) -> DataScannerViewController {
    let controller = DataScannerViewController(
                        recognizedDataTypes: [.text()],
                        qualityLevel: .balanced,
                        isHighlightingEnabled: true
                    )

    return controller
}

func updateUIViewController(_ uiViewController: DataScannerViewController, context: Context) {

    if startScanning {
        try? uiViewController.startScanning()
    } else {
        uiViewController.stopScanning()
    }
}

```

在 `makeUIViewController` 方法中，我们回传了一个 `DataScannerViewController`的实例。在 `updateUIViewController` 中，我们会根据 `startScanning` 的数值来开始或停止扫描。

然后，我们要实现以下的 `DataScannerViewControllerDelegate` 方法，来撷取扫描到的文本。

```
func dataScanner(_ dataScanner: DataScannerViewController, didTapOn item: RecognizedItem) {
  .
  .
  .
}

```

当使用者点击侦测到的文本时，就会调用这个方法，因此我们会这样实现它：

```
class Coordinator: NSObject, DataScannerViewControllerDelegate {
    var parent: DataScanner

    init(_ parent: DataScanner) {
        self.parent = parent
    }

    func dataScanner(_ dataScanner: DataScannerViewController, didTapOn item: RecognizedItem) {
        switch item {
        case .text(let text):
            parent.scanText = text.transcript
        default: break
        }
    }

}

func makeCoordinator() -> Coordinator {
    Coordinator(self)
}

```

我们会检查辨别到的物件，如果辨别到任何文本，就会把它储存。最后，在 `makeUIViewController` 方法中插入这行代码来配置 delegate：

```
controller.delegate = context.coordinator

```

现在，控制器已经可以在 SwiftUI 视图中使用了。

### 利用 DataScanner 撷取文本

举个例子，我们会构建一个 UI 简单的文本扫描器 App。在启动 App 之后，它会自动显示 Live Text 的相机视图。当侦测到文本时，使用者可以点击文本来撷取它。扫描到的文本会在显示在屏幕的下半部分。

![%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-demo.png](%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-demo.png)

图 39.2. Live text 示范

创建好一个标准的 SwiftUI 项目后，让我们打开 `ContentView.swift` 和 `VisionKit`框架。

```
import VisionKit

```

然后，我们要声明几个变量，来控制数据扫描器和扫描到的文本的操作。

```
@State private var startScanning = false
@State private var scanText = ""

```

在 `body` 的部分，让我们这样修改代码：

```
VStack(spacing: 0) {
    DataScanner(startScanning: $startScanning, scanText: $scanText)
        .frame(height: 400)

    Text(scanText)
        .frame(minWidth: 0, maxWidth: .infinity, maxHeight: .infinity)
        .background(in: Rectangle())
        .backgroundStyle(Color(uiColor: .systemGray6))

}
.task {
    if DataScannerViewController.isSupported && DataScannerViewController.isAvailable {
        startScanning.toggle()
    }
}

```

我们在 App 启动时就开始启用数据扫描器，但在此之前，我们需要调用 `DataScannerViewController.isSupported` 和 `DataScannerViewController.isAvailable`，来确保设备支持 Live Text。

这个示例 App 差不多完成了。因为 Live Text 需要相机的访问权限，所以我们要到项目配置，在 `Info.plist` 文件中添加 *Privacy – Camera Usage Description*Key，并说明 App 需要访问相机的原因。

![%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-camera-access.png](%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-camera-access.png)

图 39.3. 添加 key 以启用相机访问

完成后，我们可以在真实的 iOS 设备上测试 App 的 Live Text 功能。

Live Text 除了支持英语外，还支持法语、意大利语、德语、西班牙语、中文、葡萄牙语、日语、和韩语。

![%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-demo-chinese.png](%E5%88%A9%E7%94%A8%20Live%20Text%20API%20%E4%BB%8E%E5%9B%BE%E7%89%87%E4%B8%AD%E6%92%B7%E5%8F%96%E6%96%87%E6%9C%AC%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac546fd74a14483785e08d040560b929/swiftui-live-text-demo-chinese.png)

图 39.4. App 示范

### 总结

很高兴看到 Apple 向 iOS 开发人员开放实时文本功能。 虽然新的`DataScannerViewController`不是专门为 SwiftUI 设计的，但我们可以轻松地将其移植到我们的 Swift 项目中，并将实时文本功能整合到我们自己的应用程序中。 如果你打算发布下一个App修改，请考虑加入这一个功能，它肯定能提升用户体验。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- [https://www.appcoda.com/resources/swiftui4/SwiftUILiveText.zip](https://www.appcoda.com/resources/swiftui4/SwiftUILiveText.zip)