# Xcode 编辑器之Workspace，Project，Scheme，Target

## **[Xcode 编辑器之Workspace，Project，Scheme，Target](https://www.cnblogs.com/lxlx1798/p/9369537.html)**

# 一、 **前言**

最近老是突然对Workspace，Project，Scheme，Target四者的关系有些疑惑，所以查阅资料总结一下。

# 二、 **Workspace，Project，Scheme，Target四者的定义**

## 1. **Xcode Workspace**（可类比为“办公大楼”）

定义：Workspace是组织projects和其他协同工作的一份文档。作用：       

- 它将一些项目以及其他的文档组织在一起，以便于你基于它们进行工作。一个Workspace可以包含任意个xcode项目，你也可以往里面加一些你想要的文件。
- 它还维护project和target之间的显性及隐性的依赖关系。
- 默认workspace中的所有Xcode projects都在同一个目录下构建，称为workspace build directory，由于所有project的所有文件都在同一个目录下，所以所有文件都对每个project可见。比如两个project使用同一个库，则不用复制到另一个project目录中。
- workspace中的每个project都有其独立id,同时project可以属于多个workspace，可以单独打开project或者在其他workspace中打开，且都不用重新配置project或者workspace。
- 可以使用workspace默认的build 目录，也可以指定一个。如果project指定了构建目录，这个目录会被project构建时所在的workspace的build目录覆盖。

创建：`File->New->WorkSpace`

示例：

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726035629618-1222179777.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726035629618-1222179777.png)

## 2. **Xcode project**（可理解为“在办公大楼里的公司”）

定义：Xcode中的project是一个包含了所有文件、资源以及构建一个或者多个product所需信息的一个仓库。            

- project包含编译product所用的所有元素，并帮我们组织这些元素之间的关系。一个project可以包含一个或者多个target，target指定了如何编译product。project定义了其内部所有target的默认编译配置（每个target可以通过重新设置（重写）编译配置来定义自己的特殊编译配置）。
- project包含了以下信息：

```
1、对如下源文件的引用：
    （1）代码的.h文件和.m文件
    （2）Libraries 和 frameworks
    （3）Resource files（资源文件，如文本，xml，plist等）
    （4）图片
    （5）Interface Builder files(xib、storyboard等）
 
2、用来组织资源文件的group（比如哪个文件放在哪个文件夹路径下）
 
3、project级别的编译配置，比如你对debug、release两种环境的分别设置。
 
4、targets的配置，这些配置指定了：
    （1）工程编译的引用
    （2）工程编译所需要的资源文件的引用
    （3）工程编译的编译配置，包括对其他target和配置的依赖关系，还包括没有target重写（覆盖）的project级别的编译配置
 
5、用于debug和测试程序的执行环境，这些执行环境指定了：
    （1）xcode在run或debug时的可执行文件
    （2）传递给可执行文件的命令行参数（若有）
    （3）程序运行时设置的环境变量（若有）
```

- 一个project可以独立存在，也可以包含在一个workspace下。 你可以用Xcode schemes来指定当前的target、编译配置、可执行文件配置。

## 3. Xcode Scheme（可以理解为“公司里的项目团队”）

定义：Xcode scheme 定义了可构建的target的集合、构建时可用的设置以及可执行的测试的集合。  

- 你可以创建任意数目的scheme，但是同一时刻只有一个是活跃的。
- 你可以指定某一个scheme应该是存储在project中还是workspace中——如果是存储在project中，它在所有包含这个project的workspace中都是可用的，若是存储在workspace中，它只能在当前的workspace中可用。当你选择一个活跃的scheme，你也就同时选择了一个运行目的（也就是说，这个产品将要构建在那个硬件结构中）。

## 4. Xcode Target（可以理解为“公司里项目团队的产品”）

定义：一个Targetd指定一个产品来构建并且包含了从项目或workspace的全部（或部分）文件中构建产品的指令。一个target定义了一个单一的产品。

- 它来组织构建产品的输入文件（包含构建产品必需的所有文件：源文件，预编译这些源文件的指令）。
    - Project 可以包含一个或多个target，每个target产生一个产品，可对应一个不同的应用。
    - 构建产品的指令有两种形式：构建设置和构建阶段（Build settings 和 Build phases）, 你可以在Xcode的项目editor中检查并编辑它们。
    - target继承了project的构建设置。但是你可以在target层次覆盖任何project的设置。在同一时刻，只有一个target是活跃的。Xcode scheme指定了那个target是活跃的。
- 一个target以及由它生成的产品可以与其他的target建立关系。
    - 如果构建一个target需要另一个target的输出，我们就说第一个target依赖于第二个target。
    - 如果它们两个都在同一个workspace中，xcode可以发现这种依赖关系，那么它就会按照需要的顺序来构建产品。这种关系叫做隐式依赖。你也可以在你的构建设置中设置显式依赖，并且你可以指定某两个xcode可能判定为有隐式依赖的target之间没有依赖关系。
        
        例如：你可以构建一个库以及一个链接这个库的在同一个workspace下的应用程序。xcode可以发现这种关系并且自动先构建这个库。当然，如果你确实需要链接到一个与当前workspace下不同版本的库中，你可以在你的构建设置中创建一个显示依赖，这将覆盖原来的隐式依赖。
        

# 三、相关使用方法

## **1.在一个Xcode页面建立多个工程**

### **1.1建立一个工作空间MyWorkspace**

Xcode里面，建立一个工作空间。`File->New->Workspace`,命名为Myworkspace,存放在文件夹MyWorkspace中（名字都是可以随便命名的）。

新建一个workspace

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042309553-192788517.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042309553-192788517.png)

这样我们就建立了一个空的工作空间。然后我们就可以往这个工作空间中加入几个Xcode工程。

### **1.2 使用方法一建立一个普通的Xcode工程MyApp1添加到MyWorkspace**

`File->New->Project` 新建一个名为MyApp1的app工程文件。为了便于管理，我们把他放在MyWorkspace文件夹中。创建完成后在工作空间的Xcode工程中，`File->Add File To` "MyWorkplace"，选中刚才创建的MyApp1工程。这样MyApp1工程就添加到了MyWorkplace中了。

创建一个普通的App工程

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042428667-502953876.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042428667-502953876.png)

将MyApp1添加到MyWorkspace

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042518847-372916433.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042518847-372916433.png)

### **1.3. 使用方法二建立一个普通的Xcode工程MyApp2添加到MyWorkspace。**

我们用另外一种方法添加一个工程到MyWorkplace工作空间中。`File->New->Project` 新建一个名为MyApp2的app工程文件。在存放工程的界面中，将下面的Add to : 选择成MyWorkspace。这样MyApp2工程就添加到了MyWorkplace中。到此，我们就可以在一个Xcode的界面中同时管理两个工程了。

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042819477-994190626.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042819477-994190626.png)

建立工程的时候将工程添加到MyWorkspace中

### **1.4. 建立一个SDK工程MySDK添加到MyWorkspace。实现联编。**

但是多工程使用的精髓并不在这里，而在于两个工程连编。

我们新建一个名为MySDK的.a库。然后将这个MySDK工程添加到MyWorkplace，来实现MyApp2与MySDK联调。`File->New->Project->`选择`Static Library` ，按照方法二添加到MyWorkspace。

然后在MySDK里面创建一个sayHello类方法。在方法中打一个断点。将.a库拉到MyApp2里面。在ViewController里面调用sayHello方法。执行之后，断点就会停在MySDK工程中的里面。这样就可以在两个工程进行调试了。

具体方法如下：

- 建立一个静态库工程，添加到MyWorkspace

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042933275-2100640967.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042933275-2100640967.png)

- 在MySDK中加一个sayHello方法

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042952995-1730139095.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726042952995-1730139095.png)

- 在MyApp2中调用sayHello方法

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043018216-1594695304.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043018216-1594695304.png)

- 运行MyApp2，断点会停在MySDK工程

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043150303-902266821.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043150303-902266821.png)

## **2.在同一个xcode工程里面创建多个项目**

### 2.1首先，先创建第一个Test1工程（项目），并选中Test1工程

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043549920-1441836829.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043549920-1441836829.png)

点击下面的+号

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043655443-1160572810.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043655443-1160572810.png)

### 2.2接着，跟创建新项目一样的操作，创建项目Test2。

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043749690-355130697.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043749690-355130697.png)

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043830729-303753536.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043830729-303753536.png)

点击『Finish』，就完成在同一工程创建两个项目了，再创建多个项目，按照以上操作即可。

### 但有一点需要注意的是：

- 在同一时间，XCode只能对同一工程中的某一个项目进行操作，具体要操作哪个项目，可以按以下操作：
- 这其实就是不同的Scheme

![https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043941753-508367232.png](https://images2018.cnblogs.com/blog/1115039/201807/1115039-20180726043941753-508367232.png)

## **3.在一个工程创建两个不同版本的应用**

### **一般来说有两种实现方法:**

1. 一个把当前工程拷贝然后再修改, 这样做会导致后期维护成本过高, 每次修改都要同时改两个工程, 到后期修改简直提心掉胆, 不过操作傻瓜式.
2. 另外一个种就是在一个Project里面创建两个Target, 然后通过判断Target来修改代码, 这样都是基于同一套代码做修改, 只是部分不相同的地方通过Target来添加不同代码, 后期修改维护成本低, **推荐大家使用这种方式.**

### **第一步：创建两个Target:**

- 首先先选中Target一个已经存在的版本, 右键 Duplicate

![http://cc.cocimg.com/api/uploads/20150902/1441176864427426.png](http://cc.cocimg.com/api/uploads/20150902/1441176864427426.png)

- 然后在弹出来的选择框选择 Duplicate Only

![http://cc.cocimg.com/api/uploads/20150902/1441176874704199.png](http://cc.cocimg.com/api/uploads/20150902/1441176874704199.png)

- 创建完之后你的新Target应该是和我的一样, 接下来我们就要修改Target, Scheme, Info-plist, 如图这样修改:

![http://cc.cocimg.com/api/uploads/20150902/1441176887819248.png](http://cc.cocimg.com/api/uploads/20150902/1441176887819248.png)

![http://cc.cocimg.com/api/uploads/20150902/1441176900409531.png](http://cc.cocimg.com/api/uploads/20150902/1441176900409531.png)

- 修改完了之后Target, Scheme, plist的名字之后, 你需要在新的Target添加对应的plist文件, 修改CFBundleDisplayName就可以修改应用的名字了.

![http://cc.cocimg.com/api/uploads/20150902/1441176916766853.png](http://cc.cocimg.com/api/uploads/20150902/1441176916766853.png)

![http://cc.cocimg.com/api/uploads/20150902/1441176924972286.png](http://cc.cocimg.com/api/uploads/20150902/1441176924972286.png)

![http://cc.cocimg.com/api/uploads/20150902/1441176931361554.png](http://cc.cocimg.com/api/uploads/20150902/1441176931361554.png)

- 还要记得修改一下Product Name 不然你的Bundle Identifier的后缀名有copy和你的Target名字不一样, 你还需要在Bundle Setting做一下修改.

![http://cc.cocimg.com/api/uploads/20150902/1441176948485337.png](http://cc.cocimg.com/api/uploads/20150902/1441176948485337.png)

### **第二步，开始为两个不同的应用添加不同的AppIcon, LaunchImage**

- 在这个选择使用Images.xcassets里面设置AppIcon和LaunchImage, 注意这里一个是AppIcon,另一个是AppIcon-2, 以后编译Target的时候他就会跟随这里的设置去拿了开机图和Icon

![http://cc.cocimg.com/api/uploads/20150902/1441176965975068.png](http://cc.cocimg.com/api/uploads/20150902/1441176965975068.png)

![http://cc.cocimg.com/api/uploads/20150902/1441176985829749.png](http://cc.cocimg.com/api/uploads/20150902/1441176985829749.png)

![http://cc.cocimg.com/api/uploads/20150902/1441176995681299.png](http://cc.cocimg.com/api/uploads/20150902/1441176995681299.png)

- 进入Images.xcassets看下图片是不是都是勾选了两个Target, 保持和我下图一样的勾选, 如果没有勾选的话, 你在编译的不同Target的时候会获取不到资源.

![http://cc.cocimg.com/api/uploads/20150902/1441177007387104.png](http://cc.cocimg.com/api/uploads/20150902/1441177007387104.png)

- 选择不同Target进行编译, 你的运行结果应该和我的截图一样, 有着不同的AppName和AppIcon,还有不同的LaunchImage,但是代码是共用, 到这里你已经成功了一半了, 接下来你肯定是想知道如何在代码里面区别不同Target, 然后给它们添加其他的特性.

![http://cc.cocimg.com/api/uploads/20150902/1441177121441268.png](http://cc.cocimg.com/api/uploads/20150902/1441177121441268.png)

<aside>
💡 补充一点：如果需要出现上图那样两个不同的app，是需要修改Bundle Identifier

</aside>

![Untitled](Xcode%20%E7%BC%96%E8%BE%91%E5%99%A8%E4%B9%8BWorkspace%EF%BC%8CProject%EF%BC%8CScheme%EF%BC%8CTarget%20a5fc80fa0a5d4582a9e9478a996cf7b1/Untitled.png)

### **第三步，在代码里面利用宏定义来区分不同的Traget**

- 在Bundle Setting里面设置一下Proprecessor Macros添加一份KFREE  KPRO的参数来区分到底是那个Traget. 在代码里面为需要用到这个宏去判断代码块.

![http://cc.cocimg.com/api/uploads/20150902/1441177048343802.png](http://cc.cocimg.com/api/uploads/20150902/1441177048343802.png)

![http://cc.cocimg.com/api/uploads/20150902/1441177056673797.png](http://cc.cocimg.com/api/uploads/20150902/1441177056673797.png)

- 在代码里面添加Proprecessor Macros里面宏定义, 你就会发现编译之前Xcode就会智能的选择不同代码. 这样你就共用一个项目管理两个不同版本的应用了, 大部分的代码都复用, 维护管理非常轻松.

![http://cc.cocimg.com/api/uploads/20150902/1441177083999938.png](http://cc.cocimg.com/api/uploads/20150902/1441177083999938.png)

![http://cc.cocimg.com/api/uploads/20150902/1441177092863897.png](http://cc.cocimg.com/api/uploads/20150902/1441177092863897.png)