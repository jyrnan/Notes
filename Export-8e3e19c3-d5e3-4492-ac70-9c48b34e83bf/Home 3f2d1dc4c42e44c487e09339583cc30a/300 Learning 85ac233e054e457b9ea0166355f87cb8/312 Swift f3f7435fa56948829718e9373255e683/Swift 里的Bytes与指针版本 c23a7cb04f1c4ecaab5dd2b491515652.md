# Swift 里的Bytes与指针版本

# A：Swift里的Bytes

这一部分是@zonble 的文章，写的很棒。

[在 Swift 裡頭操作 Bytes](https://zonble.github.io/swift/2019/11/27/bytes-in-swiftl.html)

從 2014 年 Swift 語言正式公開以來，就可以很明顯看出，這個語言在設計上的一大方向之一，就是想盡辦法想要隱藏指標這件事情，讓大家不需要碰到指標，甚至 byte、bit 這些觀念，就可以寫出程式來。

像是 bit flags 就被寫成是 array 的語法，在 Objective-C 當中原本 Foundation 物件都是 reference type，到了 Swift 3 以後又統統都是 value type，然後每一代 Swift 在跟指標有關的 API 都在大改。

有的時候你還是要寫點加解密之類的東西，像是先把數字轉成字串然後加個鹽巴再 AES 加密再 Base64 再把字串反轉…諸如此類，這時候就會一定會碰到 Binary 資料。這時候，如果你還是抱持著寫 C 語言的那種習慣的話，寫起 Swift 就會十分彆腳。

但很多時候，其實你根本不需要真的操作指標，你只是想要知道指標所指向的位置有什麼東西，換句話說，你只需要知道那些 bytes。在 Swift 裡頭，很多原本需要使用指標的場合，就只要把它當成 UInt8 的 Iterable 就可以了。

### 我自己的总结参考：[Data基础及十六进制[String](https://my.oschina.net/u/2426110/blog/967583)和Data互相转换](Data%E5%9F%BA%E7%A1%80%E5%8F%8A%E5%8D%81%E5%85%AD%E8%BF%9B%E5%88%B6String%E5%92%8CData%E4%BA%92%E7%9B%B8%E8%BD%AC%E6%8D%A2%2020dea4295d624f819b13f3d05a88f310.md)

## Data: UInt8的数组

基本上現在你可以在 Swift 裡頭的 Data 型態，當成像是 [UInt8] 一樣使用。像是我們想要把一段 Data 裡頭的每個 byte 倒出來看看：

```swift
var str = "Hello, playground"
var data = str.data(using: .ascii)!
for byte in data {
	print(String(format: "%2x", byte))
}
//打印出如下
48
65
6c
6c
6f
2c
20
70
6c
61
79
67
72
6f
75
6e
64
```

也可以用 map 這些 method。

```swift
extension Data {
	func hexString() -> String {
		return self.map { String(format:"%02x", $0) }.joined()
	}
}

var str = "Hello, playground"
var data = str.data(using: .ascii)!
var hex = data.hexString()

```

其餘可以用的還包括 count、append、remove… 等等。

所以，我們就可以來寫一個簡單到一定超容易被破解的凱薩加密：

```swift
func cipher(data: Data, offset:Int) -> Data {
	let array = data.map { UInt8((Int($0) + offset) % 256) }
	return Data(array)
}

var str = "Hello, playground"
var data = cipher(data: str.data(using: .ascii)!, offset: 3)
var result = String(data: data, encoding: .utf8)
```

**Data 與 [UInt8] 的轉換也很簡單：**

```swift
var str = "Hello, playground"
var data = str.data(using: .ascii)!
var dataToArray = data.map { $0 } // 把 Data 轉成 [UInt8]
var arrayToData = Data(dataToArray) // 把 [UInt8] 轉成 Data

```

例如：把bytes转换成`[UInt8]`的表示方式

```swift
var str = "Hello, playground"
var data = str.data(using: .ascii)!
Array(data)
//[72, 101, 108, 108, 111, 44, 32, 112, 108, 97, 121, 103, 114, 111, 117, 110, 100]
//里面全是十进制的UInt8的表示。
```

数组里面的UInt8除了十进制当然也可以用其它的形式来表示，最常用的就是十六进制。例如

```swift
var data2 = Data(capacity: 50)
data2.append(contentsOf: [0x01, 0x00]) //用0x开头来表示十六进制数组
```

## UnsafeRawBufferPointer：基于指针的字节流

至於我們想要拿到組成一個數字用的 Bytes，則可以呼叫 `withUnsafeBytes` 這個 function，然後我們會在 `withUnsafeBytes` 的 callback closure 裡頭拿到一個叫做 UnsafeRawBufferPointer 的物件。這個物件用起來…也很像是 [UInt8]。

比方說，我們可以把一個 32 位元整數的 4 個 bytes 倒出來看看：

```swift
**var** int: **Int32** **=** 1

**withUnsafeBytes**(of: int) { bytes **in 
		for** byte **in** bytes {
		**print**(**String**(format: "%02x", byte))
	}
}
```

把數字轉成 Bytes 有什麼用呢？舉個在 iOS 開發會遇到的情境好了：在處理一些比較底層的行為的時候，蘋果習慣使用 OSStatus 當做發生錯誤時的錯誤代碼，這個代碼其實是一個 32 位元整數，但是這個整數到底是什麼意思，往往要把這個整數轉換成四個 char，從 char 對應的字母嘗試解讀：這四個字母通常是某個單字的縮寫。

```swift
**var** int: **UInt32** **=** 1718449215

**var** x **=** **withUnsafeBytes**(of: int**.**bigEndian) { bytes **in
		return** **String**(bytes: **Array**(bytes), encoding: **.**ascii)
}
```

我們就可以得到 “fmt?” 這個字串，大概可以當成 “format?” 的意思，代表的意義是 Core Audio 不支援我們要求播放的檔案格式。

如果我們拿到 “fmt?”，想要轉回 1718449215，也可以用 withUnsafeBytes 以及 UnsafeRawBufferPointer：

```swift
**var** str **=** "fmt?"
**var** data **=** str**.data**(using: **.**ascii)**!
withUnsafeBytes**(of:data) { bytes **in
		return** bytes**.load**(as: **UInt32.self**)**.**bigEndian
}
```

我們也可以把 [UInt8] 倒進 UnsafeRawPointer 裡頭，再轉成整數。

```swift
**var** bytes:[**UInt8**] **=** [102, 109, 116, 63]
**UnsafeRawPointer**(bytes)**.load**(as: **UInt32.self**)**.**bigEndian
```

<aside>
💡 下面的内容是结合原文与参考资料的对于Pointer的一些理解。

</aside>

# B：Swift里的Pointer

在Swift里面和Pointer相关的大概有如下几个关键字，Buffer、Raw、Mutable这几个关键词和Ponter组合表示了该指针的不同类型；Bytes则用于一个方法名中表示返回的不同的指针类型，具体如下：

- **Buffer**：表示了该指针是普通指针还是表示连续bytes的指针
- **Raw**：表示了指针是否具有类型。
- **Mutable**：这个表示指针是否可以被修改

---

- **Bytes**：这个和前面不同，其实不是直接表示指针类型，而是在一个withUnsfae…方法中表示返回的是一个连续bytes还是普通指针

[Advanced Swift L1 - Memory Layout & Pointers](https://www.youtube.com/watch?v=o-Ta5AIUSKM)

![Untitled](Swift%20%E9%87%8C%E7%9A%84Bytes%E4%B8%8E%E6%8C%87%E9%92%88%E7%89%88%E6%9C%AC%20c23a7cb04f1c4ecaab5dd2b491515652/Untitled.png)

## Buffer关键字

Buffer关键字和Pointer组合会代表是普通的指针还是代表了连续Bytes的指针。

下面是@Zonble 的文章阐述了这个问题。

### UnsafeRawPointer 與 UnsafeRawBufferPointer

[UnsafeRawPointer 與 UnsafeRawBufferPointer](https://zonble.github.io/2019/12/03/unsaferawpointer-and-unsaferawbufferpointer.html)

UnsafeRawPointer 與 UnsafeRawBufferPointer 這兩個 structure 名字差不多，不過代表的東西不同。UnsafeRawPointer 可以理解成就是 C 語言裡頭的 pointer（也就是 void *） ，至於 UnsafeRawBufferPointer 就是連續的 bytes，可以理解成我們可以從裡頭拿到 UInt8 的 Array。

簡單寫一段 code：

```swift
**var** ptr: **UnsafeMutableRawPointer** **=** **calloc**(1, **MemoryLayout<Int>.**size)
ptr**.storeBytes**(of: 123, as: **Int.self**)
**var** bytes: **UnsafeRawBufferPointer** **=** **UnsafeRawBufferPointer**(start:ptr, count:**MemoryLayout<Int>.**size)
**Array**(bytes)
**let** i1 **=** ptr**.load**(as: **Int.self**)
**let** i2 **=** bytes**.load**(as: **Int.self**)
```

<aside>
💡 **下面是我自己的补充**：

</aside>

```swift
//创建RawPointer，要用C方法来分配一块内存，大小是Int类型，默认64位机是8个字节。返回该段内存的第一个地址
//该方法返回的就是一个内存（起点）地址，并没有大小的信息
//这个方法其实是不知道具体存储类型的，所以要明确告诉所需分配内存块大小（编译成ASSM时，直接按照大小来做参数）
var ptr: UnsafeMutableRawPointer = calloc(1, MemoryLayout<Int>.size)
//UnsafeMutableRawPointer(60000022C000)

//给该指针位置存入Bytes，告知类型其实就是告知了要存储几个字节
ptr.storeBytes(of: 150, as: Int.self)
//UnsafeMutableRawPointer(60000022C000)

//其实和前面非常接近，只不过名字是Pointer，但其实返回的是连续的bytes
var bytes:UnsafeRawBufferPointer = UnsafeRawBufferPointer(start: ptr, count: MemoryLayout<Int>.size)
//UnsafeRawBufferPointer(start: 0x000060000022c000, count: 8)

//因为BufferPointer其实是连续bytes，所以可以直接数组化,转换成[UInt8]
Array(bytes)
//[150, 0, 0, 0, 0, 0, 0, 0]

//以上两种都包含了一个指向内存地址的指针（只是后者多个字节数），所以都是可以调用loads方法来载入指针指向的内容。
//但是载入loads方法是都要显性告知as的类型，也就是获得载入的字节数
let i1 = ptr.load(as: Int.self) //150
let i2 = bytes.load(as: Int.self) //150

//针对第一个ptr载入不同类型（不同字节数），就会导致产生不同的值
let i4 = ptr.load(as: Int32.self) //150 Int32还是和默认Int64位保持一致
let i5 = ptr.load(as: Int16.self) //150
let i6 = ptr.load(as: Int8.self) //-106， 数值发生变化，变成了补数
let i7 = ptr.load(as: UInt8.self) //150

//针对bytes载入不同类型（不同字节数），也就会导致产生不同的值
let i42 = bytes.load(as: Int32.self) //150
let i52 = bytes.load(as: Int16.self) //150
let i62 = bytes.load(as: Int8.self) //-106， 数值发生变化，变成了补数
let i72 = bytes.load(as: UInt8.self) //150

//关于大小端的尝试
//改成大端类型后数值变了
var i3 = ptr.load(as: Int.self).bigEndian
//-7638104968020361216

//重新打印出i3的bytes类型可以看出和原来的区别，大小端排列形式不一样
withUnsafeBytes(of: &i3){print(Array($0))}
//[0, 0, 0, 0, 0, 0, 0, 150]

//转换成大端后，bytes排序成[0, 0, 0, 0, 0, 0, 0, 150]
//因此再进行不同的类型转换情况就不一样了，只有Int64能包括到后面的有效位数
//其余的都只能获取前面若干位，也就是都为零了
withUnsafeBytes(of: &i3){print($0.load(as: Int64.self))} //-7638104968020361216
withUnsafeBytes(of: &i3){print($0.load(as: Int32.self))} //0
withUnsafeBytes(of: &i3){print($0.load(as: Int16.self))} //0
withUnsafeBytes(of: &i3){print($0.load(as: Int8.self))} //0

//可以更进一步改变成无符号整数
withUnsafeBytes(of: &i3){print($0.load(as: UInt64.self))} //10808639105689190400
withUnsafeBytes(of: &i3){print($0.load(as: UInt32.self))} //0
withUnsafeBytes(of: &i3){print($0.load(as: UInt16.self))} //0
withUnsafeBytes(of: &i3){print($0.load(as: UInt8.self))} //0
```

這邊的 calloc 就是 C 語言的 calloc，應該會回傳一個 pointer 回來，在 Swift 裡頭，就會包裝成 UnsafeMutableRawPointer 這種形態。我們在這邊，建立了只有一個整數大小的記憶體位置。

我們可以從 UnsafeMutableRawPointer 建立 UnsafeRawBufferPointer，但是在建立 UnsafeRawBufferPointer 的時候，除了指標的位置之外，還要提供這段記憶體的長度。

接著，無論是 UnsafeMutableRawPointer 或是 UnsafeRawBufferPointer，我們都可以透過呼叫 `.load(as:)`，宣告成我們所想要的形態。

<aside>
💡 **这里示范了如何调用C里面的变量和方法！！！**

</aside>

所以，在 Swift 3 的年代，我們想要呼叫 Common Crypto 呼叫 MD5，可能寫成這樣：

```swift
**import** **var** CommonCrypto**.CC_MD5_DIGEST_LENGTH
import** **func** **CommonCrypto.CC_MD5
import** **typealias** **CommonCrypto.CC_LONG
func** **MD5_1**(string: **String**) **->** **Data** {
	**let** length **=** **Int**(**CC_MD5_DIGEST_LENGTH**)
	**let** messageData **=** string**.data**(using: **.**utf8)**!var** digestData **=** **Data**(count: length)

	_ **=** digestData**.**withUnsafeMutableBytes { digestBytes **->** **UInt8** **in**messageData**.**withUnsafeBytes { messageBytes **->** **UInt8** **inif** **let** messageBytesBaseAddress **=** messageBytes**.**baseAddress, **let** digestBytesBlindMemory **=** digestBytes**.bindMemory**(to: **UInt8.self**)**.**baseAddress {
				**let** messageLength **=** **CC_LONG**(messageData**.**count)
				**CC_MD5**(messageBytesBaseAddress, messageLength, digestBytesBlindMemory)
			}
			**return** 0
		}
	}
	**return** digestData
}
```

現在可以寫成這樣：

```swift
**import** **var** CommonCrypto**.CC_MD5_DIGEST_LENGTH
import** **func** **CommonCrypto.CC_MD5
import** **typealias** **CommonCrypto.CC_LONG
func** **MD5_2**(string: **String**) **->** **Data** {
	**let** length **=** **Int**(**CC_MD5_DIGEST_LENGTH**)
	**let** digestData **=** [**UInt8**](repeating: 0, count: length)
	**let** messageData **=** **Array**(string**.data**(using: **.**utf8)**!**)
	**let** messageLength **=** **CC_LONG**(messageData**.**count)
	**CC_MD5**(**UnsafeMutableRawPointer**(mutating: messageData), messageLength,**UnsafeMutableRawPointer**(mutating: digestData)**.bindMemory**(to: **UInt8.self**, capacity: length))
	**return** **Data**(digestData)
}
```

## Raw关键字

UnsafePointer和UnsafeRawPointer区别-是否带类型

### 区别点：

**是否具有类型**

- 前者带类型，后者不带类型。也就是是否有设置内存边界
- Memory that has been bound to a type, whether it is initialized or uninitialized, is typically accessed using typed pointers—instances of `UnsafePointer` and `UnsafeMutablePointer`

**指针计算方式**

- 前者按照类型进行加减（偏移）byte数量，Pointer arithmetic with a typed pointer is counted in strides of the pointer’s `Pointee` type.

```swift
var numArr: [UInt32] = [100, 101, 500]

//设置指针<UInt32>,指针指向numArr的第一个数据空间
let interPointer:UnsafePointer<UInt32> = UnsafePointer<UInt32>(numArr)
let x = interPointer.pointee
// x == 100

//偏移2个
let offsetPointer = interPointer + 2
let y = offsetPointer.pointee
// y == 500
```

- 后者因为没有类型，所以偏移是按照byte来执行。Pointer arithmetic with raw pointers is performed at the byte level.

```swift
let bytesPointer = UnsafeMutableRawPointer.allocate(byteCount: 4, alignment: 4)
bytesPointer.storeBytes(of: 0xFFFF_FFFF, as: UInt32.self)

// Load a value from the memory referenced by 'bytesPointer'
let x = bytesPointer.load(as: UInt8.self)       // 255

// Load a value from the last two allocated bytes
let offsetPointer = bytesPointer + 2
let y = offsetPointer.load(as: UInt16.self)     // 65535
```

The code above stores the value `0xFFFF_FFFF` into the four newly allocated bytes, and then loads the first byte as a `UInt8` instance and the third and fourth bytes as a `UInt16` instance.

Always remember to deallocate any memory that you allocate yourself.

```swift
bytesPointer.deallocate()
```

### 共同点

- 都需要自己管理内存
- 都有不同的[内存状态](Swift%20%E9%87%8C%E7%9A%84Bytes%E4%B8%8E%E6%8C%87%E9%92%88%E7%89%88%E6%9C%AC%20c23a7cb04f1c4ecaab5dd2b491515652.md)

> **Understanding a Pointer’s Memory State 指针内存状态**
> 
> - untyped and uninitialized 未类型和初始化
> - bound to a type and uninitialized 已经边界化（类型化）和没有初始化
> - bound to a type and initialized to a value 已经边界化（类型化）和已经初始化

### 类型定义

****UnsafePointer:****

 `@frozen struct UnsafePointer<Pointee>`

You use instances of the `UnsafePointer` type to access data of a specific type in memory. The type of data that a pointer can access is the pointer’s `Pointee` type.

有一个`.pointee`属性，所以指针具有类型，也就是pointee的类型Pointee

****UnsafeRawPointer****

`@frozen struct UnsafeRawPointer`

可以看出这个是没有类型属性的

## Mutable关键字

这个比较好理解了，就是（指针）是否可以被改变。

## Bytes关键字

与之对应还有一个Bytes关键字用在下面的方法，分别返回连续bytes指针或普通指针

![Screenshot 2022-08-10 at 10.34.08.png](Swift%20%E9%87%8C%E7%9A%84Bytes%E4%B8%8E%E6%8C%87%E9%92%88%E7%89%88%E6%9C%AC%20c23a7cb04f1c4ecaab5dd2b491515652/Screenshot_2022-08-10_at_10.34.08.png)

# C：Swift 进阶一书中解释

- 含有 managed 的类型代表内存是自动管理的。编译器将负责为你申请，初始化并且释放内存。
- 含有 unsafe 的类型摒弃了 Swift 常规的安全特性，例如边界检查以及变量使用前必须初始化的保证。它也不提供自动化的内存管理 (这和 managed 正好相反)。你需要明确地进行内存申请，初始化，销毁和回收。
- 含有 buffer 类型作用于在连续存储空间上的多个元素，而非一个单独的元素上。因此，它也提供了 Collection 的接口。
- 含有 pointer 的类型拥有指针的语义 (和 C 中的指针是类似的)。
- 含有 raw 的类型包含无类型的原始数据，它和 C 的 void* 是等价的。名称中不含有 raw 的类型包含的都是具体类型的数据，这些具体类型都是通过各自的泛型参数指定的。
- 含有 mutable 的类型允许修改它指向的内存。

## 指针

`UnsafePointer` 是最基础的指针类型，它与 C 中的 const 指针类似，但是泛化了其指向数据的类型。所以，`UnsafePointer<Int>` 对应的就是 `const int*`。

可以通过申明指针变量为var或let来控制指针的可变性

## 针对函数参数的自动指针转换

“对于那些接受 UnsafePointer 参数的函数，Swift 还提供了一种特殊的调用语法，只要在任意类型正确的可变变量前面加上 & 符号把它变成 in-out 表达式就好了：”

### 不可修改版本

指针对应于c里面的 const Type*

```swift
var x = 5
func fetch(p:UnsafePointer<Int>)->Int {
	p.pointee
}
fetch(p:&x) //5
//“**Swift在幕后创建和传递给函数的指针只保证在函数调用期间是有效的。**切记不要尝试返回这个指针，或者在函数返回之后再去访问它，这么做的结果是未定义的。”
```

### 可修改版本

“还有一个可变版本的指针，那就是 UnsafeMutablePointer。这个结构体的行为和普通的 C 指针很像，你可以对指针进行**解引用**，**并修改内存的值**，这些改动将通过 in-out 表达式的方式传回给调用者：”

```swift
func increment(p:UnsafeMutablePointer<Int>) {
    p.pointee += 1
}
var y = 0
increment(p: &y)
print(y) //1
```

## 指针的生命周期

### 申请内存来构建指针

和C相似：申请内存后必须初始化才能使用

```swift
let z = UnsafeMutablePointer<Int>.allocate(capacity: 2)
z.initialize(repeating: 42, count: 2)
z.pointee //42
(z+1).pointee //42，指针运算
z[1] //42,下标操作指向的对象
z.deallocate()
```

如果Pointee类型是一个需要内存管理的非简单类型（估计是一个指针指向一个复杂对象），需要在deallocate这片内存前先deinitialize那片内存（其实就是管理ARC 中对于该对象的引用计数），否则就会内存泄漏

## 原始指针

指的是包含了Raw字样的指针，对应于C中的 `void*` 或者 `const void*`

通常通过 load(fromByteOffset:as:)方法转换成Unsafe[Mutabale]Pointer或者其他确定了类型的指针。

## 用可选值代表可空指针

“和 C 不同，Swift 使用可选值来区分可能为空的指针和不可能为空的指针。只有那些包含指针的可选值才能表示一个空指针。在底层，UnsafePointer<T> 和 Optional<UnsafePointer<T>> 的内存结构相同；编译器会将 Optional.none 映射为一个所有位全为零的空指针。”

## 不透明指针

是指的类型定义其实没有明确的指针？

swift中对OpaquePointer的类型安全上比较笼统。所以，理想情况下应该是一个泛型，这样OpaquePointer<T>就能区分不同类型的指针了。

## 缓冲区指针

“在 Swift 中，我们经常会使用 Array 来连续存储一系列值。在 C 里，数组通常用指向第一个元素的指针以及元素的个数来表示。如果要将 C 中的数组作为 Swift 中的集合类型使用，我们可以把它转换成 Array，但这会导致元素被复制。通常来说这是件好事 (因为一旦这些元素存在于 Array 中，它们的内存就将由 Swift 运行时管理)。然而，有时候你却不想复制每个元素。对于那些情况，我们可以使用 `Unsafe[Mutable]BufferPointer`，它通过一个指向起始元素的指针和元素的个数来进行初始化。完成后，你就拥有一个可以随机访问的集合了。缓冲区指针让 Swift 与 C 的集合协同工作变得容易很多。”

“是 `Unsafe[Mutable]RawBufferPointer`，它让我们可以直接把原始内存数据当作集合来处理 (它们在底层与 Foundation 中的 Data 是等价的)。”

“Array 还提供了一个 `withUnsafe[Mutable]BufferPointer` 方法，通过缓冲区指针，它让我们可以直接访问到数组的内部存储 (这个操作甚至是可写的)。”

“在 Swift 5 里，这些方法还有了泛型的版本`withContiguous[Mutable]StorageIfAvailable`。”