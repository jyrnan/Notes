# Frame和Bounds的区别及改变bounds属性产生的影响

# Frame和Bounds的区别

在iOS中，一个view有两个很相似的属性：`frame`和`bounds`。他们都是CGRect类型的结构体，所以他们都可以描述view在坐标系中的原点(origin)位置坐标，view自身的大小(size)，即宽(width)和高(height)。

<aside>
💡 **frame和bounds的区别简单来说就是frame是描述该view在super view坐标系统中的位置和大小，而bounds是描述该view在本地坐标系统中的位置和大小。**

</aside>

因为frame的参考坐标系是super view的坐标系，所以一个view的frame的坐标原点值就是相对于super view的坐标原点的偏移。而大小就是自身view的大小。而bounds由于是相对自身坐标系的，所以一般来说bounds的坐标原点值都是(0, 0)。大小由于和参照系无关，所以与frame中的大小值相同。

<aside>
💡 Frame因为是相对于Super View的位置和大小，所以是不能随便改变的，改变的话就导致自身在Super view的位置或大小的改变。我们可以利用这个特性来实现对view位置的改变，只需改变该View的frame属性，特别是只改变原点位置，就可以移动自己。

</aside>

<aside>
💡 Bounds是相对自身的位置和大小。但是同样也不能改变自己在Super View的位置和大小，所以只能通过改变View自己的原点位置，这样来实现自己相对于自己原点的反向移动了。

</aside>

这个时候，我们就会很好奇，既然bounds和frame这么像，而且bounds原点坐标一般都是(0, 0)，那么要这个属性有什么用呢？如果改变这个值会发生什么情况？

## 实验:

首先先在试图控制器自带的视图上面添加一个view，它的frame是(100, 100, 100, 100)。接着再在该view上面添加一个子view，frame是(0, 0, 50, 50)。

```objectivec
UIView *view = [[UIView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
view.backgroundColor = [UIColor redColor];
[self.view addSubview:view];

UIView *subView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 50, 50)];
subView.backgroundColor = [UIColor grayColor];
[view addSubview:subView];

```

运行一下效果如下：

![https://code.imerc.cc/images/the-influence-of-changing-bounds/bounds0.png](https://code.imerc.cc/images/the-influence-of-changing-bounds/bounds0.png)

打印view的bounds:

```objectivec
NSLog(@"view.bounds --->> %@", NSStringFromCGRect(view.bounds));
//2015-09-02 14:59:01.987 FrameBounds[1772:56852] view.bounds --->> {{0, 0}, {100, 100}}
```

# **修改bounds属性的origin**

现在改变bounds的原点值

```objectivec
view.bounds = CGRectMake(50, 50, 100, 100);
```

再次运行，效果就变成这样了：

![https://code.imerc.cc/images/the-influence-of-changing-bounds/bounds1.png](https://code.imerc.cc/images/the-influence-of-changing-bounds/bounds1.png)

对比之前的图可知，view的位置并没有变，而subView的位置变了，subView跑到了左上角。看起来就好像subView的frame变成了(-50, -50, 50, 50)。

可是如果打印subView的frame

```objectivec
NSLog(@"subView.frame --->> %@", NSStringFromCGRect(subView.frame));
// 2015-09-02 15:15:18.358 FrameBounds[2023:68211] subView.frame --->> {{0, 0}, {50, 50}}
```

证明subView的frame并没有改变。

那为什么subView会跑到view的左上角去呢？原因就在于改变view的bounds的原点坐标为(50, 50)后，view自身左上角的点在view自身坐标系下的坐标就变成了(50, 50)。也就是view自身坐标系的(0, 0)坐标跑到了view的左上方去了。又因为view的自身坐标系就是subView的frame的参考坐标系，而subView的frame的原点坐标是(0, 0)，所以subView就跑到了view的左上角去了。

# **修改bounds属性的size**

既然改变bounds的原点(origin)坐标会影响subView相对于当前view的相对位置，那么如果改变bounds属性的中的大小(size)的值，又会有什么影响呢？

```objectivec
view.bounds = CGRectMake(0, 0, 50, 50);
```

通过改变bounds里面的大小值来改变view的大小，这时运行一下，效果如下：

![https://code.imerc.cc/images/the-influence-of-changing-bounds/bounds2.png](https://code.imerc.cc/images/the-influence-of-changing-bounds/bounds2.png)

subView把view盖住了，这是可以预见的。但是从图中可看到view的位置和之前不太一样，好像往右下角偏移了一点，为了证实我们的猜测，打印view的frame，控制台输出

```
2015-09-02 15:22:25.004 FrameBounds[669:14432] view.frame --->> {{125, 125}, {50, 50}}
```

view的frame的原点坐标果然改变了。改变view的bounds属性的size，竟然会影响view本身的位置。这又是为什么呢？

这就要牵扯到CALayer的position和anchorPoint属性了，这里有篇文章对这两个属性有很详细的介绍——[彻底理解position与anchorPoint](http://www.cnblogs.com/benbenzhu/p/3615516.html?utm_source=tuicool)。至于CALayer和UIView的关系，网上很多，可以自行查找，这里也不详细介绍。

简单来说，position用来描述当前CALayer在父层中的位置，但是position是一个点，那么position是指CALyer上面哪一点相对父层的位置呢？这个点就是anchorPoint。position是相对父坐标系，而anchorPoint是相对自身坐标系。**这两个点实际上是在不同坐标系下的重合点**。anchorPoint的取值范围是[0,1],这个值是由该点在自身坐标系的数值与bounds的大小的比例来确定的，**anchorPoint默认值为(0.5, 0.5)。**

anrchorPoint、position、frame之间有一个变换得公式：（anchor其实就是一个比例值）

```objectivec
position.x = frame.origin.x + anchorPoint.x * bounds.size.width；
position.y = frame.origin.y + anchorPoint.y * bounds.size.height；

```

根据这个公式，本例中的view的位置改变就很好解释了：

在未改变bounds的size之前，position的坐标有上述公式可算得为(150, 150)。这个点其实也是anchorPoint点，在本例中就表现为view的中心点。

<aside>
💡 **当改变bounds的size后，position点和anchorPoint点都不会改变，bounds是以这两个点（它们重合）按比例缩放**

</aside>

这样上述公式可变为

```swift
frame.origin.x = position.x - anchorPoint.x * bounds.size.width；
frame.origin.y = position.y - anchorPoint.y * bounds.size.height；
```

代入position坐标，anchorPoint坐标还有新的bounds的size值后就可得frame的原点坐标变成了(125, 125)。

### **3、总结**

UIView的bounds属性一般不怎么改变，我们用的比较多的是frame属性，但是改变bounds属性还是会对view的位置等参数造成一定的影响。