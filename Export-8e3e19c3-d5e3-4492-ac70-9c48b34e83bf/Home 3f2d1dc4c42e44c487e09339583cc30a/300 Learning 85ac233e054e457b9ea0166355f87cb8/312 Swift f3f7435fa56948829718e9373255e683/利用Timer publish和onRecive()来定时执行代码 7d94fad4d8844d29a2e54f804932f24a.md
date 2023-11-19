# 利用Timer.publish和onRecive()来定时执行代码

If you want to run some code regularly, perhaps to make a countdown timer or similar, you should use **Timer** and the **onReceive()** modifier.

```swift
struct ContentView: View {
    let timer = Timer.publish(every: 5, on: .main, in: .common).autoconnect()
    @State var time: Date = Date()
    var body: some View {
        Text("\(self.time)").onReceive(timer, perform: { timer in
            self.time = timer
        })
    }
}
```