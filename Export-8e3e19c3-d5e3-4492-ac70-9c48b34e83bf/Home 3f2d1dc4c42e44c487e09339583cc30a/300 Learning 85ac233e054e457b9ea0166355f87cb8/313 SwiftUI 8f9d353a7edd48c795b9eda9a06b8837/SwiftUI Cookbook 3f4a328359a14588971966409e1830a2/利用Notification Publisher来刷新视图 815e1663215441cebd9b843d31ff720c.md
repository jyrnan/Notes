# 利用Notification.Publisher来刷新视图

### 步骤

- A: 创建Notification.name
- B: 创建动作发送Notification，顺带上object
- C: 接收Notification.Publisher, 顺带接收object

```swift
struct SplitSubView: View {
  var body: some View {
    VStack{
      let _ = Self._printChanges()
      Button {
				/// B：
        NotificationCenter.default.post(name: .test, object: Date())
      } label: {
        Text("Now!")
      }.buttonStyle(.borderedProminent)

      SubTextView()
    }
  }
}

struct SplitSubView_Previews: PreviewProvider {
  static var previews: some View {
    SplitSubView()
  }
}

struct SubTextView: View{
  @State var time: Date = .now
  var body: some View {
    let _ = Self._printChanges()
    Text(time.formatted(date: .abbreviated, time: .standard))
			/// C：
      .onReceive(NotificationCenter.default.publisher(for: .test)) { date in
        self.time = date.object as! Date
      }
  }
}

/// A：
extension Notification.Name {
  static var test = Notification.Name("test")
}
```

![Untitled](%E5%88%A9%E7%94%A8Notification%20Publisher%E6%9D%A5%E5%88%B7%E6%96%B0%E8%A7%86%E5%9B%BE%20815e1663215441cebd9b843d31ff720c/Untitled.png)