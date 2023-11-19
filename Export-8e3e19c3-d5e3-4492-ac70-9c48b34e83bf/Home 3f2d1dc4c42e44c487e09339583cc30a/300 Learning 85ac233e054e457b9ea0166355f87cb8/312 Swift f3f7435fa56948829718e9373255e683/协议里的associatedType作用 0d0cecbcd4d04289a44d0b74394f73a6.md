# 协议里的associatedType作用

```swift
protocol IteratorProtocol {
    associatedtype Element //A：
    mutating func next() -> Element?
}

struct ReverseIndexIterator: IteratorProtocol {
    
    typealias Element = Int
    ///就是用来明确associatedType（A）是什么的。
    ///例如协议规定的func输出的类型是是什么。
    ///因此，如果协议的func明确了输出类型（B），那么typealias也就不用了

    var index: Int
    
    init<T>(array: [T]) {
        index = array.endIndex - 1
    }
    
    mutating func next() -> Int? {  //B:
        guard index >= 0 else {return nil}
        defer {index -= 1} //这个语法表示括弧里的操作在return之前无论如何都会执行
        return index
    }
}
```

```swift
Protocol的若干概念

1. Protocol在定义的时候不能写方法具体的内容
protocol Drawing {
    mutating func addEllipse(rect: CGRect, fill: UIColor)
    mutating func addRectangle(rect: CGRect, fill: UIColor)
    mutating func addCircle(center: CGPoint, radius: CGFloat, fill: UIColor) ///A：
}

可以通过扩展协议来给协议的方法提供具体的实现方法。
extension Drawing {
    mutating func addCircle(center: CGPoint, radius: CGFloat, fill: UIColor) {
        print("YES, Got Circle!")
    }
}
通过扩展提供的协议方法，就可以用于其他遵循该协议的类型直接使用。这就是对应了类的方法和属性继承，但是又有足够多的优点！
var context: Drawing = SVG()
context.addCircle(center: CGPoint.zero, radius: 10, fill: .black)

如果该类型里面也直接定义了同样的方法，则在调用该类型的方法的时候会采用该方法
//例如这个方法调用的结果是打印"Add Circle From SVG"
extension SVG {
    mutating func addCircle(center: CGPoint, radius: CGFloat, fill: UIColor) {
        print("Add Circle From SVG")
    }
}

如果在协议申明的地方（1处）加入了A处的方法的显性申明定义，则相当于在协议增加了一个方法的入口，会采用动态派发，我的理解是： 会根据协议或类型是否有具体的方法实现来采用，应该是优先采用具体类型实现的方法，如果没有则调用协议默认的实现方法
```