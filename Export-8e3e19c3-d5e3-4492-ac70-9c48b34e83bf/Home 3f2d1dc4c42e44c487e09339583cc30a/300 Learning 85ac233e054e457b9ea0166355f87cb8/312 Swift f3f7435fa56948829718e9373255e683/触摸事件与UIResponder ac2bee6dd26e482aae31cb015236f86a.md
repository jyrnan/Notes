# 触摸事件与UIResponder

# 触摸事件

## 处理触摸事件只需要处理以下四种方法：

- 一根手指或多根手指触摸屏幕。
`- （void）touchesBegan:（NSSet *）touches withEvent:（UIEvent *）event;`
- 一根手指或多根手指在屏幕上移动（随着手指的移动，相关的对象会持续发送该消息）。
`- （void）touchesMoved:（NSSet *）touches withEvent:（UIEvent *）event;`
- 一根手指或多根手指离开屏幕。
`- （void）touchesEnded:（NSSet *）touches withEvent:（UIEvent *）event;`
- 在触摸操作正常结束前，某个系统事件（例如有电话进来）打断了触摸过程。
`- （void）touchesCancelled:（NSSet *）touches withEvent:（UIEvent *）event;`

## UITouch对象和事件响应方法的工作机制

- 一个UITouch对象对应屏幕上的一根手指。只要手指没有离开屏幕，相应的UITouch对象就会一直存在。这些UITouch对象都会保存对应的手指在屏幕上的当前位置。
- 在触摸事件的持续过程中，无论发生什么，最初发生触摸事件的那个视图都会在各个阶段收到相应的触摸事件消息。**`即使手指在移动时离开了这个视图的frame区域`**，系统还是会向该视图发送`touchesMoved:withEvent:`和`touchesEnded:withEvent:`消息。也就是说，当某个视图发生触摸事件后，该视图将永远“拥有”当时创建的所有UITouch对象。
- 读者自己编写的代码不需要也不应该保留任何UITouch对象。当某个UITouch对象的状态发生变化时，系统会向指定的对象发送特定的事件消息，并传入发生变化的UITouch对象。

当应用发生某个触摸事件后（例如触摸开始、手指移动、触摸结束），系统都会将该事件添加至一个由`**UIApplication**`单例管理的事件队列。通常情况下，很少会出现满队列的情况，所以UIApplication会立刻分发队列中的事件。分发某个触摸事件时，UIApplication会向“拥有”该事件的视图发送特定的UIResponder消息

# 相应对象链

“UIResponder对象拥有一个名为nextResponder的指针，相关UIResponder对象可以通过该指针组成一个响应对象链（见图12-5）。

![Untitled](%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6%E4%B8%8EUIResponder%20ac2bee6dd26e482aae31cb015236f86a/Untitled.png)

- 当UIView对象属于某个UIViewController对象时，其nextResponder指针就会指向包含该视图的UIViewController对象。
- 当UIView对象不属于任何UIViewController对象时，其nextResponder指针就会指向该视图的父视图。
- UIViewController对象的nextResponder通常会指向其视图的父视图。（这就是为什么UIViewController也是UIResponsder）
- 最顶层的父视图是UIWindow对象，而UIWindow对象的nextResponder指向的是UIApplication单例。

如果UIResponder对象没有处理传给它的事件，会发生什么？该对象会将未处理的消息转发给自己的nextResponder。除了由UIResponder对象向nextResponder转发消息，也可以直接向nextResponder发送消息。例如：

```objectivec
- （void）touchesBegan:（NSSet *）touches withEvent:（UIEvent *）event
　　{
　　UITouch *touch = [touches anyObject];
　　if （touch.tapCount == 2） {
　　[[self nextResponder] touchesBegan:touches withEvent:event];
　　return;
　　}
```

# UIControl

UIControl是部分Cocoa Touch类的父类，例如UIButton和UISlider

### 触发控件事件常量

对于UIControl对象，每个可能触发的控件事件（control event）都有一个对应的常量。例如UIButton对象为例，该对象的常用 控件事件是UIControlEventTouchUpInside。

### 查找动作目标对

控件会针对当前的触摸动作对自己发送一个动作消息。该消息会遍历UIControl对象的所有**目标-动作对**，根据传入的控件事件类型进行查找，然后向匹配的目标对象发送对应的动作消息。

所以UIControl总是有`addTarget:action:`这样的操作

> **UIControl对象绝对不是直接向目标对象发送消息，而是要通过UIApplication转发。**
为什么UIControl对象不能直接向目标对象发送动作消息？这是因为在UIControl对象所拥有的目标-动作对中，目标对象可以是nil。UIApplication在转发源自UIControl对象的消息时，会先判断目标对象是不是nil。如果是nil，UIApplication就会先找出UIWindow对象的第一响应对象，然后向第一响应对象发送相应的动作消息。
> 

# UIGestureRecognizer

## 基本概念

使用UIGestureRecognizer子类对象时，除了要设置目标-动作对，还要将该子类对象“附着”在某个视图上。当该子类对象根据当前附着的视图所发生的触摸事件识别出相应的手势时，就会向指定的目标对象发送指定的动作消息

由UIGestureRecognizer对象发出的动作消息都会遵守以下规范：会传递一个手势对象作为参数。从这个手势对象中我们可以获取相应的位置、状态等信息，作为action方法的输入参数。
　　`- （void）action:（UIGestureRecognizer *）gestureRecognizer;`

## Gesture与Touch事件的关系

UIGestureRecognizer对象在识别手势时，会截取本应由其附着的视图自行处理的触摸事件（见图13-2）。因此，附着了UIGestureRecognizer对象的视图可能不会收到常规的UIResponder消息，例如touchesBegan:withEvent:。

![Untitled](%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6%E4%B8%8EUIResponder%20ac2bee6dd26e482aae31cb015236f86a/Untitled%201.png)

[简单区分UIResponder与UIControl](../316%20ObjectiveC%201b5fd0ce94e443449f4cc2271235d3f6/%E7%AE%80%E5%8D%95%E5%8C%BA%E5%88%86UIResponder%E4%B8%8EUIControl%20eba48eb914df4c808bd975e9b075c5a2.md)

## GestureState

当某个UIGestureRecognizer子类对象触发特定的事件后，其state属性会发生变化。以UILongPressGestureRecognizer对象为例，和上述三种事件相对应的state属性分别为：`UIGestureRecognizerStatePossible`、`UIGestureRecognizerStateBegan`、`UIGestureRecognizerStateEnded`。
　　当某个UIGestureRecognizer子类对象的state属性发生变化时（除了切换至UIGestureRecognizerStatePossible的情况），该对象就会向其目标对象发送指定的动作消息。所以当某个长按手势开始和结束时，相应的UILongPressGestureRecognizer对象都会向其目标对象发送同一个消息。和该消息匹配的方法可以通过UIGestureRecognizer对象的state属性来判断当前的事件类型，然后根据不同的事件类型执行不同的代码。
**具体可以利用switch来依据gestrue的不同state来执行相应的代码。**

## 深入了解UIGestureRecognizer

### UIGestureRecognizer会屏蔽或延迟底层的TouchEvent

当某个UIGestureRecognizer子类对象附着在UIView对象上时，这个子类对象会自动处理和触摸事件有关的UIResponder方法（例如touchesBegan:withEvent:）。UIGestureRecognizer子类对象都是很“贪婪的”，凡是附着了该子类对象的UIView对象，通常都不会再收到和触摸事件有关的消息，或者会延迟收到这类消息。

如果要进一步修改UIGestureRecognizer对象处理触摸事件的逻辑，还可以为其实现特定的**委托方法**

### UIGestureRecognizer的各种状态

掌握UIGestureRecognizer的关键是根据不同的UIGestureRecognizer子类，理解其各种状态的含义。总的来说，UIGestureRecognizer子类对象可以有以下几种状态：
　　·`UIGestureRecognizerStatePossible` ·`UIGestureRecognizerStateFailed`
　　·`UIGestureRecognizerStateBegan` ·`UIGestureRecognizerStateCancelled`
　　·`UIGestureRecognizerStateChanged`· `UIGestureRecognizerStateRecognized`
　　·`UIGestureRecognizerStateEnded`

### 状态切换

- 多数情况下，UIGestureRecognizer子类对象会处于“可能”（`UIGestureRecognizer- StatePossible`）状态。
- 当某个UIGestureRecognizer子类对象识别出特定的手势时，其状态会切换至“开始”（`UIGestureRecognizerStateBegan`）状态。
- 如果某个UIGestureRecognizer子类对象的手势是可持续的（例如拖移），其状态就会切换至“变化后”（`UIGestureRecognizerStateChanged`）状态，而且会保持该状态直到手势结束。在“变化后”状态中，只要手势发生了变化，UIGestureRecognizer子类对象就会向目标对象发送指定的动作消息。
- 当手势结束时，UIGestureRecognizer子类对象会切换至“结束”（`UIGestureRecognizerStateEnded`）状态。”

> 不是所有的UIGestureRecognizer子类对象都会有“开始”“变化后”和“结束”这三种状态。对某些识别“不连续”手势（例如按下）的UIGestureRecognizer子类，就只有一个“已识别”（UIGestureRecognizerStateRecognized）状态（UIGesture- RecognizerStateRecognized
和UIGestureRecognizerStateEnded的值是相同的）”
> 
> 
> “UIGestureRecognizer子类对象是可以“取消的”（例如有电话进来），也可能会失败（例如根据当前的触摸事件不足以识别特定的手势）。”
>