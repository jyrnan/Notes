# String Tips

# **字符串字面量的特殊字符**

字符串字面量可以包含以下特殊字符：

- 转义字符 \0(空字符)、\\(反斜线)、\t(水平制表符)、\n(换行符)、\r(回车符)、\"(双引号)、\'(单引号)。
- Unicode 标量，写成 \u{n}(u 为小写)，其中 n 为任意一到八位十六进制数且可用的 Unicode 位码。

# **扩展字符串分隔符**

您可以将字符串文字放在扩展分隔符中，**这样字符串中的特殊字符将会被直接包含而非转义后的效果**。

将字符串放在引号（"）中并用数字符号（#）括起来。

例如，打印字符串文字 `#"Line 1 \nLine 2"#` 会打印换行符转义序列（\n）而不是给文字换行。

如果需要字符串文字中字符的特殊效果，**请匹配转义字符（\）后面添加与起始位置个数相匹配的 # 符。**

例如，如果您的字符串是 #"Line 1 \nLine 2"# 并且您想要换行，则可以使用 #"Line 1 \#nLine 2"# 来代替。

同样，###"Line1 \###nLine2"### 也可以实现换行效果。

你可以使用扩展字符串分隔符创建字符串，来包含不想作为字符串插值处理的字符。例如：

```swift
print(#"Write an interpolated string in Swift using \(multiplier)."#)
// 打印 "Write an interpolated string in Swift using \(multiplier)."
```

# **生成字符Range的问题**

字符可以定义成`Range`，但是因为不是`Stridable`，所以不能步进，也就是说不能用for循环来迭代将其逐一打印出来。但是可以用contain方法来判断是否包含有某个元素。

```swift

let interval = "0"..."~"
print(interval)
if interval.contains("b") {
    print("B")
}

if interval.contains("t") {
    print("yes")
} else {
    print("no")
}
```

同理，不是Stridable，所以不符合Collection，无法用.map()

```swift
interval.map{print($0)} //not compile
```

# **把String格式化成以首字母排序且为组的数据结构**

```swift
struct Section {
        var sectionName : String
        var rowData : [String]
    }
    var sections : [Section] = []
        
    override func viewDidLoad() {
        super.viewDidLoad()
        let states = s.components(separatedBy: " ")
        let d = Dictionary(grouping: states, by: {String($0.prefix(1))})
        self.sections = Array(d).sorted{$0.key < $1.key}.map{Section(sectionName: $0.key, rowData: $0.value)}
        print(sections)
```

# **按照指定条件（字数、字符内容）把字符串分割**

```swift
extension String {
    /**
    通过一个闭包来判断是否属于断开位置，
	实现按照指定字数和指定的字符换行的代码
     */
    func wrapped(after: Int = 70) -> String {
        var i = 0 //字数计数器，在换行后会重置为0
        ///利用了split可以利用一个闭包来判断是否属于serparator，从而对String进行分割
        let lines = self.split(omittingEmptySubsequences: false) {
            character in
            switch character {
            case "\n", " " where i >= after: //如果是转行或者空格，且超过after数量（70）的字符，则分割
                i = 0
                return true //返回true则表示是分割符
            default:
                i += 1
                return false
            }
        }.map(String.init) //对分割后的字符子串进行字符串转换
        return lines.joined(separator: "\n") //把各字符串链接起来
    }
}

let paragraph = "The quick brown fox jumped over the lazy dog."

print(paragraph.wrapped(after: 15))

/**
 可以在上面的基础上进一步实现指定的多个分隔符来分割字符串
 */
extension Collection where Iterator.Element: Equatable {
    func split<S:Sequence>(seperators: S) ->[SubSequence] where Iterator.Element == S.Iterator.Element
    {
        return split{seperators.contains($0)} //返回一个利用闭包判断字符是不是属于seperaor的split方法
    }
}
```