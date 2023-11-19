# 细说 Swift 4.2 新特性：Dynamic Member Lookup - 掘金

Swift 4.2 的新特性这两篇文章已经介绍的很清楚了：[WWDC 2018：Swift 更新了什么](https://juejin.cn/post/6844903618714271751#heading-18)，[Swift 4.2 新特性更新](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F86ca289a6e47)。但是 4.2 中实现的 dynamic member lookup 苹果在 WWDC 上却完全没有提到。然而我认为这是一个对未来有着重要影响的特性，所以这里单独介绍一下。

## 语法

这个特性中文可以叫动态查找成员。在使用`@dynamicMemberLookup`标记了对象后（对象、结构体、枚举、protocol），实现了`subscript(dynamicMember member: String)`方法后我们就可以访问到对象不存在的属性。如果访问到的属性不存在，就会调用到实现的 `subscript(dynamicMember member: String)`方法，key 作为 member 传入这个方法。 比如我们声明了一个结构体，没有声明属性。

```swift
@dynamicMemberLookup
struct Person {
    subscript(dynamicMember member: String) -> String {
        let properties = ["nickname": "Zhuo", "city": "Hangzhou"]
        return properties[member, default: "undefined"]
    }
}

//执行以下代码
let p = Person()
print(p.city)
print(p.nickname)
print(p.name)
复制代码
```

如果没有声明`@dynamicMemberLookup`的话，执行的代码肯定会编译失败。很显然作为一门类型安全语言，编译器会告诉你不存在这些属性。但是在声明了`@dynamicMemberLookup`后，虽然没有定义 `city`等属性，但是程序会在运行时动态的查找属性的值，调用`subscript(dynamicMember member: String)`方法来获取值。

[%E7%BB%86%E8%AF%B4%20Swift%204%202%20%E6%96%B0%E7%89%B9%E6%80%A7%EF%BC%9ADynamic%20Member%20Lookup%20-%20%E6%8E%98%E9%87%91%2072415d8e36974ddfb7109fa533b02324/16407b32e72020b4tplv-t2oaga2asx-zoom-in-crop-mark4536000.awebp](%E7%BB%86%E8%AF%B4%20Swift%204%202%20%E6%96%B0%E7%89%B9%E6%80%A7%EF%BC%9ADynamic%20Member%20Lookup%20-%20%E6%8E%98%E9%87%91%2072415d8e36974ddfb7109fa533b02324/16407b32e72020b4tplv-t2oaga2asx-zoom-in-crop-mark4536000.awebp)

### 这样安全吗？

Swift 面世时就大谈自己的安全特性，现在来了这么一个无限制访问的成员万一返回的是`nil`不就闪退了？是的，出于安全的原因，如果实现了这个特性，你就不能返回可选值。必须处理好意料外的情况，一定要有值返回。不像常规的`subscript`方法可以返回可空的值。

### 说好的动态查找，如果两个属性类型不一样怎么破

这个方法可以被重载。和泛型的逻辑类似，会根据你要的返回值而通过类型推断来选择对应的`subscript`方法。

```swift
@dynamicMemberLookup
struct Person {
    subscript(dynamicMember member: String) -> String {
        let properties = ["nickname": "Zhuo", "city": "Hangzhou"]
        return properties[member, default: "undefined"]
    }

    subscript(dynamicMember member: String) -> Int {
        return 18
    }
}
复制代码
```

但是执行的时候就一定要告诉编译器你要获取的属性是什么类型的，否则会编译错误。

```swift
let p = Person()
let age: Int = p.age
print(age)  // 18
复制代码
```

Swift 中函数是一等公民，所以返回函数也是可以的。

```
@dynamicMemberLookup
struct Person {
   subscript(dynamicMember member: String) -> (_ input: String) -> Void {
        return {
            print("Hello! I live at the address \($0).")
        }
    }
}
复制代码
```

### 居然可以继承！

需要注意的是如果声明在类上，那么他的子类也会具有动态查找成员的能力。

```
@dynamicMemberLookup
class User {
    subscript(dynamicMember member: String) -> String {
        return "user"
    }
}

class Developer: User { }

let dev = Developer()
dev.name // "user"
复制代码
```

虽然想起来应该是这样，但是还是很反直觉。因为大多数开发者没想过继承一个类后，会有失去属性拼写检查的副作用。这样可能不小心写错了属性的名字编译器也不会告诉你。

所以声明在类上的时候一定要特别谨慎。

当然如果想害同事，在`BaseViewController`里声明是个好主意。

## 看起来很骚有什么卵用？

这个特性的感觉就是乍一看很厉害的样子，仔细一看好像就这么回事，再冷静想想似乎没有这么简单。

这个东西本质上只是一个语法糖，和数组的`subscript`类似。

```
let numbers = [1, 2]
let firstItem = number[0]
//这个语法最后还是调用到了一个方法，如果没有这种写法，类似 oc 的时候就需要显式的调用一个方法
NSNumber *firstItem = [numnber obbjectAtIndex: 0];
复制代码
```

**原来你需要显式声明字符串参数的地方，可以不用是字符串的形式，可以直接用点语法访问**。官方举的例子是 JSON 的使用。 常规的写法是这样的：

```
json[0]?["name"]?["first"]?.stringValue
复制代码
```

如果像这样定义动态查找成员：

```swift
@dynamicMemberLookup
enum JSON {
   case intValue(Int)
   case stringValue(String)
   case arrayValue(Array<JSON>)
   case dictionaryValue(Dictionary<String, JSON>)

   var stringValue: String? {
      if case .stringValue(let str) = self {
         return str
      }
      return nil
   }

   subscript(index: Int) -> JSON? {
      if case .arrayValue(let arr) = self {
         return index < arr.count ? arr[index] : nil
      }
      return nil
   }

   subscript(key: String) -> JSON? {
      if case .dictionaryValue(let dict) = self {
         return dict[key]
      }
      return nil
   }

   subscript(dynamicMember member: String) -> JSON? {
      if case .dictionaryValue(let dict) = self {
         return dict[member]
      }
      return nil
   }
}
复制代码
```

那么写起来就会是这样：

```swift
json[0]?.name?.first?.stringValue
复制代码
```

## 实现方案

这个功能的实现原理很简单，就是编译器帮助你把点语法转化为下标的语法：

```
  a = someValue.someMember

	//编译器处理后
  a = someValue[dynamicMember: "someMember"]
复制代码
```

动态属性其实并不陌生，回忆一下 OC 里的属性就是动态合成的。声明了`@property`后，编译器帮你生成get、set方法。与之类似，在声明了动态查找成员后，编译器帮你转换成了对应的方法。

## 然而事情并没有这么简单

如果你以为这只是一个语法糖，那你就错了。

### 独有的身世暴露了你

这个 pr 是由已经离开苹果加入谷歌的 swift 创始人 CL 提出的。他不仅提了这个 pr，而且还自己实现了。果然是 swift 是亲儿子，身在曹营还不忘为 swift 添砖加瓦。而且大佬不仅提了这个，还提了一个 [@dynamicCallable](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fapple%2Fswift-evolution%2Fblob%2Fmaster%2Fproposals%2F0216-dynamic-callable.md) 。 当你给一个对象标记`@dynamicCallable`后，可以动态的给传参。

```swift
// 常规操作
a = someValue(keyword1: 42, "foo", keyword2: 19)

// dynamicallyCall
a = someValue.dynamicallyCall(withKeywordArguments: [
    "keyword1": 42, "": "foo", "keyword2": 19
])
复制代码
```

是的，这很 JS。

大佬就是大佬，要啥有啥。目前 `@dynamicCallable`的进度已经在 review 中，也许 5.0 的时候能够上？我猜测 swift 团队想这两个特性都开发完后一起宣布所以这次发布会没有介绍。

## 另有所谋：把 Python 和 JS 纳入怀中

Swift 目前可以”良好“的和 C、OC 交互。然而程序的世界里还有一些重要的动态语言，比如 Python 、 JS，emmm，还有有实力但是不太主流的 Perl、Ruby。如果 swift 能够愉快的的调用 Python 和 JS 的库，那么毫无疑问会极大的拓展的 swift 的边界。

这里需要一点想象力，因为这个设计真正的意义是`@dynamicMemberLookup`、 `@dynamicCallable`组合起来用。通过`@dynamicMemberLookup`动态的返回一个函数，再通过`@dynamicCallable`来调用。**从语法层面来讲，这种姿态下 swift 完完全全是一门动态语言**。

```swift
@dynamicCallable @dynamicMemberLookup
class WeiSuoYuWei {
}

let niuBi = WeiSuoYuWei()
niuBi.someMethod.dynamicallyCall(withKeywordArguments: ["wei_suo_yu_wei": true])
复制代码
```

就像上面的代码展示的，你不必声明过`someMethod`也可以通过动态特性调用到，合法的传参。真的可以为所欲为！

据说谷歌的 TensorFlow For Swift 能够顺利的开发就是依靠了这个特性。CL 是这么说的：

> 
> 
> 
> While this is a syntactic sugar proposal, we believe that this expands Swift to be usable in important new domains
> 

语法糖上的一小步，swift 的一大步！

## reference

[How to use Dynamic Member Lookup in Swift – Hacking with Swift](https://link.juejin.cn/?target=https%3A%2F%2Fwww.hackingwithswift.com%2Farticles%2F55%2Fhow-to-use-dynamic-member-lookup-in-swift)

[SE 195: Introduce User-defined “Dynamic Member Lookup” Types](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fapple%2Fswift-evolution%2Fblob%2Fmaster%2Fproposals%2F0195-dynamic-member-lookup.md%23future-directions-python-code-completion)

- 
    
    微博：[@没故事的卓同学](https://link.juejin.cn/?target=https%3A%2F%2Fweibo.com%2F1926303682)
    
- 
    
    如果想与我有更密切的交流也可以加入我的知识星球：[程序员生存指南](https://link.juejin.cn/?target=https%3A%2F%2Ft.zsxq.com%2FJEiqj27)