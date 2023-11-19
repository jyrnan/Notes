# Swift UI thinking

### alignment guide

- 改变的是alignment guide本身值，而不能改变对其模式中所采用的具体那个alignment guide
- 如果不采用该alignmentguide，则改变了也不起作用

### 环境值

- .font其实是对环境值的改变😺
- 环境值中读取时候设置更具体的键值路径可以减少视图重刷
- 环境值是在视图body方法中读取，在视图初始化阶段是不应该读取环境值的

### Animation

- withAnimation是显示，.animation是隐式
- 显示会被隐式所覆盖
- Transaction是真正的动画属性，有.animation属性，transaction属性会从顶层视图不断向底层传递
- 显示和隐式设置动画类型其实就是设置这个属性；
- 这个属性可以被禁用，不接受上层传下来的.animation属性。从而可以在该层transaction中设置新的.animation
- transition包含两种状态
    - active ：初始活动状态，例如插入时，opacity是0
    - identity ：常时状态，例如在屏幕显示时，opacity是1
    - 利用Anytransition.modifier(active:identity:)来创建自定义过渡效果
- **animation要放置在相对transition正确的位置，例如通过if来实现transition的视图，不能把animation直接依附在这个if内的视图上，而应该以副作用在if视图之外，通过Hstack容器来依附也是可以**

## 基于阶段的动画phaseAnimator

如果提供了触发值，则所有离散值完成计算一遍后动画停止

如果没有触发值，则会循环计算不停止

## 基于关键帧的动画keyframeAnimator

### 多个轨道

通过创建struct包含各个变量属性

每个变量属性构成自己的轨道

# 进阶布局

## 锚点

这种情况下，即使椭圆的位置可能是由较深层的视图所确定的，我们还是会希望将它绘制在靠近顶部的视图层级上。
为了实现这一点，我们可以使用首选项来传递登录按钮的 frame，然后将椭圆作为覆盖层绘制在整个视图树的顶部。

<aside>
💡 这个Anchor和原来的UIKit里面的anchor貌似不同？！

</aside>

## 用法范例

### 创建PreferenceKey

```swift
struct HighlightKey: PreferenceKey {
    static var defaultValue: Anchor<CGRect>?
    static func reduce(value: inout Anchor<CGRect>?, nextValue: () -> Anchor<CGRect>?) {
        value = value ?? nextValue()
    }
}
```

### 更新数值到PreferenceKey

`.anchorPreference` 可以把当前View的bounds和Anchor关联，并存储到key中

```swift
Text("Hello, This world!")
                .anchorPreference(key: HighlightKey.self, value: .bounds, transform: { anchor in
                    return anchor
                })
```

### 应用anchor创建view

- 从key中读取到Anchor值
- GeometryReader可以从Anchor中获取到相应CGRect值
- 用CGRect来创建View
    - 这个CGRect是相对于当前GeometryReader的空间而言
    - 所以需要offset数值为`rect.origin.x`， `rect.origin.y`

```swift
.overlayPreferenceValue(HighlightKey.self, { value in
            if let anchor = value {
                GeometryReader {proxy in
                    let rect = proxy[anchor]
                    Ellipse()
                        .stroke(.red, lineWidth: 1)
                        .frame(width: rect.width, height: rect.height)
                        .offset(x: rect.origin.x, y: rect.origin.y)
                }
            }
        })
```

### 效果

在Vstack上创建一个椭圆，按照下层的文字区域创建。

![Untitled](Swift%20UI%20thinking%20ac9fc6f92a384c0291d678cb8ce9bcf8/Untitled.png)

## 几何匹配效果

原来几何匹配效果还可以直接匹配静态的😱

不用PreferencKey和Anchor，直接实现上面的效果！

```swift
struct ContentView: View {
    @Namespace var namespace
    let groupID = "highlight"
    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            Rectangle()
                .foregroundStyle(.tint)
                .frame(width: 100, height: 100)
            Text("Hello, This world!")
                .matchedGeometryEffect(id: groupID, in: namespace)
        }
        .border(.red)
        .overlay {
            Ellipse()
                .stroke(.red, lineWidth: 1)
                .matchedGeometryEffect(id: groupID, in: namespace, isSource: true)
        }
    }
}
```

> 实际上，我们可以使用本章中讨论的技术自己实现 matchedGeometryEffect 的 API。我们需要在顶层的某个地方明确使用一 个视图修饰器，来从首选项中读取源视图的几何信息，并通过环境将它们 传递到目标视图中。作为框架来说，SwiftUI 是有能力自动处理这些操作 的。除此之外，创建一个几何匹配效果的 API 并没有什么神奇之处。
> 

<aside>
💡 看起来，底层的实现还是采用了PreferenceKey的方法！！！
**上面提到过，.matchedGeometryEffect 修饰器本质上是 .frame 和 .offset 修饰器的组合。**

</aside>