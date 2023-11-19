# 如何在SwiftUI View中增加执行代码

```swift
struct GeometryGetter: View {
    @Binding var rect: CGRect {
        didSet {
            print(#line, self.rect)
        }
    }

    var body: some View { //这是一段很好的增加执行代码的范例
        GeometryReader { geometry in
            Group { () -> AnyView in
                DispatchQueue.main.async {
                    self.rect = geometry.frame(in: .global)
                }

                return AnyView(Color.clear)。/。/返回一个透明色，其实也就是相当于没有实际效果
            }
        }
    }
}
```