# 自动闭包 @autoclosure

```swift
let evens = [2,4,6]
if !evens.isEmpty && evens[0] > 10 {
    // 执行操作
}

///定义一个and操作符，采用这种形式，第二个参数必须是一个闭包
func and(_ l: Bool, _ r: ()->Bool) -> Bool {
    guard l else {
        return false
    }
    return r()
}

///运用方式
if and(!evens.isEmpty, {evens[0] > 10}) {
    ///执行操作
}

///增加一个@autoclosure的标注
func and2(_ l: Bool, _ r: @autoclosure ()-> Bool) -> Bool {
    guard l else {return false}
    return r()
}

///第二个参数不需要闭包形式了，自动会·形成闭包
if and2(!evens.isEmpty, evens[0]>10) {
    //执行操作
}
```