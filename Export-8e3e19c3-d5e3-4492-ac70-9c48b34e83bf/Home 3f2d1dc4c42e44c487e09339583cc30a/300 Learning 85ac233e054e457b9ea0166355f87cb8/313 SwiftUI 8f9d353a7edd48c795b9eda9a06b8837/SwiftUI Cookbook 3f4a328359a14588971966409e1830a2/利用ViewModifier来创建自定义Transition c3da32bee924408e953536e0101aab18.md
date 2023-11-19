# 利用ViewModifier来创建自定义Transition

## 基本方法：

1. 创建一个ViewModifier，需要通过参数控制不同的状态
2. 利用extension来通过AnyTransition.modifier(active: identity: )创建自定义transition
3. 应用

## 实例

创建一个从屏幕外进出并且旋转的transition

### 创建Modifier

通过degrees参数来实现modifier的不同状态

因为后续需要用到modifier的两种状态

```swift
struct RotationModifier: ViewModifier {
    let degrees: Double // 该参数会控制modifer产生的不同状态效果
    init(degrees: Double = 90) {
        self.degrees = degrees
    }
    func body(content: Content) -> some View {
        content
            .rotationEffect(Angle(degrees: degrees))
            .offset(x:degrees != 0 ? UIScreen.main.bounds.width : 0,
                    y:degrees != 0 ? UIScreen.main.bounds.height : 0)
    }
}
```

### 创建Transition

- 利用`AnyTransition.modifier(active: identity: )` 可以创建一个AnyTransition的静态实例，

可以是静态变量，也可以是静态方法。如果需要传递参数进去，则采用静态方法

```swift
extension AnyTransition {
    static var rotationCopy: AnyTransition {
        return AnyTransition.modifier(active: RotationModifier(),
                               identity: RotationModifier(degrees: 0))
    }
}
```

- 该方法需要设置active和identity两种状态下的modifier，两个modifier是同一个类型，所以应该通过参数来控制modifier产生不同的状态效果。

```swift
//Returns a transition defined between an active modifier 
and an identity modifier.
static func modifier<E>( active: E, identity: E) 
-> AnyTransition where E : ViewModifier
```

### 使用

```swift
struct TransitionDemo: View {
    @State private var show: Bool = false
    var body: some View {
        VStack{
            Spacer()
            if show{
                Rectangle()
                    .frame(width: 200, height: 100)
                    .frame(maxWidth: .infinity)
                    .transition(.rotationCopy) //使用
            }
            
            Spacer()
            Button{
                withAnimation {
                    show.toggle()
                }
                
            } label: {
                Text("Click Me")
            }
        }
    }
}
```

### 效果

[Simulator Screen Recording - iPhone 14 Pro - 2023-08-31 at 09.50.19.mp4](%E5%88%A9%E7%94%A8ViewModifier%E6%9D%A5%E5%88%9B%E5%BB%BA%E8%87%AA%E5%AE%9A%E4%B9%89Transition%20c3da32bee924408e953536e0101aab18/Simulator_Screen_Recording_-_iPhone_14_Pro_-_2023-08-31_at_09.50.19.mp4)