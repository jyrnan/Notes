# 使用拖曳手势DragGesture

<aside>
💡 要理解DragGesture手势传递的位置信息是当前这次拖动相对变化信息
最终反应出来的效果应该考虑上次手势产生的影响
所以需要：

- 保留上次的位置信息
- offset的参数应该是上次位置信息+当前位置变化
</aside>

# `DragGesture`

要识别拖曳手势，你初始化一个 `DragGesture` 实例，并监听修改。在 `update` 函数中，我们传送一个手势状态属性来追踪拖曳事件。与长按手势类似，`update` 函数的闭包接收三个参数。

在这个示例中，value 参数储存拖曳的目前数据（包含移动），这就是为什么我们将 `state` 变量（实际上是 `dragOffset` ）设定为`value.translation`的缘故。

```swift
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

# 拖动时候offset的状态算法

## `@GestureState` 会让属性归位

在预览画布中运行项目，你可以拖曳图片，而当你放开图片时，它会返回原始位置。

你知道为什么图片会回到它的起点吗？如前一节所述，使用 `@GestureState` 的优点是， 当手势结束时，它会重置属性值为原始值。因此，当你放开手指结束拖曳时，`dragOffset` 会重置为`.zero`，即原始位置。

## 利用一个额外属性来保存当前位置

由于 `@GestureState` 属性包裹器将重置属性为原始值，我们需要另一个状态属性来储存最终的位置。因此，我们声明一个新的状态属性如下：

```
@State private var position = CGSize.zero

```

## 调整offset的算法

接下来，修改 `body` 变量如下：需要加入上次移动的最新位置的考虑

```swift
var body: some View {
    Image(systemName: "star.circle.fill")
        .font(.system(size: 100))
        **.offset(x: position.width + dragOffset.width, y: position.height + dragOffset.height)**
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

## 小结：

我们在代码中做了一些修改：

1. 除了 `update` 函数之外，我们还实现了 `onEnded` 函数，其在拖曳手势结束时调用。在闭包中，我们加入拖曳偏移来计算图片的新位置。
2. `.offset` 修饰器也已修改，如此我们将目前的位置列入计算。

[Rotation](%E4%BD%BF%E7%94%A8%E6%8B%96%E6%9B%B3%E6%89%8B%E5%8A%BFDragGesture%2083264c3fd9c1411bace7427b0967f9f2/Rotation%20d96ee0ccd82845fcacbc0e877870b828.md)