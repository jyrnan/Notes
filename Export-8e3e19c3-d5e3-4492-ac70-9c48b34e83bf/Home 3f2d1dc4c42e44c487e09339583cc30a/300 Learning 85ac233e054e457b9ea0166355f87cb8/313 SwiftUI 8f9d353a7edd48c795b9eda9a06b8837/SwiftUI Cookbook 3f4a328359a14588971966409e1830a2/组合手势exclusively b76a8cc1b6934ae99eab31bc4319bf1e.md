# 组合手势exclusively

```swift
.gesture(
    TapGesture()
        .onEnded({ _ in
            withAnimation(.easeOut(duration: 0.15).delay(0.1)) {
                self.isCardPressed.toggle()
                self.selectedCard = self.isCardPressed ? card : nil
            }
        })
        .exclusively(before: LongPressGesture(minimumDuration: 0.05) 
        .sequenced(before: DragGesture())
        .updating(self.$dragState, body: { (value, state, transaction) in
            switch value {
            case .first(true):
                state = .pressing(index: self.index(for: card))
            case .second(true, let drag):
                state = .dragging(index: self.index(for: card), translation: drag?.translation ?? .zero)
            default:
                break
            }

        })
        .onEnded({ (value) in

            guard case .second(true, let drag?) = value else {
                return
            }

            // 重新排列卡片
        })

    )
)

```

SwiftUI 让你可专门组合多种手势。在上列的代码中，我们告诉 SwiftUI 捕捉点击手势或长按手势，换句话说，当侦测到点击手势时，SwiftUI 将忽略长按手势。

<aside>
💡 上面代码的意思是：
如果是单点就执行单点，所以就执行.onEnded方法
如果按下不放，就变成了长按，因此单点的方法.onEnded就不执行。**长按和后面的drag变成组合手势**：分别处理长按（上移20）的和拖动（跟随）的状态

</aside>

全文：[建立像 Apple Wallet App 的动画和转场效果 | 精通 SwiftUI - iOS 16 版](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a.md) 

# 官方手册

```swift
/// A gesture that consists of two gestures where only one of them can succeed.
///
/// The `ExclusiveGesture` gives precedence to its first gesture.
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
@frozen public struct ExclusiveGesture<First, Second> : Gesture where First : Gesture, Second : Gesture
```

```swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension Gesture {

    /// Combines two gestures exclusively to create a new gesture where only one
    /// gesture succeeds, giving precedence to the first gesture.
    ///
    /// - Parameter other: A gesture you combine with your gesture, to create a
    ///   new, combined gesture.
    ///
    /// - Returns: A gesture that's the result of combining two gestures where
    ///   only one of them can succeed. SwiftUI gives precedence to the first
    ///   gesture.
    @inlinable public func exclusively<Other>(before other: Other) -> ExclusiveGesture<Self, Other> where Other : Gesture
}
```