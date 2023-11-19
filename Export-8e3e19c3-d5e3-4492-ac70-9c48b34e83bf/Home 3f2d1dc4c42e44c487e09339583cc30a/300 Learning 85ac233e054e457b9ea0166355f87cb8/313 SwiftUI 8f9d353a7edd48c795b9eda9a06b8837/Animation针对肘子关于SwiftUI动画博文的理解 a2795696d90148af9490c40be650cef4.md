# Animation针对肘子关于SwiftUI动画博文的理解

> 这是针对[肘子关于SwiftUI动画博文](https://www.fatbobman.com/posts/the_animation_mechanism_of_swiftUI/)的理解， [Notion版本](SwiftUI%20%E7%9A%84%E5%8A%A8%E7%94%BB%E6%9C%BA%E5%88%B6%20%E8%82%98%E5%AD%90%E7%9A%84Swift%E8%AE%B0%E4%BA%8B%E6%9C%AC%20039ae96064ea4b09bc041b4225e1b607.md)
> 

在 SwiftUI 中，实现一个动画需要以下三个要素：

- 一个**时序曲线算法函数**： .easeInOut等：决定AnimatableData随时间变化方式
- 将状态（**特定依赖项**）同该时序曲线函数相**关联的声明（[implicit / explicit](Animation%E9%92%88%E5%AF%B9%E8%82%98%E5%AD%90%E5%85%B3%E4%BA%8ESwiftUI%E5%8A%A8%E7%94%BB%E5%8D%9A%E6%96%87%E7%9A%84%E7%90%86%E8%A7%A3%20a2795696d90148af9490c40be650cef4.md)）**：
    - `.animation(_ animation: value: )`
    - `.withAnimation(animation:) {}`
    - 更高级的有Transaction，其实就是在内部包了animation及更多控件
- 一个依赖于该状态（特定依赖项）的**可动画部件**: .offset() 或 Transition

![Untitled](Animation%E9%92%88%E5%AF%B9%E8%82%98%E5%AD%90%E5%85%B3%E4%BA%8ESwiftUI%E5%8A%A8%E7%94%BB%E5%8D%9A%E6%96%87%E7%9A%84%E7%90%86%E8%A7%A3%20a2795696d90148af9490c40be650cef4/Untitled.png)

![Untitled](Animation%E9%92%88%E5%AF%B9%E8%82%98%E5%AD%90%E5%85%B3%E4%BA%8ESwiftUI%E5%8A%A8%E7%94%BB%E5%8D%9A%E6%96%87%E7%9A%84%E7%90%86%E8%A7%A3%20a2795696d90148af9490c40be650cef4/Untitled%201.png)

# Transaction 事物？其实是关联申明

无论是修饰符 `animation` 还是全局函数 `withAnimation` ，实际上都是在视图中声明 Transaction 的快捷方法，内部分别对应着 `transaction` 和 `withTransaction`。

# Animation**时序曲线算法函数**

决定AnimatableData随时间变化方式，例如快慢缓急等

# 可动画部件

包含了`animatableData`，这个数值决定了部件的动画数据，由系统计算；以及动作的类型，例如.offset .scale .rotation，

transition是包住了这些数值/动作类型的整合体，区别是数值可能是由自行设定。

## **AnimatableData**

Animatable 协议的要求非常简单，只需实现一个计算属性 `animatableData` 这是进行动画效果插值的依据

```swift
public protocol Animatable {

    /// The type defining the data to animate.
    associatedtype AnimatableData : VectorArithmetic

    /// The data to animate.
    var animatableData: Self.AnimatableData { get set }
}
```

Any type that conforms to *VectorArithmetic*protocol can be used as *animtableData*
. SwiftUI provides us a few types that conform to *VectorArithmetic* protocol out of the box. For example, *Float*, *Double*, *CGFloat*, and *AnimatablePair*. -[majid blog](https://swiftwithmajid.com/2020/06/17/the-magic-of-animatable-values-in-swiftui/)

## Transition 转场

SwiftUI 的转场类型（ AnyTransition ）是对可动画部件的再度包装，其实内部就是一个可动画部件。类似与`.offset(animated ? value1 : value2)`

使用方法是：在View基础上调用transition方法，返回一个带动画的View？

```swift
func transition(_ t: AnyTransition) -> some View
```

A transition in SwiftUI is what determines how a view is inserted or deleted from the hierarchy. **A transition on its own has no effect. It must be associated with an animation.** 

使用的时候也是需要如上面的两个要素：**特定依赖项**和**动画变化曲线的结合**，所以主要如下代码关键项

```swift
struct ContentView: View {
    @State var animated = false // 特定依赖项
    var body: some View {
        VStack {
            if animated {
                Text("Hello world")
                    .transition(.slide) //需要的是AnyTransision，有多个形式，.slide是其中一种
            }
            Button("Animate") {
                withAnimation{ // 和动画变化曲线结合（默认省略了easyInOut）
                    animated.toggle()
                }
            }
        }
    }
}
```

### AnyTransition

而.transition方法最终要的传入的参数是AnyTransition，其基本形式有，并可以带上更详细的参数

- `.slide`
- `.opacity`
- `.scale`

比较高级的有

- `.asymmetric`
- `.combined`

更进一步，可以自行定义， 就更灵活了

```swift
extension AnyTransition {
    static var myOpacity: AnyTransition { get {
        AnyTransition.modifier(
            active: MyOpacityModifier(opacity: 0),
            identity: MyOpacityModifier(opacity: 1))
        }
    }
}

struct MyOpacityModifier: ViewModifier {
    let opacity: Double
    
    func body(content: Content) -> some View {
        content.opacity(opacity)
    }
}
```

> 更多的Transition看[这篇博客](https://swiftui-lab.com/advanced-transitions/)
> 

## **Explicit vs. Implicit Animations**

- Implicit: `.animation(_: value:)` 现在也有关联值的参数了，所以其实也变得exlicit了
- explicit: `withAnimation{...}` 闭包里面的参数才会导致动画变化