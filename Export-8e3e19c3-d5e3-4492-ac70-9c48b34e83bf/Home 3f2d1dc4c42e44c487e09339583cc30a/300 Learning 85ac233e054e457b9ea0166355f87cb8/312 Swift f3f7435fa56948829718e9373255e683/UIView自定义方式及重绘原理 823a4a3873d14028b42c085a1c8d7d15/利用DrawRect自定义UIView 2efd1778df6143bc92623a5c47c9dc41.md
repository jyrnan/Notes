# 利用DrawRect自定义UIView

我们可以利用`drawRect`来实现对UIView的一些自定义内容。

这个方法应该是从UIKit层级来实现。

“UIImage、UIBezierPath和NSString都提供了至少一种用于在drawRect:中绘图的方法，这些绘图方法会在drawRect:执行时分别将图像、图形和文本绘制到视图的图层上。”

使用这些方法进行绘图既简单又方便，但是绘制方法内部封装了很多复杂的绘图代码。

```swift
override func draw(_ rect: CGRect) {
//        super.draw(rect)
        
        let bounds = self.bounds
        let center = self.center
        let radius = 120.0
        
        let path = UIBezierPath()
        path.addArc(withCenter: center, radius: radius, startAngle: 0.0, endAngle: .pi * 2, clockwise: true)
        path.lineWidth = 10
        UIColor.lightGray.setStroke()
        path.stroke()
    }
```

> **这里解释了graphic context和draw rect关系， 所有的绘制都必须在这个context上进行**
调用 DrawRect：之前，系统会对这个 view 进行 locks focus。每一个view 都有己的graphics context- 包含了 view 的坐标系统、当前颜色、当前字体，以及剪区域等。当view 被 locks focus 后，它的graphics context 将激活，而当 unlock focus，它的graphics context 将不再是激活状态。任何时候绘制命令都是在当前激活graphics context 上进行的。
> 

---

无论绘制JPEG、PDF还是视图的图层，都是由Core Graphics框架完成的。本章使用的UIBezierPath，其实是将Core Graphics代码封装在一系列方法中，以方便开发者调用，降低了绘图难度

<aside>
💡 所以，在drawrect里面通过调用UIKit层级的绘制命令，实现View的定制，而这些命令又是调用更底层的Core Graphics命令来实现
所以需要深入了解一些👇

</aside>

<aside>
💡 UIBezierPath这类方法必须在`drawRect`这个方面里面才能调用！大概是因为下面所说的创建图形上下文的关系？

</aside>

# 深入学习Core Graphics

Core Hraphics是一套提供2D绘图功能的C语言API，使用C结构和C函数模拟了一套面向对象的编程机制，并没有Objective-C对象和方法。Core Graphics中最重要的“对象”是图形上下文（graphics context），图形上下文是CGContextRef的“对象”，负责存储绘画状态（例如画笔颜色和线条粗细）和绘制内容所处的内存空间。

视图的drawRect:方法在执行之前，系统首先为视图的图层创建一个图形上下文，然后为绘画状态设置一些默认参数。drawRect:方法开始执行时，随着图形上下文不断执行绘图操作，图层上的内容也会随之改变。drawRect:执行完毕后，系统会将图层与其他图层一起组合成完整的图像并显示在屏幕上。

参与绘图操作的类都定义了改变绘画状态和执行绘图操作的方法，这些方法其实调用了对应的Core Graphics函数。例如，向UIColor对象发送setStroke消息时，会调用Core Graphics中的CGContextSetRGBStrokeColor函数改变当前图形上下文中的画笔颜色。

以下两段代码的效果是相同的：

```objectivec
[[UIColor colorWithRed:1.0 green:0.0 blue:1.0 alpha:1.0] setStroke];
UIBezierPath *path = [UIBezierPath bezierPath];
[path moveToPoint:a];
[path addLineToPoint:b];
[path stroke];
```

直接使用Core Graphics函数完成相同的绘图操作：

```c
CGContextSetRGBStrokeColor(currentContext, 1, 0, 0, 1);
CGMutablePathRef path = CGPathCreateMutable();
CGPathMoveToPoint(path, NULL, a.x, a.y);
CGPathAddLineToPoint(path, NULL, b.x, b.y);
CGContextAddPath(currentContext, path);
CGContextStrokePath(currentContext);
CGPathRelease(path);
```