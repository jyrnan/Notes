# How an App with UIKit launches

# 两种新旧不同的架构 Window Scene Architecture

了解App的启动过程需要先了解在iOS12和iOS13二者app的架构是不同的。最明显的区别就在于window这个属性存在于哪里。

- Window architecture
这是iOS12及之前的，是基于window的架构，window是`app delegate`的一个属性
- Scene architecture
    - iOS13及以后都是新的架构，基于scene， app会有一个`scene delegate`。 window是`scene delegate`的一个属性
    
    ```swift
    class SceneDelegate: UIResponder, UIWindowSceneDelegate {
        var window: UIWindow?
        // ...
    }
    ```
    
    - 并且`app delegate` 会有两个指向scene的方法
    
    ```swift
    class AppDelegate: UIResponder, UIApplicationDelegate {
        // ...
        func application(_ application: UIApplication,
            configurationForConnecting connectingSceneSession: UISceneSession,
            options: UIScene.ConnectionOptions) -> UISceneConfiguration {
                // ...
        }
        func application(_ application: UIApplication,
            didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
                // ...”
    	}
    }
    ```
    
    - 除此外，info.plist还包含有“`Application Scene Manifest`”章节
    
    ```swift
    <key>Application Scene Manifest</key>
    <dict>
        <!-- ... -->
    </dict>
    ```
    

> 如果需要兼容新老版本，需要将windows拷贝到appDelegate之下，并将与`sceneDelegate`相关的方法和定义通过`@available(iOS 13.0, *)`进行限制
> 

# 新架构启动过程

在Swift中，一个应用启动的过程是相比较OC要隐性的：在于App启动过程中会默认调用`UIApplicationMain` 方法。这个方法会逐一初始化许多重要的实例。如果这个app使用了main storyboard，那么这些实例就会包括window和它的root view controller

本过程仅描述新版架构。

## 启动过程

1. `UIApplicationMain` 初始化 `UIApplication`并持有。我们可以通过`UIApplication.shared`来访问。`UIApplication`然后就会初始化标记有`@main`的`AppDelegae`并持有它，这两个都会在整个App的生命周期存在
2. `UIApplicationMain` 会调用app delegate的 `application(_:didFinishLaunchingWithOptions:)`

<aside>
💡 这个方法里面可以设置一些代码执行，相比之下会执行的比较早期

</aside>

1. UIApplicationMain 接下来创建一个 UISceneSession，一个UIWindowScene以及一个scene delegate。在前面提到的[info.plist的字节](How%20an%20App%20with%20UIKit%20launches%2062d8735f5b474bc0b1d53d418dd40778.md)会**指定**这个scene delegate的类名。默认是`$(PRODUCT_MODULE_NAME).SceneDelegate`
2. UIApplicationMain会查看是否采用storyboard来初始化scene。

### 采用storyboard

1. 如果采用storyboard，storyboard是info.plsit里面的`Application Scene Manifest`章节`Storyboard Name`来决定
2. `UIApplicationMain`初始化一个storyboard上的initial view controller
3. `UIApplicationMain`初始化一个UIWindow并且把这个window设置成`sceneDelegate`的window属性
4. `UIApplicationMain`会将initial view controller作为window实例的root view controller。这个root view controller的view将成为window最基本？的subview
5. `UIApplicationMain`调用UIWindow 的实例方法makeKeyAndVisible来显示app的用户界面
6. 调用secene delegate的`scene(_:willConnectTo:options:)`

### 不采用storyboard

不采用storyboard意味着在info.plist没有相应的字节，具体是

- 老架构是没有`“ Main storyboard file base name”`字节
- 新架构是`“Application Scene Manifest”`没有`“Storyboard Name”`字节

所有`UIApplicationMain`的动作都将通过代码完成。在新架构里面这些代码放在scene delegate的`scene(_:willConnectTo:options:)`里面

```swift
func scene(_ scene: UIScene,
    willConnectTo session: UISceneSession,
    options connectionOptions: UIScene.ConnectionOptions) {
        if let windowScene = scene as? UIWindowScene {
            self.window = UIWindow(windowScene: windowScene) //1
            let vc = // ...  2                                
            self.window!.rootViewController = vc   //3          
            self.window!.makeKeyAndVisible()    //4             
        }
}
```

代码实现其实也是和采用storyboard的步骤一样。

采用代码来实现步骤可以在步骤2实现更加灵活的viewController初始化，例如针对是否登陆，可以在App启动的时候有不同的界面：

采用StoryBoard的形式来生产ViewController

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

如果不采用Storyboard来产生ViewController，可以用xib文件。**这是比较老的做法**：

```objectivec
- (void)scene:(UIScene *)scene willConnectToSession:(UISceneSession *)session options:(UISceneConnectionOptions *)connectionOptions {
    
    self.window = [[UIWindow alloc] initWithWindowScene:scene];
    MyViewController *VC = [[MyViewController alloc] init]; //需要同名的xib文件，会自动调用ViewController的initWithNibName来生成VC
    [VC.view setBackgroundColor:[UIColor blueColor]];
    self.window.rootViewController = VC;
    [self.window makeKeyAndVisible];
}
```

ViewController里重写initWithNibName方法

```objectivec
- (instancetype)initWithNibName:(NSString *)nibNameOrNil
                         bundle:(NSBundle *)nibBundleOrNil {
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    
    if (self) {
        self.questions = @[@"From what is cognac made?",
                           @"What is 7 + 7",
                           @"What is the capital of Vermont?"];
        self.answers = @[@"Grapes",
                         @"14",
                         @"Montpelier"];
    }
    return self;
}
```