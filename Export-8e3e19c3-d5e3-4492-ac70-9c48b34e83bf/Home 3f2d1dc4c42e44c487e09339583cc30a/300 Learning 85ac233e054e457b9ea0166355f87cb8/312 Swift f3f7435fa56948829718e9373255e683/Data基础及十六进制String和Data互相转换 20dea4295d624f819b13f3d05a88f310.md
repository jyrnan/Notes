# Data基础及十六进制String和Data互相转换

# 背景知识：

## Data就是一串Byte

Data是由一串Byte组成，每个Byte是一个8bit的二进制 `01000101`

可以理解成数组： [Byte]

这其实就是内存的形式！

## Data如何表示

### 实质是[UInt8]

UInt8应该是和Byte最匹配的数据类型（当然是8位）。

Swift里面的Data可以看作是[UInt8]类型的数组。参考：[Data: UInt8的数组](Swift%20%E9%87%8C%E7%9A%84Bytes%E4%B8%8E%E6%8C%87%E9%92%88%E7%89%88%E6%9C%AC%20c23a7cb04f1c4ecaab5dd2b491515652.md) ，其实就是把这8个bit的byte等价于`UInt8`这个类型。

那么每个UInt8如何用字面量（不知道这么说合不合适）表示呢？有如下：

### 十进制（默认）

为了阅读方便，UInt8可以转换成一个十进制数字表示。最多可能会是3位数字，*最大255？*

例如`[72, 101, 108, 108, 111, 44, 32, 112, 108, 97, 121, 103, 114, 111, 117, 110, 100]`  

### 双十六进制数

一个Byte其实就是8bit，相当于2 * 4bit，一个4bit就是一个十六进制数，8bit也就是两个十六进制数并列。所以Data的每个Byte（也就是8bit）可以变成2个十六进制数并列

用十六进制的好处是8位二进制可以用2位数字就表示完。而且很匀称。如果不足两位的时候，就在前面添加一个“0”

例如：`[0x13, 0x45, 0x08, 0xA9]`

这里要注意：0x前缀表示是十六进制，但有时候数字作为字符串表示（输出）有时候会省略0x前缀。

如果作为字符串表示的时候，有可能会连起来，就如下：

`“134508A9”`

# 转换

## String → Data

```swift
var str = "Hello, playground"
var data = str.data(using: .ascii)! //编码可以选其他，例如.utf8
```

## Data → [UInt8]→Data

其实在上面转换成Data后也就是内存中一串8bit的Byte的序列了，

但是还可以更显性实现通过Array将起转化成[UInt8]类型的数组，这也是最常见现形式。

```swift
let bytes:[UInt8] = Array(data)
//[72, 101, 108, 108, 111, 44, 32, 112, 108, 97, 121, 103, 114, 111, 117, 110, 100]
//数组里面默认全是十进制的UInt8的表示。

let dataBack = Data(bytes: bytes, count: bytes.count)
//将[UInt8]数组转换成Data,
```

还可以将[UInt8]数组转换成Data，这是和上面一个互逆的过程。

```swift
var bytes: [UInt8] = [72, 101, 108, 108, 111, 44, 32, 112, 108, 97, 121, 103, 114, 111, 117, 110, 100]

var data2 = Data(bytes: bytes, count: bytes.count) //init(bytes: UnsafeRawPointer, count: Int)
let str2 = String(data: data2, encoding: .ascii) //“Hello, playground”
```

<aside>
🤔 不过这个转换回Data的方法需要注意：`init(bytes: UnsafeRawPointer, count: Int)`，传入的第一个参数是一个指针。**看起来swift里面和c类似所谓array的变量其实也就是一个指向第一个字节的指针**。因此，除了传递指针外还需要知道读取多少个字节，也就是参数2 `count` 的作用了

</aside>

## Data转换成十六进制字符串表示

我们需要将Data的[UInt8]用十六进制数来表示，并将数字转换成字符串。

思路是把Data的每个Bytes转换成String，转换String的时候格式采用十六进制。方法是扩展Data，添加一个`hexString`的方法。这个方法是针对Data的每一个Byte对转换成String，并连接起来

前面讲到Data其实就是UInt8类型数组，所以可以对其使用Sequence通用方法，例如：`map`

```swift
extension Data {
	func hexString() -> String {
		return self.map { String(format:"%02x", $0) }.joined()
	}
}
```

有了这个扩展方法可以把Data转换成十六进制数字串了

```swift
var hexStr = data.hexString() // 输出：“48656c6c6f2c20706c617967726f756e64”
```

## 十六进制字符串表示转换成Data

将上面的字符串（假定为十六进制）转换成Data，可以创建String的扩展方法

- 用到了正则表达式
- 转换结果的是optional<Data>

```swift
extension String {

    /// Create `Data` from hexadecimal string representation
    ///
    /// This takes a hexadecimal representation and creates a `Data` object. Note, if the string has any spaces or non-hex characters (e.g. starts with '<' and with a '>'), those are ignored and only hex characters are processed.
    ///
    /// - returns: Data represented by this hexadecimal string.

    func hexadecimal() -> Data? {
        var data = Data(capacity: self.count / 2)

        let regex = try! NSRegularExpression(pattern: "[0-9a-f]{1,2}", options: .caseInsensitive)
        regex.enumerateMatches(in: self, range: NSMakeRange(0, utf16.count)) { match, flags, stop in
            let byteString = (self as NSString).substring(with: match!.range)
            var num = UInt8(byteString, radix: 16)!
            data.append(&num, count: 1)
        }

        guard data.count > 0 else { return nil }

        return data
    }

}
```

用法如下：

```swift
let dataBack = hexStr.hexadecimal()! //直接用上面的转换后强制解包，实际需要判断
```

## Data → String

```swift
let strBack = String(data: dataBack, encoding: .ascii) //编码和上面转换环节一致
```