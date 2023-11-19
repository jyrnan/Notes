# 手动实现autorevers范例

很有用的一个autorevers手动范例！！！可以拓展到其他例如位置，颜色等其他变化的效果

```swift
struct ContentView: View {
    @State var scalingFactor: CGFloat = 1

    var body: some View {
        Text("hello world")
        .modifier(ReversingScale(to: scalingFactor, onEnded: {
            DispatchQueue.main.async {
                self.scalingFactor = 1
            }
        }))
        .animation(.default)
        .onAppear {
            self.scalingFactor = 2
        }
    }
}

struct ReversingScale: AnimatableModifier {
    var value: CGFloat

    private var target: CGFloat
    private var onEnded: () -> () //这个很关键，用来在第一阶段动画结束是设置成初始值，从而产生第二阶段的互逆动画

    init(to value: CGFloat, onEnded: @escaping () -> () = {}) {
        self.target = value
        self.value = value
        self.onEnded = onEnded // << callback
    }
///animatableData是一个很关键的值，表示在动画过程中能够变化的值，在动画过程中就是该值在不停变化。
    var animatableData: CGFloat {
        get { value }
        set { value = newValue
            // newValue here is interpolating by engine, so changing
            // from previous to initially set, so when they got equal
            // animation ended
            //当animatableData变成了目标值，，这个时候需要执行onEnded()，
            //设置新的目标值成初始值。这样就会产生一个从目标值变成初始值的新动画
            //这个新动画和初始值变目标值的动画刚好反过来，并且类型一致，相当于
            //手动实现了autoReverse的效果，并且最终状态会恢复成初始状态
            if newValue == target {
                onEnded()
            }
        }
    }

    func body(content: Content) -> some View {
        content.scaleEffect(value)
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```