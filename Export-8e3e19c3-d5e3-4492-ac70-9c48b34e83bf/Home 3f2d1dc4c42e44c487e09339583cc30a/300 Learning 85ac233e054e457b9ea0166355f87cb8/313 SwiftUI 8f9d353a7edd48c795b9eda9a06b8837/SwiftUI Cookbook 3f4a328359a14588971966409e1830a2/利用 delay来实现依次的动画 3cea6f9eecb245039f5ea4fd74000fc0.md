# 利用.delay来实现依次的动画

SwiftUI 框架不只让你可以控制动画的持续时间，你可以通过 `delay` 函数来延迟动画， 如下所示：

```swift
Animation.default.delay(1.0)
```

![Untitled](%E5%88%A9%E7%94%A8%20delay%E6%9D%A5%E5%AE%9E%E7%8E%B0%E4%BE%9D%E6%AC%A1%E7%9A%84%E5%8A%A8%E7%94%BB%203cea6f9eecb245039f5ea4fd74000fc0/Untitled.png)

```swift
struct ContentView: View {
    @State private var isLoading = false

    var body: some View {
        HStack {
            ForEach(0...4, id: \.self) { index in
                Circle()
                    .frame(width: 10, height: 10)
                    .foregroundColor(.green)
                    .scaleEffect(self.isLoading ? 0 : 1)
                    .animation(.linear(duration: 0.6).repeatForever().delay(0.2 * Double(index)), value: isLoading)
            }
        }
        .onAppear() {
            self.isLoading = true
        }
    }
}
```