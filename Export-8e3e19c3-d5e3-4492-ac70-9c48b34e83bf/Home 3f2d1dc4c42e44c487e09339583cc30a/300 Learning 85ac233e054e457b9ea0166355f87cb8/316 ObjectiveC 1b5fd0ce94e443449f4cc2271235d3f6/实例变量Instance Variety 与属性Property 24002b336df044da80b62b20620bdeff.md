# 实例变量Instance Variety 与属性Property

<aside>
💡 “声明一个属性，等于隐含地为相应名称的实例变量声明一对存取方法。”

“如果声明了一个名为itemName的属性，编译器会自动生成实例变量_itemName、取方法itemName和存方法setItemName:。（请注意实例变量和存取方法的声明不会出现在文件中，编译器会在编译时自动加入这些代码）”

</aside>

# 区别

**在头文件类名后添加一对花括号就可以添加实例变量**

添加类实例变量后还需要能够存取这些变量对方法。这些方法有命名规范：name 和 setName 

“通过属性，也可以为类声明实例变量并实现相应的存取方法，而且更简便”

![对比图](%E5%AE%9E%E4%BE%8B%E5%8F%98%E9%87%8FInstance%20Variety%20%E4%B8%8E%E5%B1%9E%E6%80%A7Property%2024002b336df044da80b62b20620bdeff/Untitled.png)

对比图

<aside>
💡 “请注意属性的名字是实例变量的名字去掉下画线，编译器根据属性生成实例变量时会自动在变量名前加上下画线”

</aside>

# 理解关键点

- Property是对外，可以从对象外部访问到。可以直接从外部利用”.”来读取
- ivars（instance vars）是内部的，只能从内部访问。对其只能通过内部的方法来进行读取

```objectivec
@interface Circle : NSObject
{
    int innerNumber;
}
@property (nonatomic ) int outNumber;
- (void) changeInnerNumber:(int) n; 
- (void) log;
@end

//实现部分
@implementation Circle
-(void) changeInnerNumber:(int)n {
    innerNumber = n;
}
- (void) log {
    NSLog(@"test %d", innerNumber);
}
//使用方法
Circle *circle = [Circle new];
        int i;
        for (i = 1; i < 10; i++) {
            [circle changeInnerNumber: i + 18];
            [circle log]; //通过log来打印ivars
            
            circle.outNumber = i; //property可以直接赋值
            NSLog(@"circle %@.number = %d", circle.description, circle.outNumber); //直接通过'.'访问
        }
```

# 原文

An instance variable is unique to a class. By default, only the class and subclasses can access it. Therefore, as a fundamental principal of object-oriented programming, instance variables (ivars) are private—they are encapsulated by the class.

By contrast, a property is a public value that may or may not correspond to an instance variable. If you want to make an ivar public, you'd probably make a corresponding property. But at the same time, instance variables that you wish to keep private do not have corresponding properties, and so they cannot be accessed from outside of the class. You can also have a calculated property that does not correspond to an ivar.

Without a property, ivars can be kept hidden. In fact, unless an ivar is declared in a public header it is difficult to even determine that such an ivar exists.

A simple analogy would be a shrink-wrapped book. A property might be the `title`, `author` or hardcover vs. softcover. The "ivars" would be the actual contents of the book. You don't have access to the actual text until you own the book; you don't have access to the ivars unless you own the class.

---

More interestingly, properties are better integrated into the runtime. Modern 64-bit runtimes will generate an ivar for accessor properties, so you don't even need to create the ivar. Properties are in fact methods:

```objectivec
// This is not syntactically correct but gets the meaning across
(self.variable) == ([self variable];)
(self.variable = 5;) == ([self setVariable:5];)

```

For every property, there are two methods (unless the property is declared `readonly`, in which case there is only one): there is the *getter*, which returns the same type as the ivar and is of the same name as the ivar, as well as the *setter* (which is not declared with a `readonly` ivar); it returns void and its name is simply *set* prepended to the variable name.

Because they are methods, you can make dynamic calls on them. Using `[NSSelectorFromString()](http://www.cocoadev.com/index.pl?NSSelectorFromString)` and the various `[performSelector:](http://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Protocols/NSObject_Protocol/Reference/NSObject.html)` methods, you can make a very dynamic program with many possibilities.

Finally, properties are used extensively in Core Data and with [Key-Value Coding](http://cupsofcocoa.wordpress.com/2011/05/09/objective-c-lesson-13-key-value-coding/). Core Data is an advanced framework for storing data in a SQLite database while providing a clear Obj-C front-end; KVC is used throughout Core Data and is a dynamic way to access properties. It is used when encoding/decoding objects, such as when reading from XIBs.

# 属性的特性

“任何属性都有三个特性，每个特性都有多种不同的可选类型。在这些可选类型中，有一种是默认的。”

```objectivec
@property (nonatomic, readwrite, strong) NSString *itemName;
```

- **多线程特性：nonatomic / atomic**
- **读写特性：Read/write**
- **内存管理：strong、weak、copy和unsafe_unretained**

对于不指向任何对象的属性（例如int valueInDollars），不需要做内存管理，这时应该选用unsafe_unretained，它表示存取方法会直接为实例变量赋值。unsafe_unretained中的“unsafe（不安全）”可能会误导读者。该类型的“不安全”是相对于弱引用而言的。**与弱引用不同，unsafe_unretained类型的指针指向的对象被销毁时，指针不会自动设置为nil，而是成为空指针**，因此不安全

关于内存管理参考这篇，特别是采用strong或是weak，有详细说明

[Objective-C内存管理](Objective-C%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%2005e67b9b4b894deaa5c0821c73f37e62.md)

### 是否采用copy：

通常情况下，当某个属性是指向其他对象的指针，而且该对象的类有可修改的子类（例如NSString/NSMutableString或NSArray/NSMutableArray）时，应该将该属性的内存管理特性设置为copy。

这样是避免所指的对象进行了修改，但是在属性上是看不出来的。因为属性是一个指针，指针所指的对象进行了了修改，但是指针本身的内容（所指对象的内存地址）并没有变化。

这样做的原因是，如果属性指向的对象的类有可修改的子类，那么该属性可能会指向可修改的子类对象，同时，该对象可能会被其他拥有者修改。因此，最好先复制该对象，然后再将属性指向复制后的对象。

如果设置了属性的copy特性，那么在其set方法里面并不是直接将传入的对象（例如NSString)的内存地址赋值给该属性，而是向传入的对象发送一个copy消息，这样获得一个和传入对象一样的数据并保存在堆中，然后将这个新对象的地质赋值给属性。

至于所有权，copy方法返回的是拥有强引用特性的指针，而收到copy消息的NSString对象不会发生任何变化：该对象不会获得也不会失去拥有者，其数据也不会发生任何变化。

# 属性合成

如果需要自定义属性的合成方式，可以在实现文件中使用@synthesize指令：

```c
“@implementation Person
　　// 创建存取方法，方法名是age和setAge:，
　　// 同时创建实例变量_age
　　@synthesize age = _age;
　　// 其他方法
　　@end”
```

也可以不写变量名，这样实例变量的变量名会和方法名相同：
　　

```c
@synthesize age;
　　// 和以下语句效果相同：
　　@synthesize age = age;
```