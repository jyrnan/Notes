# ScrollView 与 Carousel UI 的建立 | 精通 SwiftUI - iOS 16 版

从前一章中，我相信你现在应该了解如何使用堆叠建立一个复杂的UI。当然，在你能够熟练 SwiftUI 的运用之前，需要许多的练习才行。因此，在我们深入研究 ScrollView， 学习如何让视图滚动之前，我们先进行一个挑战，来作为本章的开始。你的任务是建立一个卡片视图（card view ），如图5.1 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-1.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-1.png)

图 5.1. 卡片视图

通过使用堆叠、图片与文字视图，你应该能够建立 UI。虽然我会逐步示范如何实现， 但请先花一些时间来思考如何完成这个任务，以及找出自己的解决方案。

当你完成卡片视图的建立之后，我将与你讨论 `ScrollView`，并使用卡片视图建立一个可滚动的介面，图 5.2 即是完成后的 UI。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-2.jpg](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-2.jpg)

图 5.2. 使用 ScrollView 建立一个滚动式UI

### 建立一个卡片式 UI

如果你还没有开启 Xcode，请开启它并使用 App 模板 （于 iOS 下）。来建立一个新项目。在下一个屏幕画面中，设定项目名称为“SwiftUIScrollView”（或是任何你喜欢的名称），并填入所需要的值。请确认已选取“Interface”选项中的“SwiftUI”。

到目前为止，我们在 `ContentView.swift` 档中撰写使用者介面的代码，代码撰写在这里完全没有任何问题，不过我要介绍一个整理代码的较佳方式。为了实现卡片视图， 我们另外为它建立一个单独文件，在项目导航器中，于 `SwiftUIScrollView` 按右键，并选择“New File...”，如图 5.3 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-3.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-3.png)

图 5.3. 建立一个新文件

如图 5.4 所示，在“User Interface”区块，选取“SwiftUI View”模板，然后点选“Next”来建立文件。将文件名称命名为 `CardView`，并将其储存在项目数据夹中。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-4.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-4.png)

图 5.4 选取 SwiftUI View 模板

`CardView.swift` 中的代码与 `ContentView.swift` 中的代码很相似。同样的，你可以在画布中预览 UI，如图5.5 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-5.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-5.png)

图 5.5. 就像ContentView.swift 一样，你可以在画布中预览CardView.swift

### 准备图片档

现在，我们准备要撰写卡片视图的代码。但是，首先你需要准备图档，并将其汇入素材目录。如果你不想要准备自己的图片，则可以至 [https://www.appcoda.com/resources/swiftui/SwiftUIScrollViewImages.zip](https://www.appcoda.com/resources/swiftui/SwiftUIScrollViewImages.zip) 下载示例图片档，将图片档解压缩后，选取 `Assets`，并所有图片其拖曳至素材目录。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-6.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-6.png)

图 5.6 将图片档加入素材目录

### 实现卡片视图

现在切回 `CardView.swift` 档。若你再看一下图 5.1，这个卡片视图是由两个部分组成， 视图上部是图片，而视图下部是文字叙述。

让我们从图片开始。我将使图片可调整大小，以缩放来填满屏幕，同时保持长宽比。你可以撰写代码如下：

```swift
struct CardView: View {
    var body: some View {
        Image("swiftui-button")
            .resizable()
            .aspectRatio(contentMode: .fit)
    }
}

```

如果你忘记了这两个修饰器是什么，请返回并阅读有关 `Image` 物件的章节。接下来，我们来实现文字叙述部分，你可以撰写代码如下：

```swift
VStack(alignment: .leading) {
    Text("SwiftUI")
        .font(.headline)
        .foregroundColor(.secondary)
    Text("Drawing a Border with Rounded Corners")
        .font(.title)
        .fontWeight(.black)
        .foregroundColor(.primary)
        .lineLimit(3)
    Text("Written by Simon Ng".uppercased())
        .font(.caption)
        .foregroundColor(.secondary)
}

```

显然的，你需要使用 `Text` 来建立文字视图。由于我们在叙述中实际上有三个垂直排列的文字视图，因此我们使用一个 `VStack` 来嵌入它们。对于 `VStack`，我们指定对齐方式为 `.leading`，这会将文字视图对齐堆叠视图的左侧。

这些文字的修饰器皆在有关 `Text` 物件的章节讨论过。如果你对任何修饰器有疑问的话，可以回去参考。但是，这里会特别提到有关 `.primary` 与 `.secondary` 颜色。

虽然你可在 `foregroundColor` 修饰器指定标准颜色，像是 `.black` 与 `.purple`，但 iOS 提供一套系统颜色，其中包含主色（ primary color ）、辅色（ secondary color ）、第三级色（ tertiary color ）等变化，通过使用此颜色变化，你的 App 可以轻松支持浅色模式与深色模式。举例而言，文字视图的主色默认设定为浅色模式的黑色。当 App 切换到深色模式， 主色将被调整为白色，这是由 iOS 自动调整，因此你无须另外编写写支持深色模式的代码，我们将在后面的章节中深入探讨深色模式。

为了将图片与这些文字视图垂直排列，我们使用 `VStack` 来嵌入它们，目前的布局如图 5.7 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-7.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-7.png)

图 5.7. 将图片与文字视图嵌入到 VStack 中

还没有完成，尚有几件事情需要实现。首先，如果文字叙述区块要与图片的边缘对齐，该如何做呢？

依照我们所学，我们可以在一个 `HStack` 嵌入文字视图的 `VStack`，然后我们将使用一个留白（ `Spacer` ）来将`VStack` 往左推，我们来看看是否可行。

如果你已经修改代码，如图 5.8 所示，这个文字视图的VStack 会对齐屏幕的左侧。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-8.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-8.png)

图 5.8. 文字叙述的对齐

最好是在 `HStack` 周围加入一些间距（ padding ）。插入 `padding` 修饰器如下，如图 5.9 所示 ：

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-9.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-9.png)

图 5.9. 加入一些文字叙述的间距

最后是边框部分。我们在前面的章节中讨论过如何绘制圆角边框。我们可以使用 `overlay` 修饰器，并使用`RoundedRectangle` 来画出边框。以下是完整的代码：

```swift
struct CardView: View {
    var body: some View {
        VStack {
            Image("swiftui-button")
                .resizable()
                .aspectRatio(contentMode: .fit)

            HStack {
                VStack(alignment: .leading) {
                    Text("SwiftUI")
                        .font(.headline)
                        .foregroundColor(.secondary)
                    Text("Drawing a Border with Rounded Corners")
                        .font(.title)
                        .fontWeight(.black)
                        .foregroundColor(.primary)
                        .lineLimit(3)
                    Text("Written by Simon Ng".uppercased())
                        .font(.caption)
                        .foregroundColor(.secondary)
                }

                Spacer()

            }
            .padding()
        }
        .cornerRadius(10)
        .overlay(
            RoundedRectangle(cornerRadius: 10)
                .stroke(Color(.sRGB, red: 150/255, green: 150/255, blue: 150/255, opacity: 0.1), lineWidth: 1)
        )
        .padding([.top, .horizontal])
    }
}

```

除了边框之外，我们也在顶部、左侧、右侧分别加入了一些间距。现在，你应该已经建立好卡片视图的布局，如图 5.10 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-10.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-10.png)

图 5.10. 加入边框与圆角

### 让卡片视图更具弹性

虽然目前卡片视图看起来没问题，但我们将图片与文字写死（ Hard Code）在程序中， 为了让它更具弹性，我们要重构代码。首先，在 `CardView` 声明 image、category、heading 与author 这些变量：

```
var image: String
var category: String
var heading: String
var author: String

```

接下来，将 `Image` 与 `Text` 视图的值以下列变量替代：

```
VStack {
    Image(image)
        .resizable()
        .aspectRatio(contentMode: .fit)

    HStack {
        VStack(alignment: .leading) {
            Text(category)
                .font(.headline)
                .foregroundColor(.secondary)
            Text(heading)
                .font(.title)
                .fontWeight(.black)
                .foregroundColor(.primary)
                .lineLimit(3)
            Text("Written by \(author)".uppercased())
                .font(.caption)
                .foregroundColor(.secondary)
        }

        Spacer()
    }
    .padding()
}

```

修改完成后，你将在 `CardView_Previews` 结构中看到一个错误，如图 5.11 所示。这是因为我们在 `CardView` 导入了一些变量，当使用它时，必须指定参数给它。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-11.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-11.png)

图 5.11. 调用 CardView 时缺少参数

因此，以下列代码来取代：

```
struct CardView_Previews: PreviewProvider {
    static var previews: some View {
        CardView(image: "swiftui-button", category: "SwiftUI", heading: "Drawing a Border with Rounded Corners", author: "Simon Ng")
    }
}

```

错误将可被修正，现在你已经建立了一个能接受不同图片及文字的弹性 `CardView`。

### ScrollView 的介绍

再看一下图5.2，这就是我们要实现的使用者介面。首先，你可能认为我们可以使用一个 `VStack` 来嵌入四个卡片视图。你可以切换到 `ContentView.swift` ，于 body 内插入以下的程序：

```
VStack {
    CardView(image: "swiftui-button", category: "SwiftUI", heading: "Drawing a Border with Rounded Corners", author: "Simon Ng")
    CardView(image: "macos-programming", category: "macOS", heading: "Building a Simple Editing App", author: "Gabriel Theodoropoulos")
    CardView(image: "flutter-app", category: "Flutter", heading: "Building a Complex Layout with Flutter", author: "Lawrence Tan")
    CardView(image: "natural-language-api", category: "iOS", heading: "What's New in Natural Language API", author: "Sai Kambampati")
}

```

如果你这样做的话，这些卡片视图将被挤压，以填满屏幕，因为 `VStack` 是不可滚动的，如图 5.12 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-12.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-12.png)

图 5.12 在一个VStack 中嵌入卡片视图

要让内容可以滚动，SwiftUI 提供一个名为 `ScrollView` 的视图。当内容嵌入在一个 `ScrollView` 时，它变得可以滚动，因此你需要做的是在一个 `ScrollView` 内加入一个 `VStack`，以使视图可以滚动。在预览画布中，你可以拖曳这些视图来滚动内容。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-13.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-13.png)

图 5.13. 使用 ScrollView

### 作业 #1

你的任务是加入标题（ header ）至目前的滚动视图（ scroll view ）中，结果如图 5.14 所示。如果你完全暸解了 `Vstack`与 `Hstack`，你应该有能力建立这个布局。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-14.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-14.png)

图 5.14. 作业#1

### 使用水平 ScrollView 建立轮播式 UI

默认上，`ScrollView` 允许你以垂直方向滚动内容。另外，它还支持水平方向的可滚动内容。我们来了解如何进行一些修改，以将目前的布局转换为轮播（carousel ）UI。

修改 `ContentView` 如下：

```
struct ContentView: View {
    var body: some View {

        ScrollView(.horizontal) {

            // 作业#1的代码

            HStack {
                CardView(image: "swiftui-button", category: "SwiftUI", heading: "Drawing a Border with Rounded Corners", author: "Simon Ng")
                    .frame(width: 300)
                CardView(image: "macos-programming", category: "macOS", heading: "Building a Simple Editing App", author: "Gabriel Theodoropoulos")
                    .frame(width: 300)
                CardView(image: "flutter-app", category: "Flutter", heading: "Building a Complex Layout with Flutter", author: "Lawrence Tan")
                    .frame(width: 300)
                CardView(image: "natural-language-api", category: "iOS", heading: "What's New in Natural Language API", author: "Sai Kambampati")
                    .frame(width: 300)
            }
        }

    }
}

```

我们在上列的代码中做了三个修改：

1. 我们传送一个 `.horizontal` 值，以在 `ScrollView` 中使用一个水平滚动视图。
2. 由于我们使用一个水平滚动视图，因此我们还需要将堆叠视图从 `VStack` 修改为 `HStack`。
3. 对于每个卡片视图，我们将框架的宽度设定为“300点”。这是必要的，因为要显示的图片太宽

修改代码之后，你将看到卡片视图以水平排列且可以滚动，如图 5.15 所示。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-15.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-15.png)

图 5.15 轮播 UI

### 隐藏滚动指示器

在滚动视图时，屏幕底部附近有一个滚动指示器。这个指示器默认是显示的。如果你想要隐藏它，你可以将 `ScrollView` 的代码修改如下：

```
ScrollView(.horizontal, showsIndicators: false)

```

通过指定 `showIndicators` 为 `false`，iOS 将不再显示该指示器。

### 群组视图内容

如果你再次阅读一下代码，所有的 `CardViews` 是以 `.frame` 修饰器来限制其宽度为 300 点，是否有其他简化的方式，并移除重复的代码呢？SwiftUI 框架提供了开发者群组视图（`Group` view）的功能，可以将相关内容群组起来。更重要的是，你可以将修饰器加至群组，所有嵌入群组内的视图皆能够同步做产生效果。

举例而言，你可以将 `HStack` 内的程序重写如下来完成同样的结果：

```swift
HStack {
    Group {
        CardView(image: "swiftui-button", category: "SwiftUI", heading: "Drawing a Border with Rounded Corners", author: "Simon Ng")
        CardView(image: "macos-programming", category: "macOS", heading: "Building a Simple Editing App", author: "Gabriel Theodoropoulos")
        CardView(image: "flutter-app", category: "Flutter", heading: "Building a Complex Layout with Flutter", author: "Lawrence Tan")
        CardView(image: "natural-language-api", category: "iOS", heading: "What's New in Natural Language API", author: "Sai Kambampati")
    }
    .frame(width: 300)
}

```

### 自动调整文字

如图 5.15 所示，第一张卡片的标题被截断了，该如何修正这个问题呢？ SwiftUI 中可以使用 `.minimumScaleFactor` 修饰器来自动缩小文字。你可以切换至 CardView.swift，并于 `Text（标题）`加上以下这个修饰器：

```
.minimumScaleFactor(0.5)

```

SwiftUI 会自动缩小文字来相容可用的空间。这边的值设定了视图所允许的最小缩放量。以这个例子来看，SwiftUI 能够将文字尽量缩至原来大小的 50%。

### 作业 #2

最后有一个作业，修改目前的代码，并如图 5.16 所示来重新排列。请注意，当使用者滚动卡片视图时，用户应该可以看到标题和日期。

![ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-16.png](ScrollView%20%E4%B8%8E%20Carousel%20UI%20%E7%9A%84%E5%BB%BA%E7%AB%8B%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fde4414413464948b2a2469521767625/swiftui-scrollview-16.png)

图 5.16. 视图靠上对齐

在本章所准备的示例档中，有完整的项目与作业解答可以下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIScrollView.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIScrollView.zip))