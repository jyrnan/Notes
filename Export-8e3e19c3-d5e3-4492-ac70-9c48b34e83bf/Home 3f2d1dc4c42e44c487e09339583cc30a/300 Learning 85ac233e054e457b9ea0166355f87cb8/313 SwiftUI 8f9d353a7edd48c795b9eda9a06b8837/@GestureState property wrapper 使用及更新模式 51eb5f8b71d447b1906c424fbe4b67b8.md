# @GestureState property wrapper?使用及更新模式

SwiftUI gives us a specific property wrapper for tracking the state of gestures, helpfully called **@GestureState**. Although you can accomplish the same using a simple **@State** property wrapper, **@GestureState** comes with the added ability that it automatically sets your property back to its initial value when the gesture ends.

```swift
///以下范例实现了一个拖拽手势
struct ContentView: View {
    @GestureState var dragAmount = CGSize.zero//这是
    var body: some View {
        Image(systemName: "person.fill")
            .resizable()
            .aspectRatio(contentMode: .fill)
            .frame(width: 100, height: 100, alignment: .center)
            .offset(self.dragAmount) //来始终保持偏移的modifer
            .gesture(
                DragGesture()
                    .updating(self.$dragAmount, body: {
                        value, state, transition in
                        state = value.translation
                    }))
    }
    
}
```

<aside>
💡 `value`： 移动产生的值， 
`state`：是一个inout参数，实际上是指的updating的参数，也就是`self.$dragAmount`。在updating的过程中，就是不断将移动产生的数据value中的偏移值transition更新给`state`（`self.$dragAmount`），然后再通过`@GestureState`这个wapper来实现在view里面的更新

</aside>

### 进一步说明：

There’s quite a lot of code in there, so let’s unpack it:

1. The **`DragGesture().updating()`** code creates a new drag gesture, asking it to modify the value stored in **dragAmount** – that’s our **CGSize**.
2. It takes a closure with three parameters: **`value`**, **`state`**, and **`transaction`**.
3. The **`value`** parameter is the current data for the drag – where it started, how far it’s moved, where it’s predicted to end, and so on.
4. The **`state`** parameter is an **inout** value that is our property. So, rather than reading or writing **dragAmount** directly, inside this closure we should modify **`state`**.
5. The **`transaction`** parameter is an **inout** value that stores the whole animation context, giving us a little information about what’s going on such as whether this is a continuous or transient animation. Continuous animations might be produced by dragging a slider, whereas transient animations might be produced by tapping a button.
6. To make our view draggable, all we do is assign the current translation the drag straight to **`state`** (which is really **dragAmount** in this case), which in turn is used in the **`offset()**` modifier to move the view.

Remember, one of the advantages of **`@GestureState`** is that it automatically sets the value of your property back to its initial value when the gesture ends. In this case, it means we can drag a view around all we want, and as soon as we let go it will snap back to its original position.

里面的代码挺多的，我们来解释一下。

1. DragGesture().update()代码创建了一个新的拖动手势，要求它修改存储在dragAmount中的值--这就是我们的CGSize。

2. 2.它接受一个有三个参数的闭包：value、state和transaction。

3. value参数是拖动的当前数据--它在哪里开始，移动了多远，预测在哪里结束，等等。

4. state参数是一个inout值，是我们的属性。所以，在这个闭包里面，我们不应该直接读写dragAmount，而应该修改state。

5. 事务参数是一个inout值，它存储了整个动画上下文，给我们提供了一点信息，比如这是一个连续动画还是瞬时动画。连续动画可能是通过拖动滑块产生的，而瞬时动画可能是通过点击按钮产生的。

6. 为了使我们的视图可以拖动，我们所做的就是给当前的翻译分配拖动直达状态（在这种情况下，它实际上是dragAmount），这反过来又被用于offset()修改器来移动视图。

请记住，@GestureState 的一个优点是，当手势结束时，它会自动将你的属性值设置回初始值。在这种情况下，这意味着我们可以随意拖动视图，只要我们一松手，它就会恢复到原来的位置。