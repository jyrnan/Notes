# 认识手势（Gestures） | 精通 SwiftUI - iOS 16 版

在前面的章节中，你已经对使用 SwiftUI 建立手势有所了解。我们使用 `onTapGesture` 修饰器来处理使用者的触控，并做出相对的回应。而在本章中，我们更深入了解如何在 SwiftUI 中处理各种类型的手势。

这个框架提供一些内建手势， 例如： 我们之前使用过的点击手势。除此之外， “DragGesture”、“MagnificationGesture”与“LongPressGesture”等都是现成可用的手势。我们将研究其中几个手势，并看看如何在 SwiftUI 中使用。最重要的是，你将学习如何建立一个可以支持拖曳手势的通用视图。

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-1.gif](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-1.gif)

图 17.1. 示例展示可拖曳视图

### 使用手势修饰器

要使用 框架识别特定手势，你需要做的就是使用 `.gesture` 修饰器将手势识别器加到视图上。下面是使用 `.gesture` 修饰器加到 `TapGesture` 的示例代码片段：

```
var body: some View {
    Image(systemName: "star.circle.fill")
        .font(.system(size: 200))
        .foregroundColor(.green)
        .gesture(
            TapGesture()
                .onEnded({
                    print("Tapped!")
                })
        )
}

```

如果你想要测试代码，则使用 “App”模板来建立一个新项目， 并确认你有选取 “Interface ”选项中的“SwiftUI”，然后在ContentView.swift 中贴上代码。

通过修改上列的代码，并导入一个状态变量，我们可以在星形图片被点击时，建立一个简单的缩放动画。下列为修改后的代码：

```
struct ContentView: View {
    @State private var isPressed = false

    var body: some View {
        Image(systemName: "star.circle.fill")
            .font(.system(size: 200))
            .scaleEffect(isPressed ? 0.5 : 1.0)
            .animation(.easeInOut, value: isPressed)
            .foregroundColor(.green)
            .gesture(
                TapGesture()
                    .onEnded({
                        self.isPressed.toggle()
                    })
            )
    }
}

```

当你在画布或模拟器中运行代码时，应该会看到缩放效果，这就是如何使用 `.gesture` 修饰器来侦测与回应某些触控事件的方法。如果你忘记动画的工作原理，可以回头阅读第 9 章。

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-2.gif](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-2.gif)

图 17.2. 简单的缩放效果

### 使用长按手势

其中一个是 `LongPressGesture`，这个手势识别器可以让你侦测长按事件。举例而言，如果你想只有当使用者长按星形图片一秒时可调整其大小，你可以使用 `LongPressGesture` 来侦测触控事件。

修改 `.gesture` 修饰器中的代码如下，以实现 `LongPressGesture`：

```
.gesture(
    LongPressGesture(minimumDuration: 1.0)
        .onEnded({ _ in
            self.isPressed.toggle()
        })
)

```

在预览画布中运行项目来快速测试。现在，你必须至少长按星形图片一秒钟，才能切换其大小。

### @GestureState 属性包裹器

当你按住星形图片时，在侦测到长按事件之前，图片不会给使用者任何回应。显然地，我们可以采取一些措施来改善使用者体验，我想要做的是在使用者点击图片时给予即时回馈。任何形式的回馈都将有助于改善情况，例如：当使用者点击图片时，我们可将图片调暗一点，这只是让使用者知道我们的 App 捕捉到触控事件，并且正在进行工作。图 17.3 说明了动画如何工作。

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-3.png](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-3.png)

图 17.3. 点击图片时应用暗淡效果

要实现这个动画，其中一项任务是追踪手势的状态。在长按手势的运行期间，我们必须区分点击与长按事件，那么我们该如何做呢？

SwiftUI 提供一个名为 `@GestureState` 的属性包裹器，它可以方便地追踪手势的状态变化，并让开发者决定对应的动作。要实现我们刚才描述的动画，我们可以使 用 `@GestureState` 声明一个属性：

```
@GestureState private var longPressTap = false

```

这个手势状态变量表示“运行长按手势期间是否侦测到点击事件”。当你定义了变量后，你可以修改 `Image` 视图的代码，如下所示：

```
Image(systemName: "star.circle.fill")
    .font(.system(size: 200))
    .opacity(longPressTap ? 0.4 : 1.0)
    .scaleEffect(isPressed ? 0.5 : 1.0)
    .animation(.easeInOut, value: isPressed)
    .foregroundColor(.green)
    .gesture(
        LongPressGesture(minimumDuration: 1.0)
            .updating($longPressTap, body: { (currentState, state, transaction) in
                state = currentState
            })
            .onEnded({ _ in
                self.isPressed.toggle()
            })
    )

```

我们只在上列的代码中做了一些修改。首先，加入了 `.opacity`修饰器。当侦测到点击事件后，我们将不透明度值设定为 `0.4`，以使图片变暗。

其次是 `LongPressGesture` 的 `updating` 方法。运行长按手势的期间，将调用此方法，并接收 value、state 与transaction 等三个参数：

- “value” 参数是手势的目前状态。这个值会依照手势而有所不同，但对于长按手势，`true` 值表示侦测到点击事件。
- “state”参数实际上是一个 in-out参数，可以让你修改 `longPressTap` 属性的值。在上列的代码中，我们设定 `state` 的值为 `currentState`。换句话说，`longPressTap` 属性始终追踪长按手势的最新状态。
- “transaction” 参数储存了目前状态处理修改的内容。

修改代码后，在预览画布中运行项目来进行测试。当你点击图片时，图片会立即变暗，而持续按住一秒后，图片会自己调整尺寸。

当使用者放开手指时，图片的不透明度会自动重置为正常状态，你是否想知道为什么呢？这是 `@GestureState` 的优点，当手势结束时，它会自动将手势状态属性的值设定为初始值，而在我们的示例中为 `false`。

### 使用拖曳手势

现在你应该了解如何使用 `.gesture` 修饰器与 `@GestureState`，我们来看另一个常见的“拖曳”手势。我们要做的是，修改现有的代码来支持拖曳手势，让使用者拖曳星形图片来移动它。

现在替换 `ContentView` 结构如下：

```
struct ContentView: View {
    @GestureState private var dragOffset = CGSize.zero

    var body: some View {
        Image(systemName: "star.circle.fill")
            .font(.system(size: 100))
            .offset(x: dragOffset.width, y: dragOffset.height)
            .animation(.easeInOut, value: dragOffset)
            .foregroundColor(.green)
            .gesture(
                DragGesture()
                    .updating($dragOffset, body: { (value, state, transaction) in

                        state = value.translation
                    })
            )
    }
}

```

要识别拖曳手势，你初始化一个 `DragGesture` 实例，并监听修改。在 `update` 函数中，我们传送一个手势状态属性来追踪拖曳事件。与长按手势类似，`update` 函数的闭包接收三个参数。在这个示例中，value 参数储存拖曳的目前数据（包含移动），这就是为什么我们将 `state` 变量（实际上是 `dragOffset` ）设定为`value.translation`的缘故。

在预览画布中运行项目，你可以拖曳图片，而当你放开图片时，它会返回原始位置。

你知道为什么图片会回到它的起点吗？如前一节所述，使用 `@GestureState` 的优点是， 当手势结束时，它会重置属性值为原始值。因此，当你放开手指结束拖曳时，`dragOffset` 会重置为`.zero`，即原始位置。

不过，如果你想让图片停留在拖曳的终点，该如何做呢？给自己几分钟的时间来思考如何实现。

由于 `@GestureState` 属性包裹器将重置属性为原始值，我们需要另一个状态属性来储存最终的位置。因此，我们声明一个新的状态属性如下：

```
@State private var position = CGSize.zero

```

接下来，修改 `body` 变量如下：

```
var body: some View {
    Image(systemName: "star.circle.fill")
        .font(.system(size: 100))
        .offset(x: position.width + dragOffset.width, y: position.height + dragOffset.height)
        .animation(.easeInOut, value: dragOffset)
        .foregroundColor(.green)
        .gesture(
            DragGesture()
                .updating($dragOffset, body: { (value, state, transaction) in

                    state = value.translation
                })
                .onEnded({ (value) in
                    self.position.height += value.translation.height
                    self.position.width += value.translation.width
                })
        )
}

```

我们在代码中做了一些修改：

1. 除了 `update` 函数之外，我们还实现了 `onEnded` 函数，其在拖曳手势结束时调用。在闭包中，我们加入拖曳偏移来计算图片的新位置。
2. `.offset` 修饰器也已修改，如此我们将目前的位置列入计算。

现在，当你运行项目并拖曳图片时，拖曳结束后，图片会停留在最后的位置，如图 17.4 所示。

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-4.gif](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-4.gif)

图 17.4. 拖曳图片

### 组合手势

在某些情况下，你需要在同一个视图中使用多个手势识别器。举例而言，我们想让使用者在开始拖曳之前按住图片，则必须结合长按与拖曳手势。SwiftUI 可以让你轻松组合手势，来运行一些复杂的互动。它提供三种手势组合类型，包括：“同时”（simultaneous ）、“依序”（sequenced ）与“专门”（exclusive ）。

当你需要同时侦测多个手势时，可以使用“同时”（simultaneous ）组合类型。而当你专门组合多个手势为一个手势时，SwiftUI 会识别你指定的所有手势，但当侦测到其中一个手势后，它会忽略其他手势。

顾名思义，如果你使用“依序”（sequenced ）组合类型来组合多个手势，SwiftUI 会以特定顺序来识别手势，这正是我们将用来对长按与拖曳手势进行排序的组合类型。

要使用多个手势，代码可以修改如下：

```
struct ContentView: View {
    // 长按手势
    @GestureState private var isPressed = false

    // 拖曳手势
    @GestureState private var dragOffset = CGSize.zero
    @State private var position = CGSize.zero

    var body: some View {
        Image(systemName: "star.circle.fill")
            .font(.system(size: 100))
            .opacity(isPressed ? 0.5 : 1.0)
            .offset(x: position.width + dragOffset.width, y: position.height + dragOffset.height)
            .animation(.easeInOut, value: dragOffset)
            .foregroundColor(.green)
            .gesture(
                LongPressGesture(minimumDuration: 1.0)
                .updating($isPressed, body: { (currentState, state, transaction) in
                    state = currentState
                })
                .sequenced(before: DragGesture())
                .updating($dragOffset, body: { (value, state, transaction) in

                    switch value {
                    case .first(true):
                        print("Tapping")
                    case .second(true, let drag):
                        state = drag?.translation ?? .zero
                    default:
                        break
                    }

                })
                .onEnded({ (value) in

                    guard case .second(true, let drag?) = value else {
                        return
                    }

                    self.position.height += drag.translation.height
                    self.position.width += drag.translation.width
                })
            )
    }
}

```

你应该对部分代码片段非常熟悉，因为我们结合已建立的长按手势与拖曳手势。

我来逐行解释一下 `.gesture` 修饰器。我们要求使用者在开始拖曳之前，至少长按图片一秒钟，因此我们从建立`LongPressGesture` 来开始，与我们之前所实现的内容类似，我们有一个 `isPressed` 手势状态属性，当某人点击图片时，我们将修改图片的不透明度。

`sequenced` 关键字可将长按与拖曳手势链接在一起。我们告诉 SwiftUI，`LongPressGesture` 应该在`DragGesture` 之前发生。

`updating` 与 `onEnded` 函数中的代码看起来非常相似，不过 `value` 参数现在实际上包含了两个手势（即长按与拖曳），这就是为何我们使用 `switch` 叙述来区分手势。你可以使用 `.first` 与 `.second` case 来找出要处理的手势。由于我们应该要在拖曳手势之前识别长按手势，因此这里的第一个手势是长按手势。在代码中，我们只印出“点击”（Tapping ）信息供你参考。

当长按手势确认之后，我们会进到 `.second` case。在这里，我们取出拖曳数据，并以对应的位移来修改`dragOffset`。

当拖曳结束后，将调用 `onEnded` 函数。同样的，我们通过计算拖曳数据（也就是 `.second` case ）来修改最终的位置。

现在，你可以测试手势组合了。在预览画布中，使用 debug 来运行 App，如此你可以在主控台中看到信息。你必须按住星形图片至少一秒钟，才能拖曳它。

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-5.gif](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-5.gif)

图 17.5. 只有当使用者长按图片至少一秒时，才能开始拖曳

### 使用枚举重构代码

编写拖曳状态的更好方式是使用枚举（Enum），这可让你将 `isPressed`与 `dragOffset` 状态结合为单个属性。我们声明一个名为 `DragState` 的枚举：

```
enum DragState {
    case inactive
    case pressing
    case dragging(translation: CGSize)

    var translation: CGSize {
        switch self {
        case .inactive, .pressing:
            return .zero
        case .dragging(let translation):
            return translation
        }
    }

    var isPressing: Bool {
        switch self {
        case .pressing, .dragging:
            return true
        case .inactive:
            return false
        }
    }
}

```

这里有三种状态：“静止”（inactive ）、“按下”（pressing ）与“拖曳”（dragging ）， 这些状态足以表示长按与拖曳手势运行期间的状态。对于“拖曳”（dragging ）状态，它与拖曳的位移有关。

使用 `DragState` 枚举，我们可以修改原来的代码如下：

```
struct ContentView: View {
    @GestureState private var dragState = DragState.inactive
    @State private var position = CGSize.zero

    var body: some View {
        Image(systemName: "star.circle.fill")
            .font(.system(size: 100))
            .opacity(dragState.isPressing ? 0.5 : 1.0)
            .offset(x: position.width + dragState.translation.width, y: position.height + dragState.translation.height)
            .animation(.easeInOut, value: dragState.translation)
            .foregroundColor(.green)
            .gesture(
                LongPressGesture(minimumDuration: 1.0)
                .sequenced(before: DragGesture())
                .updating($dragState, body: { (value, state, transaction) in

                    switch value {
                    case .first(true):
                        state = .pressing
                    case .second(true, let drag):
                        state = .dragging(translation: drag?.translation ?? .zero)
                    default:
                        break
                    }

                })
                .onEnded({ (value) in

                    guard case .second(true, let drag?) = value else {
                        return
                    }

                    self.position.height += drag.translation.height
                    self.position.width += drag.translation.width
                })
            )
    }
}

```

我们现在声明一个 `dragState` 属性来追踪拖曳状态。默认上， 它设定为 `DragState. inactive`。代码几乎相同，除了它修改为使用`dragState` 而不是使用 `isPressed` 与 `dragOffset`。举例而言，对于 `.offset` 修饰器，我们从拖曳状态的相关值中取得拖曳偏移量。

代码的结果是相同的，但是使用枚举追踪手势的复杂状态是较好的做法。

### 建立通用的可拖曳视图

到目前为止，我们已建立了一个可拖曳的图片视图，若是我们想要建立可拖曳的文字视图呢？或者我们想要建立可拖曳的圆形呢？是否应复制并贴上所有的代码，来建立文字视图或圆形呢？

总是会有更好的方式来实现它，我们来看如何建立通用的可拖曳视图。

在项目导航器中，右键点击 `SwiftUIGesture` 数据夹，选择“New File”，接着选取“SwiftUI View”模板，然后将文件命名为 `DraggableView`。

声明 `DragState` 枚举，并修改 `DraggableView`结构如下：

```swift
enum DraggableState {
    case inactive
    case pressing
    case dragging(translation: CGSize)

    var translation: CGSize {
        switch self {
        case .inactive, .pressing:
            return .zero
        case .dragging(let translation):
            return translation
        }
    }

    var isPressing: Bool {
        switch self {
        case .pressing, .dragging:
            return true
        case .inactive:
            return false
        }
    }
}

struct DraggableView<Content>: View where Content: View {
    @GestureState private var dragState = DraggableState.inactive
    @State private var position = CGSize.zero

    var content: () -> Content

    var body: some View {
        content()
            .opacity(dragState.isPressing ? 0.5 : 1.0)
            .offset(x: position.width + dragState.translation.width, y: position.height + dragState.translation.height)
            .animation(.easeInOut, value: dragState.translation)
            .gesture(
                LongPressGesture(minimumDuration: 1.0)
                .sequenced(before: DragGesture())
                .updating($dragState, body: { (value, state, transaction) in

                    switch value {
                    case .first(true):
                        state = .pressing
                    case .second(true, let drag):
                        state = .dragging(translation: drag?.translation ?? .zero)
                    default:
                        break
                    }

                })
                .onEnded({ (value) in

                    guard case .second(true, let drag?) = value else {
                        return
                    }

                    self.position.height += drag.translation.height
                    self.position.width += drag.translation.width
                })
            )
    }
}

```

所有的代码都与你之前编写的代码非常相似。技巧是将 `DraggableView` 声明为通用视图， 并建立一个`content` 属性，此属性接收任何视图，并且我们使用长按与拖曳手势为 `content` 视图提供支持。

现在，你可通过替换 `DraggableView_Previews`来测试这个通用视图，如下所示：

```
struct DraggableView_Previews: PreviewProvider {
    static var previews: some View {
        DraggableView() {
            Image(systemName: "star.circle.fill")
                .font(.system(size: 100))
                .foregroundColor(.green)
        }
    }
}

```

在代码中，我们初始化一个 `DraggableView`，并提供我们自己的内容（即星形图片）。在这个示例中，你应该完成支持长按与拖曳手势的相同星形图片。

那么，如果我们要建立一个可拖曳的文字视图呢？你可以将代码片段替换为下列代码：

```
struct DraggableView_Previews: PreviewProvider {
    static var previews: some View {
        DraggableView() {
            Text("Swift")
                .font(.system(size: 50, weight: .bold, design: .rounded))
                .bold()
                .foregroundColor(.red)
        }
    }
}

```

在闭包中，我们建立了一个文字视图而不是图片视图。如果你在预览画布中运行这个项目（如图 17.6 所示），则可以拖曳文字视图来移动它，是不是很酷呢？

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-6.gif](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-6.gif)

图 17.6. 可拖曳的文字视图

如果你想要建立一个可拖曳的圆形，则可以替换代码如下：

```
struct DraggableView_Previews: PreviewProvider {
    static var previews: some View {
        DraggableView() {
            Circle()
                .frame(width: 100, height: 100)
                .foregroundColor(.purple)
        }
    }
}

```

这便是建立通用的可拖曳的方式。试着以其他视图替换圆形，来建立你自己的可拖曳视图，并享受其中的乐趣。

### 作业

在本章中，我们探索了三个内建手势，包括：点击、拖曳与长按，不过有一些手势我们还没试过。作为练习，请试着建立一个通用的可缩放视图，它可以识别 `MagnificationGesture`，并且可相应缩放任何给定的视图。图 17.7 显示了一个示例结果。

![%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-7.gif](%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-7.gif)

图 17.7. 可缩放的图片视图

### 本章小结

SwiftUI 框架让手势处理变得非常容易。正如你在本章所学到的内容，这个框架提供几个可以立即使用的手势识别器。要使视图支持某个类型的手势，你需要做的是将其加上 `.gesture` 修饰器。组合多个手势从未如此简单。

为行动应用程序建立手势驱动的使用者介面，是一种日益增长的趋势。藉由易于使用的 API，试着使用一些有用的手势来增强 App 的功能，以使你的使用者满意。

在本章所准备的示例档中，有完整的项目可以下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIGesture.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIGesture.zip))