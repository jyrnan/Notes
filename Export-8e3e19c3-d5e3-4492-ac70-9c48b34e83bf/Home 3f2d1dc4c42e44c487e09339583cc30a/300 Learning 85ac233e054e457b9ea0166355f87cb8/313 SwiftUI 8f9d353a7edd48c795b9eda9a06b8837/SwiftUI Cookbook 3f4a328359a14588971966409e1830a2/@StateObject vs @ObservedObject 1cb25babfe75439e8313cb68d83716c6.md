# @StateObject vs @ObservedObject

<aside>
💡 详细阅读肘子文章： [StateObject 与 ObservedObject | 肘子的Swift记事本](../StateObject%20%E4%B8%8E%20ObservedObject%20%E8%82%98%E5%AD%90%E7%9A%84Swift%E8%AE%B0%E4%BA%8B%E6%9C%AC%200a427b8b05454fab828c94caf3b9eb4b.md)

</aside>

# 区别

可以从两方面来理解：

如果用来作为View的初始资料这二者的区别是什么呢？

```swift
class MyObserveObject: ObservableObject {}

struct SomeView: View {
@StateObject var object: MyObserveObject = .init()
vs
@ObservedObject var object: MyObserveObject = .init()
...
```

`@StateObject` 会绑定一个View，无论这个View刷新多少次，并不会重新生产底层的MyObserveObject：ObservableObject，这个工作是SwiftUI框架来确保的。而`@ObservedObject`则做不到这一点，每次View新生成（SwiftUI的特性就是如此，不断生产新的View），都会调用.init来生成底层的MyObserveObject。

如果SomeView不会不断生成，例如初始View，二者就有可能通用。

完整范例代码：SubView里面的object采用不同的方式，会存在不一样的初始化次数。

```swift
struct ObservedView: View {
  @ObservedObject var object: MyObserveObject = .init()
    var body: some View {
      VStack{
        Text(object.name)
        
        Button {
          object.changeName()
        } label: {
          Text("Change Name")
        }
        
        SubView()

      }
    }
}

struct SubView: View {
  @StateObject var object: MyObserveObject = .init()
  var body: some View {
    Text(object.name)
  }
}

struct ObservedView_Previews: PreviewProvider {
    static var previews: some View {
        ObservedView()
    }
}

class MyObserveObject: ObservableObject {
  
  var name: String = "Observed" {
    willSet {
      objectWillChange.send()
    }
  }
  
  func changeName() {
    self.name += "add"
  }
}
```

# 正确用法

其实应该理解成`@ObservedObject`是 `@StateObject`的Binding版本😂 。

利用`@StateObject`来创建和View绑定的`ObservableObject`，然后在传给其他View中标注`@ObservedObject` 的参数。可以理解成把引用传递（自然不会新建物体）过去，并具备SwiftUI中的更新特性。

参考@jane视频中的总结。

![Untitled](@StateObject%20vs%20@ObservedObject%201cb25babfe75439e8313cb68d83716c6/Untitled.png)

[觀察 Reference Type 的變化：ObservableObject、用 MainActor 處理畫面更新 - SwiftUI 新手入門](https://www.youtube.com/watch?v=rrD2I4bVzFc&t=2s)