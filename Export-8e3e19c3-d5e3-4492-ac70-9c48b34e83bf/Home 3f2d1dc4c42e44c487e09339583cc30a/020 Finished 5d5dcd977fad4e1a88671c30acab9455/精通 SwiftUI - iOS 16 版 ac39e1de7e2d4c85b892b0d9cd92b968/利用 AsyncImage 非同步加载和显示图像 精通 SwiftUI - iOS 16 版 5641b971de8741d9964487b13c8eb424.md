# 利用 AsyncImage 非同步加载和显示图像 | 精通 SwiftUI - iOS 16 版

在 WWDC 2021 中，Apple 公布了 SwiftUI 框架的大量新功能，使开发者的开发 App 变得更得心应手。 `AsyncImage` 绝对是 iOS 15 其中一个最令人期待的功能。如果你的App需要从远程服务器加载和显示图像，这个新视图应该可以让你不必编写自己的代码来处理非同步 (asynchronous) 下载。

`AsyncImage` 是用于非同步加载和显示远程图像的内置视图。 你只需要告诉它图像的 URL， `AsyncImage` 就自动抓取远程图像并将其显示在屏幕上。

在本章中，我将向你讲解如何在 SwiftUI 项目中使用 `AsyncImage`。

### AsyncImage 的基本用法

使用 AsyncImage 的最简单方法是像这样指定图像的 URL：

```
AsyncImage(url: URL(string: imageURL))

```

AsyncImage 就会自动连接到所指定的 URL 以非同步形式从远程服务器加载图像。 当图像尚未能够显示时，它还会自动将占位符呈现为灰色。 完全下载后，`AsyncImage` 就会以其固有大小显示图像。

![%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-1.png](%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-1.png)

图 36.1. 使用 AsyncImage

假设你已经在 Xcode 中建立了一个新的 SwiftUI 项目，你可以像这样修改 `ContentView` 中的代码来试一试：

```
struct ContentView: View {
    let imageURL = "https://link.appcoda.com/testimage"

    var body: some View {
        AsyncImage(url: URL(string: imageURL))
    }
}

```

修改后，你应该会看到一个灰色的占位符。数秒后，就会显示下载的图像。

如果要使图像变小或变大，可以将缩放值传给 `scale` 参数，如下所示：

```
AsyncImage(url: URL(string: imageURL), scale: 2.0)

```

要缩小图像，你就指定一个大于 1.0 的值。 相反，小于 1 的值就会把图像变大。

![%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-2.png](%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-2.png)

图 36.2. 缩小图像

### 客制化图像尺寸和占位符

`AsyncImage` 也提供了另一个构造函数 (constructor)，让开发者可以进一步客制化图像：

```
init<I, P>(url: URL?, scale: CGFloat, content: (Image) -> I, placeholder: () -> P)

```

我们可以使用上面的 `init` 初始化 `AsyncImage`，来调整及缩放下载好的图像。更重要的是，我们可以实现自己的占位符。看看以下的示例代码片段：

```
AsyncImage(url: URL(string: imageURL)) { image in
    image
        .resizable()
        .scaledToFill()
} placeholder: {
    Color.purple.opacity(0.1)
}
.frame(width: 300, height: 500)
.cornerRadius(20)

```

在上面的代码中，`AsyncImage` 提供了下载好的图像。然后，我们应用 `resizable()` 和 `scaledToFill()` 修饰符来调整图像尺寸，并把 `AsyncImage` 视图的尺寸限制为 300×500 points。

`placeholder` 参数让我们可以创建自己的占位符，来取代默认的占位符。在以下示例中，我们把占位符设置为浅紫色。

![%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-3.png](%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-3.png)

图 36.3. 自订占位符

### 处理非同步操作的不同阶段 (Phase)

如果你需要更精准操作非同步下载，`AsyncImage` 视图提供了另一个`init`函数：

```
init(url: URL?, scale: CGFloat, transaction: Transaction, content: (AsyncImagePhase) -> Content

```

`AsyncImagePhase` 是一个枚举 (enum)，用于跟踪下载操作的当前阶段。 你可以针对每个阶段提供详细的实现，包括*empty*、*failure* 和*success*。

看看以下示例代码片段：

```
AsyncImage(url: URL(string: imageURL)) { phase in
    switch phase {
    case .empty:
        Color.purple.opacity(0.1)
    case .success(let image):
        image
            .resizable()
            .scaledToFill()
    case .failure(_):
        Image(systemName: "exclamationmark.icloud")
            .resizable()
            .scaledToFit()
    @unknown default:
        Image(systemName: "exclamationmark.icloud")
    }
}
.frame(width: 300, height: 300)
.cornerRadius(20)

```

在 *Empty* 的情况下，表示图像未加载，于是我们会显示一个占位符。在 *success* 的情况下，我们就会应用几个修饰符，并将图像显示在屏幕上。在 *failure* 的情况下，我们就可以在出现错误时提供备用视图 (alternate view)。在上面的代码中，我们就这样显示了一个系统图像。

![%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-4.png](%E5%88%A9%E7%94%A8%20AsyncImage%20%E9%9D%9E%E5%90%8C%E6%AD%A5%E5%8A%A0%E8%BD%BD%E5%92%8C%E6%98%BE%E7%A4%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205641b971de8741d9964487b13c8eb424/asyncimage-4.png)

图 36.4. 当图档下载遇到问题时，App会显示一个系统图像

### 利用 Transaction 来添加动画

同一个 `init` 可以让我们在阶段修改时指定可选 `transaction`。例如，以下代码片段在 `transaction` 参数中指定使用 `spring` 类型的动画：

```
AsyncImage(url: URL(string: imageURL), transaction: Transaction(animation: .spring())) { phase in
    switch phase {
    case .empty:
        Color.purple.opacity(0.1)

    case .success(let image):
        image
            .resizable()
            .scaledToFill()

    case .failure(_):
        Image(systemName: "exclamationmark.icloud")
            .resizable()
            .scaledToFit()

    @unknown default:
        Image(systemName: "exclamationmark.icloud")
    }
}
.frame(width: 300, height: 500)
.cornerRadius(20)

```

如此一来，在下载图像后，我们就会看到淡入 (fade-in) 动画。我们无法在预览版面中测试代码，请在模拟器中测试代码以查看动画。

你也可以把 `transition` 修饰符附加到 `image` 视图：

```
case .success(let image):
    image
        .resizable()
        .scaledToFill()
        .transition(.slide)

```

这样在显示结果图像时，就会看到滑入 (slide-in) 动画。

### 总结

在本章中，我们介绍了一个非常好用的“AsyncImage”视图。有了这个新功能，下载和显示远程图像（Remote Image）时都变得非常容易。 你只需要指定图像的 URL，AsyncImage视图就会为你完成所有繁重的工作。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUIAsyncImage.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIAsyncImage.zip)