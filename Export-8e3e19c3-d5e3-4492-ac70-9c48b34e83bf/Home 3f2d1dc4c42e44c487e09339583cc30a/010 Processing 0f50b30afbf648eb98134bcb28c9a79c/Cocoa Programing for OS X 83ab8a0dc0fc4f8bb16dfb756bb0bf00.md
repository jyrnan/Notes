# Cocoa Programing for OS X

# 1 Let’s Get Started

## Application Design

### The View Layer

Mac App的View hierarchy和UIKit类似

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled.png)

### The controller layer

控制应用的流程

“Object diagram for RandomPassword”

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%201.png)

### XIB files andNIB files

“XIB” stands for XML Interface Builder.

编译后就变成NIB文件打包到App里面，启动时候NIB文件加载生成View文件

## Showing the Window

# 5. Control 控件

## 关于Mac App视窗层级关系及自定义步骤

### AppDelegate 总是需要有一个Window属性

- 但可以通过创建一个mainWindowController及Xib来间接实现window的出

```swift
class AppDelegate: NSObject, NSApplicationDelegate {
	var mainWindowController: MainWindowController?

  func applicationDidFinishLaunching(_ aNotification: Notification) {
    // Insert code here to initialize your application
    let mainWindowController = MainWindowController()
    mainWindowController.showWindow(self) //这个重要
    self.mainWindowController = mainWindowController
  }
```

### 创建WindowController

- NSWindowController子类mainWindowController
- NSWindowController总是有一个Window的outlet属性，这个outlet连接到Xib文件中的window
- 可以通过如下代码来设置xib文件名，以确保windowController启动时候能有Xib文件可用
    
    ```swift
    override var windowNibName: String? {
            return "MainWindowController"
        }
    ```
    

### 创建Xib文件。

- 对应一个同名的Xib文件，也可以不同名，Xib文件里面包含了Window及视图架构
- Xib文件可以通过设置File’s owner来指定Window Controller
- 空Xib文件可以添加一个Window组件，这个Window总是会有一个默认的content View。这个Window可能就是AppDelegate下的window属性

### 连接Xib文件里面的Window到WindowController里面的window outlet属性

## About Control

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%202.png)

## 控件的本质

NSControl add the ability to send action on message. The `target` is a reference to the  object that receives the message; the `action` is the selector that sent to the `target` of the action message.

`selector`本质上一个表示方法名称的字符串

控件本质就是朝Target发送Action message

### 控件属性面板的层级性

<aside>
💡 以前并没有关注🤔

</aside>

“The top section is labeled Slider, the next section down is
labeled Control, and the section beneath that is labeled
View. These sections reflect the slider’s inheritance
hierarchy.”

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%203.png)

## Setting the Target Programmatically

NSControl includes the property:

```swift
var action: Selector
```

the action property is  a select, so…

```swift
var playButton: NSButton
func play(sender: NSButton) {
...
}
...
let playSelector = Selector("play:")
playButton.action = playSelector
```

“play:”的冒号不能少，因为selector是一个方法的名称，而action方法总是有一个参数“sender”，所以不能缺少这个冒号。

# 6. Delegation

## NSApplication and NSApplicationDelegate

Every Cocoa application has a single instance of NSApplication and a single instance of  a custom class named AppDelegate which conforms to the NSApplicationDelegate protocol.

和Application相关的几个代理方法，但这些方法都不是自己调用，而是应用生命周期自动调用。

## The main event loop

When a Cocoa application is launched, the NSApplication starts and maintains an `event loop` on the main thread.

- events from window server put in the queue
- event loop finds the appropriate method to handle event,
- method call other method
- when all the method return, read next event.

Your code is executed as part of the chain of method calls that event loop kicks off in response to an event.

A big part of learning Cocoa is learning the **right methods to implement - action methods, delegate methods, and notification observers - to get your code executed at the right time.**

# 8. KVC, KVO, and Bindings

Key-value coding is a mechanism. that allows. you to get and set the value of a property indirectly using a `key`, A key is the name of the property as a string.

## Key-value observing

With KVO, an object can sing up to observe changes to the value of a key. When the key’s value changes, the observer is notified.

Binding are an abstraction layer on the top of KVO and KVC which keep your Cocoa views in sync with the model objects they are bound to. 

KVC和KVO是相辅相成的，KVO的实现，必须是要依靠KVC方式的设值。**这都依赖于OC的runtime。**

### KVO本质：

在KVC的`setValue(_: forkey:)`方法前后，分别调用

- willChangeValueForKey()
- didChangeValueForKey()

**这个方法的调用需要改变原有的set方法，所以需要runtime来改变set对应的方法。**

## Swift没有runtime，如何实现？

添加`dynamic` 关键词，让Swift属性具有类似OC的runtime的特性。但这个有利有弊，因为这样就丧失了Swift对于类型的检查，变得不再安全。

> **Dynamic**
> 
> 
> Swift中也有dynamic关键字，它可以用于修饰变量或函数，它的意思也与OC完全不同。它告诉编译器使用动态分发而不是静态分发。OC区别于其他语言的一个特点在于它的动态性，任何方法调用实际上都是消息分发，而Swift则尽可能做到静态分发。
> 
> 因此，标记为dynamic的变量/函数会隐式的加上@objc关键字，它会使用OC的runtime机制。
> 
> 虽然静态分发在效率上可能更好，不过一些app分析统计的库需要依赖动态分发的特性，动态的添加一些统计代码，这一点在Swift的静态分发机制下很难完成。这种情况下，虽然使用dynamic关键字会牺牲因为使用静态分发而获得的一些性能优化，但也依然是值得的。
> 
> 作者：MiniCoder
> 
> 链接：https://www.jianshu.com/p/76760da4b3e9
> 
> 来源：简书
> 
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
> 

## Using Debugger

### Using breakpoints

- debug navigator
- stack trace
    - the method that you have written code for are in black while the methods belonging to apple are in gray.
- Debug area with variables view
    - reference type are shown with their address
    - value types are shown with their value
    
    ![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%204.png)
    
- Debugger bar

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%205.png)

### setting an exception breakpoint

<aside>
💡 这可能是一个非常有用的办法，应该是在抛出错误的地方创建一个断点。还可以选择哪种类型的错误

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%206.png)

</aside>

## LLDB console

### expr

short for expression

### p

### po

The command (print object ) can make this output more succinct.

更多内容参考下面

[LLDB Homepage — The LLDB Debugger](https://lldb.llvm.org/)

# 9 NSArrayController

这章是建立一个document-based application

a subclass of NSDocument is used to control the window or windows used to display a single document. Often, this means that an `NSDocument` act much as a window controller does: An NSDocument has a reference to the data that makes up the document, and it manages shuttling that data to the views in the window it manages.

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%207.png)

<aside>
💡 这里的范例出了错，绑定失败。😮‍💨

</aside>

## XIB文件中快捷选择

shift+ctrl+click

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%208.png)

# 10 Formatter

## Formatters and a control’s objectValue

Formatters are very closely integrated with controls. In fact, NSControl has a formatter property.

Formatter is given the opportunity to validate the input.

## Formatters and location

## Validation with Key-Value Coding

### Adding Key-Value validation to RaiseMan

![Untitled](Cocoa%20Programing%20for%20OS%20X%2083ab8a0dc0fc4f8bb16dfb756bb0bf00/Untitled%209.png)

When you enter a new value into the text field, the bindings system will look for a method named according to the following scheme:
`validateKEY(_:error:).`

```swift
func validateRaise(raiseNumberPointer: AutoreleasingUnsafeMutablePointer<NSNumber?>,
                       error outError: NSErrorPointer) -> Bool
```

The method takes a pointer to an optional NSNumber . It takes pointer parameters because they allow **pass-by-writeback.** this means you can use them to pass information back to the code that call this method.

The first parameters also contains the value that bindings is trying to validate. You access this value using the `memory` property: `raiseNumberPointer.memory`

## NSValueTransformer 转换器

NSValueTransformer used in binding read values from objects,  transform the value before it can be used.  One example is NSNegateBoolean, which transforms true into false, and false into true.