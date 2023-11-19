# SwiftUI 入门 - 文字的处理 | 精通 SwiftUI - iOS 16 版

如果你之前有使用过 UIKit，SwiftUI 的 `Text` 控制与 UIKit 中的 `UILabel` 非常相似。这是一个能够显示一 行或多行文字的视图。这个 `Text` 控制无法编辑，不过对于在屏幕上呈现唯读的资讯非常好用。举例来说，你想要在画面上呈现一个信息，你可以使用 `Text` 来实现。

本章，我将告诉你如何以 `Text` 来呈现资讯。你将会学到如何运用不同颜色、字体、背景与旋转效果来自订文字。

### 建立新项目来体验 SwiftUI

首先，开启 Xcode 并使用 *Single View App* 模板来建立一个新项目，输入项目名称，我设定为 *SwiftUIText* ，不过你可以使用任何的名称。在组织名称（ organization name），你可以设定为你的公司或组织的名字。组织识别码（organization identifier）是 App 的唯一识别码，这里我使用 *com.appcoda*，不过这里需要填入你自己的内容，如果你有一个网站，则可以将网域名称以倒过来写的方式作为设定。要使用 SwiftUI 的话，则需要在User Interface选项勾选 *SwiftUI* ，点选 *Next* 并选取一个数据夹来建立项目。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-1.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-1.png)

图 1. 建立一个新项目

储存完项目之后，Xcode 即会载入 `ContentView.swift` 档，并显示一个设计划布（ design canvas ）与预览画布（preview canvas）。如果你没有见到这个画布，你可以至 Xcode 菜单，并选取 *Editor* > *Canvas* 来启用它。

Xcode 默认会在 `ContentView.swift` 上建立一些 SwiftUI 代码。在 Xcode 14，这个预览画布会自动渲染（render）App 预览画面。Xcode 会依照你在模拟器选项（例如 iPhone 13 Pro）中的选择来将预览画面渲染在模拟器中。在旧版的 Xcode，你必须点选 *Resume* 按钮才能看到这个预览画面。

为了让程序编辑器与画布能显示，你可以隐藏项目导航器（project navigator ）与工具面板来释放更多空间。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-2.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-2.png)

图 2. 程序编辑器与画布

### 简单文字的呈现

在 `ContentView` 所产生的简单程序已经告诉你如何显示一行文字和简单图像。它使用一个 `VStack` 包装内里的 `Text` 和 `Image`。我们将在后面的章节中讨论图像和堆栈视图。 现在让我们首先关注 `Text` 的用法。

要在屏幕上显示文字，你需要初始化一个 `Text` 并将要显示的文字（例如 *Hello World*）作为参数来传递。 像这样修改 `body` 的代码：

程序初始化一个 `Text` 并将要放的文字（例如 *Hello World* ）作为参数来传传递。现在将 `body` 内的代码如下修改：

```
Text("Stay Hungry. Stay Foolish.")

```

如此，预览画布即会在屏幕上显示 *Stay Hungry. Stay Foolish.* 。这是建立一个文字视图的基本语法。你可以任意修改文字内容，画布会即时显示修改的结果。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-3.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-3.png)

图 3. 修改文字

### 修改字体与颜色

在 SwiftUI，你可以调用一些方法，也就是所谓的修饰器（*Modifiers* ）来修改属性（例如颜色）。譬如说，你想要粗体字。你可以使用名为 `fontWeight` 的修饰器，并指定你想要的字体粗细（例如 `.bold` ）：

```
Text("Stay Hungry. Stay Foolish.").fontWeight(.bold)

```

你可以使用点语法（dot syntax）来存取修饰器。当你输入一个点符号时，Xcode 则出现你可能会用到的修饰器或值。举例来说，当你在 `fontWeight` 修饰器，输入一个点符号时，你会见到不同的字体粗细选项，你可以选取 `bold` 来使用粗体字。如果你想要更粗一点，则可以选取 `heavy` 或 `black` 。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-4.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-4.png)

图 4. 挑选你喜爱的字体粗细

通过 `fontWeight` 修饰器的调用，并选取 `.bold` 的值，它会回传一个加上粗体字的视图。 SwiftUI 有趣的是，你可以进一步串连其他修饰器。譬如说，你想要大一点的粗体字，程序可以修改如下：

```
Text("Stay Hungry. Stay Foolish.").fontWeight(.bold).font(.title)

```

因为可能会串连多个修饰器，我们通常会将以上的程序写成如下的格式：

```
Text("Stay Hungry. Stay Foolish.")
    .fontWeight(.bold)
    .font(.title)

```

这个功能是一样的，不过我相信你会发现到以上的程序更容易阅读。我们将继续在本书中使用这样的程序写法。

`font` 修饰器可以让你修改字体属性。在上面的程序中，我们指定使用 *title* 字体以放大文字。SwiftUI 内有几个内建的字体样式，包括 *title* 、 *largeTitle* 、*body* 等等。如果你想要加大字体大小，则可以将 `.title` 改成 `.largeTitle` 。

*提示：你可以参考这份文件（[https://developer.apple.com/documentation/swiftui/font）来找出所有](https://developer.apple.com/documentation/swiftui/font%EF%BC%89%E6%9D%A5%E6%89%BE%E5%87%BA%E6%89%80%E6%9C%89) `font` 修饰器所支持的值。*

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-5.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-5.png)

图 5. 修改字体样式

你也可以使用 `font` 修饰器来指定字体设计，譬如说，你想要字体圆润。你可以将修饰器撰写如下：

```
.font(.system(.largeTitle, design: .rounded))

```

这里你指定使用系统字体，文字样式为 `largeTitle` ，以及 `rounded` 设计。预览画布应该会立即对修改做出反应，并显示圆润的文字。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-6.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-6.png)

图 6.使用圆字体设计

动态型态（ Dynamic Type ）是 iOS 依照使用者设定（设定 > 屏幕显示与亮度 > 文字大小）而自动调整字体大小的功能。换句话说，当你使用文字样式（例如 `.title` ），这个字体大小将会改变，你的 App 会依照使用者的偏好来自动缩放文字。

如果你想要使用一个固定大小的字体，你可以将程序修改如下：

```
.font(.system(size: 20))

```

这是告诉系统使用一个 20 点大小的固定字体。

如所述，你可以继续串连其他修饰器来客制化文字。现在我们来修改字体颜色。你可以使用 `foregroundColor` 修饰器来完成，如下所示：

```
.foregroundColor(.green)

```

`foregroundColor` 修饰器接收 `颜色` 的值。这里我们指定使用 `.green`，这个值是内建颜色，你也可以使用其他像是 `.red` 、`.purple` 等内建颜色值。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-7.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-7.png)

图 7. 修改字体颜色

虽然我们比较喜欢使用代码自订一个控制组件的属性，你也可以使用设计划布来编辑。在默认情况下，预览在 *Live* 模式下运行。 要编辑视图的属性，你必须首先切换到 *Selectable* 模式。

按住 command 键不放，并点选文字来弹出弹出菜单， 选取 *Show SwiftUI Inspector* ，然后你可以编辑 text/font 属性。 最棒的是，当你修改字体属性时，代码也会自动修改。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-8.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-8.png)

图 8. 使用 Inspect 功能来编辑文字属性

### 多行文字的处理

`Text` 默认支持多行文字，所以它可以显示一段文字，而不需要使用任何其他修饰器，将程序以下面这段替换：

```
Text("Your time is limited, so don’t waste it living someone else’s life. Don’t be trapped by dogma—which is living with the results of other people’s thinking. Don’t let the noise of others’ opinions drown out your own inner voice. And most important, have the courage to follow your heart and intuition.")
    .fontWeight(.bold)
    .font(.title)
    .foregroundColor(.gray)

```

你可以将这段文字换成你自己的内容。只要确认内容长度够长即可。做完修改后，设计划布应该会渲染一个多行文字标签。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-9.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-9.png)

图 9. 显示多行文字

要将文字置中对齐，插入 `multilineTextAlignment` 修饰器，并设定值为 `.center` 如下：

```
.multilineTextAlignment(.center)

```

在某些情况下，想要限制固定的行数的话，你可以使用 `lineLimit` 修饰器。以下为例：

```
.lineLimit(3)

```

系统默认设定是截断字尾。要修改文字的截断模式，你可以使用`truncationMode` 修饰器，并设定它的值为 `.head` 或 `.middle` ，如下所示：

```
.truncationMode(.head)

```

修改完成之后，你的文字会如下图所示。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-10.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-10.png)

图 10. 使用 .head 截断模式

先前我提到 `Text` 控制默认是显示多行文字。理由是 SwiftUI 框架默认 `lineLimit` 修饰器 为 `nil` 的值，你可以将 `.lineLimit` 设定为 `nil` 来看一下结果：

```
.lineLimit(nil)

```

### 设定间距与行距

一般默认的行距对大部分的情况而言已经够用。如果你想要改默认的设定，你可以使用 `lineSpacing` 修饰器来调整间距。

```
.lineSpacing(10)

```

如你所见，文字太靠近边缘的左侧与右侧。要赋予更多间距，你可以使用 `padding` 修饰器，为文字的每一边增加一些间距。在 `lineSpacing` 修饰器后面插入以下这行程序：

```
.padding()

```

设计划布现在的结果如下：

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-11.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-11.png)

图 11. 设定间距与文字行距

### 文字的旋转

SwiftUI 框架提供了可以轻易旋转文字的 API，你可以像这样使用 `rotateEffect` 修饰器，并传入旋转角度：

```
.rotationEffect(.degrees(45))

```

如果你在 `padding()` 后面插入以上这行程序，你将会见到文字旋转 45 度。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-12.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-12.png)

图 12. 旋转文字

默认，旋转动作会以文字视图为中心来旋转，如果你想要将文字以特定点来旋转（譬如左上角），程序的写法如下：

```
.rotationEffect(.degrees(20), anchor: UnitPoint(x: 0, y: 0))

```

我们另外传入 `anchor` 参数来指定旋转点。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-13.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-13.png)

图 13. 以文字视图的左上角来旋转

你不止可以进行 2D 旋转， SwiftUI 提供一个称作 `rotation3DEffect` 修饰器，可以让你建立 3D 效果。这个修饰器有两个参数：“旋转角度”与“旋转轴”，譬如你要建立透视文字特效，程序可以这样写：

```
.rotation3DEffect(.degrees(60), axis: (x: 1, y: 0, z: 0))

```

只要一行程序，你就可以建立星际大战透视（ Star Wars perspective ）文字

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-14.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-14.png)

图 14. 使用 3D 旋转建立惊艳的文字效果

你还可以插入以下这行程序来对透视文字建立阴影效果

```
.shadow(color: .gray, radius: 2, x: 0, y: 15)

```

这个 `shadow` 修饰器将会对文字应用阴影效果，你只需要指定颜色与阴影半径。另外，你也可以告诉系统 `x` 与 `y` 值来指定阴影位置

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-15.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-15.png)

图 15. 应用阴影效果

### 使用自订字体

在默认情况下，所有文字都是使用系统字体显示的。 假设你在 Google Fonts 上找到了一种免费字体（例如 [https://fonts.google.com/specimen/Nunito）。那如何在](https://fonts.google.com/specimen/Nunito%EF%BC%89%E3%80%82%E9%82%A3%E5%A6%82%E4%BD%95%E5%9C%A8) App 中使用自订字体？

假设你已经下载了字体文件，跟着就要将字体档加到你的 Xcode 项目中。 方法很简单，只要将字体档拖到项目导航器中并将它们放进 *SwiftUIText* 文件夹下。 在这个演示，我只添加了常规字体档（即 Nunito-Regular.ttf）。 如需使用粗体或斜体，请添加相应的文件。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-16.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-16.png)

图 16. 将字体档加到你的 Xcode 项目中

添加字体后，Xcode 即显示一个选项框。 请确保你选取 “Copy items if needed”和 *SwiftUIText* target。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-17.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-17.png)

图 17. 选择添加文件的选项

添加字体文件后，还未能直接使用字体。 Xcode 要求开发者在项目配置中设定字体。 在项目导航器中，先选择 *SwiftUIText*，然后点击 Targets 下的 *SwiftUIText*。 选择显示项目配置的 *Info* 选项。

你可以右点*Bundle*并选择*Add row*。 将 Key 设置为“Fonts provided by application”。 接下来，点击披露指示器以展开条目。 在 *item 0*，将值设置为`Nunito-Regular.ttf`，这是你刚刚添加的字体档。 如果你有多个字体档，你可以点击“+”按钮添加另一个项目。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-18.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-18.png)

图 18. 在项目配置中设置字体文件

现在你可以回到`ContentView.swift`。 要使用自定义字体，你可以将以下代码：

```
.font(.title)

```

转成:

```
.font(.custom("Nunito", size: 25))

```

上面的代码没有使用系统字体样式，而是使用 `.custom` 并指定字体名称。 字体名称可以在应用程序“Font Book”中找到。 你可以打开 Finder > Application，然后点击 *Font Book* 以启动该 App。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-19.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-19.png)

图 19. 使用自订字体

### 使用 Markdown 设置文字样式

> 
> 
> 
> Markdown 是一种简洁的标记语言，可用于将格式元素添加到纯文本文档中。 Markdown 由 [John Gruber](https://daringfireball.net/projects/markdown/) 于 2004 年开发，现已成为世界上最受欢迎的标记语言之一。
> 

SwiftUI 内置了对 Markdown 的支持。 如果你不知道 Markdown 是什么，它可让你轻松使用易于阅读的格式设置文字的样式。 要了解有关 Markdown 的资讯，可以阅读这本指南 ([https://www.markdownguide.org/getting-started/)。](https://www.markdownguide.org/getting-started/)%E3%80%82)

要使用 Markdown 设置文字样式，你只需要以 Markdown 编写文字。 `Text` 视图会自动呈现文本。 以下是一个例子：

```
Text("**This is how you bold a text**. *This is how you make text italic.* You can [click this link](https://www.appcoda.com) to go to appcoda.com")
    .font(.title)

```

如果你在 `ContentView` 中加入以上代码，Xcode 就会自动将 Markdown 文字的样式呈现出来。 要测试超链结，你必须在模拟器中运行该 App。 当你点击该链接时，iOS 会自动开启 Safari 并打开该 URL。

![SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-20.png](SwiftUI%20%E5%85%A5%E9%97%A8%20-%20%E6%96%87%E5%AD%97%E7%9A%84%E5%A4%84%E7%90%86%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%205ef32dfa6e33450481dfb140eae2725b/swiftui-text-20.png)

图 20. 使用 Markdown

### 本章小结

你喜欢以 SwiftUI 来建立使用者介面吗？我希望你会喜欢，SwiftUI 的声明式语法让程序可读性更高，且更容易理解。当你经验更多之后，你只要在 SwiftUI 写几行代码就可以建立酷炫的 3D 样式。

在本章所准备的示例档中，有最后完整的项目，您可以至以下网址下载：

- 示例项目 ( [https://www.appcoda.com/resources/swiftui4/SwiftUIText.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIText.zip) )