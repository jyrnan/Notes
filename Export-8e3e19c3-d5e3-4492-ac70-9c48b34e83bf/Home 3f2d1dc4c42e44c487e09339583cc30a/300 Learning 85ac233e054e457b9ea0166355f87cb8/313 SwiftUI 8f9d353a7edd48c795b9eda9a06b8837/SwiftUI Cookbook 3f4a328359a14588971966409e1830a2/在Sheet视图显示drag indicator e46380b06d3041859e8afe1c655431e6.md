# 在Sheet视图显示drag indicator

弹出的sheet视图默认没有drag indicator， 可以通过如下方法添加

```swift
struct CustomSheetDetentView: View {
    @State private var showSheet = false
    var body: some View {
        Button{
            showSheet.toggle()
        } label: {
            Text(/*@START_MENU_TOKEN@*/"Hello, World!"/*@END_MENU_TOKEN@*/)
        }
        .sheet(isPresented: $showSheet) {
            Color.red
                .ignoresSafeArea()
                .presentationDragIndicator(.visible) // 添加方法
                .presentationDetents([.tiny])
        }
    }
}
```

![Untitled](%E5%9C%A8Sheet%E8%A7%86%E5%9B%BE%E6%98%BE%E7%A4%BAdrag%20indicator%20e46380b06d3041859e8afe1c655431e6/Untitled.png)