# 简单区分UIResponder与UIControl

# 关系：

UIResponder类：上承NSObject，下接UIView ，UIVIewController ，UIApplacation；响应点，压，滑；

UIControl类：上承UIView，下接UIButton等开关按钮；相应操作

# 主要区别在于：

- 前者，主要是响应某个动作，执行某个行为－－

```objectivec
－（[void](https://so.csdn.net/so/search?q=void&spm=1001.2101.3001.7020)）touchesBegan:(NSSet*)touches withEvent:(UIEvent *)event；
```

- 后者，在继承了前者的属性基础上，还能够相应某个动作，为某个对象，添加动作－－

```objectivec
(void) addTarget:(id)target action:(SEL)action forControlEvents(UIControlEvents)controlEvents
```

# **触摸与事件：**

## 在UIView中判断触摸的点是否在UIView中起作用

1、- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event

在UIView中判断触摸的点是否在UIView中起作用。可以实现一些非矩形的触摸事件。

例如：圆形，判断点到圆心的距离。

## 2、在UIWebView上的触摸事件处理方法：

方法（1）继承UIWindow，重写- (void)sendEvent:(UIEvent *)event

方法（2）使用UITapGestureRecognizer

## 3、响应链

触摸事件的响应是：

`view->viewcontroller->superview->superViewcontroller->window->application，`

如果一直没有被处理，就被抛弃掉了。

响应都链排列顺序大致与视图层次结构顺序相反。

## 4、分派事件

使用下面两个方法分派事件给响应者处理：

- (void)sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event;
- (void)sendActionsForControlEvents:(UIControlEvents)controlEvents; // send all actions associated with events

事件与操作的区别：事件报告对屏幕的触摸；操作报告对控件的操纵。

## 5、事件处理

调用[self.nextResponder touchesBegan:touches withEvent:event];把事件传递

参考 http://blog.csdn.net/iefreer/article/details/4754482