# 函数

# 综述

## 核心

Swift和许多现代语言一样，都把函数看作是“头等对象”。你既可以把函数赋值给变量，也可以将它作为其他函数的参数或返回值。

> 这里值得注意的是，我们不能在 funVar 调用时包含参数标签，而在 printInt 的调用 (像是 printInt(i: 2)) 却要求有参数标签。Swift 只允许在函数声明中包含标签，这些标签不是函数类型的一部分。也就是说，现在你不能将参数标签赋值给一个类型是函数的变量，不过这在未来的 Swift 版本中可能会有改变。
> 

## 捕获

函数应用其作用域之外的变量，这个变量就被**捕获**了

## 闭包

闭包和func相比就是没有被赋予一个名字。只能用来赋值或者作为参数传递

> 闭包是一个函数以及它所捕获的所有变量的组合。
> 

# 函数的灵活性

该章节的若干都是将函数作为参数传入其他函数的范例，需要细看。特别是函数的签名形式需要理解。

```swift
extension SortDescriptor {
		func then(_ other: SortDescriptor<Root>) -> SortDescriptor<Root> {
			SortDescriptor { x, y in
				if areInIncreasingOrder(x,y) { return true } //调用方法，如果true则返回true
				if areInIncreasingOrder(y,x) { return false } //参数反过来调用如果true则返回false
				return other.areInIncreasingOrder(x,y) //上面两个都不满足，则返回新的SortDescriptor内部的排序方法
		}
	}
}
//这段太抽象了。😳
```

# 函数作为属性来实现回调或代理功能

这个范例需要认真消化，完整版在P118页

```swift
class AlertView {
    var buttons: [String]
    var buttonTapped:((_ buttonIndex: Int) -> Void)?
    
    init(buttons: [String] = ["OK", "Cancel"]) {
        self.buttons = buttons
    }
    
    func fire() {
        buttonTapped?(1)
    }
}

struct TapLogger {
    var taps:[Int] = []
    
    mutating func logTap(index: Int) {
        taps.append(index)
    }
}

let alert = AlertView()
var logger = TapLogger()

//直接这样赋值会报错，因为这是一个结构体内部方法。“在上面的代码中，这个赋值的结果不明确。是 logger 需要复制一份呢，还是 buttonTapped 需要改变它原来的状态 (即 logger 被捕获) 呢？”
//alert.buttonTapped = logger.logTap // Cannot reference 'mutating' method as function value

alert.buttonTapped = {logger.logTap(index: $0)}// 需要将这个方法包在一个闭包中，可以将方法的主体logger捕获，闭包的主体方法实现靠的是捕获的logger内部方法来实际实现，闭包本身只是一个壳，同时实现参数的传递
```

### 代理vs闭包回调

如果代理的方法只有一个的时候，可以如上所述设置回调方法的变量来实现。

在通过闭包实现回调（代理功能）的时候，要小心闭包对主体的循环引用。需要通过`[weak self]`来避免。对比通过实现协议采用delegate的形式，delegate可以显式的标注成weak，和闭包里面的做法其实是一样的。

# inout参数和可变方法

如果你有些 C 或 C++ 的背景，Swift 中用在 inout 参数前面的 & 可能会给你一种这是在传递引用的错觉。但事实并非如此，inout 做的事情是传值，然后复制回来，并不是传递引用。 引用官方《Swift 编程语言》中的话：

> 一个 in-out 参数持有一个传入函数的值，函数可以改变这个值，然后从函数中传出并替换掉原来的值。
> 

> 编译器可能会把 inout 变量优化成引用传递，而非传入和传出时的复制。不过，文档已经明确指出我们不应该依赖这个行为。 HaHa🤣
> 

## lvalue & rvalue

lvalue是左侧值，描述的是一个内存地址。它是可以存在于赋值语句左侧的表达式。例如arr[0]

rvalue是右侧值，描述的是一个值，例如2+ 2

对于inout参数，只能传递lvalue。rvalue是一个值是不能作为inout参数传递的，类似的所有没有set方法的属性也是不行。

## 嵌套函数和inout

inout嵌套没有问题，但是不能逃逸。可以这么理解：在函数返回之前需要把inout参数的值复制回去。所以……

<aside>
💡 看起来确实和引用不一样吧，但是原理还是有点不清楚

</aside>

# &不意味着inout的情况

这时候，是真的引用，例如把一个函数参数转换成不安全指针。

在Swift中，是不能把指针传递出当前方法的作用域的。例如： **我不是很理解**😳

```swift
“func incref(pointer: UnsafeMutablePointer<Int>) -> () -> Int {
// 将指针的的复制存储在闭包中
	return {
		pointer.pointee += 1
		return pointer.pointee
	}
}

//我们会在后面的章节介绍，Swift 的数组可以无缝地隐式退化为指针，这使得将 Swift 和 C 一起使用的时候非常方便。现在，假设在调用这个函数之前，你传入的数组已经离开其作用域了：
let fun: () -> Int
	do {
		var array = [0]
		fun = incref(pointer: &array)
}
/*_*/fun()

```

# 下标

下标操作类似于函数，因为接受参数。又有点像属性，因为可以get和set。

可以通过重载overloading 不同类型的下标操作符

## 自定义下标操作

如同定义一个函数一样，只是这个函数的名字就叫subscript，也不需要func关键词。或者说关键词和名字合二为一。

```swift
extension Collection {
    subscript(indies indexList:Index...) -> [Element] {
        var result: [Element] = []
        for index in indexList {
            result.append(self[index])
        }
        return result
    }
}
```

# @escaping标注

“一个被保存在某个地方 (比如一个属性中) 等待稍后再调用的闭包就叫做逃逸闭包。相对的，永远不会离开一个函数的局部作用域的闭包就是非逃逸闭包。对于逃逸闭包，编译器强制我们在闭包表达式中显式地使用 self，因为无意中对于 self 的强引用，是发生引用循环的最常见原因之一。当一个函数返回的时候，非逃逸闭包会自动销毁，所以它不会创建一个固定的引用循环。”

<aside>
💡 所以，对比之下最大的区别就是这个闭包使用的环境是不是当前的函数的Scope
逃逸闭包是保存在堆中？

</aside>

# Result Builder

在标注了`@resutlBuilder`的方法中，编译器会使用定义在相应result Builder类型上的一些静态方法，例如将多个View合成一个View。

Result builder 类型被 `@resultBuilder` 标记，它可以实现一系列不同的静态 build... 方法，我们在本节剩余的部分将会检查更多细节。哪些方法被实现，将决定在 result builder 函数中可以使用哪些类型的表达式和语句。

## Block和表达式

参考

[Result Builder的理解](../Result%20Builder%E7%9A%84%E7%90%86%E8%A7%A3%20583d6123702c4972b234ec5a483611dc.md)