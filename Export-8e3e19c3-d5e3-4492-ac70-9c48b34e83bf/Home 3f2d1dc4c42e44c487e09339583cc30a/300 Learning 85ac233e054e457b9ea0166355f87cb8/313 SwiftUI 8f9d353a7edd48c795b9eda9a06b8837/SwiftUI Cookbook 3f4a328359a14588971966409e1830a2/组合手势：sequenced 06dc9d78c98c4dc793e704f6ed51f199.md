# 组合手势：sequenced

<aside>
💡 -sequenced会创建一个新的手势，因此也有一个.updating, .onEnded等闭包处理方法
-闭包的参数数量和单手势的类似，例如value，但会相对复杂，可能会是两个组合的手势各自的数据，因此需要对其进行判断，通过switch先识别这个value代表的是哪个手势的数据，例如：.first或.second，然后在将数据分别处理

</aside>

```swift
**@inlinable** **public** **func** sequenced<Other>(before other: Other) -> SequenceGesture<Self, Other> **where** Other : Gesture
```

在某些情况下，你需要在同一个视图中使用多个手势识别器。举例而言，我们想让使用者在开始拖曳之前按住图片，则必须结合长按与拖曳手势。SwiftUI 可以让你轻松组合手势，来运行一些复杂的互动。它提供三种手势组合类型，包括：

- “同时”（simultaneous ）、
- “依序”（sequenced ）与
- “专门”（exclusive ）。

当你需要同时侦测多个手势时，可以使用“同时”（simultaneous ）组合类型。而当你专门组合多个手势为一个手势时，SwiftUI 会识别你指定的所有手势，但当侦测到其中一个手势后，它会忽略其他手势。

顾名思义，如果你使用“依序”（sequenced ）组合类型来组合多个手势，SwiftUI 会以特定顺序来识别手势，这正是我们将用来对长按与拖曳手势进行排序的组合类型。

要使用多个手势，代码可以修改如下：

```swift
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
                **.sequenced(before: DragGesture())**
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

我来逐行解释一下 `.gesture` 修饰器。我们要求使用者在开始拖曳之前，至少长按图片一秒钟，因此我们从建立`LongPressGesture` 来开始，与我们之前所实现的内容类似，我们有一个 `isPressed` 手势状态属性，当某人点击图片时，我们将修改图片的不透明度。

`sequenced` 关键字可将长按与拖曳手势链接在一起。我们告诉 SwiftUI，`LongPressGesture` 应该在`DragGesture` 之前发生。

`updating` 与 `onEnded` 函数中的代码看起来非常相似，**不过 `value` 参数现在实际上包含了两个手势（即长按与拖曳），这就是为何我们使用 `switch` 叙述来区分手势。你可以使用 `.first` 与 `.second` case 来找出要处理的手势。由于我们应该要在拖曳手势之前识别长按手势，因此这里的第一个手势是长按手势。在代码中，我们只印出“点击”（Tapping ）信息供你参考。**

**当长按手势确认之后，我们会进到 `.second` case。在这里，我们取出拖曳数据，并以对应的位移来修改`dragOffset`。**

当拖曳结束后，将调用 `onEnded` 函数。同样的，我们通过计算拖曳数据（也就是 `.second` case ）来修改最终的位置。

现在，你可以测试手势组合了。在预览画布中，使用 debug 来运行 App，如此你可以在主控台中看到信息。你必须按住星形图片至少一秒钟，才能拖曳它。

![../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-5.gif](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44/swiftui-gestures-5.gif)

# 进一步提升(值得一看）

可以通过枚举来重构上面的代码

并可以将这个可拖拽的手势变成一个通用方法

[使用枚举重构代码](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E8%AE%A4%E8%AF%86%E6%89%8B%E5%8A%BF%EF%BC%88Gestures%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20c71882a83f6e4944bdf595f821bdda44.md)