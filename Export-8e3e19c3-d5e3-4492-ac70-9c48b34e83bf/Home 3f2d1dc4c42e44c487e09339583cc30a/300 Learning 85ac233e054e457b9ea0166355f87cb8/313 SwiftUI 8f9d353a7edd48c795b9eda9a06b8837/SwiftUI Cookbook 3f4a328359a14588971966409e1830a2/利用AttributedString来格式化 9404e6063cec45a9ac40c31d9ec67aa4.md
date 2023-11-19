# 利用AttributedString来格式化

# 基本方法

利用AttributedString来格式化内容，然后把它嵌入到其他Text中

## 范例

```swift
var myString: AttributedString {
            var myString = AttributedString("Important Information")
            myString.foregroundColor = .red
            myString.backgroundColor = .yellow
            myString.underlineStyle = .single
            return myString
        }

Text(myString) //嵌入
Text("This is \(myString) so make sure you read it.")
```

![Untitled](%E5%88%A9%E7%94%A8AttributedString%E6%9D%A5%E6%A0%BC%E5%BC%8F%E5%8C%96%209404e6063cec45a9ac40c31d9ec67aa4/Untitled.png)

利用方法生成更多的AttributedString

```swift
func highlight(_ string: String, color: Color = .yellow) -> AttributedString {
        var myString = AttributedString(string)
        myString.backgroundColor = color
        myString.underlineStyle = .single
        return myString
    }

let swiftUI = highlight("SwiftUI")
let uiKit = highlight("UIKit", color: .pink.opacity(0.4))

//嵌入到
Text("Do you use \(swiftUI) or \(uiKit)? in your code")
```

![嵌入效果](%E5%88%A9%E7%94%A8AttributedString%E6%9D%A5%E6%A0%BC%E5%BC%8F%E5%8C%96%209404e6063cec45a9ac40c31d9ec67aa4/Untitled%201.png)

嵌入效果

## 相对更复杂的棋盘格效果范例

AttributedString可以通过“+”链接

```swift
func checkerBoard(_ string: String) -> AttributedString {
        var result = AttributedString()
        for (index, letter) in string.enumerated() {
            var letterString = AttributedString(String(letter))
            letterString.backgroundColor = index % 2 == 0 ? .white : .black
            letterString.foregroundColor = index % 2 == 0 ? .black : .white
            result += letterString //通过加法构成完整
        }
        result.font = .monospacedSystemFont(ofSize: 40, weight: .bold)
        return result
    }

...
Section("Advanced") {
                        Text(checkerBoard("Checkmate")) //嵌入AttributedString
                            .border(.black)
                    }
```

效果

![棋盘格效果](%E5%88%A9%E7%94%A8AttributedString%E6%9D%A5%E6%A0%BC%E5%BC%8F%E5%8C%96%209404e6063cec45a9ac40c31d9ec67aa4/Untitled%202.png)

棋盘格效果