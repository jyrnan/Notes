# Objective-C 入门教程 | 菜鸟教程

Objective-C 是一种简单的计算机语言，设计为可以支持真正的面向对象编程。

Objective-C 通过提供类定义，方法以及属性的语法，还有其他可以提高类的动态扩展能力的结构等，扩展了标准的 ANSI C 语言。类的语法和设计主要是基于 Smalltalk，最早的面向对象编程语言之一。

如果你以前使用过其他面向对象编程语言，那么下面的信息可以帮助你学习 Objective-C 的基本语法。许多传统的面向对象概念，例如封装，继承以及多态，在 Objective-C 中都有所体现。这里有一些重要的不同，但是这些不同在这文章会表现出来，而且如果你需要还有更多详细的信息存在。

如果你从来没有使用任何编程语言编过程序，那么你至少需要在开始之前，对相关概念进行一些基础的了解。对象的使用和对象对象架构是 iPhone 程序设计的基础，理解他们如何交互对创建你的程序非常重要。想了解面向对象概念的，请参看使用 Objective-C 进行面向对象编程。

## Objective-C：C的超集

Objective-Objective-C是C语言的严格超集－－任何C语言程序不经修改就可以直接通过Objective-C编译器，在Objective-C中使用C语言代码也是完全合法的。Objective-C被描述为盖在C语言上的薄薄一层，因为Objective-C的原意就是在C语言主体上加入面向对象的特性。

### Objective-C代码的文件扩展名

| 扩展名 | 内容类型 |
| --- | --- |
| .h | 头文件。头文件包含类，类型，函数和常数的声明。 |
| .m | 源代码文件。这是典型的源代码文件扩展名，可以包含 Objective-C 和 C 代码。 |
| .mm | 源代码文件。带有这种扩展名的源代码文件，除了可以包含Objective-C和C代码以外还可以包含C++代码。仅在你的Objective-C代码中确实需要使用C++类或者特性的时候才用这种扩展名。 |

当你需要在源代码中包含头文件的时候，你可以使用标准的 #include 编译选项，但是 Objective-C 提供了更好的方法。#import 选项和 #include 选项完全相同，只是它可以确保相同的文件只会被包含一次。Objective-C 的例子和文档都倾向于使用 #import，你的代码也应该是这样的。

### 语法

Objective-C的面向对象语法源于Smalltalk消息传递风格。所有其他非面向对象的语法，包括变量类型，预处理器（preprocessing），流程控制，函数声明与调用皆与C语言完全一致。但有些C语言语法合法代码在objective-c中表达的意思不一定相同，比如某些布尔表达式，在C语言中返回值为true，但在Objective-C若与yes直接相比较，函数将会出错，因为在Objective-C中yes的值只表示为1。

第一个 Objective-C 程序，基于Xcode 4.3.1:

```objectivec
#import <Foundation/Foundation.h>

int main(int argc, char *argv[]) {

    @autoreleasepool {
        NSLog(@"Hello World!");
    }

   return 0;}
```

## 消息传递

Objective-C最大的特色是承自Smalltalk的消息传递模型（message passing），此机制与今日C++式之主流风格差异甚大。Objective-C里，与其说对象互相调用方法，不如说对象之间互相传递消息更为精确。此二种风格的主要差异在于调用方法/消息传递这个动作。C++里类别与方法的关系严格清楚，一个方法必定属于一个类别，而且在编译时（compile time）就已经紧密绑定，不可能调用一个不存在类别里的方法。但在Objective-C，类别与消息的关系比较松散，调用方法视为对对象发送消息，所有方法都被视为对消息的回应。所有消息处理直到运行时（runtime）才会动态决定，并交由类别自行决定如何处理收到的消息。也就是说，一个类别不保证一定会回应收到的消息，如果类别收到了一个无法处理的消息，程序只会抛出异常，不会出错或崩溃。

C++里，送一个消息给对象（或者说调用一个方法）的语法如下：

```cpp
obj.method(argument);
```

Objective-C则写成：

```objectivec
[obj method: argument];
```

此二者并不仅仅是语法上的差异，还有基本行为上的不同。

这里以一个汽车类（car class）的简单例子来解释Objective-C的消息传递特性：

```objectivec
[car fly];
```

典型的C++意义解读是"调用car类别的fly方法"。若car类别里头没有定义fly方法，那编译肯定不会通过。但是Objective-C里，我们应当解读为"发提交一个fly的消息给car对象"，fly是消息，而car是消息的接收者。car收到消息后会决定如何回应这个消息，若car类别内定义有fly方法就运行方法内之代码，若car内不存在fly方法，则程序依旧可以通过编译，运行期则抛出异常。

此二种风格各有优劣。C++强制要求所有的方法都必须有对应的动作，且编译期绑定使得函数调用非常快速。缺点是仅能借由virtual关键字提供有限的动态绑定能力。Objective-C天生即具备鸭子类型之动态绑定能力，因为运行期才处理消息，允许发送未知消息给对象。可以送消息给整个对象集合而不需要一一检查每个对象的类型，也具备消息转送机制。同时空对象nil接受消息后默认为不做事，所以送消息给nil也不用担心程序崩溃。

## 字符串

作为C语言的超集，Objective-C 支持C语言字符串方面的约定。也就是说，单个字符被单引号包括，字符串被双引号包括。然而，大多数Objective-C通常不使用C语言风格的字符串。反之，大多数框架把字符串传递给NSString对象。NSString类提供了字符串的类包装，包含了所有你期望的优点，包括对保存任意长度字符串的内建内存管理机制，支持Unicode，printf风格的格式化工具，等等。因为这种字符串使用的非常频繁，Objective-C提供了一个助记符可以方便地从常量值创建NSString对象。要使用这个助记符，你需要做的全部事情，是在普通的双引号字符串前放置一个@符号，如下面的例子所示：

```objectivec
NSString* myString = @"My String\n";
NSString* anotherString = [NSString stringWithFormat:@"%d %s", 1, @"String"];// 从一个C语言字符串创建Objective-C字符串
NSString*  fromCString = [NSString stringWithCString:"A C string" encoding:NSASCIIStringEncoding];
```

## 类

如同所有其他的面向对象语言，类是 Objective-C 用来封装数据，以及操作数据的行为的基础结构。对象就是类的运行期间实例，它包含了类声明的实例变量自己的内存拷贝，以及类成员的指针。Objective-C 的类规格说明包含了两个部分：定义（interface）与实现（implementation）。定义（interface）部分包含了类声明和实例变量的定义，以及类相关的方法。实现（implementation）部分包含了类方法的实际代码。

下图展现了声明一个叫做 MyClass 的类的语法，这个类继承自 NSObject 基础类。类声明总是由 @interface 编译选项开始，由 @end 编译选项结束。类名之后的（用冒号分隔的）是父类的名字。类的实例（或者成员）变量声明在被大括号包含的代码块中。实例变量块后面就是类声明的方法的列表。每个实例变量和方法声明都以分号结尾。

类的定义文件遵循C语言之惯例以.h为后缀，实现文件以.m为后缀。

### 类声明图

![Objective-C%20%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B%20%E8%8F%9C%E9%B8%9F%E6%95%99%E7%A8%8B%20bc370f2bf71c4c5fac3b440af77f20b3/2011021920525849.jpg](Objective-C%20%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B%20%E8%8F%9C%E9%B8%9F%E6%95%99%E7%A8%8B%20bc370f2bf71c4c5fac3b440af77f20b3/2011021920525849.jpg)

### Interface

定义部分，清楚定义了类的名称、数据成员和方法。 以关键字@interface作为开始，@end作为结束。

```objectivec
@interface MyObject : NSObject 
{
    int memberVar1; // 实体变量
    id  memberVar2;
}
+(return_type) class_method; // 类方法
-(return_type) instance_method1; // 实例方法
-(return_type) instance_method2: (int) p1;
-(return_type) instance_method3: (int) p1 andPar: (int) p2;
@end
```

方法前面的 +/- 号代表函数的类型：

- 加号（+）代表类方法（class method），不需要实例就可以调用，与C++ 的静态函数（static member function）相似。
- 减号（-）即是一般的实例方法（instance method）。

这里提供了一份意义相近的C++语法对照，如下：

```cpp
class MyObject : public NSObject {protected:
    int memberVar1;  // 实体变量
    void * memberVar2;

  public:
    static return_type class_method(); // 類方法

    return_type instance_method1();    // 实例方法
    return_type instance_method2( int p1 );
    return_type instance_method3( int p1, int p2 );}
```

Objective-C定义一个新的方法时，名称内的冒号（:）代表参数传递，不同于C语言以数学函数的括号来传递参数。

Objective-C方法使得参数可以夹杂于名称中间，不必全部附缀于方法名称的尾端，可以提高程序可读性。设定颜色RGB值的方法为例：

```objectivec
- (void) setColorToRed: (float)red Green: (float)green Blue:(float)blue; /* 宣告方法*/
[myColor setColorToRed: 1.0 Green: 0.8 Blue: 0.2]; /* 呼叫方法*/
```

这个方法的签名是setColorToRed:Green:Blue:。每个冒号后面都带着一个float类别的参数，分别代表红，绿，蓝三色。

### Implementation

实现区块则包含了公开方法的实现，以及定义私有（private）变量及方法。 以关键字@implementation作为区块起头，@end结尾。

```objectivec
@implementation MyObject 
{
  int memberVar3; //私有實體變數}
+(return_type) class_method {
    .... //method implementation
}
-(return_type) instance_method1 {
     ....
}
-(return_type) instance_method2: (int) p1 {
    ....
}
-(return_type) instance_method3: (int) p1 andPar: (int) p2 {
    ....
}
@end
```

值得一提的是不只Interface区块可定义实体变量，Implementation区块也可以定义实体变量，两者的差别在于访问权限的不同，Interface区块内的实体变量默认权限为protected，宣告于implementation区块的实体变量则默认为private，故在Implementation区块定义私有成员更匹配面向对象之封装原则，因为如此类别之私有信息就不需曝露于公开interface（.h文件）中。

### 创建对象

Objective-C创建对象需通过alloc以及init两个消息。alloc的作用是分配内存，init则是初始化对象。 init与alloc都是定义在NSObject里的方法，父对象收到这两个信息并做出正确回应后，新对象才创建完毕。以下为范例：

```objectivec
MyObject * my = [[MyObject alloc] init];
```

在Objective-C 2.0里，若创建对象不需要参数，则可直接使用new

```objectivec
MyObject * my = [MyObject new];
```

仅仅是语法上的精简，效果完全相同。

若要自己定义初始化的过程，可以重写init方法，来添加额外的工作。（用途类似C++ 的构造函数constructor）

## 方法

Objective-C 中的类可以声明两种类型的方法：实例方法和类方法。

实例方法就是一个方法，它在类的一个具体实例的范围内执行。也就是说，在你调用一个实例方法前，你必须首先创建类的一个实例。而类方法，比较起来，也就是说，不需要你创建一个实例。

方法声明包括方法类型标识符，返回值类型，一个或多个方法标识关键字，参数类型和名信息。下图展示 insertObject:atIndex: 实例方法的声明。声明由一个减号(-)开始，这表明这是一个实例方法。方法实际的名字(insertObject:atIndex:)是所有方法标识关键的级联，包含了冒号。冒号表明了参数的出现。如果方法没有参数，你可以省略第一个(也是唯一的)方法标识关键字后面的冒号。本例中，这个方法有两个参数。

### 方法声明语法

![Objective-C%20%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B%20%E8%8F%9C%E9%B8%9F%E6%95%99%E7%A8%8B%20bc370f2bf71c4c5fac3b440af77f20b3/2011022010310145.jpg](Objective-C%20%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B%20%E8%8F%9C%E9%B8%9F%E6%95%99%E7%A8%8B%20bc370f2bf71c4c5fac3b440af77f20b3/2011022010310145.jpg)

当你想调用一个方法，你传递消息到对应的对象。这里消息就是方法标识符，以及传递给方法的参数信息。发送给对象的所有消息都会动态分发，这样有利于实现Objective-C类的多态行为。也就是说，如果子类定义了跟父类的具有相同标识符的方法，那么子类首先收到消息，然后可以有选择的把消息转发（也可以不转发）给他的父类。

消息被中括号( [ 和 ] )包括。中括号中间，接收消息的对象在左边，消息（包括消息需要的任何参数）在右边。例如，给myArray变量传递消息insertObject:atIndex:消息，你需要使用如下的语法：

```objectivec
[myArray insertObject:anObj atIndex:0];
```

为了避免声明过多的本地变量保存临时结果，Objective-C允许你使用嵌套消息。每个嵌套消息的返回值可以作为其他消息的参数或者目标。例如，你可以用任何获取这种值的消息来代替前面例子里面的任何变量。所以，如果你有另外一个对象叫做myAppObject拥有方法，可以访问数组对象，以及插入对象到一个数组，你可以把前面的例子写成如下的样子：

```objectivec
[[myAppObject getArray] insertObject:[myAppObject getObjectToInsert] atIndex:0];
```

虽然前面的例子都是传递消息给某个类的实例，但是你也可以传递消息给类本身。当给类发消息，你指定的方法必须被定义为类方法，而不是实例方法。你可以认为类方法跟C++类里面的静态成员有点像（但是不是完全相同的）。

类方法的典型用途是用做创建新的类实例的工厂方法，或者是访问类相关的共享信息的途径。类方法声明的语法跟实例方法的几乎完全一样，只有一点小差别。与实例方法使用减号作为方法类型标识符不同，类方法使用加号( + )。

下面的例子演示了一个类方法如何作为类的工厂方法。在这里，arrayWithCapacity是NSMutableArray类的类方法，为类的新实例分配内容并初始化，然后返回给你。

```objectivec
NSMutableArray*   myArray = nil; // nil 基本上等同于 NULL// 创建一个新的数组，并把它赋值给 myArray 变量
myArray = [NSMutableArray arrayWithCapacity:0];
```

## 属性

**属性是用来代替声明存取方法的便捷方式。**

属性不会在你的类声明中创建一个新的实例变量。他们仅仅是定义方法访问已有的实例变量的速记方式而已。暴露实例变量的类，可以使用属性记号代替getter和setter语法。类还可以使用属性暴露一些“虚拟”的实例变量，他们是部分数据动态计算的结果，而不是确实保存在实例变量内的。

实际上可以说，属性节约了你必须要写的大量多余的代码。因为大多数存取方法都是用类似的方式实现的，属性避免了为类暴露的每个实例变量提供不同的getter和setter的需求。取而代之的是，你用属性声明指定你希望的行为，然后在编译期间合成基于声明的实际的getter和setter方法。

**属性声明应该放在类接口的方法声明那里**。基本的定义使用@property编译选项，紧跟着类型信息和属性的名字。你还可以用定制选项对属性进行配置，这决定了存取方法的行为。下面的例子展示了一些简单的属性声明：

```objectivec
@interface Person : NSObject {
    @public NSString *name;
    @private int age;
}
@property(copy) NSString *name;
@property(readonly) int age;
-(id)initWithAge:(int)age;
@end
```

属性的访问方法由@synthesize关键字来实现，它由属性的声明自动的产生一对访问方法。另外，也可以选择使用@dynamic关键字表明访问方法会由程序员手工提供。

```objectivec
@implementation Person
@synthesize name;
@dynamic age;

-(id)initWithAge:(int)initAge{
    age = initAge; // 注意：直接赋给成员变量，而非属性
    return self;
}

-(int)age{
    return 29; // 注意：并非返回真正的年龄
}
@end
```

### 属性的访问方式

属性可以利用传统的消息表达式、点表达式或"valueForKey:"/"setValue:forKey:"方法对来访问。

```objectivec
Person *aPerson = [[Person alloc] initWithAge: 53];
aPerson.name = @"Steve"; // 注意：点表达式，等于[aPerson setName: @"Steve"];
NSLog(@"Access by message (%@), dot notation(%@), property name(%@) and direct instance variable access (%@)",
      [aPerson name], aPerson.name, [aPerson valueForKey:@"name"], aPerson->name);
```

为了利用点表达式来访问实例的属性，需要使用"self"关键字：

```objectivec
-(void) introduceMyselfWithProperties:(BOOL)useGetter
{
    NSLog(@"Hi, my name is %@.", (useGetter ? self.name : name)); // NOTE: getter vs. ivar access
}
```

类或协议的属性可以被动态的读取。

```objectivec
int i;
int propertyCount = 0;
objc_property_t *propertyList = class_copyPropertyList([aPerson class], &propertyCount);
for ( i=0; i < propertyCount; i++ ) {
    objc_property_t *thisProperty = propertyList + i;
    const char* propertyName = property_getName(*thisProperty);
    NSLog(@"Person has a property: '%s'", propertyName);}
```

## 快速枚举

比起利用NSEnumerator对象或在集合中依次枚举，Objective-C 2.0提供了快速枚举的语法。在Objective-C 2.0中，以下循环的功能是相等的，但性能特性不同。

```objectivec
// 使用NSEnumerator
NSEnumerator *enumerator = [thePeople objectEnumerator];
Person *p;
while ( (p = [enumerator nextObject]) != nil ) {
    NSLog(@"%@ is %i years old.", [p name], [p age]);
}
```

```objectivec
// 使用依次枚举
for ( int i = 0; i < [thePeople count]; i++ ) {
    Person *p = [thePeople objectAtIndex:i];
    NSLog(@"%@ is %i years old.", [p name], [p age]);
}
```

```objectivec
// 使用快速枚举
for (Person *p in thePeople) {
    NSLog(@"%@ is %i years old.", [p name], [p age]);
}
```

快速枚举可以比标准枚举产生更有效的代码，由于枚举所调用的方法被使用NSFastEnumeration协议提供的指针算术运算所代替了。

## 协议（Protocol）

**协议是一组没有实现的方法列表，任何的类均可采纳协议并具体实现这组方法。**

Objective-C在NeXT时期曾经试图引入多重继承的概念，但由于协议的出现而没有实现之。

协议类似于Java与C#语言中的"接口"。在Objective-C中，有两种定义协议的方式：由编译器保证的"正式协议"，以及为特定目的设定的"非正式协议"。

**非正式协议**为一个可以选择性实现的一系列方法列表。非正式协议虽名为协议，但实际上是挂于NSObject上的未实现分类（Unimplemented Category）的一种称谓，Objetive-C语言机制上并没有非正式协议这种东西，OSX 10.6版本之后由于引入@optional关键字，使得正式协议已具备同样的能力，所以非正式协议已经被废弃不再使用。

**正式协议**类似于Java中的"接口"，它是一系列方法的列表，任何类都可以声明自身实现了某个协议。在Objective-C 2.0之前，一个类必须实现它声明匹配的协议中的所有方法，否则编译器会报告错误，表明这个类没有实现它声明匹配的协议中的全部方法。

**bjective-C 2.0版本允许标记协议中某些方法为可选的（Optional），这样编译器就不会强制实现这些可选的方法。**

协议经常应用于Cocoa中的委托及事件触发。例如文本框类通常会包括一个委托（delegate）对象，该对象可以实现一个协议，该协议中可能包含一个实现文字输入的自动完成方法。若这个委托对象实现了这个方法，那么文本框类就会在适当的时候触发自动完成事件，并调用这个方法用于自动完成功能。

Objective-C中协议的概念与Java中接口的概念并不完全相同，即一个类可以在不声明它匹配某个协议的情况下，实现这个协议所包含的方法，也即实质上匹配这个协议，而这种差别对外部代码而言是不可见的。正式协议的声明不提供实现，它只是简单地表明匹配该协议的类实现了该协议的方法，保证调用端可以安全调用方法。

### 语法

协议以关键字@protocol作为区块起始，@end结束，中间为方法列表。

```objectivec
@protocol Locking
- (void)lock;
- (void)unlock;
@end
```

这是一个协议的例子，多线程编程中经常要确保一份共享资源同时只有一个线程可以使用，会在使用前给该资源挂上锁 ，以上即为一个表明有"锁"的概念的协议，协议中有两个方法，只有名称但尚未实现。

下面的SomeClass宣称他采纳了Locking协议：

```objectivec
@interface SomeClass : SomeSuperClass <Locking>
@end
```

一旦SomeClass表明他采纳了Locking协议，SomeClass就有义务实现Locking协议中的两个方法。

```objectivec
@implementation SomeClass
- (void)lock {
  // 實現lock方法...
}
- (void)unlock {
  // 實現unlock方法...
}
@end
```

由于SomeClass已经确实遵从了Locking协议，故调用端可以安全的发送lock或unlock消息给SomeClass实体变量，不需担心他没有办法回应消息。

插件是另一个使用抽象定义的例子，可以在不关心插件的实现的情况下定义其希望的行为。

## 动态类型

类似于Smalltalk，Objective-C具备动态类型：即消息可以发送给任何对象实体，无论该对象实体的公开接口中有没有对应的方法。对比于C++这种静态类型的语言，编译器会挡下对（void*）指针调用方法的行为。但在Objective-C中，你可以对id发送任何消息（id很像void*，但是被严格限制只能使用在对象上），编译器仅会发出"该对象可能无法回应消息"的警告，程序可以通过编译，而实际发生的事则取决于运行期该对象的真正形态，若该对象的确可以回应消息，则依旧运行对应的方法。

一个对象收到消息之后，他有三种处理消息的可能手段，**第一是回应该消息并运行方法，若无法回应，则可以转发消息给其他对象，若以上两者均无，就要处理无法回应而抛出的例外**。只要进行三者之其一，该消息就算完成任务而被丢弃。若对"nil"（空对象指针）发送消息，该消息通常会被忽略，取决于编译器选项可能会抛出例外。

虽然Objective-C具备动态类型的能力，但编译期的静态类型检查依旧可以应用到变量上。以下三种声明在运行时效力是完全相同的，但是三种声明提供了一个比一个更明显的类型信息，附加的类型信息让编译器在编译时可以检查变量类型，并对类型不符的变量提出警告。

下面三个方法，差异仅在于参数的形态：

```objectivec
- setMyValue:(id) foo;
```

id形态表示参数"foo"可以是任何类的实例。

```objectivec
- setMyValue:(id <aProtocol>) foo;
```

id<aProtocol>表示"foo"可以是任何类的实例，但必须采纳"aProtocol"协议。

```objectivec
- setMyValue:(NSNumber*) foo;
```

该声明表示"foo"必须是"NSNumber"的实例。

动态类型是一种强大的特性。在缺少泛型的静态类型语言（如Java 5以前的版本）中实现容器类时，程序员需要写一种针对通用类型对象的容器类，然后在通用类型和实际类型中不停的强制类型转换。无论如何，类型转换会破坏静态类型，例如写入一个"整数"而将其读取为"字符串"会产生运行时错误。这样的问题被泛型解决，但容器类需要其内容对象的类型一致，而对于动态类型语言则完全没有这方面的问题。

## 转发

Objective-C允许对一个对象发送消息，不管它是否能够响应之。除了响应或丢弃消息以外，对象也可以将消息转发到可以响应该消息的对象。转发可以用于简化特定的设计模式，例如观测器模式或代理模式。

Objective-C运行时在Object中定义了一对方法：

### 转发方法：

```objectivec
- (retval_t) forward:(SEL) sel :(arglist_t) args; // with GCC
- (id) forward:(SEL) sel :(marg_list) args; // with NeXT/Apple systems
```

### 响应方法：

```objectivec
- (retval_t) performv:(SEL) sel :(arglist_t) args;  // with GCC
- (id) performv:(SEL) sel :(marg_list) args; // with NeXT/Apple systems
```

希望实现转发的对象只需用新的方法覆盖以上方法来定义其转发行为。无需重写响应方法performv::，由于该方法只是单纯的对响应对象发送消息并传递参数。其中，SEL类型是Objective-C中消息的类型。

以下代码演示了转发的基本概念：

### Forwarder.h 文件代码：

```objectivec
#import <objc/Object.h>
@interface Forwarder : Object{
    id recipient; //该对象是我们希望转发到的对象。
}
@property (assign, nonatomic) id recipient;
@end
```

### Forwarder.m 文件代码：

```objectivec
#import "Forwarder.h"
@implementation Forwarder
@synthesize recipient;
- (retval_t) forward: (SEL) sel : (arglist_t) args {
    /*
     *检查转发对象是否响应该消息。
     *若转发对象不响应该消息，则不会转发，而产生一个错误。
     */
    if([recipient respondsTo:sel])
       return [recipient performv: sel : args];
    else
       return [self error:"Recipient does not respond"];
}
```

### Recipient.h 文件代码：

```objectivec
#import <objc/Object.h>// A simple Recipient object.
@interface Recipient : Object
- (id) hello;
@end
```

### Recipient.m 文件代码：

```objectivec
#import "Recipient.h"
@implementation Recipient
- (id) hello {
    printf("Recipient says hello!\n");

    return self;
}
@end
```

### main.m 文件代码：

```objectivec
#import "Forwarder.h"
#import "Recipient.h"
int main(void){
    Forwarder *forwarder = [Forwarder new];
    Recipient *recipient = [Recipient new];

    forwarder.recipient = recipient; //Set the recipient.
    /*
     *转发者不响应hello消息！该消息将被转发到转发对象。
     *（若转发对象响应该消息）
     */
    [forwarder hello];

    return 0;}
```

利用GCC编译时，编译器报告：

```
$ gcc -x objective-c -Wno-import Forwarder.m Recipient.m main.m -lobjc
main.m: In function `main':
main.m:12: warning: `Forwarder' does not respond to `hello'
$
```

如前文所提到的，编译器报告Forwarder类不响应hello消息。在这种情况下，由于实现了转发，可以忽略这个警告。 运行该程序产生如下输出：

```objectivec
$ ./a.outRecipient says hello!
```

## 类别 (Category)

在Objective-C的设计中，一个主要的考虑即为大型代码框架的维护。结构化编程的经验显示，改进代码的一种主要方法即为将其分解为更小的片段。Objective-C借用并扩展了Smalltalk实现中的"分类"概念，用以帮助达到分解代码的目的。

一个分类可以将方法的实现分解进一系列分离的文件。程序员可以将一组相关的方法放进一个分类，使程序更具可读性。举例来讲，可以在字符串类中增加一个名为"拼写检查"的分类，并将拼写检查的相关代码放进这个分类中。

进一步的，**分类中的方法是在运行时被加入类中的**，这一特性允许程序员向现存的类中增加方法，而无需持有原有的代码，或是重新编译原有的类。例如若系统提供的字符串类的实现中不包含拼写检查的功能，可以增加这样的功能而无需更改原有的字符串类的代码。

在运行时，分类中的方法与类原有的方法并无区别，其代码可以访问包括私有类成员变量在内的所有成员变量。

**若分类声明了与类中原有方法同名的函数，则分类中的方法会被调用**。因此分类不仅可以增加类的方法，也可以代替原有的方法。这个特性可以用于修正原有代码中的错误，更可以从根本上改变程序中原有类的行为。若两个分类中的方法同名，则被调用的方法是不可预测的。

其它语言也尝试了通过不同方法增加这一语言特性。TOM在这方面走的更远，不仅允许增加方法，更允许增加成员变量。也有其它语言使用面向声明的解决方案，其中最值得注意的是Self语言。

C#与Visual Basic.NET语言以扩展函数的与不完全类的方式实现了类似的功能。Ruby与一些动态语言则以"monkey patch"的名字称呼这种技术。

### 使用分类的例子

这个例子创建了Integer类，其本身只定义了integer属性，然后增加了两个分类Arithmetic与Display以扩展类的功能。**虽然分类可以访问类的私有成员，但通常利用属性的访问方法来访问是一种更好的做法，可以使得分类与原有类更加独立**。这是分类的一种典型应用—另外的应用是利用分类来替换原有类中的方法，虽然用分类而不是继承来替换方法不被认为是一种好的做法。

### Integer.h 文件代码：

```objectivec
#import <objc/Object.h>

@interface Integer : Object{
@private int integer;
}

@property (assign, nonatomic) integer;

@end
```

### Integer.m 文件代码：

```objectivec
#import "Integer.h"
@implementation Integer

@synthesize integer;

@end
```

### Arithmetic.h 文件代码：

```objectivec
#import "Integer.h"

@interface Integer(Arithmetic)
- (id) add: (Integer *) addend;
- (id) sub: (Integer *) subtrahend;
@end
```

### Arithmetic.m 文件代码：

```objectivec
#import "Arithmetic.h"

@implementation Integer(Arithmetic)

- (id) add: (Integer *) addend{
    self.integer = self.integer + addend.integer;
    return self;
}

- (id) sub: (Integer *) subtrahend{
    self.integer = self.integer - subtrahend.integer;
    return self;
}
@end
```

### Display.h 文件代码：

```objectivec
#import "Integer.h"

@interface Integer(Display)
- (id) showstars;
- (id) showint;
@end
```

### Display.m 文件代码：

```objectivec
#import "Display.h"
@implementation Integer(Display)
- (id) showstars{
    int i, x = self.integer;
    for(i=0; i < x; i++)
       printf("*");
    printf("\n");

    return self;
}

- (id) showint{
    printf("%d\n", self.integer);

    return self;
}
@end
```

### main.m 文件代码：

```objectivec
#import "Integer.h"
#import "Arithmetic.h"
#import "Display.h"

int main(void){
    Integer *num1 = [Integer new], *num2 = [Integer new];
    int x;

    printf("Enter an integer: ");
    scanf("%d", &x);

    num1.integer = x;
    [num1 showstars];

    printf("Enter an integer: ");
    scanf("%d", &x);

    num2.integer = x;
    [num2 showstars];

    [num1 add:num2];
    [num1 showint];

    return 0;
}
```

利用以下命令来编译：

```bash
gcc -x objective-c main.m Integer.m Arithmetic.m Display.m -lobjc
```

在编译时间，可以利用省略#import "Arithmetic.h" 与[num1 add:num2]命令，以及Arithmetic.m文件来实验。程序仍然可以运行，这表明了允许动态的、按需的加载分类；若不需要某一分类提供的功能，可以简单的不编译之。

## 垃圾收集

Objective-C 2.0提供了一个可选的垃圾收集器。在向后兼容模式中，Objective-C运行时会将引用计数操作，例如"retain"与"release"变为无操作。当垃圾收集启用时，所有的对象都是收集器的工作对象。普通的C指针可以以"__strong"修饰，标记指针指向的对象仍在使用中。被标记为"__weak"的指针不被计入收集器的计数中，并在对象被回收时改写为"nil"。iOS上的Objective-C 2.0实现中不包含垃圾收集器。垃圾收集器运行在一个低优先级的后台线程中，并可以在用户动作时暂停，从而保持良好的用户体验。

> 参考链接：https://zh.wikipedia.org/wiki/Objective-C
>