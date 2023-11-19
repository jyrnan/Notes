# Container ViewController

一些ViewController具有Container的属性：例如

- UINavigationController
- UITabbarViewController
- UISplitViewController

# UINavigationController

## UINavigationController主要由两部分构成：

- `navigationBar`：始终显示在ViewController上面，但可以关闭
- `viewControllers`：UINavigationController对象有一个名为`viewControllers`的属性，指向一个负责保存视图控制器的数组对象。在这个数组对象中，根视图控制器是第一个对象

![对象的栈图](Container%20ViewController%20e8d2221bbb154fd9bda176feb7d83ec7/Untitled.png)

对象的栈图

## rootViewController

初始化UINavigationController对象时，需要传入一个UIViewController对象。这个UIViewController对象将成为UINavigationController对象的根视图控制器（`root view controller`），且根视图控制器将永远位于栈底。

创建NavigationController的时候就可以选择某个ViewController直接embed in到这个NVC中，也就是成为这个NVC的rootVC，方法如下

```objectivec
UINavigationController *navController = [[UINavigationController alloc] initWithRootViewController:itemsViewController];
```

## UINavigationController的导航功能

### 将视图控制器压入栈/推出栈

向UINavigationController对象发送`pushViewController:animated:`消息。类似还有`popViewController:animated:` 执行反向操作

通过在栈中的VC，例如rootVC的navigationController属性总能获得UINavigationController对象，从而对其发送消息

<aside>
💡 使用UINavigationController对象时，经常会由当前处于栈顶的视图控制器来负责压入另一个视图控制器，这是常见的使用模式。也就是说，UINavigationController对象的根视图控制器会负责创建并压入下一个视图控制器，而下一个视图控制器会负责创建并压入再下一个视图控制器，依此类推。

</aside>

### 视图的显示和消失

当UINavigationController对象切换视图时，其包含的两个UIViewController对象会分别收到`viewWillDisappear`:消息和`viewWillAppear`:消息。即将出栈的UIViewController对象会收到`viewWillDisappear`:消息，即将入栈的UIViewController对象会收到`viewWillAppear`:消息。

## UINavigationBar

导航栏的空间是由NavigationViewController来通过UINavigationBar提供。但是里面的内容其实是由各个Container内部的VC的`navigationItem`来提供。

![Untitled](Container%20ViewController%20e8d2221bbb154fd9bda176feb7d83ec7/Untitled%201.png)

- UIViewController对象有一个名为`navigationItem`的属性，类型为UINavigationItem。和UINavigationBar不同，UINavigationItem不是UIView的子类，不能在屏幕上显示。
- UINavigationItem对象的作用是为UINavigationBar对象提供绘图所需的内容。当某个UIViewController对象成为UINavigationController的栈顶对象时，UINavigationBar对象就会访问该UIViewController对象的navigationItem，获取和界面显示有关的内容

### navigationItem内容

- UINavigationItem对象除了可以设置标题字符串（title属性）外，还可以设置若干其他的界面属性，如图10-16所示。这些可以自定义的属性包括：`leftBarButtonItem`、`rightBarButtonItem`和`titleView`

![Untitled](Container%20ViewController%20e8d2221bbb154fd9bda176feb7d83ec7/Untitled%202.png)

- UINavigationBar对象包含两种标题显示模式。第一种模式是前面介绍过的：显示一个简单的字符串。第二种模式是在UINavigationBar对象正中显示一个视图。两种模式不能共存。

# UITabbarViewController

## 设置标签项

标签栏上的每一个标签项都可以显示标题和图片，具体数据需要由视图控制器的tabBarItem属性提供。当UITabBarController对象加入一个视图控制器时，就会为标签栏增加一个标签项，并根据新加入的视图控制器的`tabBarItem`属性设置该标签项的内容，包括标题和图片

<aside>
💡 这个视图控制器是指嵌入到UITabbarViewController里面的ViewController

</aside>

# UISplitViewController

这个VC主要是用于iPad，和其他VCContainer一样，它需要传入一组视图控制器，但UISplitViewController对象只能包含两个视图控制器：一个主视图控制器和一个从视图控制器。某个视图控制器是主还是从，是由该对象在数组中的位置决定的。数组中的第一项是主视图控制器，第二项是从视图控制器。

如下面图示结构：

- 第一级SVC有两个视图，分别是master和detail，
- 第二级的master和detail都是NVC，
- 第三级master NVC的rootVC是一个TableVC了，detail NVC的rootVC是一个WebVC
- 可能的第四级：第三级的WebVC也可能会成为第四级，作为TableVC下的detail显示界面

以上的层级结构，如果iPad就同时显示两个NVC，两个NVC分别显示TableVC和WebVC，而如下关系图所示，TableVC来持有WebVC，并根据在TableVC里面选择的栏目来确定WebVC的内容。如果是iPhone就仅显示masterNVC，并在这个NVC里面实现TableVC和WebVC的导航交替显示。

<aside>
💡 为什么要采用NVC做第二级原因：

- 考虑iPad和iPhone的universal程序，所以在appDelegate创建各个VC的时候，会进行设备类型判断，如果是iPhone，SVC就只显示master NVC，这样masterNVC下的TableVC就可以直接将条目利用WebVC来显示，并push到masterNVC里面，就如同iPhone单屏幕下的正常NVC下的内容更替
- detail视图也采用NVC的原因是在iPad竖屏状态下master视图会被隐藏，需要一个UItabBarButton来显示它，所以这个BarItemButton可以用detal的NVC导航栏来显示。要实现这个功能，就还需要把在detail视图里面真正提供显示内容的WebVC遵循`UISplitViewControllerDelegate`协议设置成SVC的delegate。来实现必要的时候显示/隐藏这个button
</aside>

![Untitled](Container%20ViewController%20e8d2221bbb154fd9bda176feb7d83ec7/Untitled%203.png)