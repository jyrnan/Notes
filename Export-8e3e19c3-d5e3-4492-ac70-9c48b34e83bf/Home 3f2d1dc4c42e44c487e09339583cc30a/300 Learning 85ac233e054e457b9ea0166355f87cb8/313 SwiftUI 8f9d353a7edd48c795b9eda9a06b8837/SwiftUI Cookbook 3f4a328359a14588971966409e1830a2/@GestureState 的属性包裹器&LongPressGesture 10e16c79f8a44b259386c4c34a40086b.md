# @GestureState 的属性包裹器&LongPressGesture

[@GestureState 属性包裹器](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44.md) 

<aside>
💡 **简单来说就是在View里定义的一个类似@State的状态参数**`@GestureState` **，
-用来接受并保存后面的闭包随时更新的数值，
-同时它的变化会导致View的改变。**

</aside>

SwiftUI 提供一个名为 `@GestureState` 的属性包裹器，它可以方便地追踪手势的状态变化，并让开发者决定对应的动作。要实现我们刚才描述的动画，我们可以使 用 `@GestureState` 声明一个属性：

```swift
@GestureState private var longPressTap = false
```

# 用法(核心）

需要利用手势的.updating方法来更新这个状态参数

```swift
@GestureState private var longPressTap = false
@State private var isPressed = false
...
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

其次是 `LongPressGesture` 的 `updating` 方法。运行长按手势的期间，将调用此方法，并接收 value、state 与transaction 等三个参数：

- “value” 参数是手势的目前状态。**这个值会依照手势而有所不同**，但对于长按手势，`true` 值表示侦测到点击事件。
- “state”参数实际上是一个 in-out参数，可以让你修改 `longPressTap` 属性的值。在上列的代码中，我们设定 `state` 的值为 `currentState`。换句话说，`longPressTap` 属性始终追踪长按手势的最新状态。
- “transaction” 参数储存了目前状态处理修改的内容。

修改代码后，在预览画布中运行项目来进行测试。当你点击图片时，图片会立即变暗，而持续按住一秒后，图片会自己调整尺寸。

当使用者放开手指时，图片的不透明度会自动重置为正常状态，你是否想知道为什么呢？**这是 `@GestureState` 的优点，当手势结束时，它会自动将手势状态属性的值设定为初始值，而在我们的示例中为 `false`**。