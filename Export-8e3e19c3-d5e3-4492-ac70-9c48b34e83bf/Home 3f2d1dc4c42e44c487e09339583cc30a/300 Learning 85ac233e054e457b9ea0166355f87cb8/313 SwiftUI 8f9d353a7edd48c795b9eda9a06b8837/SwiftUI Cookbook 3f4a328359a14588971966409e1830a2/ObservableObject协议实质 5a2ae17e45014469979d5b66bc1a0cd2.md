# ObservableObject协议实质

关于深入了解可以参考这篇：[@PublishedObject思考](../../312%20Swift%20f3f7435fa56948829718e9373255e683/@PublishedObject%E6%80%9D%E8%80%83%20d90786b23f414624a1d74423defb3020.md) 

# 协议本质

ObservableObject的协议其实有一个`var objectWillChange: Self.ObjectWillChangePublisher`， 这个Publisher会被一些View订阅，在必要的时候会发送通知订阅方。

这个通知需要在相应属性的willSet方法里面发送，这样就实现了一旦属性发生改变，订阅了这个Publisher的subscriber就会收到通知。

```swift
class MyObserveObject: ObservableObject {
  
  var name: String = "Observed" {
    willSet {
      objectWillChange.send() // 发送通知
    }
  }
  
  func changeName() {
    self.name += "add"
  }
}
```

# @Published

其实，@Published就是把上面那段代码用包装器来实现了。这就是实质所在

```swift
class MyObserveObject: ObservableObject {
  
  @Published var name: String = "Observed" 
  
	func changeName() {
    self.name += "add"
  }
}
```