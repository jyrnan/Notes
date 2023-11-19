# Evrionment变量存取的两个层级概念及Value和Object区别

# Environment**Value**

**设置及自定义实现方式**

先定义一个Environment的Key，这个Key的要求就是要有default值

```swift
struct PointSizeKey: EnvironmentKey {
    typealias Value = CGFloat
    static var defaultValue: Value = 0.1
}
```

再定义一个Enviroment的变量，这个变量的存取方式就是通过，Enviroment本身的某个key对应的值进行存取，
这个key就是上面定义的key
这个Key和Value的对应方式可以理解成Enviroemnt内部的数据的存储方式
和下面表现的从Enviroment外存取变量的区别

```swift
extension EnvironmentValues {
    var knobPointSize: CGFloat {
        get {
            self[PointSizeKey.self]
        }
        set {
            self[PointSizeKey.self] = newValue
        }
    }
}
```

```swift
///定义好key以及该key对应的Value后，就可以对这个Value进行读取了。
///这个可以理解成是Enviroment对外的展现形式
///所以访问的方式是\.变量名来进行访问读取。注意和上面的区别
extension View {
    func knobPointSize(size: CGFloat) -> some View {
        environment(\.knobPointSize, size)
    }
}
```

```swift
///使用范例
struct KnobView: View {
    @Environment(\.knobPointSize) var envPointSize
    var body: some View {
        Text("Hello")
    }
    
    func setEnvPoint() -> some View {
        Text("hello")
            .knobPointSize(size: 0.2)
    }
}
```

# EnvironmentObject

和上面的区别是这个是Environment Object，而不是Value。所以这个Object的更新会导致View的改变，这个Object必须是符合`ObservableObject`才可以

```swift
///定义需要注入环境变量的Object，必须是ObservableObject
class DatabaseConnection: ObservableObject {
    @Published var isConnected: Bool = true
}

///在上层View注入这个Object的实例
struct TopView: View {
    var body: some View {
        NavigationView {
            MyView()
                .environmentObject(DatabaseConnection())
        }
    }
}

///在下层的View里面引入上层注入环境的Object
struct MyView: View {
    @EnvironmentObject var connection: DatabaseConnection
    var body: some View {
        VStack {
            if connection.isConnected {
                Text("Connected")
            }
        }
    }
}
```

> “虽然 @EnvironmentObject 模式很有用，我们还是建议在可能的时候用带有 key 的 @Environment 并传递值类型，因为这相对来说更安全：EnvironmentKey 要求我们提供默认值。当使用 @EnvironmentObject 的时候，我们非常容易忘记注入对象从而引起严重崩溃。”
Excerpt From: Chris Eidhof. “SwiftUI 编程思想.” Apple Books.
>