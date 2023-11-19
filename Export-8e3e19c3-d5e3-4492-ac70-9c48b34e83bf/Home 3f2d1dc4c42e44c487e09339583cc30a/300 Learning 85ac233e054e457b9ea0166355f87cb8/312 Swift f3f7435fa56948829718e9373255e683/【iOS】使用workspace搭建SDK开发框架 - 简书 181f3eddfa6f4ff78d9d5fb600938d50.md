# 【iOS】使用workspace搭建SDK开发框架 - 简书

# **前言**

SDK开发和APP并不一样，APP开发简单点直接开个项目撸就是了，但是SDK需要打包成库，然后才能拿这个库去用。所以，SDK开发一般都需要创建3个项目：SDK项目、测试项目（自己单元测试用的）、demo项目（给用户看的，要有完整的使用代码）。如果每个项目都是独立的，那么就需要打包好库，然后放到测试项目里去测试，测试好了再搞到demo里，这样子太麻烦了，还很low。其实苹果对于这种情况早就有解决方案了，那就是workspace，即工作空间。

> workspace允许你把多个项目放到一个工程里，他们既是独立的，也能有所联系。正是这种特性使得我们能快速开发而不需要过多的考虑其它。
> 

---

---

***接下来我会给大家演示怎么用workspace来搭建开发SDK的架构***

# **库的选用 -- 为什么用Framework**

- .a是一个纯二进制文件，而.framework中除了有二进制文件之外还有头文件和资源文件。
- .a文件不能直接使用，至少要有.h文件配合，.framework文件可以直接使用。
- .a + .h + sourceFile = .framework。
- framework可以是动态库，也可以是静态库。

综上所述，如果是.a的话，资源和头文件与库就会很零散，被弄乱了都不知道；而framework可以很好的把一个所需的文件库集合在一起。同时framework既能做动态库也能做静态库，在库类型的切换上有天然的优势。故而选framework更为友好。

# **动态库与静态库的区别**

- 静态库在程序编译时会被链接到目标代码中，程序运行时将不再需要该静态库，所以静态库是相对于编译期的；动态库在程序编译时并不会被链接到目标代码中，只是在程序运行时才被载入，所以动态库是相当于运行期的。
- 静态库在链接时会被完整的复制到可执行文件中，被多次使用就有多份拷贝；动态库在链接时不复制，程序运行时由系统动态加载到内存，系统只加载一次，主APP和App Extension之间共享动态库，节省内存。
- 静态库没有自己的独立空间，用的是主APP的空间（*刚才也说到，静态库是在编译的时候复制过去的*），所以会有符号重复的问题，比如库用了AFN，主工程也用了AFN，在编译的时候就会报符号重复的问题；而动态库是动态加载的，有自己的独立空间，所以能内置bundle，也不会出现符号重复的问题。

> 注意：  动态库上架不能带模拟器版本，所以上架的时候不要合并模拟器版本。iOS8开始支持动态库了，所以动态库是能上架的，请放心使用动态库吧（本人已经用动态库上架很多APP了）。如果动态库编译报错，在SDK的General - Linked Framework and Librarles里点击“+”号，搜索libSystem并添加进去。
> 

# **开始搭建框架**

# **创建workspace**

---

先在桌面创建一个文件夹，我把它命名为iOS（***因为创建workspace是不会像创建项目一样自动帮你生成文件夹***）。打开Xcode，选择`File-New-Workspac`，名字也命名为iOS。然后选择我们刚才创建的iOS文件夹，点击保存。一个空的workspace创建好了。

![https://upload-images.jianshu.io/upload_images/14154456-8e7bbfc602d8a8d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/841](https://upload-images.jianshu.io/upload_images/14154456-8e7bbfc602d8a8d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/841)

创建workspace.png

# **创建测试项目**

---

选择`File-New-Project-Single View App`，名字命名为SDKTest。然后选择我们刚才创建的iOS工作空间，点击创建。一个空的测试项目创建好了。

![https://upload-images.jianshu.io/upload_images/14154456-7936e49f73651c27.png?imageMogr2/auto-orient/strip|imageView2/2/w/799](https://upload-images.jianshu.io/upload_images/14154456-7936e49f73651c27.png?imageMogr2/auto-orient/strip|imageView2/2/w/799)

创建SDKTest.png

# **创建demo项目**

---

选择`File-New-Project-Single View App`，名字命名为SDKDemo。然后选择我们刚才创建的iOS工作空间，点击创建。一个空的demo项目创建好了。

![https://upload-images.jianshu.io/upload_images/14154456-33712f56dfa53098.png?imageMogr2/auto-orient/strip|imageView2/2/w/799](https://upload-images.jianshu.io/upload_images/14154456-33712f56dfa53098.png?imageMogr2/auto-orient/strip|imageView2/2/w/799)

创建SDKDemo.png

# **创建SDK项目**

---

选择`File-New-Project-Cocoa Touch Framework`，命名为SDK。然后选择我们刚才创建的iOS工作空间，点击创建。一个空的SDK项目创建好了。

![https://upload-images.jianshu.io/upload_images/14154456-2678f86905f1a2eb.png?imageMogr2/auto-orient/strip|imageView2/2/w/799](https://upload-images.jianshu.io/upload_images/14154456-2678f86905f1a2eb.png?imageMogr2/auto-orient/strip|imageView2/2/w/799)

创建SDK.png

> 项目结构一览
> 
> 
> ![https://upload-images.jianshu.io/upload_images/14154456-2c6dbdeeef94597e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200](https://upload-images.jianshu.io/upload_images/14154456-2c6dbdeeef94597e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)
> 
> 项目结构.png
> 

# **创建脚本target**

---

> 单单只是有上面的项目还是不行的，无法实现联调，现在我的需求是，当运行SDKDemo或者SDKTest时，Xcode自动帮我编译好SDK。为了实现这个需求，要用到脚本。
> 

我们选中SDK项目，点击`File-New-Target`，选中`Cross-platform`，然后选择`Aggregate`，命名为SDKBuildScript，点击完成。

![https://upload-images.jianshu.io/upload_images/14154456-5e72238a53034c19.png?imageMogr2/auto-orient/strip|imageView2/2/w/730](https://upload-images.jianshu.io/upload_images/14154456-5e72238a53034c19.png?imageMogr2/auto-orient/strip|imageView2/2/w/730)

创建SDKBuildScript.png

点击`File-New-File`，选择`Shell Script`，命名为SDKBuild，点击创建（***不要把它添加到Targets，不然会被编译到Framework里的***）。

![https://upload-images.jianshu.io/upload_images/14154456-9ef02f2e78879c86.png?imageMogr2/auto-orient/strip|imageView2/2/w/841](https://upload-images.jianshu.io/upload_images/14154456-9ef02f2e78879c86.png?imageMogr2/auto-orient/strip|imageView2/2/w/841)

创建SDKBuild.png

> 把这段脚本复制到SDKBuild中，其中TARGET_NAME默认是当前项目名，如果你的target名字和项目名不一样，请修改TARGET_NAME为正确的target名字；OUTPUT_FOLDER、TEST_FOLDER、DEMO_FOLDER也是根据你实际情况来赋值的。其它的不需要修改，能直接用。完成脚本编写后，把Xcode里的SDKBuild文件删了（但不要移除到废纸篓）。
> 

```bash
# 要build的target名
TARGET_NAME="${PROJECT_NAME}" # 如果和target名不一致，需要修改为正确的target名
CONFIG="Release" # "Release" "${CONFIGURATION}" "Debug" 编译模式，使用Release即可

# 项目里存放Framework的路径
OUTPUT_FOLDER="${SRCROOT}/${PROJECT_NAME}/"
TEST_FOLDER="${SRCROOT}/../SDKTest/SDKTest"
DEMO_FOLDER="${SRCROOT}/../SDKDemo/SDKDemo"

# ---------- 以上配置是可以修改的，下面的配置则不需要改 ----------

# 编译时存放framework的路径
IPHONE_DIR="build/${CONFIG}-iphoneos/${TARGET_NAME}.framework"
SIMULATOR_DIR="build/${CONFIG}-iphonesimulator/${TARGET_NAME}.framework"

# 定义函数，用来清除编译产生的文件
function removeBuild ()
{
# 判断build文件夹是否存在，存在则删除
if [ -d "${SRCROOT}/build" ]
then
rm -rf "${SRCROOT}/build"
fi
}

removeBuild # 编译前先删除之前留下来的，防止干扰

# 分别编译模拟器和真机的Framework
xcodebuild -configuration "${CONFIG}" -target "${TARGET_NAME}" -sdk iphoneos clean
xcodebuild -configuration "${CONFIG}" -target "${TARGET_NAME}" -sdk iphonesimulator clean
xcodebuild -configuration "${CONFIG}" -target "${TARGET_NAME}" -sdk iphoneos build
xcodebuild -configuration "${CONFIG}" -target "${TARGET_NAME}" -sdk iphonesimulator build

# 判断IPHONE_DIR和SIMULATOR_DIR是否存在，存在就可能是编译成功
if [[ -d "${IPHONE_DIR}" && -d "${SIMULATOR_DIR}" ]]
then
# 删除之前的Framework文件
rm -rf "${OUTPUT_FOLDER}/${TARGET_NAME}.framework"
rm -rf "${TEST_FOLDER}/${TARGET_NAME}.framework"
rm -rf "${DEMO_FOLDER}/${TARGET_NAME}.framework"

# 拷贝Framework到目录
cp -R "${IPHONE_DIR}" "${OUTPUT_FOLDER}"

# 合并Framework
lipo "${SIMULATOR_DIR}/${TARGET_NAME}" -remove arm64 -output "${SIMULATOR_DIR}/${TARGET_NAME}" # Xcode12开始，模拟器版本也有arm64，会导致合并失败，所以需要先移除了
lipo -create "${IPHONE_DIR}/${TARGET_NAME}" "${SIMULATOR_DIR}/${TARGET_NAME}" -output "${OUTPUT_FOLDER}/${TARGET_NAME}.framework/${TARGET_NAME}"

# 删除编译之后生成的无关的配置文件
DIR_PATH="${OUTPUT_FOLDER}/${TARGET_NAME}.framework"
for FILE in $(ls "${DIR_PATH}"|tr " " "?") # 解决名字带空格的问题
do
if [[ "${FILE}" =~ ".xcconfig" ]]
then
rm -f "${DIR_PATH}/${FILE}"
fi
done

# 拷贝Framework到别的工程
cp -R "${OUTPUT_FOLDER}/${TARGET_NAME}.framework" "${TEST_FOLDER}"
cp -R "${OUTPUT_FOLDER}/${TARGET_NAME}.framework" "${DEMO_FOLDER}"

fi

removeBuild
```

点击SDK项目，然后在TARGETS里选中SDKBuildScript，上面选中`Build Phases`，点击左上角的“+”号，选择`New Run Script Phase`。

![添加Script Phase.png](https://upload-images.jianshu.io/upload_images/14154456-5707992a55ba8045.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

添加Script Phase.png

在黑框里输入*./SDKBuild.sh*。

![https://upload-images.jianshu.io/upload_images/14154456-ebb014ea6aeb4837.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200](https://upload-images.jianshu.io/upload_images/14154456-ebb014ea6aeb4837.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

黑框.png

> 当然你也可以直接把脚本写在黑框里，这样子就不需要创建脚本文件了。如果使用脚本文件的话，会报没权限的错误，所以需要使用命令行来打开权限。（因为使用文件方便管理和编写代码，所以这里我选择了使用文件的方式）打开命令行，cd到SDKBuild.sh所在的目录，然后执行sudo chmod +x SDKBuild.sh即可。
> 

# **使用Bundle管理资源**

---

> 当我们的库需要用到一些资源时，如果资源分散乱放，对使用者而言是件很痛苦的事，所以我们需要把资源集中到一起，这个时候bundle就很有用了。
> 

如何创建Bundle我就不说了，网上大把的资料，我主要说的是使用Bundle的一些细节。目前普遍使用Bundle的方式都是把Bundle作为一个独立体的存在，即打包好Bundle，然后把Bundle和库一起给开发者去使用，这样子开发者就能使用到我们打包好的资源了。但是这里有个问题，那就是资源可能会被替换，从而引发未知问题。我们使用framework就是为了资源和代码成为一个整体，也就是说，开发者只需要导入framework就能使用到Bundle的资源了。基于此，所以这里对*“Bundle作为一个独立体的存在”*这种情况不讨论了，网上也一大把资料。我曾经使用很多方法尝试了获取framework里Bundle的资源，但都失败了，虽然Bundle确实存在于framework里，但是怎么都读取不出来。然后我读了这篇文章：[iOS：NSBundle的一些理解](https://www.jianshu.com/p/b64ff9d8e7ce)  我终于知道了， ~~静态库是拿不到该Bundle的资源的，所以只能使用动态库~~（***事实证明静态库也是能读取资源的，具体之后有详细说明***）。经验证，在动态库里确实是能读取到资源的，以读取图片为例，有3种方式：

```bash
NSBundle *resourceBundle = [NSBundle bundleForClass:[SDKData class]]; // 获取类所在的bundle
NSString *bundlePath = [resourceBundle pathForResource:@"SDK" ofType:@"bundle"]; // 获取资源bundle路径

    // 方式1 直接拼路径
//    NSString *imagePath = [bundlePath stringByAppendingPathComponent:@"user.jpg"];
//    UIImage *image = [UIImage imageWithContentsOfFile:imagePath];
    
    // 方式2 通过获取bundle来操作
//    NSBundle *bundle = [NSBundle bundleWithPath:bundlePath];
//    NSString *imagePath = [bundle pathForResource:@"user.jpg" ofType:nil];
//    UIImage *image = [UIImage imageWithContentsOfFile:imagePath];

    // 方式3 通过传入bundle来获取数据
NSBundle *bundle = [NSBundle bundleWithPath:bundlePath];
UIImage *image = [UIImage imageNamed:@"user.jpg" inBundle:bundle compatibleWithTraitCollection:nil];
```

***静态库想要获取到资源文件，需要在`Build Phases - Copy Bundle Resources`里把你的库添加进去，这样子就能用以下方法获取资源了：***

```bash
NSString *bundlePath = [[NSBundle mainBundle] pathForResource:@"SDK.framework/SDK" ofType:@"bundle"];
NSBundle *bundle = [NSBundle bundleWithPath:bundlePath];
NSString *imagePath = [bundle pathForResource:@"user" ofType:@"jpg"];
UIImage *image = [[UIImage alloc] initWithContentsOfFile:imagePath];
```

> 对比下动态库和静态库的读取bundle方式：
> 
> - 动态库不需要额外的设定都能读取资源，而静态库需要额外设置。
> - 动态库读取资源的方式有3种，而静态库只有1种。
> - 动态库可以通过class获取bundle所在的位置，较为灵活，而静态库只能拼路径，这种写死路径的做法其实风险还是比较大的。
> - 如果把bundle通过`Copy Bundle Resources`方式添加的话，上架是会报错的。（*所以，事实上静态库内置bundle也只是能方便本地调试，并不能上架，上架还是要把bundle拿出来*）

---

---

---

***一个简单的workspace工程就完成了，接下来说说SDK的一些配置、库的使用和workspace联调的一些注意事项***

# **项目配置**

# **SDK项目配置**

---

- 在`Build Settings`里找到`Build Active Architecture Only`，把它设置为NO。 这个配置的作用是生成所有平台的二进制文件。
- 在`Build Settings`里找到`Architectures`，点击`Other..`增加armv7s支持，不过如果不需要支持iPhone5和iPhone5C的话，则不需要加（***同样的，当少了一个平台时，编译出来库的大小就会相应的变小***）。
- 在`Build Phases - Headers`里可以看出有三个选项，分别是 Public、Private、Project；把需要公开给别人的头文件拖到Public 中，把不想公开的（即隐藏的）头文件拖到Project中。（***Private下的头文件依然是可以暴露出来的，因此名字可能有些误导。事实上Project下的头文件对你的工程来说才是“私有”的，因此，一般的头文件或者在Public或者Project下***）
- 在默认生成的.h文件中（我的是SDK.h），把所有需要暴露的头文件都用 `#import <SDK/XX.h>`的方式引入；记住，被包含的头文件一定要在`Headers - Public`中，不然编译后生成的framework在引用的时候会有警告。

### **可选配置**

---

---

> 这些配置不是SDK必须的，但可以根据实际需求进行配置。
> 
- 我们创建的framework默认是动态库，如果需要静态库则在`Build Settings`里找到在`Mach-O Type`并设置为`Static Library`。
- 在`Build Phases - Link Binary With Libraries`里添加项目的依赖库。
- 在`Build Phases - Copy Bundle Resources`里添加项目中使用到的资源文件，如图片、XIB文件、plist文件等。
- 在`Build Settings`里找到`Product Name`，这里设置的名称是编译出来的framework文件名称。
- 在`Build Settings`里找到`Base SDK`，设置成当前Xcode最新版本。
- 在`Build Settings`里找到`Dead Code Stripping`，设置为YES。这时会过滤掉”dead”、”unreachable”的代码，即不会执行到的代码。
- 在`Build Settings`里找到`Debug Information Level`，设置为`Line tables only`。这时调试信息允许获得带有函数名、文件名和行号的函数调用栈，但是不包含其他数据（比如局部变量和函数参数），即断点依然会中断，但是无法在调试器中查看局部变量的值。
- 在`Build Settings`里找到`Link With Standard Libraries`，设置为NO。这时会避免重复链接。（***不过有可能造成链接库找不到而报错***）
- 在`Build Settings`里找到`Strip Style`，设置为`Non-Global Symbols`。（***`Strip Linked Product`为YES时`Strip Style`才生效；对于库而言，最高去除符号的级别为`Non-Global Symbols`，如果为`All Symbols`则无法找到符号，从而引发报错***）
- 在`Build Settings`里找到`Strip Linked Product`，设置为YES，此时运行APP，断点不会中断，在程序中打印[NSThreadcallStackSymbols]也无法看到类名和方法名。而在程序崩溃时，函数调用栈中也无法看到类名和方法名；当该项为NO时，包的体积会变大，因为它容纳了本要去除掉的调试信息。（***`Strip Linked Product`选项在 `Deployment Postprocessing`设置为YES的时候才生效，而在 Archive的时候Xcode总是会把`Deployment Postprocessing`设置为 YES 。所以我们可以打开`Strip Linked Product`并且把`Deployment Postprocessing`设置为NO，而不用担心调试的时候会影响断点和符号化，同时打包的时候又会自动去除符号信息***）
- 在`Build Settings`里找到`Strip Debug Symbols During Copy`，设置为YES；与`Strip Linked Product`类似，但是这个是将那些拷贝进项目包的第三方库、资源或者Extension的Debug Symbol去除掉，同样也是使用的strip命令。这个选项没有前置条件，所以我们只需要在Release模式下开启，不然就不能对第三方库进行断点调试和符号化了。（***如果依赖的Target是独立签名的（比如App Extension），strip操作就会失效，并伴随着Warning：warning: skipping copy phase strip, binary is code signed: xxxx。此情况将依赖的Target 中的`Strip Linked Product`修改为YES，保证依赖的Target是已经去除了符号即可，Waning忽略掉就可以了***）
- 在`Build Settings`里找到`Strip Swift Symbols`，设置为YES；此时移除相应Target中的所有的Swift符号，这个选项是默认打开的。（***补充一点：Swift ABI稳定之前，Swift标准库是会打进目标文件的，想要同时移除Swift标准库里面的符号的话需要在发布选项中勾选`Strip Swift symbols`***）
- 在`Build Settings`里找到`Generate Debug Symbols`，设置为NO来禁用Debug符号生成；当值为YES时，APP crash会跳进framework源码，泄露了framework所有源代码，很不安全。（***当为YES时，可以通过`Level of Debug Symbols`来控制生成符号的级别***）
- 在`Build Settings`里找到`Debug Information Format`，设置为`DWARF`。这一项是设置是否将调试信息加入到可执行文件中。改为`DWARF`后，如果程序崩溃，将无法输出崩溃位置对应的函数堆栈，但由于Debug模式下可以在XCode中查看调试信息，所以改为`DWARF`影响并不大。不过，既然这个设置叫做`Debug Information Format`，所以首先得有调试信息。如果此时`Generate Debug Symbols`选择的是NO的话，是没法产出dSYM文件的。dSYM文件的生成，是在Strip等命令执行之前。所以无论`Strip Linked Product`是否开启，生成的dSYM文件都不会受影响。注意，***静态库是无法生成dSYM文件的，即使设置为`DWARF with dSYM File`***，构建过程中依然不会有生成dSYM文件的步骤。（***需要注意的是，将`Debug Information Format`改为`DWARF`之后，会导致在Debug窗口无法查看相关类类型的成员变量的值。当需要查看这些值时，可以将`Debug Information Format`改回`DWARF with dSYM file`，clean（必须）之后重新编译即可***）

> Level of Debug Symbols有3个值，分别是：
> 
> - used：只引用符号
> - full：所有符号
> - default：使用编译器默认值
> 
> ---
> 
> Strip Style表示的是我们需要去除的符号的类型的选项，其分为三个选择项:
> 
> - All Symbols：去除所有符号，一般是在主工程中开启。
> - Non-Global Symbols：去除一些非全局的Symbol（保留全局符号，Debug Symbols同样会被去除），链接时会被重定向的那些符号不会被去除，此选项是静态库/动态库的建议选项。
> - Debug Symbols：去除调试符号，去除之后将无法断点调试。
> 
> ---
> 
> iOS的调试符号是DWARF格式，相关概念如下：
> 
> - Mach-O: 可执行文件，源文件编译链接的结果。包含映射调试信息(对象文件)具体存储位置的Debug Map。
> - DWARF：一种通用的调试文件格式，支持源码级别的调试，调试信息存在于对象文件中，一般都比较大。Xcode调试模式下一般都是使用DWARF来进行符号化的。
> - dSYM：独立的符号表文件，主要用来做发布产品的崩溃符号化。dSYM 是一个压缩包，里面包含了DWARF文件。使用Xcode编译打包的时候会先通过可执行文件的Debug Map获取到所有对象文件的位置，然后使用dsymutil来将对象文件中的DWARF提取出来生成dSYM文件。

# **通用配置**

---

> 以上的配置只是针对SDK的配置，这里的配置是针对所有项目的配置（即SDK和APP项目）。
> 
- 设置好最低支持的iOS系统版本。
- 在`Build Settings`里找到`Precompile Prefix Header`设置为YES，提升编译速度。
- 选择`File-New-File-PCH File`，名字不能用默认名，一般都通过下划线加上项目名来命名，如PrefixHeader_SDK.pch。然后选择Targets，点击创建。然后去`Build Settings`里找到`Prefix Header`，设置为`$(PROJECT_DIR)/$(PRODUCT_NAME)/PrefixHeader_SDK.pch`（***最后的是文件名，改为你实际上的文件名就行了***）。

> 附上我pch文件的一些定义。
> 

```objectivec
#ifndef PrefixHeader_SDK_pch
#define PrefixHeader_SDK_pch

#ifdef __OBJC__ // 防止非OC文件包含OC的头文件而引发的编译报错
// OC相关的应该在这里包含
// #import "Tools.h"  // OC的工具类

// frame相关
#define kScreenHeight [[UIScreen mainScreen] bounds].size.height // 物理屏幕高度
#define kScreenWidth [[UIScreen mainScreen] bounds].size.width   // 物理屏幕宽度
#define kIsFullScreen ((([[[UIDevice currentDevice] systemVersion] floatValue] >= 11.0f) && ([[[[UIApplication sharedApplication] delegate] window] safeAreaInsets].bottom > 0.0))? YES : NO) // 判断是否全面屏
#define kIsiPhoneX CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) // 判断是否是iPhone X
#define kStatusBarHeight (kIsFullScreen ? 44.f : 20.f)       // 状态栏高度
#define kNavigationBarHeight (kIsFullScreen ? 88.f : 64.f)   // 导航栏高度
#define kTabBarHeight (kIsFullScreen? (49.f + 34.f) : 49.f)  // tabBar高度
#define kHomeIndicatorHeight (kIsFullScreen ? 34.f : 0.f)    // home指示器高度

// 颜色相关
#define kRGBA255(R, G, B, A) [UIColor colorWithRed:((R) / 255.0f) green:((G) / 255.0f) blue:((B) / 255.0f) alpha:(A)]
#define kRGBA(R, G, B, A) [UIColor colorWithRed:(R) green:(G) blue:(B) alpha:(A)]
#define kRGB255(R, G, B) [UIColor colorWithRed:((R) / 255.0f) green:((G) / 255.0f) blue:((B) / 255.0f) alpha:1.0f]
#define kRGB(R, G, B) [UIColor colorWithRed:(R) green:(G) blue:(B) alpha:1.0f]
#define kUIColorFromRGB(rgbValue) [UIColor colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 green:((float)((rgbValue & 0xFF00) >> 8))/255.0 blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0] // 0xf8ff格式（16进制格式）
#define kUIColorFronHSB(h, s, b) [UIColor colorWithHue:h saturation:s brightness:b alpha:1.0f]
#define kUIColorFronHSBA(h, s, b, a) [UIColor colorWithHue:h saturation:s brightness:b alpha:a]

// 定义通用颜色
#define kBlackColor         [UIColor blackColor]
#define kDarkGrayColor      [UIColor darkGrayColor]
#define kLightGrayColor     [UIColor lightGrayColor]
#define kWhiteColor         [UIColor whiteColor]
#define kGrayColor          [UIColor grayColor]
#define kRedColor           [UIColor redColor]
#define kGreenColor         [UIColor greenColor]
#define kBlueColor          [UIColor blueColor]
#define kCyanColor          [UIColor cyanColor]
#define kYellowColor        [UIColor yellowColor]
#define kMagentaColor       [UIColor magentaColor]
#define kOrangeColor        [UIColor orangeColor]
#define kPurpleColor        [UIColor purpleColor]
#define kClearColor         [UIColor clearColor]

// 路径相关
#define kDocumentPath [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject] // 获取沙盒Document路径
#define kTempPath NSTemporaryDirectory() // 获取沙盒temp路径
#define kCachePath [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject] // 获取沙盒Cache路径

#define kWeakSelf(x) __weak typeof(self) x = self     // 弱引用
#define kStrongSelf(x) __strong typeof(self) x = self // 强引用

#ifdef DEBUG
#define kLog(...) NSLog(__VA_ARGS__)
#else#define kLog(...)
#endif#endif#endif /* PrefixHeader_pch */
```

### **可选配置**

---

---

> 这些配置不是必须的，可以根据实际需求来进行配置（SDK和APP项目）。
> 
- 在`Build Settings`里找到`Enable Bitcode`设置为NO来关闭该功能。Bitcode有一致性要求，这就意味着工程开启Bitcode之后必须要求所有打进Bundle的Binary都需要支持Bitcode，也就是说我们依赖的静态库都要含有Bitcode的，不然会报错。（***如果你要开启Bitcode，开启之后需要特别注意崩溃定位的问题：由于最终的可执行文件是Apple自动生成的，同时产生新的符号表文件，所以我们使用原本打包生成的dSYM符号化文件是无法完成符号化的。所以我们需要在上传至App Store时需要勾选`Include app symbols for your application to receive symboilcated crash logs from Apple`，勾选之后Apple会给我们生成dSYM，然后就可以在`Xcode -> Organizer`或者`iTunes Connect`中下载对应的dSYM来进行符号化了***）
- 允许项目进行HTTP请求：打开`info.plist`文件，增加`App Transport Security Settings`（字典类型），然后在该item下增加`Allow Arbitrary Loads`（布尔类型）并设为YES。
- 配置隐私权限，且必须要写上权限的用途，不能写***“是否允许APP使用XXX权限”***，更不能留空，不然会出问题，APP也无法上线。

# **注意事项**

- 第一次编译需要选中SDKBuildScript来运行，编译成功后framework就会拷贝到之前配置的目录里（我这里把路径配置到了项目里），然后自己手动添加到测试和demo里去。
- 当测试和demo项目添加了库以后，修改SDK的代码不需要手动去运行SDKBuildScript了，只需要运行测试或者demo项目就能享受SDK修改后带来的变化了。不过需要注意的是，如果SDK新增或者删除某个类或声明，还是需要运行SDKBuildScript来重新生成包（***运行测试或者demo项目时，Xcode并不会自动运行SDKBuildScript来生成新的SDK包，只会使用缓存，所以并不会生成新的资源列表，也就无法改变原有头文件包含和声明。但会根据缓存和改变的内容生成新的库文件。总的来说，虽然库的资源文件不会改变，但它的二进制已经改变，所以运行时能运行最新的内容***）。不过该特性只适用于模拟器，在真机上必须要用脚本重新编译来生成一个新的SDK包。
- 如果确实需要每次运行项目前都让Xcode自动运行SDKBuildScript，这也是有办法的：***在Xcode左上角选择target那里，选择`Edit Scheme`，选中你要自动运行脚本的target，选中`Build`，点击`+`号，选中脚本的target，点击Add完成添加，最后把脚本的target拖到最顶部即可。原理是从顶部向下，target的顺序就是编译的顺序。***（因为每次运行项目之前都会运行脚本打包，等待时间会比较长，特别是大项目，所以并不建议如此做。）
- 在使用每次编译都运行脚本的功能时，在模拟器是没问题的，但是在真机上你会发现本次运行的SDK代码是上次编译SDK的代码，这是因为Xcode为了加快编译速度，默认启用了并行编译，所以虽然是脚本先运行，但是因为是多线程编译，所以APP编译完成的时候，脚本还没编译完呢，也就使用了上次的SDK了。要解决这个问题很简单，***在`Edit Scheme`那里，选中自动运行脚本的target，把`Parallelize Build`的勾去掉即可。***不过因为会变成串行编译，所以编译会变慢。
    
    ![https://upload-images.jianshu.io/upload_images/14154456-c66ef769fa2ef5d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/896](https://upload-images.jianshu.io/upload_images/14154456-c66ef769fa2ef5d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/896)
    
    自动运行脚本设置.png
    

> 打包出来的framework库使用方法：
> 
> - 点击项目名称，选中你想配置的TARGETS，选择`General`。
> - 如果你的framework是静态库，到`Linked Framework and Librarles`里，点击“+”号，然后点击`Add Other..`，找到库文件添加进去就OK了。
> - 如果你的framework是动态库，到`Embedded Binarles`里，点击“+”号，然后点击`Add Other..`，找到库文件添加进去就OK了（***不需要导入到`Linked Framework and Librarles`里，Xcode会自动帮我们导入好***）。
> - 以上是Xcode11之前的导入方法，从Xcode11开始，静态库和动态库的导入地方为同一个，即`Frameworks, Librarles,and Embedded Content`，不同的是静态库的`Embed`为`Do Not Embed`，动态库的`Embed`为`Embed & Sign`。
> 
> ---
> 
> 当你的SDK项目使用了分类，别人在用你的库时需要在`Build Settings - Other Linker Flags`里面加入`-ObjC`参数，以下是这些参数的意义：
> 
> - ObjC：加了这个参数后，链接器就会把静态库中所有的Objective-C类和分类都加载到最后的可执行文件中，如果使用了分类就要加这个参数。
> - all_load：会让链接器把所有找到的目标文件都加载到可执行文件中，但是千万不要随便使用这个参数！假如你使用了不止一个静态库文件，然后又使用了这个参数，那么你很有可能会遇到`ld: duplicate symbol`错误，因为不同的库文件里面可能会有相同的目标文件，所以建议在遇到-ObjC失效的情况下使用-force_load参数。
> - force_load：所做的事情跟-all_load其实是一样的，但是-force_load需要指定要进行全部加载的库文件的路径，这样的话，你就只是完全加载了一个库文件，不影响其余库文件的按需加载。

---

***项目配置也完事了，接下来介绍下SDK开发时的一些代码风格***

# **SDK编写的代码风格**

> 别人刚使用你的SDK时，如果看到乱乱的代码，就会心生畏怯，就会觉得你的SDK可能很复杂很难用，所以，怎么样才写好一个SDK是很重要的。现在来介绍编写SDK需要注意的地方：
> 
- 类名、静态变量、全局变量、宏定义、分类名、枚举、结构体等都要有前缀，前缀一般取项目名的缩写，并且整个SDK只用一个前缀。
- 如果你在封装SDK时使用了第三方开源库，需在说明文件声明，以免开发者重复导入引起冲突。（***或者自己修改前缀来解决冲突问题，括类名、delegate协议、常量名、宏定义、枚举、结构体，尤其需要注意Category的方法名要修改***）
- 文件组织清晰明了。
- 方法命名可以参考系统的命名方式，特别是代理，一般都是以类名起头，并且第一个参数是把自己传出去。
- 必要时写上一定的注释。但注释应该简单明了，不应该长篇大论，不然一个很简单的东西，因为文字太多，别人也以为很复杂。
- 容器类型需要注明容器里装的什么类型，如：NSArray<NSString *> *。
- 当外界需要使用到你内部定义的字符串时，不要用宏定义或者直接让开发者去书写这个字符串，应该用`extern NSString * const`去声明该字符串，如我在一个方法里返回一个字典，外界需要用字符串去取值的时候，就需要用到`extern NSString * const`了。
- 不要使用XIB等，应该使用纯代码。
- 最好能使用自动布局而不是frame，如果要使用frame，则应该对某些因素可能影响到view，需要frame改变的情况做好处理（比如键盘升高等）。

---

---

---

***这里只介绍SDK独占的一些注意事项，别的请期待我之后编写的代码风格文章，里面详细介绍了APP编写代码的风格***

# **使用git子模块来管理Workspace**

> 每个项目都有一个git来管理，他们是独立的，因为各个项目都有了git，所以在workspace里的git对workspace的各个项目是不起作用的。但是，我们既想每个项目的git都独立，而在workspace又能统一去管理，那怎么办？事实上git已经给我们提供了该功能，那就是子模块（Submodule）。
> 

之前我们并没有为workspace创建git，所以我们首先要创建git。添加子模块的命令为`git submodule add <url> <path>`，其中url可以是远程地址和本地地址，本地地址要用绝对对路径，path则是该子模块存储的目录路径（使用相对路径）。操作如下：打开`终端`并`cd`到workspace所在的文件夹。

```bash
$ git init
$ git submodule add /Users/cer/Desktop/iOS/SDK ./SDK
$ git submodule add /Users/cer/Desktop/iOS/SDKTest ./SDKTest
$ git submodule add /Users/cer/Desktop/iOS/SDKDemo ./SDKDemo
```

如此就把3个项目成功添加为workspace的子模块了。

> 注意！！！创建完子项目之前workspace不能创建git，不然在创建子项目的时候，Xcode无法勾选创建git的选项。在子项目没有git时，workspace的git会记录这些文件，导致后面为子项目创建git后可能无法把子项目添加为子模块。如果你已经这么做了，解决的方法是：把workspace的git和子模块文件都删了，没有git的子项目也删了，然后重新创建带git的子项目，再是创建workspace的git，最后把子项目添加为子模块。
> 
> 
> ---
> 
> ***子模块虽然能添加本地路径，但最好不要这样子做。因为提交的时候子模块只能提交到它自己的远程地址，如果没有，当你在别的地方拉取的时候会就发生子模块缺失。如果你已经这么做了，解决的方法是：为所有子模块增加远程地址，然后编辑`.gitmodules`文件，把里面的URL替换成相应的远程地址。如果使用SourceTree，那么还需要点击右上角的设置，然后点击`编辑配置文件...`，把里面的URL也替换成相应的远程地址。***
> 
> ---
> 
> 想git对子模块了解更多，可以参考以下文章：[Git Submodule管理项目子模块](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fnicksheng%2Fp%2F6201711.html)  [【Git】子模块：一个仓库包含另一个仓库](https://www.jianshu.com/p/491609b1c426)  [git中submodule子模块的添加、使用和删除](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fguotianqing%2Farticle%2Fdetails%2F82391665)
> 

---

> demo地址在这里：[iOS_SDK](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FOCer%2FiOS_SDK)
> 

---

***关于SDK开发的相关知识就到这了，各位有啥有不懂可以到我的QQ群（139322447）找我***

---

---

记录下踩到的坑：在我的理解里，SDK设置了debug和release的区别，那么我编译出来的库就应该有所区别。比如，SDK的debug，把Generate Debug Symbols设置为YES让其产生符号，方便调试。然后在release下设置为NO，让其打包出来后不能调试，防止泄露源码。然后使用以下案例测试：

- SDK使用debug，Test项目使用debug -- 能进入断点和查看源码
- SDK使用release，Test项目使用debug -- 能进入断点和查看源码
- SDK使用debug，Test项目使用release -- 不能进入断点，但能点击进去源码
- SDK使用release，Test项目使用release -- 不能进入断点，但能点击进去源码

这结果令我很意外，我本以为与配置的SDK有关的，但事实上却与运行的项目有关。然后我想，这会不会与workspace有关呢，于是我新创建了一个独立的项目，然后分别测试debug和release包：

- SDK使用debug，Test项目使用debug -- 不能进入断点，也不能点击进去源码
- SDK使用release，Test项目使用debug -- 不能进入断点，也不能点击进去源码
- SDK使用debug，Test项目使用release -- 不能进入断点，也不能点击进去源码
- SDK使用release，Test项目使用release -- 不能进入断点，也不能点击进去源码

噗，这。。。着实又让我震惊一次。产生的debug包那么大，但好像没有用？当然，workspace也是与SDK的配置有关系的，如果debug的Generate Debug Symbols不设置为YES，那么无论如何都不会响应断点的。但对独立项目不影响，不管是YES还是NO，都不响应断点。

附我SDK项目的配置：

| 配置名称 | debug | release | 备注 |
| --- | --- | --- | --- |
| Build Active Architecture Only | NO | NO | 为了方便调试，debug也需要生成全平台 |
| Dead Code Stripping | YES | YES | 过滤没用到的代码，缩小库的体积 |
| Debug Information Level | Compiler default | Line tables only | release时不让查看局部变量的值 |
| Link With Standard Libraries | YES | YES | 如果不设置为YES，可能会报错 |
| Strip Style | Debug Symbols | Non-Global Symbols | debug下保留调试符号，release下最大程度删除符号 |
| Strip Linked Product | NO | YES | debug时能中断断点，release时不让中断断点 |
| Deployment Postprocessing | NO | YES | debug时不打开Strip Linked Product，release时打开 |
| Strip Debug Symbols During Copy | NO | YES | debug时允许调试第三方库，release时不需要 |
| Strip Swift Symbols | YES | YES | 移除Target中所有的Swift符号 |
| Generate Debug Symbols | YES | NO | debug时允许调试，release不允许调试 |
| Debug Information Format | DWARF | DWARF | debug也是能调试的，所以影响不大 |
| Enable Bitcode | NO | NO | 防止第三方库不支持而报错 |

> 目前项目比较忙，还没空细究这个，待有时间想起来了再回头研究下。当然，如果你们懂，也欢迎告诉我，大家一起共同进步。
> 

---

好了，我来填坑了。上面的测试都是通过模拟器去测试的，然后我猜可能是因为苹果为了让开发者方便调试，所以在workspace才会以当前的工程环境为主，然后我用真机也进行了测试。workspace：

- SDK是release，测试是debug -- 不会泄露源码 但能点击进去 响应断点但无法进入 无法用符号捕获
- SDK是debug，测试是debug -- 不会泄露源码 但能点击进去 响应断点但无法进入 无法用符号捕获
- SDK是release，测试是release -- 不会泄露源码 但能点击进去 响应断点但无法进入 无法用符号捕获
- SDK是debug，测试是release -- 不会泄露源码 但能点击进去 响应断点但无法进入 无法用符号捕获

总结：在真机的workspace里，以SDK为主，和当前工程没关系。

独立工程：

- SDK是release，测试是debug -- 不会泄露源码 不能点击进去 响应断点但无法进入 无法用符号捕获
- SDK是debug，测试是debug -- 不会泄露源码 不能点击进去 响应断点但无法进入 无法用符号捕获
- SDK是release，测试是release -- 不会泄露源码 不能点击进去 响应断点但无法进入 无法用符号捕获
- SDK是debug，测试是release -- 不会泄露源码 不能点击进去 响应断点但无法进入 无法用符号捕获

总结：都不会响应断点和符号捕获，也不能进入源码，但如果有符号捕获，会在右边显示调用的符号。

---

最后的总结：在模拟器的workspace里，为了开发人员方便调试，SDK不管是什么版本，都会用当前项目的环境重新编译一次。而在其它的情况下，则是真实情况，均以SDK版本为主，SDK不会受到当前环境的影响。

iOS OC Swift Flutter开发群 139322447