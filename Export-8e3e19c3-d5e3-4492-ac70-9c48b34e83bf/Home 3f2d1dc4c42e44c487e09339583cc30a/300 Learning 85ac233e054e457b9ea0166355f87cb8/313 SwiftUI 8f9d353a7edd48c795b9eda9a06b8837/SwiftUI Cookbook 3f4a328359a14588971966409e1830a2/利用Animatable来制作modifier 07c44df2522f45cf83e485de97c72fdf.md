# 利用Animatable来制作modifier

利用一个文字的动画变化来说明：

例如从0.1变到0.5，如果不指定则是从0.1通过透明度变化直接变到0.5，但可以实现成是数值的变化，例如0.1 → 0.11 →0.12 … 0.49 → 0.5

<aside>
💡 关键就在于实现AnimatableData协议，来定义一个AnimatableData

</aside>

所以要达到上面的效果，必须指定AnimatableData是这个数值

范例：

```swift
struct ProgressTextModifier: Animatable, ViewModifier {

    var progress: Double = 0.0
    var textColor: Color = .primary

    private var progressText: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .percent
        formatter.percentSymbol = "%"

        return formatter.string(from: NSNumber(value: progress)) ?? ""
    }

    var animatableData: Double {
        get { progress }
        set { progress = newValue }
    } // 这里指定了animatableData其实就是progress这个Double类型的变量。因此，动画也就是以这个数值变化为内容。

    func body(content: Content) -> some View {
        content
            .overlay(
                Text(progressText)
                    .font(.system(.largeTitle, design: .rounded))
                    .fontWeight(.bold)
                    .foregroundColor(textColor)
                    .animation(nil)
            )
    }
}
```

> AnimatableModifier已经Deprecated，可以直接通过Animatable, ViewModifier来实现
>