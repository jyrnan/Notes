# 类似于NavigationViewHeader

实现一个类似于NavigationViewHeader的效果，可以下拉更新，向上滑动时候头部会固定

![Untitled](%E7%B1%BB%E4%BC%BC%E4%BA%8ENavigationViewHeader%20d2fc8ace88c44011aa6ac8b26438703e/Untitled.png)

关键代码是`LazyVStack(pinnedViews: [.sectionHeaders])`

```swift
struct SectionPin: View {
  var body: some View {
    ScrollView {
      LazyVStack(pinnedViews: [.sectionHeaders]) {
        Section {
          ForEach(1 ..< 100, id: \.self) { num in
            
            Text("Hello, World!\(num)")
              .padding()
              .listRowSeparator(.visible)
              .listRowInsets(EdgeInsets(top: 0, leading: 0, bottom: 0, trailing: 0))
          }
          .onDelete { index in
            print(index)
          }
        } header: {
          
          Text("Pin Header")
            .font(.title)
            .bold()
            .padding()
            .frame(maxWidth: .infinity,maxHeight: .infinity, alignment: .bottomLeading)
            .frame(height: 150)

            .background(Color.green)
        }
      }
      .listStyle(.plain)
    }
  }
}
```