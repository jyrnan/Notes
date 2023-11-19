# 通过 ShareLink 来分享文本和图像等数据 | 精通 SwiftUI - iOS 16 版

在 iOS 16 中，SwiftUI 带来了一个新的视图 `ShareLink`。使用者点击 Share Link 时，视图就会显示一个 Share Sheet，让使用者可以分享内容到其他 App 或复制数据。

`ShareLink` 视图可以分享任何类型的数据。此章我们会一起看看如何使用 `ShareLink` 让使用者分享文本、URL 和图像。

### ShareLink 的基础使用

让我们先看看以下例子。如果想建立一个 Share Link 来分享一个 URL，我们可以如此编写代码：

```
struct ContentView: View {
    private let url = URL(string: "https://www.appcoda.com")!

    var body: some View {
        VStack {
            ShareLink(item: url)
        }
    }
}

```

SwiftUI 会自动显示一个 *Share* 按钮和小图标。

![%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-basic.png](%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-basic.png)

图 40.1. Share 按钮

使用者点击按钮后，iOS 就会显示一个 Share Sheet，让使用者选择下一步操作，例如复制、或是把 Link 添加到 Reminders。

![%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-sharesheet.png](%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-sharesheet.png)

图 40.2. 显示 share sheet

如果我们想分享文本，我们只需要把字串 (string) 传递到 `item` 参数 (parameter) 即可。

```
ShareLink(item: "Check out this new feature on iOS 16")

```

### 客制化 Share Link 的外观

如果想客制化 Share Link 的外观，我们可以在闭包 (closure) 中提供视图内容

```
ShareLink(item: url) {
    Image(systemName: "square.and.arrow.up")
}

```

在这个例子中，SwiftUI 就会只显示 Share Link 的图标。

![%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-tapmeshare.png](%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-tapmeshare.png)

图 40.3. 客制化 Share 按钮

或者，我们可以显示一个 Label，并指定系统或客制化的图像：

```
ShareLink(item: url) {
    Label("Tap me to share", systemImage:  "square.and.arrow.up")
}

```

在初始化 `ShareLink` 实例时，我们可以添加 2 个附加参数，以提供更多有关分享项目的数据：

```
ShareLink(item: url, subject: Text("Check out this link"), message: Text("If you want to learn Swift, take a look at this website.")) {
    Image(systemName: "square.and.arrow.up")
}

```

我们可以使用 `subject` 参数和 `message` 参数，分别为 URL 或想分享的项目添加 title 和描述。iOS 会根据使用者想要分享的途径，去决定要显示 subject、message、或是两者都显示。比如说，如果我们想要把 URL 添加到 *Reminders*，Reminders App 就会显示默认的信息。

![%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-reminders.png](%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-reminders.png)

图 40.4. Reminders App 显示默认的信息

### 分享图像

除了 URL 外，我们还可以利用 `ShareLink` 来分享图像。让我们看看以下的示例代码：

```
struct ContentView: View {
    private let photo = Image("bigben")

    var body: some View {
        VStack(alignment: .leading, spacing: 10) {
            photo
                .resizable()
                .scaledToFit()

            ShareLink(item: photo, preview: SharePreview("Big Ben", image: photo))

        }
        .padding(.horizontal)
    }
}

```

在 `item` 参数中，我们指定了要分享的图像。另外，我们也传递了一个 `SharePreview` 的实例，来提供图像的预览。在预览中，我们指定了图像的 title 和 thumbnail。使用者点击 *Share* 按钮后，iOS 就会显示一个有图像预览的 Share Sheet。

![%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-image.png](%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-image.png)

图 40.5. 显示一个有图像预览的 Share Sheet

## 符合 Transferable 协议

除了 URL 外，`item` 参数也接受任何遵守 `Transferable` 协议的物件。在 iOS 中，以下都是标准的 `Transferable` 类型：

- String
- Data
- URL
- Attributed String
- Image

> 
> 
> 
> *Transferable 协议可以用来描述类型如何与 Transport API 交互，例如拖放动作 (drag and drop)、或复制和贴上 (copy and paste)。*
> 

那我们如何让一个客制物件变成 Transferable 呢？假设，我们建立了以下的 `Photo` 结构：

```
struct Photo: Identifiable {
    var id = UUID()
    var image: Image
    var caption: String
    var description: String
}

```

要让 `ShareLink` 可以分享这个物件，我们就要让 `Photo` 遵守 `Transferable` 协议，并实现 `transferRepresentation` 属性 (property)：

```
extension Photo: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        ProxyRepresentation(exporting: \.image)
    }
}

```

我们有不同的 Transfer Representation，包括：`ProxyRepresentation`、`CodableRepresentation`、`DataRepresentation`、和 `FileRepresentation`。在以上的代码中，我们使用了 `ProxyRepresentation`，它会使用另一种类型的 Transfer Representation 作为自己的 Representation。 在这里，我们使用了 `Image` 内建的 `Transferable` 协议。

![%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-photo-transferable.png](%E9%80%9A%E8%BF%87%20ShareLink%20%E6%9D%A5%E5%88%86%E4%BA%AB%E6%96%87%E6%9C%AC%E5%92%8C%E5%9B%BE%E5%83%8F%E7%AD%89%E6%95%B0%E6%8D%AE%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2053493668aad64f1d952293588f0340fa/swiftui-sharelink-photo-transferable.png)

图 40.6. 使用了 Image 内建的 Transferable 协议

因为 `Photo` 现在遵守 `Transferable` 协议，我们就可以把 `Photo` 实例传递给 `ShareLink`：

```
ShareLink(item: photo, preview: SharePreview(photo.caption, image: photo.image))

```

现在，当使用者点击 *Share* 按钮，App 就会显示分享图片的 Share Sheet。

### 总结

这章教了大家如何使用 `ShareLink` 分享文本、URL、和图像。事实上，只要类型遵守 `Transferable` 协议，我们都可以用这个新的视图分享任何类型的数据。

要分享客制化类型，我们可以采用协议，并使用其中一个内建的 `TransferRepresentation` 类型，来提供 Transfer Representation。 我们简单讨论了 `ProxyRepresentation` 类型。如果我们想要在 App 之间分享一个文件，就可以使用 `FileRepresentation` 类型。我们会在之后的教学文章中，再深入探讨这个题目。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- [https://www.appcoda.com/resources/swiftui4/SwiftUILiveText.zip](https://www.appcoda.com/resources/swiftui4/SwiftUILiveText.zip)