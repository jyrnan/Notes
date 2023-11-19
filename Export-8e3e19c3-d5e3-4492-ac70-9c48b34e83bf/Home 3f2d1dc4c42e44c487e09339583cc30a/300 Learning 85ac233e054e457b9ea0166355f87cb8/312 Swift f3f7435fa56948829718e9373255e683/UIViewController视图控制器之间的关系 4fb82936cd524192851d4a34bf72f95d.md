# UIViewController视图控制器之间的关系

Excerpt From
iOS编程（第4版）([elib.cc](http://elib.cc/))
[美] Christian Keur[美] Aaron Hillegass[美] Joe Conway [elib.cc](http://elib.cc/)[https://itunes.apple.com/WebObjects/MZStore.woa/wa/viewBook?id=0](https://itunes.apple.com/WebObjects/MZStore.woa/wa/viewBook?id=0)

# 父-子关系

当使用视图控制器容器（view controller container）时，就会产生拥有父-子关系的视图控制器。

`UINavigationController`对象、`UITabBarController`对象和`UISplitViewController`对象（第22章会介绍）都是视图控制器容器。这些容器的共性是都有一个类型为数组对象的viewControllers属性，用于保存一组视图控制器。

无论哪种视图控制器容器，都是UIViewController的子类对象，因此也有一个名为view的属性。**视图控制器容器的特性是：容器对象会将viewControllers中的视图作为子视图加入自己的视图。**

处在同一个父-子关系下的视图控制器形成一个族系（family）。一个族系可以有多个级别

![Untitled](UIViewController%E8%A7%86%E5%9B%BE%E6%8E%A7%E5%88%B6%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB%204fb82936cd524192851d4a34bf72f95d/Untitled.png)

任何容器对象都可以通过ViewControllers这个数组来访问其子对象。而子对象又可以通过四个特定的属性来访问其容器对象。

- navigationController
- tabBarController
- splitViewController
- `**parentViewController**`

`parentViewController`的属性比较特别：该属性会指向族系中“最近”的那个容器对象。因此，根据族系的形成方式，parentViewController可能会指向UINavigationController对象、UITabBarController对象或UISplitViewController对象。”

# 显示-被显示关系

视图控制器的另一类关系是显示-被显示关系（presenting-presenter relationship）。当某个视图控制器以模态形式显示另一个视图控制器时，就会产生拥有这种关系的视图控制器。

## 与父子关系的区别

当某个视图控制器（A）以模态形式显示另一个视图控制器（B）时，B的视图会覆盖A的视图。这和之前介绍的父-子关系不同，父-子关系中的子视图只会在容器对象的视图内显示。

此外，任何一个UIViewController对象都可以以模态的形式显示另一个视图控制器。

## presenting和presentedViewController指针

通过UIViewController对象的presentingViewController属性和presentedViewController属性，可以得到指向相应视图控制器的指针。

![Untitled](UIViewController%E8%A7%86%E5%9B%BE%E6%8E%A7%E5%88%B6%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB%204fb82936cd524192851d4a34bf72f95d/Untitled%201.png)

# 显示-被显示关系中的族系

这是最复杂的关系，是上面两种关系的一个集合。

**在显示-被显示关系中，位于关系两头的视图控制器不会处于同一个族系中。**被显示的视图控制器会有自己的族系，这个族系可以只有一个UIViewController对象，也可以由多个UIViewController对象组成。

UIViewController中的presentedViewController和navigationController属性是和族系有关的

![Untitled](UIViewController%E8%A7%86%E5%9B%BE%E6%8E%A7%E5%88%B6%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB%204fb82936cd524192851d4a34bf72f95d/Untitled%202.png)

从上图我们可以做出如下结论：

## 同一族系内

**凡是针对父-子关系的属性，其指向的对象都会在当前族系的范围内。**

向族系2中的视图控制器发送tabBarController消息，并不会返回族系1中的UITabBarController对象，而会返回nil。同理，向族系2中的视图控制器发送navigationController消息，返回的是族系2中的UINavigationController对象，而不是族系1中的。

## 不同族系间-顶部控制器的显示权特性

不同族系中的视图控制器的关系稍微复杂一些。**当应用以模态形式显示某个视图控制器时，负责显示该视图控制器的将是相关族系中的顶部视图控制器（或者可以说“最老”的那个）**。以图17-7中的对象为例，族系2中的视图控制器的presentingViewController属性指向的都是UITabBarController对象。无论应用是向族系1中的哪个视图控制器发送

对某个族系中的所有视图控制器，其presentingViewController属性和presentedViewController属性都是有效的，并且总会指向另一个族系中的顶部对象。

> 所以会出现如果UITabViewController包含一个UINavigationController，通过presenting一个detailViewController（另一个族系）会覆盖前面的UINavigationController。因为实现presenting功能的是最上方的UITabViewController
> 

## 底部控制器获取显示权

通过编写代码，可以改变这种“顶部对象负责以模态形式显示其他视图控制器”的行为（**只能在iPad中使用**），这样就可以限定视图的显示位置。以BNRDetailViewController对象为例，可以要求包含了该对象的UINavigationController对象显示在UITableView对象上，从而避免覆盖UINavigationBar。
　　为此，UIViewController提供了`definesPresentationContext`属性，其默认值是NO。当某个视图控制器的definesPresentationContext是NO时，会将“显示权（presentation off）”传递给父视图控制器，并沿着族系依次向上传递，直到最顶层视图控制器。也就是说，最后会由最顶层视图控制器负责显示新的视图控制器；相反，在传递过程中，如果某个视图控制器的definesPresentationContext是YES，该视图控制器就不会再将“显示权”传递给父视图控制器，而是由自己负责显示新的视图控制器（见图17-8）。”

![Untitled](UIViewController%E8%A7%86%E5%9B%BE%E6%8E%A7%E5%88%B6%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB%204fb82936cd524192851d4a34bf72f95d/Untitled%203.png)