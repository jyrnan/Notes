# 如何建立图像轮播（Image Carousel） | 精通 SwiftUI - iOS 16 版

Carousel （轮播）是你在大多数手机和 Web App中看到最常见的 UI 模式之一。 有些人称它为图像滑块（Image Slider）或旋转器（Rotator）。 但是，无论你如何称呼它，轮播的设计都旨在在有限的屏幕空间显示一组数据。 例如，图像轮播会显示数组或集合中的其中一个图片，使用者可以滑动屏幕来浏览图像集的其他图片。 Instagram 也有用这种轮播设计，当要显示多张图片时，使用者就要滑动屏幕来浏览图像集。 另外，你也可以在 Apple 的 Music App 和 App Store App 找到类似的轮播应用。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-1.jpg](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-1.jpg)

图 27.1. Music, App Store 和 Instagram app 都有用轮播设计

在本章中，你将学习如何实现图像轮播，当然是用 SwiftUI。有多种方法可以实现轮播，其中一种方法是整合 UIKit 内的 `UIPageViewController` 。 但是，我们将探讨另一种方法，并完全以 SwiftUI 框架来建立轮播。

让我们开始吧。

### 旅行示例 App 简介

像其他章节一样，我通过构建一个示例 App来引导你了解当中的技术问题。 今次的 App 以轮播形式显示不同的旅行目的地。 要浏览行程，用户可以向右滑动查看下一个目的地，或者向左滑动查看之前的行程。 为了使这个示例 App更具吸引力，使用者可以点击一个目的地来查看它的详细数据。 因此，除了实现轮播外，你还会学习一些动画技术。 图 27.2 显示了示例 App的一些屏幕截图。想知它实际的运作，你可以到 [https://link.appcoda.com/carousel-demo](https://link.appcoda.com/carousel-demo) 上观看示范影片。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-2.jpg](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-2.jpg)

图 27.2. 示例 App

为了节省你的时间并专注于开发轮播，我已为建立了一个Starter项目。 请从 [https://www.appcoda.com/resources/swiftui4/SwiftUICarouselStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUICarouselStarter.zip) 下载并解压文件。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-3.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-3.png)

图 27.3. Starter 项目

Starter项目具有以下特点：

1. 它已包含所需的图像并加至Assets 中。
2. `ContentView.swift` 是 Xcode 自动建立的默认视图。
3. `Trip.swift` 包含 `Trip` 结构，它代表App内的旅行目的地。 出于测试目的，该文件还包含一些测试数据（`sampleTrips`）。 你可以随便修改这些测试数据。
4. `TripCardView.swift` 已建立了卡片视图的 UI。 每个卡片视图都旨在显示目的地的图像。 `isShowDetails` 绑定控制文字标签的外观。 当 `isShowDetails` 设定为 true 时，标签将被隐藏。

### 滚动视图的问题

那么，你将如何在 SwiftUI 中实现轮播呢？ 你或会想到使用滚动视图来创建轮播。 可能你会像这样在 `ContentView.swift` 中加入以下代码：

```
struct ContentView: View {

    @State private var isCardTapped = false

    var body: some View {
        GeometryReader { outerView in
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(alignment: .center) {
                    ForEach(sampleTrips.indices, id: \.self) { index in
                        GeometryReader { innerView in
                            TripCardView(destination: sampleTrips[index].destination, imageName: sampleTrips[index].image, isShowDetails: self.$isCardTapped)
                        }
                        .padding(.horizontal, 20)
                        .frame(width: outerView.size.width, height: 450)
                    }
                }
            }
            .frame(width: outerView.size.width, height: outerView.size.height, alignment: .leading)
        }
    }
}

```

在上面的代码中，我们嵌入了一个带有水平 ScrollView 的 HStack 来创建图像滑块。 在 `HStack` 中，我们为每个行程创建一个 `TripCardView`。 为了更好地控制卡片大小，我们有两个 GeometryReader：*outerView* 和 *innerView*，其中*outerView*存放装置屏幕的大小，*innerView*包裹着卡片视图以控制其大小。 如果你不明白什么是`GeometryReader`，请先读第26章。

这看起来很简单，对吧？ 如果你在预览画面中运行代码，它应该会产生一个水平滚动视图。 你可以滑动屏幕浏览所有卡片视图。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-4.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-4.png)

图 27.4. 滑动屏幕浏览图片

这是否意味着我们已经实现了轮播？ 还未。 有两个主要问题：

1. 当前的设计并不支持分页（Paging）。 换句话说，你可以滑动屏幕以无间断地滚动内容。 滚动视图可以在任何位置停止。 举例，看一下图 27.4，使用者可以按制在两个卡片视图之间停止滚动。 这不是我们想要的特点， 当滚动停止，屏幕只会显示其中一个内容视图。
2. 当一个卡片视图被点击时，我们需要找出它的索引并在一个单独的视图中显示它的细节。 问题是要怎样得知使用者点击了哪个卡片视图？

这两个问题都与内置的`ScrollView`有关。 UIKit 版本的滚动视图支持分页。 然而，Apple 并未将该功能引入 SwiftUI 框架的 `ScrollView`。 为了解决这个问题，我们需要构建我们自己的支持分页（Paging）的滚动视图。

### 使用 HStack 和 DragGesture 建立轮播

起初，你可能认为很难开发我们自己的滚动视图。 但实际上，这并不难。 如果你了解 `HStack` 和 `DragGesture` 的用法，就可以构建一个支持分页的滚动视图。

想法是将所有卡片视图（即行程）布局在水平堆栈（`HStack`）中。 `HStack` 要有足够长度以容纳所有卡片视图，但在任何时候只显示一个卡片视图。 在默认情况下，水平堆栈是不支持滚动的。 因此，我们需要将拖动手势识别器附加到堆栈视图并自行处理拖动。 图 27.5 以图解方式说明了如何实现水平滚动视图。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-5.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-5.png)

图 27.5. 了解如何使用 HStack 和 DragGesture 创建水平滚动视图

### 自建ScrollView

现在让我们看看如何将这个想法转化为代码。 请容忍一下，因为你需要多次修改代码。 我想将每一个步骤都展现出来。 打开 `Content.swift` 并像这样修改 `body`：

```
var body: some View {
    HStack {
        ForEach(sampleTrips.indices) { index in
            TripCardView(destination: sampleTrips[index].destination, imageName: sampleTrips[index].image, isShowDetails: self.$isCardTapped)
        }
    }
}

```

在上面的代码，我们首先在 `HStack` 中布置所有卡片视图。 水平堆栈会尽力在可用的屏幕空间容纳所有卡片视图。 你应该在预览画面中看到类似于图 27.6 的画面。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-6.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-6.png)

图 27.6. 挤压所有卡片视图以填满整个屏幕

这显然不是我们想要构建的水平堆栈。 我们希望每个卡片视图都占据屏幕的宽度。 为此，我们必须将 HStack 嵌入在 GeometryReader 中以读取屏幕尺寸。 像这样修改 `body` 中的代码：

```
var body: some View {
    GeometryReader { outerView in
        HStack {
            ForEach(sampleTrips.indices, id: \.self) { index in
                GeometryReader { innerView in
                    TripCardView(destination: sampleTrips[index].destination, imageName: sampleTrips[index].image, isShowDetails: self.$isCardTapped)
                }
                .frame(width: outerView.size.width, height: 500)
            }
        }
        .frame(width: outerView.size.width, height: outerView.size.height)
    }
}

```

`outerView` 参数为我们提供了屏幕的宽度和高度，而`innerView` 参数可以让我们更好地控制卡片视图的大小和位置。

在上面的程码中，我们将`.frame`修饰器附加到卡片视图并将其宽度设定为屏幕宽度（即`outerView.size.width`）。 这样可以确保每个卡片视图占据整个屏幕。 对于卡片视图的高度，我们将其设置为 500 点以使其更小一些。 进行修改后，你应该会看到“London”图像的卡片视图。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-7.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-7.png)

图 27.7. 水平堆栈现在只显示一个卡片视图

为什么是“London”卡片视图？ 如果你将预览模式转至 Selectable，预览画面应显示如图 27.8 所示的内容。我们在 `sampleTrips` 数组中有 13 个项目。 由于每个卡片视图的宽度都等于屏幕宽度，因此水平堆栈视图必须扩展到屏幕之外。 碰巧“London”卡片视图是数组的中间（第 7 个）项。 这就是你看到“London”卡片视图的原因。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-8.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-8.png)

图 27.8. 这个水平堆栈其实包含 13 个卡片视图

### 修改对齐方式

那么，我们如何才能显示数组的第一项而不是中间（第 7 个）项？ 诀窍是将`.frame`修饰器附加到`HStack`，对齐设定为`.leading`，如下所示：

```
.frame(width: outerView.size.width, height: outerView.size.height, alignment: .leading)

```

SwiftUI 默认的对齐方式为`.center`。 这就是为什么水平视图的第 7 个项目会显示在屏幕上的原因。 将`alignment`修改为 `.leading` 后，你就会看到第一张图。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-9.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-9.png)

图 27.9. 现在显示第一张图

如果你想了解`alignment`如何影响水平堆栈视图，可以将其值修改为 `.center` 或 `.trailing` 试一下。 图 27.10 显示了堆栈视图在不同对齐设置下的样子。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-10.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-10.png)

图 27.10. 堆栈视图在不同对齐设置下的样子

你是否注意到每个卡片视图之间的差距？ 这也与`HStack`的默认值有关。 要减少间距，你可以修改 `HStack` 并将`spacing`参数设定为`0`，如下所示：

```
HStack(spacing: 0)

```

### 加入 padding

想做得好一点，你可以为图像添加padding， 我认为这会使卡片视图看起来更好。 加入以下代码并将它附加到包裹卡片视图的`GeometryReader`（在`.frame（width：outerView.size.width，height：500）`之前）：

```
.padding(.horizontal, self.isCardTapped ? 0 : 20)

```

虽然现在谈论详细视图的实现还是有点过早，但我们为`padding`添加了一个条件。 当使用者点击卡片视图时，水平填充将被删除。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-11.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-11.png)

图 27.11. 加入 padding

### 让卡片视图滚动

现在我们已经建立了一个能显示第一个卡片视图的水平堆栈，下一个问题是我们如何移动堆栈以显示特定卡片图？

这只是简单的数学！ 卡片视图的宽度等于屏幕的宽度。 假设屏幕宽度为 300 点，我们要显示第三个卡片视图，我们可以将堆栈向左移动 600 点（300 x 2）。 图 27.12 显示了结果。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-12.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-12.png)

图 27.12. 将堆栈向左移动

要将上面的描述翻译成代码，我们首先声明一个状态变量来存放显示于屏幕卡片视图的索引：

```
@State private var currentTripIndex = 2

```

在默认情况下，我想显示第三张卡片视图。 这就是我将 `currentTripIndex` 变量设定为 2 的原因。你可以修改它为其他值。

要将堆栈向左移动，我们可以将 `.offset` 修饰器附加到 `HStack`，如下所示：

```
.offset(x: -CGFloat(self.currentTripIndex) * outerView.size.width)

```

`outerView` 的宽度实际上是屏幕的宽度。 为了显示第三个卡片视图，如前所述，我们需要将堆栈移动 “2 x 屏幕宽度”。 这就是我们将 `currentTripIndex` 与 `outerView` 的宽度相乘的原因。 如果水平的 offset 值为负数的话，就会将堆栈视图向左移动。

进行修改后，你应该会在预览画面中看到“Amsterdam”卡片视图。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-13.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-13.png)

图 27.13. 预览画面显示Amsterdam卡片视图

### 添加拖动手势

现在，我们已可通过改变 `currentTripIndex` 的值来改变屏幕显示的卡片视图。 请记住，水平堆栈（HStack）不允许用户拖动视图。 这就是我们要自己建立此功能的原因。以下的内容是假设你已经了解 SwiftUI 手势的运作原理。 如果你不理解手势或`@GestureState`，请先阅读第 17 章。

堆栈是这样处理使用者的拖动手势：

1. 使用者可以点击图像并开始拖动。
2. 使用者可以向左或右移动视图。
3. 当拖动距离超过一定数值时，堆栈会根据拖动方向移动到下一个或上一个卡片视图。
4. 否则，堆栈视图返回原本的位置并显示当前卡片视图。

要将上面的描述翻译成代码，我们首先声明一个变量来保存拖动的偏移值：

```
@GestureState private var dragOffset: CGFloat = 0

```

接下来，我们将`.gesture`修饰器附加到`HStack`并像这样初始化`DragGesture`：

```
.gesture(
    !self.isCardTapped ?

    DragGesture()
        .updating(self.$dragOffset, body: { (value, state, transaction) in
            state = value.translation.width
        })
        .onEnded({ (value) in
            let threshold = outerView.size.width * 0.65
            var newIndex = Int(-value.translation.width / threshold) + self.currentTripIndex
            newIndex = min(max(newIndex, 0), sampleTrips.count - 1)

            self.currentTripIndex = newIndex
        })

    : nil
)

```

当你拖动水平堆栈时，App会自动调用`updating`函数。 我们将水平拖动距离存放到 `dragOffset` 变量中。 当拖动结束时，就检查拖动距离是否超过屏幕宽度的 65%，并计算新的索引（index）。 当计算了 `newIndex`，我们会先确定它是否在 `sampleTrips` 数组的范围内。 最后，我们将 `newIndex` 的值指定至 `currentTripIndex`。 然后， SwiftUI 会自动修改 UI 并显示相应的卡片视图。

请注意，我们有一个启用拖动手势的条件。 点击卡片视图时，没有手势识别器。

要在拖动过程中移动堆栈视图，我们必须再进行一项修改。 附加一个额外的 `.offset` 修饰器到 `HStack`（紧跟在前一个 .offset 之后），如下所示：

```
.offset(x: self.dragOffset)

```

在这里，我们将堆栈视图的水平偏移值修改为拖动偏移值。 现在你可以准备测试App。 在模拟器或预览画面中运行App，你应该能够拖动堆栈视图。 当你的拖动超过一定距离时，堆栈视图会显示下一个行程。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-14.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-14.png)

图 27.14. 拖动堆栈视图

### 加入动画

为了提升使用者体验，我想在App从一个卡片视图移动到另一个卡片视图时添加一个漂亮的动画。 首先，修改以下这行代码：

```
.frame(width: outerView.size.width, height: 500)

```

将它改写成：

```
.frame(width: outerView.size.width, height: self.currentTripIndex == index ? (self.isCardTapped ? outerView.size.height : 450) : 400)

```

我们将屏幕显示的卡片视图稍为放大一点。 另外，将 `.opacity` 修饰器附加到卡片视图，如下所示：

```
.opacity(self.currentTripIndex == index ? 1.0 : 0.7)

```

除了卡片视图的高度，我们还想为可见和隐藏的卡片视图设置不同的`opacity`值。 以上这些变化还没有动画化， 现在将以下代码加入到外部视图的 GeometryReader 中：

```
.animation(.interpolatingSpring(mass: 0.6, stiffness: 100, damping: 10, initialVelocity: 0.3), value: dragOffset)

```

就是这样， SwiftUI 会自动为卡片视图的移动加入动画。 在预览画布中运行App以测试修改。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-15.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-15.png)

图 27.15. 加入动画效果

### 加入标题

既然我们已经构建了图像轮播，现在就再为这 App 实现详细视图， 让我们由添加标题开始。

按Command键再点击外部视图（outerView）的`GeometryReader`并选择*embed in ZStack*。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-16.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-16.png)

图 27.16. 选择 Embed in Stack

接下来，在 `ZStack` 的开头插入以下代码：

```
VStack(alignment: .leading) {
    Text("Discover")
        .font(.system(.largeTitle, design: .rounded))
        .fontWeight(.black)

    Text("Explore your next destination")
        .font(.system(.headline, design: .rounded))
}
.frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity, alignment: .topLeading)
.padding(.top, 25)
.padding(.leading, 20)
.opacity(self.isCardTapped ? 0.1 : 1.0)
.offset(y: self.isCardTapped ? -100 : 0)

```

上面的代码应该不用解释吧，但我想特别指出两行代码。 `.opacity` 和 `.offset` 都是可有可无的。 `.opacity` 修饰器的目的是在点击卡片视图时隐藏标题，而`.offset`会进一步提升使用者体验。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-17.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-17.png)

图 27.17. 加入标题

### 练习：处理详细视图

让我们通过一个练习来开始详细视图的实现。 我假设你对 SwiftUI 有一定的经验，并且应该能够建立图 27.18 的详细视图。你可以建立一个名为`TripDetailView.swift`的文件并在那里编写代码。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-18.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-18.png)

图 27.18. 行程详细视图

为简单起见，评级和描述都只是一些 hardcode 的数据。 *Book Now* 按钮也是如此，它不能运作的。 这个详细视图的框架大概就是这样：

```
struct TripDetailView: View {
    let destination: String

    var body: some View {
        .
        .
        .
    }
}

```

请花一些时间来开发详细视图， 我将在后面的部分中介绍我的解决方案。

### 实现行程详细视图

你是否能够开发详细视图？ 我希望能完成这个练习。 让我讲解一下我的解决方案。 首先，使用 *SwiftUI View* 模板建立一个名为 `TripDetailView.swift` 的新文件。

接下来，像这样写 `TripDetailView` 结构：

```
struct TripDetailView: View {
    let destination: String

    var body: some View {
        GeometryReader { geometry in
            ScrollView {
                ZStack {
                    VStack(alignment: .leading, spacing: 5) {

                        VStack(alignment: .leading, spacing: 5) {
                            Text(self.destination)
                                .font(.system(.title, design: .rounded))
                                .fontWeight(.heavy)

                            HStack(spacing: 3) {
                                ForEach(1...5, id: \.self) { _ in
                                    Image(systemName: "star.fill")
                                        .foregroundColor(.yellow)
                                        .font(.system(size: 15))
                                }

                                Text("5.0")
                                    .font(.system(.headline))
                                    .padding(.leading, 10)
                            }

                        }
                        .padding(.bottom, 30)

                        Text("Description")
                            .font(.system(.headline))
                            .fontWeight(.medium)

                        Text("Growing up in Michigan, I was lucky enough to experience one part of the Great Lakes. And let me assure you, they are great. As a photojournalist, I have had endless opportunities to travel the world and to see a variety of lakes as well as each of the major oceans. And let me tell you, you will be hard pressed to find water as beautiful as the Great Lakes.")
                            .padding(.bottom, 40)

                        Button(action: {
                            // tap me
                        }) {
                            Text("Book Now")
                                .font(.system(.headline, design: .rounded))
                                .fontWeight(.heavy)
                                .foregroundColor(.white)
                                .padding()
                                .frame(minWidth: 0, maxWidth: .infinity)
                                .background(Color(red: 0.97, green: 0.369, blue: 0.212))
                                .cornerRadius(20)
                        }
                    }
                    .padding()
                    .frame(width: geometry.size.width, height: geometry.size.height, alignment: .topLeading)
                    .background(Color.white)
                    .cornerRadius(15)

                    Image(systemName: "bookmark.fill")
                        .font(.system(size: 40))
                        .foregroundColor(Color(red: 0.97, green: 0.369, blue: 0.212))
                        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity, alignment: .topTrailing)
                        .offset(x: -15, y: -5)
                }
                .offset(y: 15)
            }
        }

    }
}

```

基本上，我们将整个内容嵌入到滚动视图中。 在滚动视图中，我们使用 ZStack 来布局内容和书签图像。 由于 `TripDetailView` 需要提供`destination` 参数才能正常运作，因此你需要像这样修改预览代码：

```
struct TripDetailView_Previews: PreviewProvider {
    static var previews: some View {
        TripDetailView(destination: "London").background(Color.black)
    }
}

```

另外，我还将背景颜色修改为*黑色*，以便我们可以看到详细视图的圆角。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-19.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-19.png)

图 27.19. 预览图

### 弹出详细视图

现在让我们回到 `ContentView.swift` ，我们要作少少修改。 当使用者点击卡片视图时，我们将显示带有动画过渡的详细视图。 由于`ContentView`有一个 ZStack，因此我们很容易与详细视图整合。

在 `ZStack` 中插入以下代码：

```
if self.isCardTapped {
    TripDetailView(destination: sampleTrips[currentTripIndex].destination)
        .offset(y: 200)
        .transition(.move(edge: .bottom))
        .animation(.interpolatingSpring(mass: 0.5, stiffness: 100, damping: 10, initialVelocity: 0.3))

    Button(action: {
        self.isCardTapped = false
    }) {
        Image(systemName: "xmark.circle.fill")
            .font(.system(size: 30))
            .foregroundColor(.black)
            .opacity(0.7)
            .contentShape(Rectangle())
    }
    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity, alignment: .topTrailing)
    .padding(.trailing)

}

```

`TripDetailView` 只有在卡片视图被点击时才会出现， 而详细视图将从屏幕底部出现并以动画方式向上移动。 这就是我们将 `.transition` 和 `.animation` 修饰器附加到详细视图的原因。 为了让使用者能关闭详细视图，我们还添加了一个关闭按钮。该按钮显示在屏幕的右上角。 如果你不确定在那里加入上面的代码，请参考图 27.20。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-20.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-20.png)

图 27.20. 就在这里加入代码

还未完成喔！因为我们还没有加入代码侦测点击手势。 将 `.onTapGesture` 函数附加到卡片视图，如下所示：

```
.onTapGesture {
    self.isCardTapped = true
}

```

当使用者点击卡片视图时，我们只需将 `isCardTapped` 状态变量修改为 `true`。 运行App并点击任何卡片视图，它应显示详细视图。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-21.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-21.png)

图 27.21. 加入 onTapGesture

成功了！ 但是，动画效果并不太好。 当详细视图弹出时，卡片视图会变大一点，这是下面代码做成的：

```
.frame(width: outerView.size.width, height: self.currentTripIndex == index ? (self.isCardTapped ? outerView.size.height : 450) : 400)

```

为了让动画看起来更流畅，让我们在详细视图出现时将图像向上移动。 将 `.offset` 修饰器附加到 `TripCardView`：

```
.offset(y: self.isCardTapped ? -innerView.size.height * 0.3 : 0)

```

我将垂直 offset 设定为卡片视图高度的 30%， 你可以自由修改该值。 现在再次运行App，你应该会看到更流畅的动画。

![%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-22.png](%E5%A6%82%E4%BD%95%E5%BB%BA%E7%AB%8B%E5%9B%BE%E5%83%8F%E8%BD%AE%E6%92%AD%EF%BC%88Image%20Carousel%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ee691321eefa4d01805f45caed06dce8/swiftui-carousel-22.png)

图 27.22. 在 TripCardView 添加offset修饰器

### 总结

酷！ 你已经构建了一个支持分页的客制化滚动视图，并学习了如何加入过场动画。 该技术不限于图像轮播。 实际上，你可以修改代码以建立一组onboarding screens。 我希望你喜欢本章所介绍的东西，并将它应用到你的下一个App。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUICarousel.zip](https://www.appcoda.com/resources/swiftui4/SwiftUICarousel.zip)