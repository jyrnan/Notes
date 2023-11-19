# Main函数和UIApplication的启动关系

App启动的顺序是分成如下几个步骤

# main产生UIApplicationMain

用C语言编写的程序，其执行入口都是`main（）`。用Objective-C语言编写的程序也是这样，”

```objectivec
int main(int argc, char *argv[])
　　{
　　@autoreleasepool {
　　　　return UIApplicationMain(argc, argv, nil,
　　NSStringFromClass([BNRAppDelegate class]));
　　　　}
　　}
```

# UIApplicationMain产生UIApplication和delegate

`UIApplicationMain`函数会创建一个`UIApplication`对象。该对象的作用是维护运行循环。一旦程序创建了某个UIApplication对象，该对象的运行循环就会一直循环下去，**`main（）`的执行也会因此阻塞。**

`UIApplicationMain`函数还会创建某个指定类的对象，并将其设置为UIApplication对象的`delegate`

# application:didFinishLaunchingWithOptions:

在应用启动运行循环并开始接收事件前，`UIApplication`对象会向其`委托`发送一个特定的消息，使应用能有机会完成相应的初始化工作。这个消息的名称是`application:didFinishLaunchingWithOptions`:。”

该方法中创建了UIWindow对象和多个视图控制对象

# 接下来App的runloop会一直循环♻️