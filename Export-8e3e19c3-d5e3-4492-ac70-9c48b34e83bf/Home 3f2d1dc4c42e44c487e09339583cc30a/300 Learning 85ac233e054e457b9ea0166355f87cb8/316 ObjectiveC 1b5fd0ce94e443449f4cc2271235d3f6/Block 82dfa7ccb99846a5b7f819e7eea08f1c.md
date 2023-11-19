# Block

# Block格式

![Untitled](Block%2082dfa7ccb99846a5b7f819e7eea08f1c/Untitled.png)

# block的使用方式及属性特点

例如，我们在某个类需要通过传入的block来实现这个操作。那在类中先申明了一个block的属性，请注意，actionBlock被声明为`copy`。原因如后

```objectivec
@interface Cell;
@property （nonatomic, copy） void （^actionBlock）（void）；
@end
```

并在某个方法中把相应的block设置给这个属性，以便后续调用。

```objectivec
cell.actionBlock = ^{
　　NSLog（@“Going to show image for %@”, item）；
　　};” //设置相应的block
```

> 为什么前面设置的block属性需要声明为`copy` ：
系统对Block对象和其他对象的内存管理方式不同，Block对象是在栈中创建的，而其他对象是在堆中创建的。这意味着，即使应用针对新创建的Block对象保留了强引用类型的指针，一旦创建该对象的方法返回，那么与方法内部的其他局部变量相同，新创建的Block对象也会被立即释放。为了在声明Block对象的方法返回后仍然保留该对象，必须向其发送copy消息。拷贝某个Block对象时，应用会在堆中创建该对象的备份。这样，即使应用释放了当前方法的栈，堆中的Block对象也不会被释放。
> 

后续我们可以在代码中执行这个block：调用之前需要先检查对象是否存在

```objectivec
if （self.actionBlock） {
　　self.actionBlock（）；
　　}
```

# 捕获变量与循环引用

Block对象可以使用其封闭作用域（enclosing scope）内的所有变量。对声明了某个Block对象的方法，该方法的作用域就是这个Block对象的封闭作用域。因此，这个Block对象可以访问该方法的所有局部变量、传入该方法的实参以及所属对象的实例变量。

如果捕获变量是Objective-C对象，那么Block对象对捕获变量具有强引用。如果捕获变量也对Block对象具有强引用，就会导致强引用循环。例如定义Block的时候，block内的内容会有针对cell的引用，所以block会捕捉周围的变量并持有，其中就包含cell。同时cell因为持有这个block，也对block是强引用，由此产生了循环引用，导致内存无法释放。

![Untitled](Block%2082dfa7ccb99846a5b7f819e7eea08f1c/Untitled%201.png)

解决方法是将block中可能对于cell的引用改成弱引用。

```objectivec
__weak *weakCell = cell; //定义block之前先定义对于cell的弱引用
cell.actionBlock = ^{
BNRCell *strongCell = weakCell; //block内部转换成对这个弱引用进行引用
//针对strongCell的若干操作
}
```

**记住**‼️在Block对象执行过程中，必须保证Block对象始终可以访问cell。因此，以上代码在actionBlock内部创建了strongCell，以保持对cell的强引用。这与Block对象对捕获变量的强引用不同，strongCell只是在Block对象执行过程中对cell保持强引用。这样可以避免了和Cell的互相强引用了。（其实这段不是特别理解）

### 在Block中无意使用到Self

如果在block中直接使用实例变量，那么block会捕捉self。例如

```objectivec
__weak BNREmployee *weakSelf = self;
myBlock = ^{
    BNREmployee *innerSelf = weakSelf; // a block-local strong reference
    NSLog(@"Employee: %@", innerSelf);
    NSLog(@"Employee ID: %d", _employeeID); //这里直接访问了实例变量
};
```

这样直接访问实例变量其实相当让编译器是这样理解：

```objectivec
NSLog(@"Employee ID: %d", self->employeeID); //类似与堆上指针的访问方式
```

解决办法是采用**存取方法**，也就是用点语法的形式。充分利用内部创建的引用：innerSelf

```objectivec
NSLog(@"Employee ID: %d", innerSelf.employeeID); //使用存取方法，不直接访问实例变量
```

### 修改外部变量

Block使用的时候会捕捉外部的变量，但是会捕获成常量拷贝到内部。

如果需要在Block内部修改捕捉到的外部变量，则可以在申明外部变量的时候先加上`__block`关键字

```objectivec
__block int counter = 0;
void (^counterBlock)() = ^{ counter++; };
...
counterBlock(); // Increments counter to 1
counterBlock(); // Increments counter to 2
```