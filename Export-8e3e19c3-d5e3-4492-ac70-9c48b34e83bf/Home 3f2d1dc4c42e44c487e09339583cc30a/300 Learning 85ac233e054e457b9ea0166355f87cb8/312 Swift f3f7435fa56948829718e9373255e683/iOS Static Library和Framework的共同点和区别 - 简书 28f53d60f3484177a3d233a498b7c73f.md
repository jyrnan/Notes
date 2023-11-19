# iOS Static Library和Framework的共同点和区别 - 简书

参考文章：- [http://my.oschina.net/leejan97/blog/284193](https://links.jianshu.com/go?to=http%3A%2F%2Fmy.oschina.net%2Fleejan97%2Fblog%2F284193)

- [https://blog.csdn.net/jingcheng345413/article/details/54969324](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fjingcheng345413%2Farticle%2Fdetails%2F54969324)

- [https://www.jianshu.com/p/a04310199d7b](https://www.jianshu.com/p/a04310199d7b)

- [http://www.cocoachina.com/articles/23082](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cocoachina.com%2Farticles%2F23082)

### 一、共同点：

两者其实都是静态库。

### 二、区别

1.承载的内容范畴：

(1)Static Library的产出物只是一个.a文件，为二进制执行文件。分享给别人的时候，头文件、静态资源文件需要另外提供。

(2)Framework为一站式分享方案，其实是一个文件夹，其中包含代码签名、头文件、二进制执行文件、静态资源文件等。

2.头文件搜索路径的区别：StaticLibrary需要设置头文件搜索路径，Framework不需要。

3.当存在对外部代码库依赖的时候

(1)StaticLibrary：能够只引用外部库的头文件，调用外部库的公开方法，而不引入其库实现，实现与引用库的分离部署。

(2)Framework：要引用一个外部库，就必须要将此外部库的实现放入Framework内编译才可以。如果要想达到StaticLibrary的效果，可以使用运行时方式调用。

4.运行环境(对3的理解升级)

(1)StaticLibrary：共享其运行环境，假如其运行环境中包换库中同一个类，会发生代码冲突，必须剥离其中一方的此类，然后共享此类。

(2)Framework：与其运行环境隔离，假如其运行环境中包换库中同一个类，不会发生冲突，同名的两个类会在各自的环境中独立运行，互不干扰，哪怕是单例类。

### 综合，如何选择使用Framework还是StaticLibrary

(1)假如不想在同一个App中包含多份三方库(减小包大小)，可以使用StaticLibrary，库本身和App共享第三方库。但是产出物的结构可能会比较乱。

(2)假如不想考虑和App的代码冲突问题，库本身独立使用需要的库，想提供比较好的库结构，可以使用Framework。但是假如库本身和App都使用了同一个三方库，会存在两份三方库，会增加包大小。

## 三、创建Static Library

1、创建Static Library，点击File--> New --> Target（如图）

屏幕快照 2020-10-06 下午6.07.42.png

2.创建了静态库Static Library之后，Xcode自动为我们创建了XXX.h/.m文件

3.编译项目，生成对应的静态库.a文件，编译的时候，记得选上项目（当然你的是什么名字的，就选对应的就好），然后用模拟器环境和真机环境都编译一下，红色的libXXXX.a文件就变成黑色了

屏幕快照 2020-10-06 下午6.31.39.png

4、合并静态库

针对真机和模拟器的静态库文件只能在一个平台下面使用，好在我们可以将真机和模拟器上面的静态库文件合并成一个在真机和模拟器都可以使用的静态库文件，通过在终端输入命令即可完成该目的，

完整的命令：

```
lipo -create /Users/doudou/Library/Developer/Xcode/vedData/JJPickView-eqviycrqcwweiretrodyzdijhulx/Build/Products/Debug-iphoneos/libXXX.a /Users/doudou/Library/Developer/Xcode/DerivedData/JJPickView-eqviycrqcwweiretrodyzdijhulx/Build/Products/Debug-iphonesimulator/libXXX.a -output /Users/doudou/Desktop/libXXX.a

```

这个是自己的文件路径，改成自己相应的路径

5、使用静态库文件

这时候我们就可以使用自己创建、编译生成的静态库文件了，将相关xxx.h文件和桌面上面的libXXX.a文件拖到想要使用的项目中

## 四、创建framework

1. 打开Xcode，新建工程。
    
    选择 create framework & library
    
    选择 framework
    
2. 创建我们所需要的文件类,比如继承与NSobject 的testH 类
3. 更改参数
    
    接下来对我们的这个.framework静态库进行一些简单的设置。
    

（1）首先是Dead Code Stripping设置为NO，网上对此项的解释如下，大致意思是如果开启此项就会对代码中的”dead”、”unreachable”的代码过滤，不过这个开关是否关闭，似乎没有多大影响，不过为了完整还原framework中的代码，将此项关闭也未曾不可。

（2）然后将Link With Standard Libraries关闭，我想可能是为了避免重复链接

（3）最后将Mach-O Type设为Static Library，framework可以是动态库也可以是静态库，对于系统的framework是动态库，而用户制作的framework只能是静态库。

(4)还要记得把要公开的类添加到我们的FrameWorkTest.h中

1. 打包 Framework

### 使用 lipo 命令

1. 首先要支持多个架构，设置路径 Build Settings -> Build Active -> NO 支持多个架构的静态库。然后选择模拟器和Build Only Device，分别编译一次。 2.通过终端命令将两个framework合为一个模拟器和真机都可使用的framework。

```
lipo -create  /Users/chenjiazhen/Library/Developer/Xcode/DerivedData/CharTest-dxlrsuvcbsfyohedbbfitrvbqygx/Build/Products/Debug-iphoneos/CharTest.framework/CharTest /Users/chenjiazhen/Library/Developer/Xcode/DerivedData/CharTest-dxlrsuvcbsfyohedbbfitrvbqygx/Build/Products/Debug-iphonesimulator/CharTest.framework/CharTest -output /Users/chenjiazhen/Desktop/CharTest

```

然后 会生成一个.lipo文件

此时 修改后缀.framework 即可生成我们需要的framework包

最后需要总结的：

1、在制作framework或者lib的时候，如果使用了category，则使用改FMWK的程序运行时会crash，此时需要在该工程中 other linker flags添加两个参数 -ObjC -all_load。（这点没有亲测）

2、带有图片资源的需要把图片打包成Bundle文件，和framework一起拷贝到相应的项目中。

3、公开的类中如果引用的private的类，打包以后对外会报错，找不到那个private的类，可以把那个private的.h放到（也没亲测）

4、namespace 冲突。静态库用了某第三方库，项目也用了同样的第三方库，在编译的时候就会有 duplicate symbol 错误，因为有两份同样的第三方库。解决办法就是把用到的第三方库加上自定义前缀，包括类名、delegate 协议、常量名，尤其需要注意 Category 的方法名要修改。