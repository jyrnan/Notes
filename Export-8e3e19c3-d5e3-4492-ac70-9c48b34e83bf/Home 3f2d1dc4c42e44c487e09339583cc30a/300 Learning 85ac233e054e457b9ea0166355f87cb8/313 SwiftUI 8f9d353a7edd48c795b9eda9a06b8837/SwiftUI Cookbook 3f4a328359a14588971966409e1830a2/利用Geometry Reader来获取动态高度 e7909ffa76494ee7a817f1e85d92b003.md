# 利用Geometry Reader来获取动态高度

### 方法一：利用PreferenceKey实现

先创建PreferenceKey

```swift
struct TextHeightKey: PreferenceKey {
  static var defaultValue: CGFloat = 0
  
  static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
    value = value + nextValue()
  }
}
```

在需要量取的View创建相应方法

- preference
- onPreferenceChange

```swift
@State var textHeight: CGFloat = 0
...

ScrollView(showsIndicators: false) {
          Text(text)
            .font(.custom("PingFangSC-Regular", size: 16))
            .foregroundColor(.init("TextColor2"))
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(GeometryReader {
              Color.clear
                .preference(key: TextHeightKey.self,
                            value: $0.frame(in: .local).size.height)
            })
        }
        .frame(height: scrollViewHeight)
        .onPreferenceChange(TextHeightKey.self) {
          textHeight = $0
          ///整个窗口的高度是ScrollView的高度加上上部和下部空间
          ///这个高度会返回到上层View来决定偏移动画数值
          newVersionViewHeight = 188 + scrollViewHeight + 108
        }
```

### 方法二（肘子博客）

```swift
// 获取视图尺寸
struct SizeInfoModifier: ViewModifier {
    @Binding var size: CGSize
    func body(content: Content) -> some View {
        content
            .background(
                GeometryReader { proxy in
                    Color.clear
                        .task(id: proxy.size) {
                            size = proxy.size
                        }
                }
            )
    }
}

extension View {
    func sizeInfo(_ size: Binding<CGSize>) -> some View {
        self
            .modifier(SizeInfoModifier(size: size))
    }
}
```