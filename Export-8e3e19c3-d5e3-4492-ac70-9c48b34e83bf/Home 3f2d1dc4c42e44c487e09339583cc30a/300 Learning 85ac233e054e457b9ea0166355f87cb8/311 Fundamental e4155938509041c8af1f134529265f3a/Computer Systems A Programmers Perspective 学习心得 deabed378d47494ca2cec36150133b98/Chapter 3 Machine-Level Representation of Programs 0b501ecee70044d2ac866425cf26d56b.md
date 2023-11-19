# Chapter 3 Machine-Level Representation  of Programs

## 3.2 Program Encodings

C程序的编译过程： `linux gcc -Og -O p p1.c p2.c`

- **扩展** #include 和 #define 的内容
- **编译**：编译成汇编语言代码，文件结尾是`p1.s` 和 `p2.s`
- **汇编**：编译成机器代码/二进制代码 binary object-code，文件结尾是`p1.o`和`p2.o`

> object-code是一种机器代码，包含了所有指令的的二进制表示方法，但是这个时候全局变量的地址还没有填入
> 
- **链接**：linker最后把两个文件和其中引入的其它函数的代码，例如`printf`组合成一个执行文件`p`（`-O p`设定的）

### 3.2.1 **Machine-Level Code**

计算机系统存在着抽象，有两点需要记住：

- 第一种是 由指令集体系结构或指令集架构 (Instruction Set Arehiteeture, ISA)来定义机器级程序的 格式和行为，它定义了处理器状态、指令的格式，以及每条指令对状态的影响
- 供机器级程序使用的内存是虚拟地址，内存模型看上去就是一个内存字节的数组。这是由计算机存储硬件和操作系统提供

### **一些通常对 C 语言程序员隐藏的处理器状态都是可见的: 需要记住？**

- Program Conter / 程序计数器，也叫`%rip`，保存了下一条执行指令的内存地址
- integer register / 整数寄存器， 共有16个，每个记录64位值，可以存储地址（对应C的指针）或整数数据。有的寄存器被用来记录某些重要的程序状态，而其 他的寄存器用来保存临时数据，例如过程的参数和局部变量，以及函数的返回值。
- conditon code register / 状态寄存器 保存着最近执行的算术或逻辑指令的状态信息。它们用来实现控制或 数据流中的条件变化，比如说用来实现 if 和 while 语句。
- vector register / 向量计算器 保存一个或多个整数或浮点数值

### **内存**

在目前的实现中， 这些地址的高 16 位必须设置为 o, 所以一个地址实际上能够指定的是 248或 64TB 范围内 的一个字节。较为典型的程序只会访问几兆字节或几千兆字节的数据。操作系统负责管理 虚拟地址空间，将虚拟地址翻译成实际处理器内存中的物理地址。

一条机器指令只执行一个非常基本的操作。例如，将存放在寄存器中的两个数字相加， 在存储器和寄存器之间传送数据，或是条件分支转移到新的指令地址。**编译器必须产生这些 指令的序列，从而实现(像算术表达式求值、循环或过程调用和返回这样的)程序结构。**

### 3.2.2 代码示例：

这是我摘抄的一个范例的过程，更详细可以完整阅读原章节

```c
long mult2(long, long);

void multstore(long x, long y, long *dest) {
    long t = mult2(x, y);
    *dest = t;
}
```

- 将以上代码进行编译 `linux> gee -Og -S mstore.e` 产生一个汇编文件 `mstore.s` 内含如下的**汇编代码**：

```c
multstore:
pushq %rbx
movq %rdx, %rbx call mult2
movq %rax, (%rbx) popq %rbx
ret
```

- 如果用-C命令 `linux> gee -Og -C mstore.c` 产出机器码文件 `mstore.o` 内含如下**机器代码**

`53 48 89 d3 e8 00 00 00 00 48 89 03 5b c3`

- 如果把o文件进行反汇编 `linux> objdump -d mstore.o`  则获得如下内容：可以看出**机器代码对应了汇编代码**

![**机器码对应了右边的汇编码**](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled.png)

**机器码对应了右边的汇编码**

- 生成实际可执行的代码需要对一组目标代码文件运行链接器，而这一组目标代码文件
中必须含有一个 main 函数。把上述代码加入到 main.c 中有下面这样的函数:，

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%201.png)

再用`linux> gee -Og -o prog main.c mstore.c` 来生成，会发现新生成的执行文件中包含来和上面一样的机器代码，但是多了很多其它的代码；并且所有的代码都由**连接器添加了地址**，并且其中调用call指令调用的函数mult2所需要**地址也被填上了**。**链接器的任务之一就是为函数调用找到匹配的函数的可执行代码的位置**

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%202.png)

> 其中一些关千机器代码和它的反汇编表示的特性值得注意:
> 
> - x86-64 的指令长度从 1 到 15 个字节不等。常用的指令以及操作数较少的指令所需的字节数少，而那些不太常用或操作数较多的指令所需字节数较多。
> - 设计指令格式的方式是，从某个给定位置开始 ，可以将字节唯一 地解码成机器指
> 令。例如，只有指令 pushq %rbx 是 以字节值 53 开头的。
> - 反汇编器只是基于机器代码文件中的字节序列来确定汇编代码。它不需要访问该程
> 序的源代码或汇编代码。
> - 反汇编器使用的指令命名规则与 GCC 生成的汇编代码使用的有些细微的差别 。 在
> 我们的示例中，它省略了很多指令结尾的 'q'。这些后缀是大小指示符，在大多数
> 情况中可以省略。相反，反汇编器给 call 和 ret 指令添加了'q’后缀，同样，省略
> 这些后缀也没有问题。

## 3.3 Data Formats  数据格式

### word( Intel data type)

16bit: words

32bit: double words

64bit: quard words.

### float

single-precision: `float` in C

double-precision: `double` in C

### 汇编代码后缀

指令都有一个字符的后缀，表明操作数的大 小。例如，数据传送指令有四个变种: movb(传送字节)、 movw( 传送字)、 movl( 传送双 字)和 movq(传送四字)。后缀 'l' 用来表示双字，因为 32 位数被看成是“长字 (long word)"。注意，汇编代码也使用后缀 'l'来表示 4 字节整数和 8 字节双精度浮点数。这不 会产生歧义，因为浮点数使用的是一组完全不同的指令和寄存辞。

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%203.png)

## 3.4 Accessing Information 访问信息

一个 x86-64 的中央处理单元 (CPU)包含一组 16 个存储 64 位值的通用目的寄存器。 这些寄存器用来存储整数数据和指针 。（随着CPU架构这些存储器数量和名字也逐渐变化，从8到现在16个，位数也由16位增加到64位）

指令可以对这 16 个寄存器的低位字节中存放的不同 大小的数据进行操作。部分或全部利用寄存器到空间。

这些寄存器都用途有些差异，其中最特别的是栈指针 `%rsp,` 用来指明运行时栈的结束位置。

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%204.png)

![Screenshot 2023-06-26 at 17.14.00.png](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Screenshot_2023-06-26_at_17.14.00.png)

[【CSAPP-深入理解计算机系统】3-2.寄存器与数据传送指令_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Lp4y167im/?p=14&spm_id_from=pageDriver)

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%205.png)

## 3.5.3 Shift Operation

移位操作两种类型：

- 左移有两种操作SAL和SHL，效果一样都是填上0
- 右移也有两种
    - SAR： 算数右移位，填上符号位数值（可能是1）
    - SHR：逻辑右移位， 填上0

## 3.6 Control

机器代码提供两种基本的低级机制来实现有条件的 行为:测试数据值，然后根据测试的结果来改变控制流或者数据流。

后者比较常见。

### 3.6.1 Condition Codes 条件码

除了整数寄存器， CPU 还维护着一组单个位的条件码 (condition code) 寄存器，它们
描述了最近的算术或逻辑操作的属性。可以检测这些寄存器来执行条件分支指令。最常用
的条件码有:

- CF: 进位标志。最近的操作使最高位产生了进位。可用来检查无符号操作的溢出 。
- ZF: 零标志。最近的操作得出的结果为 0。
- SF: 符号标志。最近的操作得到的结果为负数。
- OF: 溢出标志。最近的操作导致一个补码溢出 正溢出或负溢出。

`leaq` 指令不改变任何条件码，因为它是用来进行地址计算的。除此之外，图 3-10 中
列出的所有指令都会设置条件码。

还有两类指令(有 8、 16、 32 和 64 位形式)，它们只设置条件码而不改变任 何其他寄存器：`CMP`和`TEST`

### 3.6.3 Jump Instructions 跳转指令

跳转 (jump)指令会导 致执行切换到程序中一个全新的位置。在汇编代码中，这些跳转的目的地通常用一个标号(label)指明。

- `jmp`指令是无条件跳转。 它可以是直接跳转， 即跳转 目标是作为指令的一部分编码的。直接跳转 是给出一个标号作 为跳转目标的，例如标号". Ll”
- 间接跳转，即跳转目标是从寄存器或内存位置中读出的，间接跳转的写法是`*'后面跟一个操作数指示符。这里有两种：
    - `jmp *%rax` 用寄存器 %rax 中的值作为跳转目标
    - `jmp *(%rax)` 以%rax 中的值作为读地址，从内存中读出跳转目标。（多了一道手续：指针的指针？）
    - 
    
    ![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%206.png)
    
- 其它的跳转指令都是有条件的，这些条件码的组合和SET指令的条件码是相匹配的

### 3.6.4 Jump Instruction Encodings 跳转编码

跳转位置的编码方式有两种：相对或绝对

C语言里面的goto就对应了汇编的jump

### 3.6.7 loops 循环

其实C里面的for loop, while , do while等最终都是翻译成汇编的goto语句了

通过一个判断后执行是否跳转到一个loop位置

> for循环先转化成while 循环，while循环先执行一个跳转到后面的判断代码，然后就变成了do-while循环形式
> 

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%207.png)

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%208.png)

### 3.6.8 switch

C的Switch语句是靠维护一个跳转表来实现多个分支的跳转

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%209.png)

## 3.7 Procedures

本章节主要讨论程序/过程直接的调用问题。涉及到三方面（假设P调用Q：

- 传递控制：靠PC计数器来保存新过程的起始内存位置以及返回时的返回内存位置
- 传递数据：P向Q传递数据是参数，Q向P传递的返回值
- 分配和释放内存：

### 3.7.1 The Run-Time Stack 运行时栈

程序可以用栈来管理它的过程所需要的存储空间，栈和程序寄存器存放着传递控制和数据、分配内存所需要的信息。

- x86-64的栈向低地 址方向增长，而栈指针 %rsp 指向栈顶元素。
- 以用 pushq 和 popq 指令将数据存入栈中或是 从栈中取出
- 当前正在执行的过程的帧总是在栈顶。
- 有些过程太短以至于不需要栈帧，数据通过计数器传递（有些可称为叶子过程）

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%2010.png)

### 3.7.2 Control Transfer

需要 告知 Q关于P的返回地址，用`call` Q

可以看到，这种把返回地址压入栈的简单的机制能够让函数在稍后返回到程序中正确

的点 。 C 语言(以及大多数程序语言)标准的调用/ 返回机制刚好 与栈提供的后 进先出的内
存管理方法吻合。

### 3.7.3 Data Transfer 数据传递

x86-64 中， 大部分过程间的数据传送 是通过寄 存器实现的 。 例如，我们已经看到无数的函数示例，参 数在寄存器 %rdi、%rsi ;和其他寄存器中传递 。当 过程 P 调用过程 Q 时， P 的代码必须首先 把参数复制到适当的寄存器中。类似地，当Q返回到P时， P的代码可以访问寄存器%rax 中的返回值。

x86-64 中，可以通过寄存楛最多传递 6 个整型(例如整数和指针)参数。其余超过六个以上的通过Stack来传递。这片区域叫“Argument build area“

参数区域应该是在返回地址之后。（参考上图和下图）

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%2011.png)

### 3.7.4 Local Storage on the Stack

数据有时候必须保存在内存中，例如以下情形：

- 寄存器数量不够
- 用了&符号来产生一个指向内存位置的指针
- 数据是数组和结构体，必须保存在内存空间中

这些需要保存在栈的“Local variables“区域中

### **3.7.5 Local Storage in Registers**

- 保存着寄存器：必须为调用者P保留好
- 调用者保存寄存器： 供Q随意调用读写，P必须自己保存好原有值

## 3.8 Array Allocation and Access

这段没什么好摘抄的

## 3.9 Heterogeneous Data Structures

C语言提供了两种将不同类型的对象组合到一起创建数据类型的机制:结构 (struc­ ture) , 用键字 struct来声明，将多个对象集合到一个单位中;联合 (union), 用关键 字 union 来声明，允许用几种不同的类型来引用一个对象。

<aside>
💡 二者的最大区别是：联合只能其中一种数据可能性，具有排他性
结构的所需总空间是内部所有的变量的总和；联合所需空间是内部某种所需空间最大的类型所需空间值

</aside>

### 3.9.1 联合

利用联合来实现数据的转换

因为联合只能存其中一种值，所以可以利用其中一种类型来实现存储，同时通过另外一种类型来读取。这样该联合的bit位特征是一样的，但是展现出来数据类型不一样了，从而表现的值也不一样。是一种数据转换？例如下面代码，从一个 double产生一个 unsigned long类型的值:

```c
unsigned long double2bits(double d) 
{ 
	union 
	{
		doubled;
		unsigned long u; } temp;
		temp.d = d;
		return temp.u;
};
```

## 3.10 Combining Control and Data in Machine-Level Programs

### 3.10.1 Understanding Pointer

Pinter serve as a universal way to generate references to elenments within different data structures.

- **Every pointer has an associated type.** Pointer types are not a part of machine code, they are an abstraction provieded by C to help programmers avoid addressing error.

```c
int *ip; char  *cpp;
```

- **Every pointer has an value**, the value is a memory adress of some object with designated type.
- **Pointers are created with the “&” operator。**我们已经看到，因为 leaq 指令是设计用来计算内存引用的地 址的，&运算符的机器代码实现常常用这条指令来计算表达式的值。
- **Pointers are dereferenced by “*” operator.**其结果是一个值，它的类型与该指针的类型一致。
- **Arrays and pointers are closely  related.**
    - 数组名可以当作和指针类似的方式使用，但是数组是只读的（‼️）
- **Casting form one type of pointer to another changes the type of pointer but not the value.**
    - 例如，如果 p 是一个 char* 类型 的指针，它的值为 p, 那么表达式 `(int*)p+7` 计算为 p+28, 而 `(int * ) (p+ 7)` 计算为 p+7。(回想一下，**强制类型转换的优先级高于加法**。)
- **Pointers can also point functions.**这提供了一个很强大的存储和向代码传递引用的功能，这些引用可以被程序的某个其他部分调用 。范例值得记忆

```c
Int fun(int x, int *y); // 定义一个函数原型！
Int (*fp)(Int, Int *); // 定义一个指向上一行类型函数的函数指针
fp = fun; // 可以赋值，所以，fun其实也是一个指针指向一个函数原型？

int y = 1;
int fp(3, &y); 
```

函数指针的值是该函数机器代码表示中第一条指令的地址。

<aside>
💡 函数指针声明的语法对程序员新手来说特别难以理解。对于以下声明:
`int (*f)(int);`
要从里(从 "f" 开始)往外读。因此，我们看到像 "(f)" 表明的那样， f 是一个指针; 而 "(f) (int)" 表明 f 是一个指向函数的指针，这个函数以一个 int 作为参数。 最后，我们看到，它是指向以 int 为参数并返回 int 的函数的指针。
f 两边的括号是必需的，否则声明变成
`int f（int);`
它会被解读成
`(int***) f(int**);`
也就是说，它会被解释成一个函数原型，声明了一个函数 f, 它以一个 int *作为参数 并迈回一个 int* 。

</aside>

![Untitled](Chapter%203%20Machine-Level%20Representation%20of%20Programs%200b501ecee70044d2ac866425cf26d56b/Untitled%2012.png)

## 3.11 浮点代码

处理器的浮点体系结构包括多个方面，会影响对浮点数据操作的程序如何被映射到机器上，包括:

- 如何存储和访问浮点数值 。 通常是通过某种寄存器方式来完成。
- 对浮点数据操作的指令 。
- 向函数传递浮点数参数和从函数返回浮点数结果的规则。
- 函数调用过程中保存寄存器的规则 例如，一些寄存器被指定为调用者保存，而其他的被指定为被调用者保存 。

### 3.11.2 过程中的浮点代码

在 x86-64 中， XMM 寄存器用来向函数传递浮点参数，以及从函数返回浮点值 。

- XMM 寄存器 %xmm0~ %xmm7 最多可以传递 8 个浮点参数
- 函数使用寄存器 %xmm0 来返回浮点值。
- 所有的 XMM 寄存器都是调用者保存的

<aside>
💡 当函数包含指针、整数和浮点数混合的参数时，指针和整数通过通用寄存器传递，而 浮点值通过 XMM 寄存器传递

</aside>

## 3.12 小结

我们看到 C 语言中缺乏边界检查，使得许多程序容易出现缓冲区溢出。

C++ 的对象用结构来表示，类似千 C 的 struct。 C++ 的方法是用指向实现方法的代码的指针来表示的。