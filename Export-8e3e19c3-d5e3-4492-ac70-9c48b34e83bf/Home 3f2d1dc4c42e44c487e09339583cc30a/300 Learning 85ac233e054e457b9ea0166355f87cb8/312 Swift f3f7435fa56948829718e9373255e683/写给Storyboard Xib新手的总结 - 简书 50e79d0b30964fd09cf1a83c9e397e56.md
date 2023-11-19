# 写给Storyboard / Xib新手的总结 - 简书

### 写在前面

我不算是个资深码农，有些iOS的编程经验。希望找到一种高效的方式来创作出自己的iOS应用。大家都知道纯代码写应用的成本是很高的，特别是涉及到UI界面的实现，相当耗费时间。之前自己写应用时有了解过Storyboard，也简单使用过，但随着最近深入了解它之后，发现自己低估了它的作用和影响力，因此在这里总结下最近段时间学习到的内容，希望对Storyboard初学者有所帮助。Interface Builder的界面布局如下图：

![%E5%86%99%E7%BB%99Storyboard%20Xib%E6%96%B0%E6%89%8B%E7%9A%84%E6%80%BB%E7%BB%93%20-%20%E7%AE%80%E4%B9%A6%2050e79d0b30964fd09cf1a83c9e397e56/3079374-8bef2d935674e28e.png](%E5%86%99%E7%BB%99Storyboard%20Xib%E6%96%B0%E6%89%8B%E7%9A%84%E6%80%BB%E7%BB%93%20-%20%E7%AE%80%E4%B9%A6%2050e79d0b30964fd09cf1a83c9e397e56/3079374-8bef2d935674e28e.png)

*(图片来自Apple官网）*

### 1.基础概念

在学习Storyboard的使用，有三个概念是最容易混淆的：xib、nib、storyboard。

- xib：是一个可视化文件，可通过拖拽文件进行界面创作和布局。xib实际是个xml文件，xib = XML nib。
- nib：xib编译之后就得到nib文件，nib= NeXT Interface Builder
- storyboard：大家可以理解为是升级版的xib，可以同时管理多个xib文件并处理场景与场景之间的跳转。

### 2.拖放元素

在storyboard/xib上通过拖放对象到界面上就能实现界面元素的创建，与纯代码UI相比可以节省很多调试的时间，**所见即所得是高效的重要方式**。

*(图片来自Apple官网）*
 拖放元素后，可以通过Attributes Inspector上的各种滑块、调色板、大小设定等方式来进行元素属性的配置（注意只支持大部分，而不是所有。没有提供的属性可通过User Defined Runtime Attributes来进行设置）

### 3.连线——IBOutlet和IBAction

在Storyboard/xib上通过拖放对象来完成界面布局后，我们就得到一个静态的界面，那接下来应该怎么把这些界面元素与代码关联起来，秘密就在于“连线”。

把storyboard上的界面对象与代码关联起来，我们俗称为连线。因为在xcode上的操作就是拖出一条线来把它们进行关联。

进行连线有两种方法：

1. 选中某个界面元素，按下Control后拖出线条来进行关联。
2. 选中某个界面元素，切换到Connections Inspector界面，在每项后面的圆圈拖出线条来进行关联。

### IBOutlet

第一种连线是IBOutlet，它的作用是把静态的界面元素与代码进行关联，使得代码中的变量与界面元素对应起来。关联之后就可通过代码对界面元素属性进行操作，如更换背景颜色，增加描边之类的。

*(图片来自Apple官网）*

### IBAction

第二种连线是IBAction，它的作用是把界面元素的事件与代码中的方法关联起来，当用户对元素触发了某个事件之后，让系统执行特定的代码对用户的操作进行响应。例如按钮被按下后，会执行跳转到另外一个界面的代码。

*(图片来自Apple官网）*

### 4.File’s Owner

看到这里，大家可能有个疑问：xcode怎么知道浙西界面要与哪个文件（类）进行联系，以进行接下来的连线操作？答案在于File’s Owner。

File’s Owner描述了xib文件的拥有者是谁（这也是字面的意思），决定了界面元素可以与哪些文件的变量进行IBOutlet，决定了界面的事件可以与哪些文件的方法进行IBAciton。

**设置File’s Owner**：在Xib文件（注意不是storyboard），选中File’s Owner通过Identity Inspector中设置xib的Class来起对指定应的拥有者。

- 基于VC创建的xib，默认已经设置好对应的File Owner，所以一般无需自己设置，如果特殊情况可以指定其他的文件作为其拥有者。
- 而基于View创建的xib，一般要指定对应的File’s Owner。

注意：在storyboard已经没有file’s owner这个概念，但并不妨碍连线。每个VC上有三个按钮，连线时只要拖线到第一个按钮即可进行关联。

### 5.Storyboard VS Xib

storyboard和xib在界面和操作上大体相同，这里聊下它们之间的差别（欢迎进行补充）

- storyboard的文件以.storyboard结尾，xib文件以.xib结尾
- storyboard更注重于多个场景（页面）的层级关系及跳转，而xib更注重于单个页面的布局和复用性，这可以作为选择使用storyboard还是xib的参考。
- storyboard上只能使用VC，不能单独使用UIview，UIView只能基于VC上进行使用，而xib同时支持两者。这一点也印证了上面提到的两者的不同倾向。
- storyboard上没有file’s Owner的概念，默认通过VC的class的指定其拥有者。
- storyboard上可以通过segue实现无需代码的界面跳转，而xib由于管理的是单个界面，因此只能通过代码来实现界面的切换。

### 6.页面切换Segue

Segue的出现，让无需代码进行页面切换成为可能，这是**工作高效的第二个很重要的技能**，只需要简单的设置几下技能实现之前通过代码实现的跳转，相当赞！Segue实际是UIStoryboardSegue对象。

通过segue可以实现：

1. 从一个页面切换到另外一个页面
2. 通过unwind Segue回退到某个页面
3. 实现传参数
4. 可以指定跳转到具体的页面，也可以进行不确定的页面跳转（实际上是通过代码来控制要跳转的目标页面）

### 跳转到一个指定的页面

一般情况下，用户触发了界面上的某个元素进行特定操作后，应用执行界面的跳转。

**具体操作**：在Storyboard的VC上，选中触发跳转的元素，按Control拖线条到目标的VC，之后在出现的Action Segue菜单中选则切换方式，成功设置segue后，在两个VC间就会出现连起来的线。就这么简单！是不是很实用。

*(图片来自Apple官网）*

**注意观**：不同类型的Segue会对应不同的图标，见下图：

### unwind Segue实现页面回退

之前大部分情况下是通过代理来实现页面间的回退，例如要取消一个模态视图，需调用dismissViewControllerAnimation()的方法。现在可以通过Unwind Segue更简单方便的实现这样的回退，**这又是个高效的技能**。
 具体步骤如下：

1. 在回退的**目标页面**上提供回退处理方法。例如从VC2返回到VC1，那么回退的方法应该写在VC1上。
2. 定义回退方法。回退方法的**返回值**必须是IBAciton，并带上UIStoryboardSegue**参数**。例如：

```
@IBAction func close (segue:UIStoryboardSegue)。

```

该方法方法只需要定义，不需要方法体。

1. 关联。在VC2触发返回操作的元素上按control键拖线到**Exit图标**（每个VC上方三个图标中的第三个），在出来的Action Segue菜单中选刚才定义好的回退方法close()。这样就可以实现页面的回退。

### segue传参

有时在页面跳转的时候，需要把参数传到新的页面，使用Segue实现页面跳转时应该怎么实现这一点？

在跳转即将发生时会调用*prepareForSegue:*，因此可以在这个方法上做文章。示例代码：

```
     override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
         if segue.identifier=="goToB"{
         let bVC = segue.destinationViewController
         bVC.testTite = @"hello world"
         }
       }

```

segue对象有个destinationViewController的属性可以获取跳转到的目标VC，这样就可以在跳转之前把当前VC的值赋予目标VC的某个属性，从而实现参数的传递。

顺便提一下，当segue不止一个的时候就需要**给它进行标识**，以便在处理跳转的过程中进行定制化，就像上面的代码，当判断segue的标识为goToB的时候才进行下面的操作。设定标识符的方法很简单，选中segue连线，在attribute inspector中设置identifier就可以了。

### segue配合代码进行不确定的页面跳转

有时会出现这样的情况，我们确定从A页面会跳转到B、C、D页面，但具体跳转到哪个页面是需要通过逻辑触发的。这种情况应该如何用Segue来实现？

1. 在.storyboard文件的导航视图中找到起始VC，右键菜单找到Triggered Segues下的manual后面的+，拖线到可能要跳转到的VC。
2. 给Segue设置标识。
3. 代码实现Segue跳转。在触发的事件响应方法IBAciton中嵌入跳转页面的代码。如点击按钮之后，通过逻辑判断跳转到不同的页面。这时候，就可以在按钮的按下事件的响应方法上嵌入下面的跳转代码。

```
     self.performSegue(withIdentifier: "id", sender: sender)

```

1. 这样就实现了通过segue设置页面的跳转关系，通过代码来控制具体的目标页面，从而实现不确定的页面跳转。

**以下内容与界面适配有关**，发现网络上已经有比我自己总结更好的文章，因此就不在这里搬砖了。大家可以参考外链来学习。

### 7.屏幕大小适配

在xcode8之前，为了在一个Storyboard中适配不同的屏幕大小，需要使用相对抽象的概念——Size Class。但xcode8之后你可以直接选择不同设备查看对应界面布局，甚至进行界面元素的差异化。这样比理解Size Class概念更为人性化。

有兴趣了解Size Class的可参考这篇文章：[初探 iOS8 中的 Size Class](https://link.jianshu.com/?t=http://blog.callmewhy.com/2014/09/12/learn-ios8-size-class/)

### 8.AutoLayout

单靠Size Class无法解决适配问题，真正起作用的是AutoLayout，它为我们处理同一个屏幕尺寸下不同旋转方向上的的界面布局提供简单可行的方案。
 用Apple官方的说法来描述:

> 
> 
> 
> AutoLayout是一种基于约束的，描述性的布局系统。 Auto Layout Is a Constraint-Based, Descriptive Layout System.
> 

AutoLayout的位置是使用相对位置的约束来定义的，简单来说就是通过与B的相对距离来定义A的位置。这样，不管屏幕的旋转情况如何，我只要告诉你这个元素相对于屏幕边缘的距离就可以定义元素的位置。这样比逐个处理屏幕的旋转情况来得**更加高效**。
 具体介绍可参考：[WWDC 2012 Session笔记——AutoLayout（自动布局）入门](https://link.jianshu.com/?t=https://onevcat.com/2012/09/autoayout/)

### 9.来点新玩意

### IBInspectable

@IBInspectable是xcode6之后才出现的，用来修饰属性，使得自己定义的属性可以出现在interface builder的界面上，进行可视化编程。

```
@property(nonatomic,assign) IBInspectable CGFloat cornerRadius;

```

定义后，在storyboard查看对应VC的Attributes Inspector就会神奇的发现多了一个可设置的属性。

### IBDesignable

@IBdesignable也是Xcode6之后才引入的，主要作用是，可以在不运行的情况下，就能把代码效果显示在Storyboard/Xib上。
 不过有几点要注意的：

1. @IBDesignable需写在Class前面。
2. 要想IBDesignable起作用必须把代码写在drawRecr方法中才能显示出来。

更多IBInspectable和IBDesignable的介绍，可戳[这里](https://link.jianshu.com/?t=http://nshipster.cn/ibinspectable-ibdesignable/)。

### 总结

最后再次强调下几个提高产出效率的点：

1. 在storyboard/Xib进行UI界面的布局。
2. 通过Segue来是实现页面间的跳转。
3. 通过AutoLayou来适配不同的屏幕尺寸和旋转方向。

另外贴个  [Xcode的官方介绍文档](https://link.jianshu.com/?t=http://help.apple.com/xcode/mac/8.0/#/) ，可更深入的了解Xcode的使用。

*(如发现内容出现纰漏，望各位大牛指正。如有补充，欢迎留言，谢谢。)*