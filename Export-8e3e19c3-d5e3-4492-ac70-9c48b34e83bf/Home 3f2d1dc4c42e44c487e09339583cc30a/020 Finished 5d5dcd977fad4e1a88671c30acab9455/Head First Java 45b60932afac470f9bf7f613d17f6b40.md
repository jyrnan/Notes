# Head First Java

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%201.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%202.png)

# primitive 主数据类型和引用

## 变量有两种

- primitive 主数据类型
- 引用

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%203.png)

## Java是注重类型的

例如Java里面创建了某种类型的数列，只能存入该类型的东西

# 第四章 方法操作实例变量

## Java是通过值传递的

Java是通过值传递的 也就是说通过拷贝传递

## 要点

- 类定义对象所知及所为。
- 对象所知者是实例变量。
- 对象所为者是方法。
- 方法可依据实例变量来展现不同的行为。
- 方法 可 使 用 参 数 ， 这 代 表 你 可以 传 人一 个 或 多个值给方法。
- 传给方法的參数必须符合声明时的数量up序和类型。
- 传人与传出 方法的值 类型可以 隐含地 放 大或 是明确地縮小。
- 传给方法 的 参 数 值 可 以是 直 接 指 定 的 文 字 或 数字(例如2或 c"等)或者是与所声明 参教相同类型的变量 (还有共他东西可以传 给方法，但我们的进度还不到那边)。
- 方法必须声明返回类型。使用void 类型代表 方法不返回任何东西。
- 如 果 方 法 声 明了非void 的 返 回 类 型 ， 那 就 定要返回与声明类型相同的值。

## 实例变量永远都会有 默认值。

如果你没有 明确的赋值给实例变 量 ，或者没有调用 setter 实例变量还 是会有值

- integers 0
- floating 0.0
- booleans false
- reference null

## 实例变量与局部变量之间的 差别

1. 实例变量是声明在类内而不是方法中
2. 局部变量是声明在方法中的
3. 局部变量在使用前必须初始化。局部变量没有默认值! 如果在变量被初始前就 要使用的话，编译器会 显示错误。
    
    

# 第五章 编写程序

## for循环

```java
for(int cell : locationCells){ }
```

这是加强版循环语法。也可以用类似C风格的语法。

# 第六章 认识Java的API

## ArrayList

### 类似于OC的Array

- 效率比Array低
- 只能放对象，不能直接放Primitive主数据类型，必须通过类型包装起来。（所以，其实里面放的就是指针，这样才能保持数组每个单元的空间长度一致

### 在Java5.0中ArrayList有参数化类型（类似泛型）

```java
ArrayList<String>
```

## 使用函数库（Java API）

要使用API中的类，你必须知道它被放在哪个包中；你必须指明程序代码中所使用到 的类的完整名称

![Screenshot 2023-10-04 at 13.51.20.png](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Screenshot_2023-10-04_at_13.51.20.png)

### 使用方法

- Import：放一个impon述句在程序源文件的最前面:
- Type：或者在程序代码中打出全名。不管在哪里，只要有使 用到就打出全名。

### javax 开头的包代表什么

标准版的扩展都以javax作为包名称的 开头

# 第七章 继承与多态

## 调用方法

当你调用对象引用的方法时，你会调用到与该对象类型最接近的方法。换句话说， 最低阶的会胜出!

## IS-A

还有! IS-A测试适用在继承层次的任何地方

## 四种存取权限

1. public ： 继承
2. private ：不继承
3. protected
4. default

## 多态

运用多态时，引用类型可以是实际对象类型的父类

类型可以设置成父类，而对象是子类，此时调用的方法必须是父类定义的方法，但是方法具体内容则可能是子类已经重写的。（C++的虚函数？）

## 实现类不能被继承的方法

1. 通过存取控制，不公开，这样类只能被同一个包内其他类继承
2. 使用`final`这个修饰符，表示是继承树的末端，不能再被继承
    1. `final`还可以标识在某个方法，表示不能被改写
3. 把构造方法变成`private`

## 方法的重写（override ）

- 参数必须要一样，且返回类型必须要兼容
- 不能降低方法的存取权限

## 方法的重裁 (overload)

重载的意义是两个方法的名称相同，但参数不同。 所以，重载与多态毫无关系

- 返回类型可以不同
- 不能只改变返回类型
- 可以更改存取权限

# 第八章 接口与抽象类（协议？）

到底接口又是什么呢?它是一种100%纯抽象的类。

什么是抽象类?它是无法初始化的类。有些类就不应该被初始化，例如：Animal

- 抽象类不能被new方法初始化
- 设计抽象的类很简单 — — 在 类的声明前面加上抽象类的关键词`abstract`

```java
	abstract class Canine extends Animal {
		public void rome() {}
}
```

# 第九章 构造器与垃圾收集器

## 堆于栈：生存空间

- 堆： 对象的生存空间
- 栈：方法调用及变量的生存空间

### 变量的生存位置

要看具体的类型

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%204.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%205.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%206.png)

## 构造函数

构造函数看起很像方法，感觉上也很像方法，但它并不是方法。 它带有new的时候会执行的程序代码。换句话说，这段程序代码会 在你初始一个对象的时候执行。
唯 一 能 够 调 用 构 造 函 数 的 办 法 就 是 新 建 一 个 类 。( 严 格 说 起 来 ， 这是唯一 在构造函数 之外能够调用构 造函数的 方式，本章稍后会 讨论这个部分)

### 无参数构造函数

如果你已经写了 一个有参数的构造函数， 并且你需要一个没有参数的构造函数，则你必须自己动手写!

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%207.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%208.png)

> 上图中父类的Foo a是一个包装了某个属性的方法，所以，在堆上保存（分配空间）a这个变量的值实际是这个包装方法的指针。**函数（方法）其实也是一个指针！**
> 

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%209.png)

### 如何调用父类的构造函数

调 用 父 类 构 造 函 数 的 唯 一方 法 是 调 用 `super()`

<aside>
💡 编译器会自动帮你加上super（）

```java
//如果你没有写构造函数：
public className() {
	super()
}
//如果写了构造函数但是没有调用super()，编译器会给每一个重载版本的构造函数自动加上，但一定是**无参数版本**

```

</aside>

## 对象的生命周期

### 父类的构造函数必须在子类的构造函数完成之前完成

理解堆栈你会发现Hippo的构造函数是第一个被调用的 (在栈上的第一个) ，也是最后一个完成的

每个子类的 构造函数会立即调用父类的构造函数，如此 一 路 往 上 直 到 Object。 等 到Object完 成 后 会 回去执行Animal的，然后等Animal完成后 又回去执行Hippo剩下的构造函数。这是因 为:

**对super()的 调 用 必 须 是 构 造 函 数 的 第 一 个 语句。**

### 有参数的父类构造函数的情形

<aside>
💡 类似于Designed Init？

</aside>

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2010.png)

### 利用this调用自身的构造函数

<aside>
💡 convinient init?

</aside>

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2011.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2012.png)

### 局部变量的生命周期

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2013.png)

当局部变量活着的时候，它的状态会被保存。只 要 d o S t u f f O )还 在 堆 栈 上， b 变 量 就 会 保 持 它 的 值 。 但 b 变 量 只 能 在 d o S t u f f ()待 在 𣏾 顶 时 才 能 使 用 。 也 就是说局部变量只能在声明它的方法在执 行中才 能被使用

### 引用变量的生命周期

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2014.png)

**“ 变量的生命周期如何影响对象的生命周期?”**

# 第十章 数字与静态

static关键词标注了不需要实例对象的静态方法

静态的方法不能调用非静态的变量

静态的方法也不能调用非静态的方法

## 静态变量初始化

**静态变量会在类的任何静态方法执行之前实现初始化**

## 静态的final变量是常数

常数变量是需要全部大写

## 静态初始化程序 static initializer

静 态 初 始 化 程 序 (s t a t i c
i n i t i a l i z e r ) 是 一 段 在 加 载 类 时 会 执行的程序代码，它会在其他程序 可以使用该类之前款执行，所以很 适合放静态final 变量的起始程序

```java
class Foo {
	final static int x;
	static {
		x = 42;
	}
}
```

## final

- final变量代表不能修改
- final方法代表不能覆盖
- final的类代表不能继承

## Primitive主数据类型的包装

每一个primitive 主数据类型都有个包装用的类， 且因为这些包装类都在java.lang这个包中、所以你无需去import 它们

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2015.png)

<aside>
💡 在新版的5.0后这种界限消失了，可以在对象和Primitive数据直接直接转换🤣 `autoboxing`可以用于
* 方法的参数
* 返回值
* boolean表达式
* 数值运算
* 赋值

</aside>

```java
	ArrayList<Integer> //其实这就是泛型，泛型类型只能是Class或者Interface，所以不能是inter 
```

## 数字的格式化

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2016.png)

### 格式化说明的格式

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2017.png)

### 唯一必填的项目是类型

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2018.png)

### 超过一个以上的参数

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2019.png)

### 日期的格式

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2020.png)

# 第十一章 异常处理

## 处理语句

```java
try {
	//打算执行的语句，如果错误就跳过
} catch (Exception ex) {
 // 捕捉到错误的语句
} finally {
	// 无论如何都会v执行的语句
}
```

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2021.png)

> RuntimeException是不会被编译器检查的
> 

## 执行流程

- 如果try块失败了，抛出异常，流程会马上 转移到catch块。 当catch块完成时，会执行
`finally`块。 当`finally`完成 时 ， 就 会继 续 执 行 其 余的部 分。
- 如果try块成功，流程会跳过c atch块并移动到 `finally` 块 ， 当`finally`完 成 时 ， 就 会 继 续 执 行 其 余的部分。
- 如 果`try` 或 `catch` 块 有return指令 ，`finally`还是会执行，流程会跳到`finally`然后再回到 return指令。

## 处理多重异常

```java
try {
} catch (Exception1 ex1) {
} catch (Exception2 ex2) {
}
```

### 异常也可以是多态

所以可以申明一个异常的父类

也可以按照子类来申明catch处理多个异常，但catch对应的子类范围应该是由小到大

## 异常的进一步抛出 duck （rethrow）

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2022.png)

# 第十二章 图形用户接口

## Windows绘图

J F r a m e 是 个 代 表 屏 幕 上w i n d o w 的 对 象

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2023.png)

### 取得用户事件

通过实现一个接口，并把自己注册到提供事件的对象中，就能获得用户事件调用。

## 创建自己的绘图组件

创 建 J P a n e l 的 子 类 井 覆 盖 掉 p a i n t C o m p o n e n t ()这 个 方 法

### 在PaitComponent()中可以做的事情

- 显示JPG

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2024.png)

### 方法背后的Graphic2D

实际上该方法传入的参数类型是graphic，但其实是一个Graphic2D的对象，有众多方法。

可以通过把g转换成Graphic2D类型，获得更多方法。例如渐变

```java
public void paintComponent(Graphics g){}
```

### 要点

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2025.png)

## 内部类

内部类可以在在外部类内部进行定义

内部类可以使用外部类的所有属性和方法，但必须是作为属于外部类的某个实例变量存在。

### 创建内部类

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2026.png)

### 利用内部类可以存取外部类属性来实现两种监听方法的调用

提供在一个类中实现同 一接口的多次机会

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2027.png)

# 第13章 Swing组件与容器

## Swing的组件（类似于View？）

组件（component）是放在UI上的东西，例如Text Field， Button等

### 组件是可以嵌套的

在Swing中，几乎所有组件都能够安置其他的组件。

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2028.png)

## 布局管理器（Layout Manager）

类似于View互相addSubview，构成层级

### 三大布局管理器

- BoarderLayout： 分五个区
- FlowLayout：依照组件理想大小从左到右从上到下
- BoxLayout：竖排或者横排

<aside>
💡 内容略……

</aside>

### 框架和组件的关系

> JFrame 会这么将珠是因为它是让事物显示在面面上的接点。因为Swing的组件纯粹由Java柏成，**JFrame 必 蘋要连接到底层的操作系统以使来存取显示装工**。我们可 以把面板想做事安置在JFrame 上的100%纯Java 层 。或者 把JFr ame 想做是支撑面板的拒架。你甚至可以用自定义的 JPanel 米换拌柜果的面板:
> 

# 第十四章 序列化和文件的输入输出

对象可以被序列化也可以展开。

**对象有状态和行为两种属性。行为存在于类中，而状态存在于个别的对象中。**

### 如果对象中还有引用

当对象被序列化时，被该对象引用的实例变量也会被序列化。且所有被引用的对象也会被序列化......最棒的是，这些操作都是自动进行的

### 如果要让类能够被序列化，就实现Serializable

Serializable接 又 又被 称 为m a r k e r 或t a g 类的 标 记 用 接 又 ，因 为此 接 又 并 没有任何方法需要实现的。它的唯一目的就是声明有实现它的类是可 以被序列化的。也就是说，此类型的对象可以通过序列化的机制来存储。如果某类是可序列化的，则它的子类也自动地可以序列化 (接又 的本意就是如此)。

### 序列化是全有或全无的

- 如果某实例变量不能或不应该被序列化，就把 它标记 `transient` (瞬时)的
- 如果你把某个对象序列化，`transient`的引用实例变量会以`null`返回

## 序列化

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2029.png)

## 解序列化

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2030.png)

## 要点

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2031.png)

## 把字符串写入文件

`fileWriter.write ("My first String to save")`

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2032.png)

## Java.io.File class (URL?)

File这个类代表磁盘上的文件，但并不是文件中的内容。 啥?你 可以把File 对象想象成文件的路径，而不是文件本身

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2033.png)

## 缓冲区

`BufferedWriter writer = new BufferedWriter (new FileWriter (aFile));`

缓 冲区的奥妙之处在于使用缓冲区比没有使用缓冲区的效率更好。你也可以
BufferedWriter 直接使用FileWriter，调用 它的write()来写文件，但它每次都会直接写下去。
你 应 该 不 会 喜欢 这 种 方 式 额 外 的 成 本 ， 因 为 每 趙 磁 盘 操 作 都 比 内 存 操 作 要 花
费 更 多时 间 。 通 过 B u f f e r e d W r i t e r 和 F i l e W r i t e r 的 链 接 ， B u f f c r e d W r i t e r 可 以 暂 存一堆数据，然后到满的时候再实际 写人磁盘，这样就可以减 少对磁盘操作
的次数。

如果你想要强制缓冲区立即写人，只要调用`writer.flush()`这个方法就可以要求 缓冲区马上把内容写下去。

## 读取文本文件

- 使用`File`对象来表示文件，
- 以`FileReader`来执行实际的读取，并用`BufferedReader`来让读取更有效率。
- 读取是以while循环来逐行进行， 一直到`readLine()`的结果为null为止

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2034.png)

> 输 入 / 输 出的 A P I 使 用 一 种 模 块 化 的 “ 链 接 「 概 念 来让 你 可以 把 连 接 串 流 与链 接 串 流以 各 种 可能 应用到的排列组合连起米。
这 个串流链并不是只能衔接两种层水，你可以连 接多种 链接串流来达成所需的处置。
通 常你只会用到 少数几个类而已， 如果想 要诶写 文本 文件，或许BufferedReader与BufferedWriter (链接到 FileReader 与FileWriter 上)就够用了.
换 言 之 ， 本 书 有 通 盖 刮 九 成 以 上 你 所 会 用 到 的 J a V a 输 人/ 输出。
> 

# 第十五章 网络与线程

## socket链接

```java
Socket chatSocket = new Socket ("196.164.1.103", 5000) ;
```

### 使用BufferedReader 从Socket 上 读取数据

Java的好处就在于大部分的输人/输出工作并不在乎链接串流的上游实际上是什么。也就是说你可以使用`BufferedReader`而不管是串流来自文件或Socket

<aside>
💡 第2步有一个创建inputStream的过程

</aside>

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2035.png)

### 用PrintWriter 写 数 据 到 Socket

需要一个outputStream，和上面的对应，但相反。

```java
PrintWriter writer = new PrintWriter(chatSocket.getOutputStream())；
writer.printIn("message to send");
```

## 多线程

### 建立多线程

```java
Thread thread = new Thread();
thread.start();
```

> 当你看到我们讨论线程时代表的是独立的线程，也就是**独立的执行空间**。
> 

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2036.png)

### 如何启动新线程

```java
Runnable threadJob = new MyRunnable () ; //建立Runnable对象 (线程的任务)
Thread myThread = new Thread (threadJob) ;
myThread.start() ;
```

- Runnable是一个接口，新建的任务类应该满足这个接口，从而就变成可以在线程中执行的方法。Runnable的必须带有`run()`方法

```java
public void run(){}
```

- 对Thread而言， 它是个工人，而runnable就是这个工 人的工作。当 你把 R u n n a b i e 传 给T h r e a d 的 构 造 函 数 时 ， 实际 上就 是 在 给 T h r e a d 取 得 r u n ( )的 办 法 。 这 就 等 于 你 给 了 T h r e a d 一 项 任 务 。

### 线程导致的数据竞争

这 一切 都 来 自 于 可 能 发 生 的 一 种 状 况 : 两 个 或 以 上 的 线 程 存 取单一对象的数据。也就是说两个不同执行空间上的方法都 **在堆上对同一个对象执行getter 或setler。**

> 使用`synchzonized`这个关键词来修饰方法使它每次只能被单一的线 程存取。
> 

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2037.png)

### 容易导致死锁

- 每个对象都有一个锁来保护实例属性的修改访问
- 每个类也有一个锁用来对静态变量进行保护

使用同步化的程序代码要小心，因为没有其他的东西能够像线程 的 死 锁 (d e a d l o c k ) 这 样 伤 害 你 的 程 序 。 死 锁 会 发 生 是 因 为 两 个 线程互相持有对方正在等待的东西。没有方法可以脱离 送个情 况，所以两个线程只好停下来等， 一直等， 一直等，海枯石烂还 在继续等。

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2038.png)

## 要点

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2039.png)

# 第十六章 泛型和集合

## 泛型的格式

如果类本身没有使用类型参数 ，你还是可以通过在一个不寻 常但可行的位置上指定给方法— 在返回类型之前。这个方 法意味着T可以是“任何一种Animal” 。

```java
public <T extends Animal> void takeThing (ArrayList<T> list)
```

更复杂的范例

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2040.png)

extends可以是类或者接口。虽然接口是说implements

## 集合的类型

- List
- Set
- Map
- 

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2041.png)

## 万用字符

```java
public void takeAnimals (ArrayList<? extends Animal> animals) {
	for (Animal animal: animals) {
		animal.eat();
	}
}
```

实际上在使用带有< ?>的声明时，编译器不会让你加入任何东西到集合中!

在方法参数中使用万用字符时，编译器会阻止任何可能破 坏引用参数所指集合的行为: 你能够调用list 中任何元素的方法，但不能加入元素也就是说，你可以擦作集合元素，但不能新增集合元素。 如此才能保障执行期间的安全性、因为编译器会阻止执行 期的恐怖行动

### 相同功能的写法

```java
// 二者等效
public <T extends Animal> void takeThing(ArrayList<T> list)
public void takeThing (ArrayList‹? extends Animal> list )

```

# 第十七章 包，jar存档文件和部署

## 指定编译文件的目录

可以通过添加-d参数来设置编译文件的放置目录

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2042.png)

## 把程序包进JAR

JAR就是Java ARchive。

可执行的I AR代表用户不需把文件抽出来就能运行。程序可以在类文件保存在JAR的情 况下执行。秘诀在于创建出manifest 文件，它会带有JAR的信息，告诉Java虚拟机哪个 类含有main()这个方法!

### 创建可执行的JAR

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2043.png)

### 执行JAR

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2044.png)

## 把类包进包里

- 用包可以防止类名的冲突
- 同时也要防止包名的冲突，利用反向域名来解决

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2045.png)

### 方法

1. 选择包名
2. 在类中加入包指令。这 必 须是 程 序 源 文件 的 第 一 个语 句 ， 比 i m p o r t 语句还要靠上。每个原始文件只能有一个包指 今，因此同 一文件中的类都会在同一个包中。 当然也包括内部。

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2046.png)

1. 设定相对应的目景结构。这个目录结构和反向域名相对应

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2047.png)

## 编译与执行的目录

编译的时候关键是需要加上-d选项。加 上 -d 来编译是很棒的 事情，因为它不仅让你把编译 结果输出到别 的地方，它还可以把类依照包的组织放到正确的目录上。

> d选项会要求编译器将编译结果根据 包的结构来建立目录并输出，如果目录还没有建好，编译器会自动地处理 这些工作。
> 

存放的时候就需要按照反向域名的层级来创建目录，确保按照这个目录层级来找到所有java文件，编译时候会在class目录中创建相应的子目录，并将生成的class文件保存其中

执行的时候写出包含反向域名的包名全称，也会自动到相应的目录层级中查找class文件。

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2048.png)

## 以包创建可执行的JAR

当你把类包进包中，包 自录结构必须在JAR中!你不能只是把类 装到JAR里面，还必须确定目录结构没有多往上走。包的第一层 目 录 ( 通 常 是 c o m ) 必 须 是 J A R 的 第 一 层 目 录 ! 如 果 你 不 小 心 从 士上 面 的 目 录 包 下 来 (例 如 从 c l a s s e s 开 始 包 ) ， J A R 就 无 法 正 确 运 行 。

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2049.png)

## 要点

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2050.png)

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2051.png)

# 第十八章 远程部署的RMI

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2052.png)

Java RMI 提供客户端和服务器端的 辅助设施对象!

# 附录B

## 静态嵌套类

静态嵌套很像一般非嵌套的，他们并未与外层对象产生 特殊关联。但因为还被认为是外层的一个成员， 所以能够存取任何外层的私用成员。然而只限于也是静态的，这是因为静态并没有实例。

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2053.png)

## 匿名的内部类

现场创建一个嵌套类😺

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2054.png)

## 存取权限

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2055.png)

## String和StringBuffer/StringBuilder

Java API中敏常用到的类包括了String和StringBuffer (因为String是不变的，使用StringBuffer/ StringBuilder来操作String比较有效率)。从Java 5.0起，你应该以StringBuilder取代St ringBuffer，除非这 些操作必须在不常见的thread安全环境中进行。下面列出 一些比较重要的方法:

## enum

可以在内部有变量，方法

可以利用if/switch来判断

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2056.png)

### 基本用法：

- 构造方法
- 覆盖内部函数

![Untitled](Head%20First%20Java%2045b60932afac470f9bf7f613d17f6b40/Untitled%2057.png)