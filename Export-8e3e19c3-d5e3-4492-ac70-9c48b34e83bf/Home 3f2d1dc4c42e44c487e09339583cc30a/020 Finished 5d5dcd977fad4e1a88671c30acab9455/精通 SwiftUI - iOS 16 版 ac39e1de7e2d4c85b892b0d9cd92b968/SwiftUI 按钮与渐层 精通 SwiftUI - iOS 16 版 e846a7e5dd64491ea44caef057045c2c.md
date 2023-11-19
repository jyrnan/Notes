# SwiftUI 按钮与渐层 | 精通 SwiftUI - iOS 16 版

> 
> 
> 
> 按钮可启动App 的特定动作，其有自订背景，并可加入一个标题或一个图示。这个系统为大部分的使用情况提供许多预定义的按钮样式。你也可以充分设计自订按钮。
> 
> - Apple 的官方文件 ([https://developer.apple.com/design/human-interface-guidelines/ios/controls/buttons/](https://developer.apple.com/design/human-interface-guidelines/ios/controls/buttons/))

我想应该不需要去解释什么是按钮，它是一个非常基本的 UI 控制组件，你可以在所有的 App 中看到它，按钮可处理使用者的触控动作，并触发特定的动作。倘若你之前有学过 iOS 程序设计的话，SwiftUI 的 Button 与 UIKit 的UIButton 非常相似，但更具弹性与可客制化，待会你将会了解我的意思。在本章中，我将详细介绍 SwiftUI 控制组件，你将学到下列的技巧：

1. 如何建立一个简单的按钮，并处理使用者的选择。
2. 如何自订按钮的背景、间距与字型。
3. 如何在按钮上加入边框。
4. 如何建立包含图片和文字的按钮。
5. 如何建立具有渐层背景与阴影的按钮。
6. 如何建立一个全宽度（full-width）的按钮。
7. 如何建立一个可重复使用的按钮样式。
8. 如何加入一个点击按钮动画。

### 启用 SwiftUI 来建立新项目

好的，让我们从基础开始，使用 SwiftUI 来建立一个简单的按钮。首先开启Xcode， 并使用“App”模板建立一个新项目。接着于下一个画面输入项目的名称，我将它设定为“SwiftUIButton”，不过你可以自由使用其他的名称。你只需确保于 Interface 选项下已选取“ SwiftUI”即可，如图 6.1 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-1.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-1.png)

图 6.1. 建立一个新项目

当你储存项目后，Xcode 应会载入 `ContentView.swift` 档，并在设计划布中显示一个预览画面，如图 6.2 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-2.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-2.png)

图 6.2 预览默认的内容视图

使用 SwiftUI 建立按钮非常简单。基本上，你可以使用下列的代码片段来建立按钮：

```
Button {
    // 所需运行的内容
} label: {
    // 按钮的外观设定
}

```

建立按钮时，你需要提供这两个代码区块：

1. **所需运行的内容** - 使用者点击或选择按钮后运行的代码。
2. **按钮的外观设定** - 描述外观的代码区块。

举例而言，如果你只想要将“Hello World”标签变成一个按钮，则可以将代码修改如下：

```
struct ContentView: View {
    var body: some View {
        Button {
            print("Hello World tapped!")
        } label: {
            Text("Hello World")
        }
    }
}

```

另外，其实你也可以将代码写成这样：

```
struct ContentView: View {
    var body: some View {
        Button(action: {
            print("Hello World tapped!")
        }) label: {
            Text("Hello World")
        }
    }
}

```

两段代码的作用完全相同，这只是编码风格的问题。 在此书中，我们会使用第一种方法。

现在，你可以在画布中看到“Hello World”文字变成可点击的按钮，如图 6.3 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-3.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-3.png)

图 6.3. 建立一个简单的按钮

`print` 语句会将信息输出到控制台（Console）。要对其测试的话，请点选“Play”按钮来运行这个 App。如此当你点击按钮时，你将在主控台中看到该信息，如图 6.4 所示。如果你无法见到主控台，至 Xcode 菜单，并选取 View > Debug Area > Activate Console 。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-4.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-4.png)

图 6.4. 主控台信息

### 自订按钮的字型与背景

现在，你应该知道如何建立一个简单的按钮，我们来了解如何使用内建的修饰器来自订其外观。举例而言，要改变背景与文字颜色，你可以使用 `background` 与 `foreground Color` 修饰器，如下所示：

```
Text("Hello World")
    .background(Color.purple)
    .foregroundColor(.white)

```

如果你要修改字型，则可进一步使用 `font` 修饰器，并指定字型样式（例如：`.title` ）：

```
Text("Hello World")
    .background(Color.purple)
    .foregroundColor(.white)
    .font(.title)

```

修改完成后，你的按钮应该看起来如图 6.5 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-5.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-5.png)

图 6.5 自订按钮的背景色与前景色

如你所见，这个按钮看起来不怎么好看，如果能在文字周围加入一些间距不是会更好吗？为此，你可以使用padding 修饰器，如下所示：

```
Text("Hello World")
    .padding()
    .background(Color.purple)
    .foregroundColor(.white)
    .font(.title)

```

修改完成后，画布会据此修改按钮。现在按钮看起来好看多了，如图 6.6 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-6.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-6.png)

图 6.6 在按钮加入间距

### 修饰器顺序的重要性

这里要强调的一件事，即“padding 修饰器要放置于 background 之后”。如果你将代码撰写如下，则最终结果将会完全不同。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-7.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-7.png)

图 6.7 将 padding 修饰器放置于 background 修饰器之后

如果你将 `padding` 修饰器放置于 `background` 修饰器之后，你依然可以加入一些间距至按钮中，但是间距没有套用所选的背景色。如果你想知道原因，则可如下解读这些修饰器：

```
Text("Hello World")
    .background(Color.purple) // 1. 将背景色修改为紫色
    .foregroundColor(.white)  // 2. 将前景色 / 字型颜色设定为白色
    .font(.title)             // 3. 修改字型样式
    .padding()                // 4. 使用主色来加入间距（ 即白色）

```

反之，如果 `padding` 修饰器放置于 `background` 修饰器之前，则修饰器会像这样工作：

```
Text("Hello World")
    .padding()                // 1. 加入间距
    .background(Color.purple) // 2. 将背景色修改为紫色（ 包含间距）
    .foregroundColor(.white)  // 3. 将前景色/ 字型颜色设定为白色
    .font(.title)             // 4. 修改字型样式

```

### 按钮加上边框

`padding` 修饰器并非一定要始终放在最前面，而是取决于你的按钮设计，例如：你要建立一个具有图 6.8 所示的边框的按钮。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-8.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-8.png)

图 6.8. 具有边框的按钮

你可以如下修改 `Text` 控制组件的代码：

```
Text("Hello World")
    .foregroundColor(.purple)
    .font(.title)
    .padding()
    .border(Color.purple, width: 5)

```

这里我们设定前景色为“紫色”，然后在文字周围加入一些空白的间距。`border` 修饰器是用来定义边框的宽度与颜色，请自行修改 `width` 参数的值来查看效果。

再举一个例子，例如：一位设计师向你展示如图 6.9 所示的按钮设计，你将如何利用目前所学的内容来实现它呢？在阅读到下一个段落之前，请自行花个几分钟来思考解决方案。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-9.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-9.png)

图 6.9 具有背景色及边框的按钮

那么，你所需要的代码如下：

```
Text("Hello World")
    .fontWeight(.bold)
    .font(.title)
    .padding()
    .background(Color.purple)
    .foregroundColor(.white)
    .padding(10)
    .border(Color.purple, width: 5)

```

我们使用两个 `padding` 修饰器来设计按钮。第一个 `padding` 与 `background` 修饰器一起建立了具有间距及紫色背景的按钮，`padding(10)` 修饰器在按钮周围加入了额外的间距，而 `border` 修饰器则指定以紫色绘制边框。

我们来看一个更复杂的示例。如果你想要设计如图 6.10 所示、具有圆角外框的按钮， 该如何做呢？

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-10.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-10.png)

图 6.10. 具有圆角边框的按钮

SwiftUI 内建了一个名为 `cornerRadius` 修饰器，可以让你轻松建立圆角。要渲染圆角按钮的背景，你只需使用这个修饰器并指定圆角即可，如下所示：

```
.cornerRadius(40)

```

要做出圆角边框，需要多花一点工夫，因为这个 `border` 修饰器无法让你建立圆角。因此，我们需要绘制边框并将其叠在按钮上。以下为最终的代码：

```
Text("Hello World")
    .fontWeight(.bold)
    .font(.title)
    .padding()
    .background(.purple)
    .cornerRadius(40)
    .foregroundColor(.white)
    .padding(10)
    .overlay {
        RoundedRectangle(cornerRadius: 40)
            .stroke(.purple, lineWidth: 5)
    }

```

`overlay` 修饰器可以让你将另一个视图叠在目前的视图之上。在代码中，我们使用 `RoundedRectangle` 物件的`stroke` 修饰器来绘制一个圆角边框，而 `stroke` 修饰器可以让你设定框线的颜色与线宽。

### 建立具有图片与文字的按钮

到目前为止，我们只使用了文字按钮。在现实世界的项目中，你或你的设计师可能想要显示具有图片的按钮，而建立图片按钮的语法是相同的，除了你是使用 `Image` 控制组件来代替 `Text` 控制组件，如下所示：

```
Button(action: {
    print("Delete button tapped!")
}) {
    Image(systemName: "trash")
        .font(.largeTitle)
        .foregroundColor(.red)
}

```

为了方便起见，我们使用 SF Symbols 内建图示库（即垃圾桶图示）来建立图片按钮。我们在 `font` 修饰器中指定使用 `.largeTitle`，以使图片大一点。你的按钮看起来应如图 6.11 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-11.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-11.png)

图 6.11 图片按钮

同样的，如果你想要建立一个具有纯背景色的圆形图片按，则可以应用我们之前讨论过的修饰器图 6.12 显示了一个示例。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-12.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-12.png)

图 6.12 圆形图片按钮

你可以同时使用文字与图片来建立一个按钮。例如：你想要将“Delete”文字放在图示旁边，则可将代码替换如下：

```
Button {
    print("Delete button tapped")
} label: {
    HStack {
        Image(systemName: "trash")
            .font(.title)
        Text("Delete")
            .fontWeight(.semibold)
            .font(.title)
    }
    .padding()
    .foregroundColor(.white)
    .background(Color.red)
    .cornerRadius(40)
}

```

这里，我们将图片与文字控制组件皆嵌入在一个水平堆叠中，这将使垃圾桶图示与“Delete”文字并排。在`HStack` 中，修饰器设定了背景色、间距与按钮的圆角，图 6.13 显示了最后结果的按钮。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-13.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-13.png)

图 6.13. 具有图示与文字的按钮

### 标签的用法

SwiftUI 于 iOS 14 导入了一个新的视图称作 `Label`，可以将图片与文字并排，因此，你可以使用 `Label` 来替代 `HStack` 来实现同样的布局。

```
Button {
    print("Delete button tapped")
} label: {
    Label(
        title: {
            Text("Delete")
                .fontWeight(.semibold)
                .font(.title)
        },
        icon: {
            Image(systemName: "trash")
                .font(.title)
        }
    )
    .padding()
    .foregroundColor(.white)
    .background(.red)
    .cornerRadius(40)
}

```

### 建立具有渐层背景与阴影的按钮

使用 SwiftUI，你便可轻松以渐层背景设计按钮。你不仅可在 `background` 修饰器设定特定颜色，还可轻松为任何按钮设定渐层效果，你只需将下列这行代码：

```
.background(.red)

```

替代为：

```
.background(LinearGradient(gradient: Gradient(colors: [Color.red, Color.blue]), startPoint: .leading, endPoint: .trailing))

```

SwiftUI 框架有几个内建的渐层效果，上列的代码从左（ `.leading` ）至右（ `.trailing` ）应用了线性渐层，其从左侧的红色开始，至右侧的蓝色结束，如图 6.14 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-14.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-14.png)

图 6.14. 渐层背景的按钮

如果你想要由上而下应用渐层效果，则可以将代码修改如下：

```
.background(LinearGradient(gradient: Gradient(colors: [Color.red, Color.blue]), startPoint: .top, endPoint: .bottom))

```

你可以自由使用自己喜好的颜色来渲染渐层效果。例如：你的设计师告诉你使用了如图 6.15 所示的渐层。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-15.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-15.png)

图 6.15. uigradients.com 的渐层示例

有多种方式可以将色码（ color code ）从十六进制（ hex ）转换为Swift 相容的格式。这里我将展示其中一种方式。在项目导航器中，选取素材目录（也就是 `Assets` ），然后在空白区域（ AppIcon 下方）按右键，并选择“New Color Set”，如图 6.16 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-16.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-16.png)

图 6.16. 在素材目录定义一个新颜色集

接下来， 于 “Any Appearance”选择好颜色并点选“Show inspector”按钮， 然后点选属性检阅器（Attributes Inspector ）图示来显示颜色集的属性。在“Name”栏位中，将名称设定为“DarkGreen”，接着在“Color”区块中，修改“Input Method”（输入方法）栏位为“8-bit Hexadecimal”，如图 6.17 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-17.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-17.png)

图 6.17 编辑颜色集的属性

现在，你可以在“Hex”栏位设定色码。对于此示例，输入 `#11998e`来定义颜色并将颜色集命名为“LightGreen”。重复上述步骤，输入`#38ef7d`来定义另一个颜色集，你可以将颜色集命名为“DarkGreen”， 如图 6.18 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-18.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-18.png)

图 6.18. 定义两个颜色集

现在你已经定义了两个颜色集，我们回到 `ContentView.swift` 并修改色码。要使用颜色集，你只需使指定颜色集的名称，如下所示：

```
Color("DarkGreen")
Color("LightGreen")

```

因此， 要使用“DarkGreen”与“LightGreen”颜色集来渲染渐层， 你只需要修改 `background` 修饰器，如下所示：

```
.background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))

```

若是你已正确修改的话，则你的按钮应该有很棒的渐层背景，如图 6.19 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-19.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-19.png)

图 6.19. 使用自己喜爱的颜色来产生渐层

在本小节中，我要介绍另一个修饰器。`shadow` 修饰器可以让你在按钮（或任何视图） 周围绘制阴影，只需在`cornerRadius` 修饰器之后加入下列这行代码，即可看到结果：

```
.shadow(radius: 5.0)

```

此外，你可以控制阴影的颜色、半径与位置。以下为示例代码：

```
.shadow(color: .gray, radius: 20.0, x: 20, y: 10)

```

### 建立全宽度按钮

大一点的按钮通常可以吸引使用者的注意。有时，你可能需要建立一个占满屏幕宽度的全宽度按钮。`frame` 修饰器是设计用来让你控制视图的尺寸，不论你是想要建立一个固定大小的按钮，还是可变化宽度的按钮，都可以使用这个修饰器。要建立全宽度按钮，你可以修改 `Button` 代码，如下所示：

```
Button(action: {
    print("Delete tapped!")
}) {
    HStack {
        Image(systemName: "trash")
            .font(.title)
        Text("Delete")
            .fontWeight(.semibold)
            .font(.title)
    }
    .frame(minWidth: 0, maxWidth: .infinity)
    .padding()
    .foregroundColor(.white)
    .background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))
    .cornerRadius(40)
}

```

该代码与我们刚才编写代码非常相似，除了在 `padding` 之前加入 `frame` 修饰器。这里我们为按钮定义了弹性宽度，并设定 `maxWidth` 参数为 `.infinity`，这表示该按钮将填满容器视图的宽度。在画布中，它现在应该显示了一个全宽度按钮，如图 6.20 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-20.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-20.png)

图 6.20. 全宽度按钮

通过将 `maxWidth` 定义为 `.infinity`，按钮宽度将根据装置的屏幕宽度来自动调整。若是你想要给此按钮更多的水平空间的话，则在 `.cornerRadius(40)` 后插入一个 `padding` 修饰器：

```
.padding(.horizontal, 20)

```

### 使用 ButtonStyle 设计按钮

在现实世界的 App 中，相同的按钮设计将运用于多个按钮上，例如：你建立了Delete、Edit 与 Share 等三个按钮，都具有相同的按钮样式，如图 6.21 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-21.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-21.png)

图 6.21. 全宽度按钮

你可能撰写下列的代码：

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Button {
                print("Share button tapped")
            } label: {
                Label(
                    title: {
                        Text("Share")
                            .fontWeight(.semibold)
                            .font(.title)
                    },
                    icon: {
                        Image(systemName: "square.and.arrow.up")
                            .font(.title)
                    }
                )
                .frame(minWidth: 0, maxWidth: .infinity)
                .padding()
                .foregroundColor(.white)
                .background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))
                .cornerRadius(40)
                .padding(.horizontal, 20)
            }

            Button {
                print("Edit button tapped")
            } label: {
                Label(
                    title: {
                        Text("Edit")
                            .fontWeight(.semibold)
                            .font(.title)
                    },
                    icon: {
                        Image(systemName: "square.and.pencil")
                            .font(.title)
                    }
                )
                .frame(minWidth: 0, maxWidth: .infinity)
                .padding()
                .foregroundColor(.white)
                .background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))
                .cornerRadius(40)
                .padding(.horizontal, 20)
            }

            Button {
                print("Delete button tapped")
            } label: {
                Label(
                    title: {
                        Text("Delete")
                            .fontWeight(.semibold)
                            .font(.title)
                    },
                    icon: {
                        Image(systemName: "trash")
                            .font(.title)
                    }
                )
                .frame(minWidth: 0, maxWidth: .infinity)
                .padding()
                .foregroundColor(.white)
                .background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))
                .cornerRadius(40)
                .padding(.horizontal, 20)
            }
        }

    }
}

```

从上列的代码中可以见到，你需要为每个按钮复制所有的修饰器。当你或你的设计师想要修改按钮样式时，你将需要修改所有的修饰器，这会是一个非常繁锁的工作，并非是程序设计的最佳实践，那么你如何归纳样式并重复使用呢？

SwiftUI 提供了一个名为 `ButtonStyle`的协议，可以让你建立自己的按钮样式。要为我们的按钮建立一个可以重复使用的样式， 则我们建立一个名为 `Gradient BackgroundStyle` 的新结构，该结构遵守 ButtonStyle 协议。在 `struct ContentPreview_Previews`上方，插入下列这段代码：

```swift
struct GradientBackgroundStyle: ButtonStyle {

    func makeBody(configuration: Self.Configuration) -> some View {
        configuration.label
            .frame(minWidth: 0, maxWidth: .infinity)
            .padding()
            .foregroundColor(.white)
            .background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))
            .cornerRadius(40)
            .padding(.horizontal, 20)
    }
}

```

该协议要求我们提供接受 `configuration` 参数的 `makeBody` 函数的实现，configuration 参数包含一个 `label` 属性，你可以应用修饰器来修改按钮的样式。在上列的代码中，我们只应用了之前所使用的同一个修饰器。

因此，你如何将自订样式应用于按钮上呢？ SwiftUI 提供了一个名为 `.buttonStyle`的修饰器，让你可以应用按钮样式，如下所示：

```
Button {
    print("Delete button tapped")
} label: {
    Label(
        title: {
            Text("Delete")
                .fontWeight(.semibold)
                .font(.title)
        },
        icon: {
            Image(systemName: "trash")
                .font(.title)
        }
    )
}
.buttonStyle(GradientBackgroundStyle())

```

很酷，是吧？代码现在已经被简化了，你只要一行代码，即可轻松应用按钮样式于任何按钮上。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-22.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-22.png)

图 6.22. 使用.buttonStyle 修饰器来应用自订样式

你也可以通过 configuration 的 `isPressed 属性`，来判断按钮是否有被按下，这可以让你在使用者按下它时改变按钮的样式。举例而言，我们想要制作一个当使用者按下去时会变小一点的按钮，你可以加入一行程序，如图 6.23 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-23.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-23.png)

图 6.23. 使用 .buttonStyle 修饰器来自订样式

`scaleEffect` 修饰器可以让你缩放按钮（或任何视图）。当要放大按钮时，则要传送大于 1.0 的值，而小于1.0 的值，会让按钮变小。

```
.scaleEffect(configuration.isPressed ? 0.9 : 1.0)

```

因此，这行代码的作用是，当按下按钮时会缩小按钮（即 `0.9`），而当使用者放开手指后，会缩放回原来的大小（即 `1.0`）。如果你已运行这个App，则在按钮缩放时，你应该会看到一个不错的动画效果，这是 SwiftUI 的强大之处，你不需要额外撰写代码，它已经内建了动画特效。

### 作业

作业内容是建立一个具有＋图示的动画按钮，可以出现 + 图示。当使用者按下按钮时，这个＋图示将旋转（顺时针/ 逆时针）成一个 × 图示，如图 6.24 所示。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-24.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-24.png)

图 6.24. 使用者按下时旋转图示

提示一下，`rotationEffect` 修饰器可以用来旋转按钮（或者其他视图）。

### 在 iOS 15 中设置按钮样式

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-25.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-25.png)

图 6.25. 圆角按钮

相信你已经知道如何实现如图 25 所示的按钮。在 iOS 15 中，Apple 为 `Button` 视图引入了许多修饰符。 要建立类似按钮，你可以编写如下代码：

```
Button {

} label: {
    Text("Buy me a coffee")
}
.tint(.purple)
.buttonStyle(.borderedProminent)
.buttonBorderShape(.roundedRectangle(radius: 5))
.controlSize(.large)

```

`tint` 修饰器指定按钮的颜色。 通过应用`.borderedProminent` 样式，iOS 将按钮呈现为紫色背景并以白色显示文字。 `.buttonBorderShape` 修饰器可让你设置按钮的边框形状。 在这里，我们将它设置为 `.roundedRectangle` 使按钮变圆角。

`.controlSize` 允许你修改按钮的大小。 默认大小为 `.regular`。 其他有效值包括 `.large`、`.small` 和 `.mini` 。 下图显示了按钮在不同尺寸下的外观。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-26.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-26.png)

图 6.26. 不同组件大小的按钮

除了使用`.roundedRectangle`，SwiftUI 还提供了另一种名为`.capsule`的边框形状，供开发者创建胶囊形状按钮。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-27.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-27.png)

图 6.27. 使用胶囊形状

你还可以使用 `.automatic` 选项让系统自动调整按钮的形状。到目前为止，我们使用`.borderProminent` 按钮样式。 新版 SwiftUI 还提供了其他内置样式，包括 `.bordered`、`.borderless` 和 `.plain`。 `.bordered` 样式是经常会使用的样式。 下图显示了一个使用 `.bordered` 样式的示例按钮。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-28.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-28.png)

图 6.28. 使用边框样式

### 将样式应用到多个按钮

使用按钮样式，你可以轻松地将相同的样式应用于一组按钮。 下面是一个例子：

```
VStack {
    Button(action: {}) {
        Text("Add to Cart")
            .font(.headline)
    }

    Button(action: {}) {
        Text("Discover")
            .font(.headline)
            .frame(maxWidth: 300)
    }

    Button(action: {}) {
        Text("Check out")
            .font(.headline)
    }
}
.tint(.purple)
.buttonStyle(.bordered)
.controlSize(.large)

```

### 使用按钮角色

自 iOS 15 版本开始，SwiftUI 框架为`Button`引入了一个新的`role`选项。 此选项描述按钮的语义角色。 根据指定的角色，iOS 会自动为按钮呈现适当的外观。

例如，如下所示将角色定义为 `.destructive` ：

```
Button("Delete", role: .destructive) {
    print("Delete")
}
.buttonStyle(.borderedProminent)
.controlSize(.large)

```

iOS 会自动以红色显示*删除* 按钮。 图 29 显示了不同角色和样式的按钮外观。

![SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-29.png](SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-29.png)

图 6.29. 不同角色和样式的按钮外观

### 本章小结

在本章中，我们介绍了 SwiftUI 中建立按钮的基本概念。按钮在手机介面（或确切地说在任何应用程序 UI）中扮演着关键角色，良好设计的按钮不仅可以使你的 UI 更具吸引力，还可以提升你的 App 的使用者体验。如你所学，通过结合 SF Symbols、渐层与动画的应用，你可以轻松建立一个有吸引力且实用的按钮。

在本章所准备的示例档中，有完整的项目可以下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIButton.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIButton.zip))