# Task(Unfinished)

[結構化的 Concurrency & Task 的取消 - Swift 新手入門](https://www.youtube.com/watch?v=LiALF5XqB-s&t=22s)

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled.png)

# 结构化写法

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled%201.png)

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled%202.png)

仔细品味二者的区别：在于切到手return时候，程序前者可以判断后面的rice和fish是不会被继续执行了，所以就会自动cancel；而后者因为是直接丢到Task里面去了，所以，就相当于放手不管了，不管后面是不是切到手return，这两个task都会跑完（没有引用，所以不能主动停止。）

<aside>
💡 Task必须被赋值持有才有可能对它进行取消！

</aside>

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled%203.png)

## 范例

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled%204.png)

下面的虽然也是看是结构化，但是并不会自动取消，因为是采用的Task方式来设置任务，这个任务创建了就立刻开始执行了。在后面仅仅是获得这个task的返回值。可以理解成这个task即便这里用不上，别的地方也会用，所以不会自动取消。

而async let则是先创建任务，并没有立即开始执行（也可能开始执行了？），但是log部分是着实等待这个任务完成，没有其他地方需要这个任务（迷😳），所以会自动对它取消。

## 取消任务时间点

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled%205.png)

# Task创建线程继承性

Task 可以繼承優先度、local變數和Actor。

- Global Actor：預設直接繼承。
- Actor：如果有用到local變數才繼承。（默认不继承？）
- 不想繼承的話要使用 Task.detached。

# 利用isolated actor来实现代码连续执行

![Untitled](Task(Unfinished)%20df8d7d71e9f44215bd37dc36036247aa/Untitled%206.png)

- 上面的方法中work和print两个方法虽然都是有prson完成，但是是从外部调用，所以二者中都有await，这期间会发生不可预料的变化，person有可能执行很多其他操作
- 下面的方法中两个步骤因为都确保由person内部执行操作，所以这两条指令都没有await，所以是连续操作，不会出现不可预料变化