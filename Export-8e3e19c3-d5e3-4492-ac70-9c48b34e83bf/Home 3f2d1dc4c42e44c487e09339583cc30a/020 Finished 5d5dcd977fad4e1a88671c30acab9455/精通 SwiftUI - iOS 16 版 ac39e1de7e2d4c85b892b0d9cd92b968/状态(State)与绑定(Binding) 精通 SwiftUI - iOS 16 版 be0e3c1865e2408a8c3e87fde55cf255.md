# 状态(State)与绑定(Binding) | 精通 SwiftUI - iOS 16 版

状态管理（state management）是每个开发者在应用程序开发中必须处理的事情。想像一下，你正在开发一个音乐播放器 App，当使用者点击“播放”按钮时，该按钮会变为“停止”按钮。在你的实现中，必须有一些方式来追踪应用程序状态，以让你知道何时修改按钮的外观。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-1.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-1.png)

图 7.1. “停止”与“播放”按钮

在 SwiftUI 中，内建了一些状态管理的功能，特别是它导入了一个名为“@State”的属性包装器（ Property Wrapper ）。当你使用 `@State` 来标注一个属性时，SwiftUI 会自动将其储存在你的应用程序中的某处。此外，使用该属性的视图会自动监听属性值的修改，当状态改变时，SwiftUI 将重新计算这些视图，并修改应用程序的外观。

听起来不错，不是吗？还是你对于状态管理觉得困惑？

总之，通过本章的示例代码，你将对状态与绑定有更多的了解。而且，我为你准备了一些作业，请花一点时间来练习一下，这将帮助你掌握 SwiftUI 的重要概念。

### 启用SwiftUI 建立新项目

我们从刚才提到的简单示例来开始，以了解如何通过追踪应用程序的状态，来切换“播放”与“停止”按钮。首先，开启Xcode 并使用“App”模板，来建立一个新项目。设定项目名称为“SwiftUIState”，不过你可以自由使用其他的名称，而你只需确保已选取“SwiftUI”选项，如图 7 .2 所示。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-2.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-2.png)

图 7.2. 建立一个新项目

当你储存项目后，Xcode 应该要载入 `ContentView.swift` 档，并且在设计划布中显示一个预览。现在我们建立“播放”按钮，如下所示：

```swift
Button {
    // 在“播放”与“停止”按钮之间切换
} label: {
    Image(systemName: "play.circle.fill")
        .font(.system(size: 150))
        .foregroundColor(.green)
}

```

我们使用系统图片，并将按钮涂成绿色，如图 7.3 所示。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-3.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-3.png)

图 7.3 预览“播放”按钮

### 控制按钮的状态

按钮的动作现在是空的，我们要做的是当使用者点击按钮时，将按钮的外观从“播放”改为“停止”。显示“停止”按钮时，按钮的颜色也应变成红色。

那么，我们要如何实现呢？显然的，我们需要一个变量来追踪按钮的状态。我们将其命名为 `isPlaying`，它是一个布林变量，指示 App 是否处于“播放”状态。如果将变量设定为“true”，则 App 应显示一个“停止”按钮；反之，App 显示一个“播放”按钮。代码如下所示：

```swift
struct ContentView: View {

    private var isPlaying = false

    var body: some View {
        Button {
            // 在“播放”与“停止”按钮之间切换
        } label: {
            Image(systemName: isPlaying ? "stop.circle.fill" : "play.circle.fill")
            .font(.system(size: 150))
            .foregroundColor(isPlaying ? .red : .green)
        }

    }
}

```

我们参照 `isPlaying` 变量的值来修改图片的名称与颜色。如果修改在你的项目中的代码，则应该会在预览画布中看到一个“播放”按钮。不过，若是你将 `isPlaying` 的默认值设定为 `true`，则会见到一个“停止”按钮。

现在的问题是，App 如何监听状态（即 `isPlaying` ）的变化，并自动修改按钮呢？使用 SwiftUI，你需要做的是在 `isPlaying` 属性前面加上 `@State`。

```
@State private var isPlaying = false

```

当我们声明属性为一个状态变量时，SwiftUI 就会管理isPlaying 的储存区，并监听其值的变化。当 `isPlaying` 的值修改时，SwiftUI 会参照 `isPlaying`状态，来自动重新计算视图。这里的视图指的是 `Button`。

> 只能从视图的 body（或者从被它调用的函数）内部存取一个状态属性。由于这个缘故，你应该声明你的状态属性为 private，以防止你的视图的用户端存取它。
> 
> - Apple 的官方文件 ([https://developer.apple.com/documentation/swiftui/state](https://developer.apple.com/documentation/swiftui/state))

我们还没有实现按钮的动作。因此，修改代码如下：

```swift
Button {
    // 在“播放”与“停止”按钮之间切换
    self.isPlaying.toggle()
} label: {
    Image(systemName: isPlaying ? "stop.circle.fill" : "play.circle.fill")
    .font(.system(size: 150))
    .foregroundColor(isPlaying ? .red : .green)
}

```

在 `action` 闭包（closure ）中，我们调用 `toggle()` 方法来将布林值从 `false` 切换为 `true`，或者从 `true` 切换为 `false`。在预览画布中，试着在“播放”与“停止”按钮之间切换，如图 7.4 所示。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-4.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-4.png)

图 7.4 在“播放”与“停止”按钮之间切换

你是否注意到，当你在按钮之间切换时，SwiftUI 会渲染一个淡入淡出动画？这个动画是内建且自动为你产生的。我们将在后续的章节中讨论更多有关动画的内容。不过，如你所见，SwiftUI 让所有的开发者对UI 动画可立即上手。

### 作业 #1

你的作业是建立一个计数器按钮，以显示点击次数。当使用者点击按钮时，该计数器会自动增加数字，并显示点击总次数，如图 7.5 所示。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-5.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-5.png)

图 7.5. 计数器按钮

### 使用绑定

你是否能够建立计数器按钮呢？这里我们不将布林变量声明为状态，而是使用一个整数状态变量来追踪计数。当点击按钮时，这个计数器会增加 1。图 7.6 的代码片段可供你参考。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-6.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-6.png)

图 7.6. 计数器按钮

好的，现在我们进一步修改代码，以显示三个计数器按钮，如图 7.7 所示。这三个按钮都共享相同的计数器，不论哪一个按钮被点击，该计数器将会增加 1，所有的按钮会同时一起显示修改后的计数。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-7.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-7.png)

图 7.7. 三个计数按钮

如你所见，所有的按钮共享相同的外观。就如我在前面章节内容所说明的，与其复制代码，较好的做法是取出一个共用视图作为可重复使用的子视图。因此，我们可以取出 Button 来建立一个独立视图，如下所示：

```
struct CounterButton: View {
    @Binding var counter: Int

    var color: Color

    var body: some View {
        Button {
            counter += 1
        } label: {
            Circle()
                .frame(width: 200, height: 200)
                .foregroundColor(color)
                .overlay {
                    Text("\(counter)")
                        .font(.system(size: 100, weight: .bold, design: .rounded))
                        .foregroundColor(.white)
                }
        }
    }
}

```

`CounterButton` 视图接收 “counter” 与 “color” 等两个参数，你可以使用红色来建立按钮，如下所示：

```
CounterButton(counter: $counter, color: .red)

```

你应该会注意到 `counter` 变量以 `@Binding` 来做标注。当你建立一个 `CounterButton` 实例时，counter 会加上一个 $ 符号作为前缀。

这是什么意思呢？

我们取出按钮至独立的视图后，`CounterButton` 变成 `ContentView` 的子视图。现在，计数器递增是在`CounterButton` 视图中完成的，而不是在 `ContentView` 中。`CounterButton` 必须在 `ContentView` 中有一个管理状态变量的方式。

这个 `@Binding` 关键字指示调用者必须提供状态变量的绑定。这样做就如同建立了 `ContentView` 中的 `counter` 以及 `CounterButton` 中的 `counter` 之间的双向连接。修改 `CounterButton` 视图中的 `counter`，会将其值传送回 `ContentView` 中的 `counter` 状态。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-8.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-8.png)

图 7.8 了解绑定

那么 `$` 符号是什么呢? 在 SwiftUI 中， 你使用 $ 前缀运算子从状态变量取得绑定。

如果你了解绑定的原理，则可以继续建立其他两个按钮，并使用 `VStack` 来垂直对齐， 如下所示：

```
struct ContentView: View {

    @State private var counter = 1

    var body: some View {
        VStack {
            CounterButton(counter: $counter, color: .blue)
            CounterButton(counter: $counter, color: .green)
            CounterButton(counter: $counter, color: .red)
        }
    }
}

```

修改完成后，你可以运行 App 并做测试。点击任何一个按钮，将使计数增加 1，如图 7.9 所示。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-9.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-9.png)

图 7.9. 测试三个计数器按钮

### 作业 #2

目前，所有的按钮共享相同的计数，而本作业需要修改代码，以使每个按钮都有其计数器。例如：当使用者点击蓝色按钮，则该 App 中只有蓝色按钮的计数器会加 1。除此之外，你需要提供一个主计数器，以计算所有按钮。图 7.10 为本作业的示范布局。

![%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-10.png](%E7%8A%B6%E6%80%81(State)%E4%B8%8E%E7%BB%91%E5%AE%9A(Binding)%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20be0e3c1865e2408a8c3e87fde55cf255/swiftui-state-10.png)

图 7.10. 每个按钮都有其计数器

### 本章小结

在 SwiftUI 中，状态的支持可简化应用程序开发中的状态管理。了解什么是 `@State` 与 `@Binding` 是非常重要的，因为它们对于在 SwiftUI 中做“状态管理”与“UI 修改”而言， 发挥了很大的作用。本章介绍了 SwiftUI 中状态管理的基础概念，之后你将学习到更多有关如何在视图动画应用 `@State`，以及如何管理多个视图之间的共享状态。

在本章所准备的示例档中，有完整的项目与作业解答可以下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUICounter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUICounter.zip))
- 作业解答 ([https://www.appcoda.com/resources/swiftui4/SwiftUIMasterCounter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIMasterCounter.zip))