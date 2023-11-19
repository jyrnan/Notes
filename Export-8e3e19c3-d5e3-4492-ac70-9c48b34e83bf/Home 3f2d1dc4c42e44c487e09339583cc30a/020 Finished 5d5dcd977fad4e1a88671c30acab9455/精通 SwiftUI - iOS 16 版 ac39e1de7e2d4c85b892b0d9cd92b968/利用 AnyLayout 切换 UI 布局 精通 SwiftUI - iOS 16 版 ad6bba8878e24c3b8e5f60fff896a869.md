# 利用 AnyLayout 切换 UI 布局 | 精通 SwiftUI - iOS 16 版

从 iOS 16 开始，SwiftUI 推出了 `AnyLayout` 和 `Layout` 协议，让开发者构建客制化和复杂的 UI 布局。`AnyLayout` 是 layout 协议的 type-erased 实例。我们可以使用 `AnyLayout` 来创建动态 UI 布局，它可以回应使用者的互动或环境变化。

在这章节，我们会看看如何使用 `AnyLayout` 来切换垂直和水平布局。

### 如何使用 AnyLayout

首先，让我们用 *App* 模板创建一个新的 Xcode 项目，并为项目命名，我会把项目命名为 `SwiftUIAnyLayout`。我们会构建一个简单的示例 App，在使用者点击堆叠视图时切换 UI 布局。UI 布局在不同方向看起来会是这样的：

![%E5%88%A9%E7%94%A8%20AnyLayout%20%E5%88%87%E6%8D%A2%20UI%20%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ad6bba8878e24c3b8e5f60fff896a869/swiftui-anylayout-1.png](%E5%88%A9%E7%94%A8%20AnyLayout%20%E5%88%87%E6%8D%A2%20UI%20%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ad6bba8878e24c3b8e5f60fff896a869/swiftui-anylayout-1.png)

图 45.1. 使用 AnyLayout 在垂直和水平堆栈之间切换

在开始时，示例 App 利用 `VStack` 把三个图像垂直排列。当使用者点击堆叠视图时，就会变成水平堆叠。我们可以这样使用 `AnyLayout` 来实现：

```
struct ContentView: View {
    @State private var changeLayout = false

    var body: some View {
        let layout = changeLayout ? AnyLayout(HStackLayout()) : AnyLayout(VStackLayout())

        layout {
            Image(systemName: "bus")
                .font(.system(size: 80))
                .frame(width: 120, height: 120)
                .background(in: RoundedRectangle(cornerRadius: 5.0))
                .backgroundStyle(.green)
                .foregroundColor(.white)

            Image(systemName: "ferry")
                .font(.system(size: 80))
                .frame(width: 120, height: 120)
                .background(in: RoundedRectangle(cornerRadius: 5.0))
                .backgroundStyle(.yellow)
                .foregroundColor(.white)

            Image(systemName: "scooter")
                .font(.system(size: 80))
                .frame(width: 120, height: 120)
                .background(in: RoundedRectangle(cornerRadius: 5.0))
                .backgroundStyle(.indigo)
                .foregroundColor(.white)

        }
        .animation(.default, value: changeLayout)
        .onTapGesture {
            changeLayout.toggle()
        }
    }
}

```

我们定义了一个 layout 变量，来保存 `AnyLayout` 的实例。这个 layout 会根据 `changeLayout` 的数值，来切换水平和垂直 layout。`HStackLayout`（或 `VStackLayout`）的行为与 `HStack`（或 `VStack`）类似；但因为它符合 `Layout` 协议，我们就可以在 conditional layout 中使用它。

我们还可以把动画附加到 layout，来动画化布局的改变。现在，当我们点击堆叠视图时，它就会切换垂直或水平布局。

## 根据装置的方向切换 layout

现在，示例 App 让使用者点击堆叠视图来切换 layout。在某些 App 中，我们可能会想根据装置方向和屏幕大小来切换 layout。在这个情况下，就可以利用 `.horizontalSizeClass` 变量来捕捉装置方向的改变。

```
@Environment(\.horizontalSizeClass) var horizontalSizeClass

```

然后，让我们如此修改 `layout` 变量：

```
let layout = horizontalSizeClass == .regular ? AnyLayout(HStackLayout()) : AnyLayout(VStackLayout())

```

举个例子，如果我们把 iPhone 14 Pro Max 转为横向，layout 就会切换为横向堆叠视图。

![%E5%88%A9%E7%94%A8%20AnyLayout%20%E5%88%87%E6%8D%A2%20UI%20%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ad6bba8878e24c3b8e5f60fff896a869/swiftui-anylayout-2.png](%E5%88%A9%E7%94%A8%20AnyLayout%20%E5%88%87%E6%8D%A2%20UI%20%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ad6bba8878e24c3b8e5f60fff896a869/swiftui-anylayout-2.png)

图 45.2. 当设备处于横向时切换到水平堆栈视图

在大多数情况下，我们会使用 SwiftUI 内建的 layout container 来创建 layout，像是 `HStackLayout` 和 `VStackLayout`。但如果这些 layout container 无法实现我们需要的 layout 类型，该怎么办呢？iOS 16 引入的 Layout 协议就让我们可以定义自己客制化的 layout。我们只需要创建一个符合 `Layout` 协议的类型，并实现以下所需的方法，来定义一个客制化的 layout container：

- `sizeThatFits(proposal:subviews:cache:)` - 这个方法会报告合成 layout 视图的大小。
- `placeSubviews(in:proposal:subviews:cache:)` - 这个方法会为 container 的子视图分配位置。

### 总结

推出了 `AnyLayout` 后，我们只需要几行代码就可以客制化或修改 UI layout，这绝对可以帮助我们构建更优雅和吸引的 UI。在这篇文章的示例 App 中，大家都学会了如何根据屏幕方向切换 layout。其实同样的技术也可以应用于其他情况中，例如是 Dynamic Type 的大小。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- [https://www.appcoda.com/resources/swiftui4/SwiftUIAnyLayout.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIAnyLayout.zip)