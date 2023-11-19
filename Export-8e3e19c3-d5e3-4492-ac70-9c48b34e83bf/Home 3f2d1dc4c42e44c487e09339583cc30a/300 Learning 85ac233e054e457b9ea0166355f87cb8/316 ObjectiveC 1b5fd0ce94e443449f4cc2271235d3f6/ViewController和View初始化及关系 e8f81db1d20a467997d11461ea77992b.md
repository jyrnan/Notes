# ViewController和View初始化及关系

# 基本概念

## 视觉文件 xib/nib和storyboard

在学习Storyboard的使用，有三个概念是最容易混淆的：xib、nib、storyboard。

- xib/nib：是一个可视化文件，可通过拖拽文件进行界面创作和布局。xib实际是个xml文件，xib = XML nib。 nib：xib编译之后就得到nib文件，nib= NeXT Interface Builder
- storyboard：大家可以理解为是升级版的xib，可以同时管理**多个xib**文件并处理场景与场景之间的跳转。

## VC（ViewController）和View

ViewController和View其实是相配套的，每个VC都会对应一个rootView，也就是VC下view属性所指。所以Apple提出其实利用视觉文件来生成VC/View这样一套。

每个VC生成后会利用`-(void)loadView` 来生成它的View

## 视觉文件和VC/View的关系

每个视觉文件（Nib）其实都有一个FileOwner，这个FileOwner其实就对应了VC，而视觉文件本身就默认用来生成一个View作为VC的rootView。但也不完全如此，因为可以通过VC的`-(void)loadView`  来自定义rootView

File's Owner对象是一个占位符对象（placeholder），它是XIB文件特意留下的一个“空洞”。当某个视图控制器将XIB文件加载为NIB文件时，首先会创建XIB文件中的所有视图对象，然后会将自己填入相应的File's Owner空洞，并建立之前在Interface Builder中设置的关联。

NIB文件用来生成VC/View这一套，其实包含两块：如下图可以做关系以及动作说明：

- NIB的fileOwner指定成VC的实例，——》**关联VC到fileOwer**
- NIB的视觉元素来生成View实例 ——》**关联View到视图控制器的view属性”**
- VC通过调用自己的loadView来默认指定生成的View是rootView，那么NIB里面的子元件（也出现在生成的View里面）自然就可以和VC的实例建立其联系，——》**关联IBOutlet和IBAction 到视图的其他组件**

<aside>
💡 再补充一点：storyboard是多个nib文件的集合，所以可以生成多套VC/View的组合。和单个nib文件的区别就是storyboard可以定义nib之间的segue关系，也就是各个VC/View之间的segue关系。

</aside>

![Untitled](ViewController%E5%92%8CView%E5%88%9D%E5%A7%8B%E5%8C%96%E5%8F%8A%E5%85%B3%E7%B3%BB%20e8f81db1d20a467997d11461ea77992b/Untitled.png)

# 加载ViewController及相关View的过程

## 生成ViewController

基础方法 ****`init(nibName:bundle:)`****

指定初始化的方法就是通过nib来生成VC，所以我们可以通过nib来生成单个VC，或storyboard的某个nib来生成VC

### 采用StoryBoard的形式来成ViewController

需要指定storyboard里面的哪个nib，用identifier来区别

```swift
func scene(_ scene: UIScene,
    willConnectTo session: UISceneSession,
    options connectionOptions: UIScene.ConnectionOptions) {
        if let windowScene = scene as? UIWindowScene {
            self.window = UIWindow(windowScene: windowScene)
            let userHasLoggedIn : Bool = // ...
            let vc = UIStoryboard(name: "Main", bundle: nil)
                .instantiateViewController(identifier: userHasLoggedIn ?
                    "UserHasLoggedIn" : "LoginScreen") // *
            self.window!.rootViewController = vc
            self.window!.makeKeyAndVisible()
        }
}
```

### 用xib（nib）文件来成ViewController

如果不采用Storyboard来产生ViewController，可以用xib文件。**这是比较老的做法**：默认采用同名的xib文件，会自动调用ViewController的initWithNibName来生成VC

```objectivec
- (void)scene:(UIScene *)scene willConnectToSession:(UISceneSession *)session options:(UISceneConnectionOptions *)connectionOptions {
    
    self.window = [[UIWindow alloc] initWithWindowScene:scene];
    MyViewController *VC = [[MyViewController alloc] init]; //需要同名的xib文件，会自动调用ViewController的initWithNibName来生成VC
    [VC.view setBackgroundColor:[UIColor blueColor]];
    self.window.rootViewController = VC;
    [self.window makeKeyAndVisible];
}
```

## ViewController指定View

VC生成后需要指定一个View作为它的rootView。通过ViewController的 `-(void)loadView` 方法可以修改/指定默认的view。

- 默认的view应该是nib文件对应生成的一个view。当视图控制器从NIB文件中创建视图层次结构时，不需要覆盖`loadView`方法，默认的loadView方法会自动处理NIB文件中包含的视图层次结构。
- 如果不用默认的view，可以自行生成。生成view的指定初始化方法：****`init(frame:)`****

例如：调用sending the message `loadNibNamed:owner:options**:**` to the `application bundle`.

这个方法是利用指定的nib文件来来生成一个view，并把VC自己设置成这个nib的fileOwner，然后顺便把nib生成的这个view作为VC的view（通过IBOutlet链接）。如果没有nib文件，那就新建一个view来作为VC的view

```objectivec
- (void)loadView
{
    // Which bundle is the NIB in?
    // Was a bundle passed to initWithNibName:bundle:?
    NSBundle *bundle = [self nibBundle];
    if (!bundle) {
        // Use the default
        bundle = [NSBundle mainBundle];
    }
    // What is the NIB named?
    // Was a name passed to initWithNibName:bundle:?
    NSString *nibName = [self nibName];
    if (!nibName) {
        // Use the default
        nibName = NSStringFromClass([self class]);
    }
    // Try to find the NIB in the bundle
    NSString *nibPath = [bundle pathForResource:nibName
                                         ofType:@"nib"];
    // Does it exist?
    if (nibPath) {
        // 顺便把nib生成的这个view作为VC的view（通过IBOutlet链接）
        [bundle loadNibNamed:nibName owner:self options:nil]; //**关键代码**
    } else {
        // 如果没有nib文件，那就新建一个view来作为VC的view
        self.view = [[UIView alloc] init];
    }
}
```

又如下面这个实现方法：实际上任何对象都可以通过向主NSBundle对象发送`loadNibNamed:owner:options:`消息手动载入XIB文件。”

```objectivec
//在TableViewController里面设置的headView属性的get方法
- (UIView *)headerView
　　{
　　// 如果还没有载入headerView...
　　if (!_headerView) {
　　　　// 载入HeaderView.xib
　　　　[[NSBundle mainBundle] loadNibNamed:@"HeaderView" owner:self options:nil];
　　}
　　return _headerView;
　　}
```

### UIWindow ，viewController和View的关系图

iew属性指向一个UIView对象。之前介绍过，UIViewController对象可以管理一个视图层次结构，view就是这个视图层次结构的根视图，当程序将view作为子视图加入窗口时，也会加入UIViewController对象所管理的整个视图层次结构。

为了将视图控制器的视图层次结构加入应用窗口，UIWindow对象提供了一个方法：setRootViewController:。当程序将某个视图控制器设置为UIWindow对象的rootViewController时，UIWindow对象会将该视图控制器的view作为子视图加入窗口。此外，还**会自动调整view的大小，将其设置为与窗口的大小相同。**

![视图层次结构](ViewController%E5%92%8CView%E5%88%9D%E5%A7%8B%E5%8C%96%E5%8F%8A%E5%85%B3%E7%B3%BB%20e8f81db1d20a467997d11461ea77992b/Untitled%201.png)

视图层次结构

[****iOS之xib和Storyboard加载原理****](ViewController%E5%92%8CView%E5%88%9D%E5%A7%8B%E5%8C%96%E5%8F%8A%E5%85%B3%E7%B3%BB%20e8f81db1d20a467997d11461ea77992b/iOS%E4%B9%8Bxib%E5%92%8CStoryboard%E5%8A%A0%E8%BD%BD%E5%8E%9F%E7%90%86%2000723d3df4e948479cd26f0d0b796b6c.md)

[UIView自定义方式及重绘原理](../312%20Swift%20f3f7435fa56948829718e9373255e683/UIView%E8%87%AA%E5%AE%9A%E4%B9%89%E6%96%B9%E5%BC%8F%E5%8F%8A%E9%87%8D%E7%BB%98%E5%8E%9F%E7%90%86%20823a4a3873d14028b42c085a1c8d7d15.md)

# 视图控制器及其视图各个层级定制方法（重要‼️）

这几级结构可以通过重写各个级别相应的方法来实现各个层级的自定义

window层级

- `application:didFinishLaunchingWithOptions`在该方法中设置和初始化应用窗口的根视图控制器。该方法只会在应用启动完毕后调用一次，设置`rootViewController` 属性来实现不同的ViewController

ViewController层级

- `initWithNibName:bundle`该方法是UIViewController的指定初始化方法，创建视图控制器时，就会调用该方法。请注意，某些情况下，需要在同一个应用中创建多个相同的UIViewController子类对象，每次创建一个该类的对象时，都会调用一次该类的initWithNibName:bundle:方法。
- `loadView`可以覆盖该方法，使用代码方式设置视图控制器的view属性。
- `viewDidLoad`可以覆盖该方法，设置使用NIB文件创建的视图对象。该方法会在视图控制器加载完视图后被调用。
- `viewWillAppear`可以覆盖该方法，设置使用NIB文件创建的视图对象。该方法和viewDidAppear:会在每次视图控制器的view显示在屏幕上时被调用；相反，viewWillDisappear:和viewDidDisappear:方法会在每次视图控制器的view从屏幕上消失时被调用。因此，如果打开HypnoNerd应用并在Hypnosis和Reminder两个标签项之间来回切换，那么BNRReminderViewController的viewDidLoad方法只会被调用一次，而viewWillAppear:方法会被调用很多次。”

View层级

- 重写`drawRect:`来实现view的具体内容定制