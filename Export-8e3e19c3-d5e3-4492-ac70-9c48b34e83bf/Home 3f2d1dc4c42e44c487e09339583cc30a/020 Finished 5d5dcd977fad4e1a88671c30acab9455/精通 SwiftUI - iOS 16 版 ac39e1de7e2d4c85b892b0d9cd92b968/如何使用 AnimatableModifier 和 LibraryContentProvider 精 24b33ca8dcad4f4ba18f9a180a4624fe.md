# 如何使用 AnimatableModifier 和 LibraryContentProvider | 精通 SwiftUI - iOS 16 版

之前，你学习了如何使用 `Animatable` 和 `AnimatableData` 为环形进度条设置动画。 在本章中，我们将更进一步，向你展示如何使用另一个名为 `AnimatableModifier` 的协议为视图设置动画。 此外，我将向你介绍 SwiftUI 的一个新功能，该功能将允许开发者轻松地将客制化视图共享到视图库让你更易重用自制的组件。 稍后，我将向你展示如何将进度环视图加到视图库中以供重用。 先睹为快，你可以看看图 31.1 或观看此示范视频 ([https://link.appcoda.com/librarycontentprovider](https://link.appcoda.com/librarycontentprovider)) 了解如何LibraryContentProvider如何运作。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-1.jpg](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-1.jpg)

图 31.1. 在视图库中使用客制化视图

### AnimatableModifier 简介

我们先来看看 `AnimatableModifier` 协议。 顾名思义，`AnimatableModifier` 是一个视图修饰器，而它符合 `Animatable` 协议。 也因为此，这修饰器可以将不同类型视图的改变动画化。

```
protocol AnimatableModifier : Animatable, ViewModifier

```

那么，我们要制作什么动画呢？ 我们将以在前一章的示例为基础再添加一个文字标签。这标签会显示当前进度百分比。 随着进度条的移动，标签也相应修改。 图 31.2 显示了标签的外观。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-2.gif](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-2.gif)

图 31.2. 带动画的进度标签

### 使用 AnimatableModifer 建立文字动画

我强烈建议你先阅读第 30 章，因为这个示范项目是建基于前一个项目之上的。 如果你还没有做过该项目，你可以在 [https://www.appcoda.com/resources/swiftui4/SwiftUIProgressRingExercise.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIProgressRingExercise.zip) 下载。

在我们深入了解 `AnimatableModifier` 协议之前，让我问你。 你将如何布局进度标签并为其设置动画？ 如果你还记得，其实我们在第 9 章中构建了一个类似的进度指示器。根据你所学，可以像这样布局进度标签（在 `ProgressRingView.swift` 中）：

```
ZStack {
    Circle()
        .stroke(Color(.systemGray6), lineWidth: thickness)

    Text(progressText)
        .font(.system(.largeTitle, design: .rounded))
        .fontWeight(.bold)
        .foregroundColor(.black)

    ...
}

```

你可以在 `ZStack` 中添加一个 `Text` 视图，并使用以下方式格式化文本以显示当前进度：

```
private var progressText: String {
    let formatter = NumberFormatter()
    formatter.numberStyle = .percent
    formatter.percentSymbol = "%"

    return formatter.string(from: NSNumber(value: progress)) ?? ""
}

```

由于 `progress` 变量是一个状态变量，所以每当 `progress` 的值发生变化时，`progressText` 都会自动修改。 但是，解决方案存在一个问题，就是文字的动画效果不太好。

如果你在 `ProgressRingView.swift` 中修改了代码，则可以返回 `ContentView.swift` 查看结果。 此App确实显示了进度标签，但是当你将进度从一个值改为另一个值时，进度标签会立即使用淡出的动画效果来显示新值。

这不是我们所寄望的结果。 进度标签不应直接从一个值（例如 100%）跳转到另一个值（例如 50%）。 我们期望进度标签跟随进度条的动画并逐步修改其值，如下所示：

100 -> 99 -> 98 -> 97 -> 96 ... ... ... ... ... ... ... ... ... ... 53 -> 52 -> 51 -> 50

当前的做法不允许你为文字的变化设置动画， 这就是为什么我必须向你介绍 `AnimatableModifier` 协议的原因。

为了制作文字动画，我们将在 `ProgressRingView.swift` 中创建一个名为 `ProgressTextModifier` 的新结构，并采用 `AnimatableModifier`：

```
struct ProgressTextModifier: AnimatableModifier {

    var progress: Double = 0.0
    var textColor: Color = .primary

    private var progressText: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .percent
        formatter.percentSymbol = "%"

        return formatter.string(from: NSNumber(value: progress)) ?? ""
    }

    var animatableData: Double {
        get { progress }
        set { progress = newValue }
    }

    func body(content: Content) -> some View {
        content
            .overlay(
                Text(progressText)
                    .font(.system(.largeTitle, design: .rounded))
                    .fontWeight(.bold)
                    .foregroundColor(textColor)
                    .animation(nil)
            )
    }
}

```

对你来说，这些代码是不是很熟悉？ 如前所述，`AnimatableModifier` 协议同时符合 `Animatable` 和 `ViewModifier`。 因此，我们在 `animatableData` 属性中指定动画的值。 这里是`progress`。 为了符合 `ViewModifier` 的要求，我们实现了 `body` 函数并添加了 `Text` 视图。

就是这样使用 `AnimatableModifier` 为文字加置动画。 为方便起见，在 `ProgressRingView` 的末尾插入以下代码，以创建用于 `ProgressTextModifier` 的扩展：

```
extension View {
    func animatableProgressText(progress: Double, textColor: Color = Color.primary) -> some View {
        self.modifier(ProgressTextModifier(progress: progress, textColor: textColor))
    }
}

```

现在你可以像以下代码将 `animatableProgressText` 修饰器附加到 `RingShape` 上：

```
RingShape(progress: progress, thickness: thickness)
    .fill(AngularGradient(gradient: gradient, center: .center, startAngle: .degrees(startAngle), endAngle: .degrees(360 * progress + startAngle)))
    .animatableProgressText(progress: progress)

```

修改后，你应该会在预览列看到进度标签。 要测试动画，请在 iPhone 模拟器上运行App或在 `ContentView.swift` 中运行App。 当你修改进度时，进度文字已经带有动画。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-3.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-3.png)

图 31.3. 应用客制化修饰符

### 使用 LibraryContentProvider

从 Xcode 12 开始，Apple 在 SwiftUI 框架添加了一项功能，允许开发者将任何客制化视图加到 View 图库中。 如果你忘记了 View 图库是什么，只需按 *command-shift-L* 即可把它弹出来。 该图库可让你轻松找寻所有可用的 UI 组件。 你可以从库中拖动组件并将其直接添加到App UI。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-4.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-4.png)

图 4. 视图库

Xcode 允许开发者使用名为`LibraryContentProvider`的协议将客制化视图添加到图库中。 要将客制化视图视图添加到视图库，你需要建立一个符合 `LibraryContentProvider` 协议的新结构。

例如，要将进度环视图共享到视图库，我们可以在 `ProgressRingView.swift` 中创建一个名为 `ProgressBar_Library` 的结构，如下所示：

```
struct ProgressBar_Library: LibraryContentProvider {
    @LibraryContentBuilder var views: [LibraryItem] {
        LibraryItem(ProgressRingView(progress: .constant(1.0), thickness: 12.0, width: 130.0, gradient: Gradient(colors: [.darkYellow, .lightYellow])), title: "Progress Ring", category: .control)
    }
}

```

你建立一个符合 `LibraryContentProvider` 的结构并覆载 `views` 属性以回一个客制化视图数组。 在上面的代码，我们回了带有一些默认值的进度环视图，将其命名为`Progress Ring`，并将其放入组件类别中。

或者，如果你想添加多个库项目，你可以编写如下代码：

```
struct ProgressBar_Library: LibraryContentProvider {
    @LibraryContentBuilder var views: [LibraryItem] {
        LibraryItem(ProgressRingView(progress: .constant(1.0), thickness: 12.0, width: 130.0, gradient: Gradient(colors: [.darkYellow, .lightYellow])), title: "Progress Ring", category: .control)

        LibraryItem(ProgressRingView(progress: .constant(1.0), thickness: 30.0, width: 250.0, gradient: Gradient(colors: [.darkPurple, .lightYellow])), title: "Progress Ring - Bigger", category: .control)
    }
}

```

再讲多一点点，你可以为项目的类别提供四种值，具体取决于图库项目所代表的内容：

- `control`
- `effect`
- `layout`
- `other`

你可能还未知道 `@LibraryContentBuilder` 属性包装器是什么？ 它只是使你免于编写用于创建`LibraryItem`数组的代码。 上面的代码其实可以改写成这样：

```
struct ProgressBar_Library: LibraryContentProvider {
    var views: [LibraryItem] {
        return [LibraryItem(ProgressRingView(progress: .constant(1.0), thickness: 12.0, width: 130.0, gradient: Gradient(colors: [.darkYellow, .lightYellow])), title: "Progress Ring", category: .control),

                LibraryItem(ProgressRingView(progress: .constant(1.0), thickness: 30.0, width: 250.0, gradient: Gradient(colors: [.darkPurple, .lightYellow])), title: "Progress Ring - Bigger", category: .control)
    }
}

```

当你加入代码后，Xcode 会自动发现项目中新加入的`LibraryContentProvider`协议，并将进度环视图添加到视图库中。 你现在可以轻易将进度环视图添加到App UI。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-5.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-5.png)

图 31.5. 将进度环视图添加到视图库

你不仅可以将客制化视图加至 Xcode 的视图库，还可以通过实现 `modifiers` 方法添加自己制的修饰器。 你可以通过如下方法将 `animatableProgressText` 修饰器添加到视图库：

```
struct ProgressBar_Library: LibraryContentProvider {
    .
    .
    .

    @LibraryContentBuilder
    func modifiers(base: Circle) -> [LibraryItem] {
        LibraryItem(base.animatableProgressText(progress: 1.0), title: "Progress Indicator", category: .control)
    }
}

```

`base` 参数允许你指定可以由修饰器修改的组件类型。 在上面的代码，那个组件就是 `Circle` 视图。 同样地，一旦你将代码加入`ProgressBar_Library`，Xcode 就会自动扫描相关项目并将其添加到修改器库中。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-6.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-6.png)

图 31.6. 将 Progress Indicator 加至 Modifier 库

### 练习

进度环现在已合并到视图库中， 尝试使用它并构建一个如下图所示的App。 该App有 4 个 sliders，用于调整不同任务的进度。 除此之外，它也会计算并显示总体进度。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-7.gif](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20AnimatableModifier%20%E5%92%8C%20LibraryContentProvider%20%E7%B2%BE%2024b33ca8dcad4f4ba18f9a180a4624fe/swiftui-library-7.gif)

图 7. Daily Task 练习 App

### 总结

`AnimatableModifier` 协议是一个非常强大的协议，可为任何视图的变化建立动画。 在本章中，我们向你介绍了如何为标签的文字设置动画。 你可以应用此技巧为其他值设置动画，例如颜色和大小。

`LibraryContentProvider` 使开发者非常轻松地共享客制化视图并鼓励重用代码。 想像一下，你可以构建一个客制化组件库并将它们放入 View/Modifier 库中，团队中的每个成员都可以轻松取得并使用组件。 在这一章，你只学习如何在同一个 Xcode 项目中使用这些组件。 往后，我们将讨论如何使用 Swift *Package* 来让你将组件分享至不同Xcode项目。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUITextAnimation.zip](https://www.appcoda.com/resources/swiftui4/SwiftUITextAnimation.zip)

该练习的解决方案也包含在Xcode 项目中。 请参考 `TaskGridView.swift` 档。