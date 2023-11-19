# 隐藏List的滚动背景（ iOS16 only)

List的背景可以通过.scrollContentBackground(:) 来进行隐藏，这样就可以显示出下面的背景了

```swift
struct SwiftUIView: View {
  @State private var show: Bool = false
  
  private var purpleGradient = LinearGradient(colors: [Color.purple, .green], startPoint: .leading, endPoint: .trailing)
  var body: some View {
    
    List(1..<15) {idx in
      Text("Hello \(idx)")
    }
    .background {
      Color.red
    }
    .ignoresSafeArea()
    .scrollContentBackground(.hidden)
  }
}
```

<aside>
💡 注意：**@available**(**iOS** 16.0, **macOS** 13.0, **watchOS** 9.0, *)

</aside>

![Untitled](%E9%9A%90%E8%97%8FList%E7%9A%84%E6%BB%9A%E5%8A%A8%E8%83%8C%E6%99%AF%EF%BC%88%20iOS16%20only)%204c0742b1c0de42ae9af5468b74bbc5a2/Untitled.png)