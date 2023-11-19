# iOS手动编译并搭建FFmpeg - 掘金

[https://juejin.cn/post/6844903857097539591](https://juejin.cn/post/6844903857097539591)

## 需求

手动编译打开x264功能的FFmpeg并放入新建的项目中,可以编译成功.以便后续使用.

## 背景

移动端学习音视频开发,FFmpeg可以说是必学的框架,FFmpeg在linux平台下开发，但它同样也可以在其他操作系统环境中编译运行，包括Windows、Mac OS X等。FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序，它包括了目前领先的音/视频编码库libavcodec。

FFmpeg有非常强大的功能，包括视频采集功能、视频格式转换、视频抓图、给视频加水印等。同时还支持以RTP方式将视频流传送给支持RTSP的流媒体服务器，支持直播应用。

## 安装方式

可以通过如下两种方式安装ffmpeg

- 1.下载iOS版本[ffmpeg静态库](https://link.juejin.cn/?target=https%3A%2F%2Fsourceforge.net%2Fprojects%2Fffmpeg-ios%2Ffiles%2Fffmpeg-ios-master.tar.bz2%2Fdownload%3Fuse_mirror%3Dversaweb): 即不用手动编译,我们只需要下载得到头文件及.a库文件.
- 2.手动编译: 下载源码, 可以在更改一些flag或源码后再编译脚本,较为灵活.

如何选择

- 如果仅仅是想简单直接使用ffmeg可以下载一个稳定版本的静态库, 建议用第一种方式.
- 如果需要在iOS项目中自定义使用ffmpeg, 以及修改一些ffmpeg中的源码以适应项目,使用第二种方式.

## 阅读前提:

- 音视频基础
- 基本终端命令行
- FFmpeg基础

### 注意: 文本仅仅编译真机使用的arm64环境,所以模拟器下无法运行项目,如需添加其他架构自行更改两个脚本文件.

本文需要的所有安装包均在以下链接中,因为下文中官网地址是外网,没有VPN的同学可能下载会很慢.故本人整理好最新所有版本放在下面的百度云盘中.

```
复制代码链接: https://pan.baidu.com/s/1nYBQUi8drEpkjmwlIpbetQ 提取码: ercw

```

### 1.下载[gas-preprocessor](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flibav%2Fgas-preprocessor)

此文件是编译FFmpeg必备的脚本文件,使用如下命令将其拷贝进bin下

```
复制代码cp -f /xxx/gas-preprocessor.pl /usr/local/bin/

```

### 2.安装[yasm](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fyasm%2Fyasm)

`Yasm`是一个完全重写的`NASM`汇编并且支持x86和AMD64指令集.

```
复制代码brew install yasm

```

### 3. 下载[x264-iOS编译脚本](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fkewlbear%2Fx264-ios)及[源码](https://link.juejin.cn/?target=http%3A%2F%2Fdownload.videolan.org%2Fpub%2Fvideolan%2Fx264%2Fsnapshots%2F)

- 
    
    下载x264编译脚本解压后如下
    
    ![iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddf2cef0aetplv-t2oaga2asx-jj-mark3024000q75.png](iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddf2cef0aetplv-t2oaga2asx-jj-mark3024000q75.png)
    
- 
    
    然后下载最新版源码解压后如下
    
    ![iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddd727c967tplv-t2oaga2asx-jj-mark3024000q75.png](iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddd727c967tplv-t2oaga2asx-jj-mark3024000q75.png)
    
- 
    
    将源码文件夹改名为`x264`并放至编译脚本文件夹(`x264-ios-master`)下
    
    ![iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddee331707tplv-t2oaga2asx-jj-mark3024000q75.png](iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddee331707tplv-t2oaga2asx-jj-mark3024000q75.png)
    

> 
> 
> 
> 因为编译脚本中指定文件目录为`x264`,所以需要改名,也可以改编译脚本 最好手动强制设置下GCC位置,否则可能会报错,然后执行命令:
> 

```
复制代码sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
./build-x264.sh

```

- 注意

`No working C compiler found` : 新版mac直接编译会报错,原因是gcc一些必要工具找不到.可能是xcode位置换了的原因,因为我们主要使用真机调试,所以我们在编译脚本中只保留arm64即可,如下修改后,可以直接编译通过. 不过像模拟器这样的设置是无法使用x264的,因为我们相当于仅编译了真机所需的库.

```
复制代码ARCHS="arm64 x86_64 i386 armv7 armv7s"
改为如下
ARCHS="arm64"

```

执行完成之后可以看到生成了x264-iOS文件夹

### 4. 下载[FFmpeg-iOS编译脚本](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fkewlbear%2FFFmpeg-iOS-build-script)及[源码](https://link.juejin.cn/?target=https%3A%2F%2Fsourceforge.net%2Fprojects%2Fffmpeg-ios%2Ffiles%2Fffmpeg-ios-master.tar.bz2%2Fdownload%3Fuse_mirror%3Dversaweb)

> 
> 
> 
> 注意: 在这里可以仅下载FFmpeg-iOS编译脚本,不用下载源码,执行脚本会自动下载源码,如果不想每次自动下载,可以手动下载源码,稍微修改下FFmpeg编译脚本即可.这里不做过多说明.
> 

修改脚本(`build-ffmpeg.sh`文件)内容

![iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddd5957045tplv-t2oaga2asx-jj-mark3024000q75.png](iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddd5957045tplv-t2oaga2asx-jj-mark3024000q75.png)

x264编译好的文件夹必须在当前目录并且命名为fat-x264,所以我们第3步编译后生成的x264-iOS文件夹改名成fat-264，放在FFmpeg-iOS-build-script这个文件夹中。目录结构如下:

![iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddc245b9c1tplv-t2oaga2asx-jj-mark3024000q75.png](iOS%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91%E5%B9%B6%E6%90%AD%E5%BB%BAFFmpeg%20-%20%E6%8E%98%E9%87%91%207f624f36a85f477b904e777bbac2bca9/16b095ddc245b9c1tplv-t2oaga2asx-jj-mark3024000q75.png)

并修改如下内容

```
复制代码CFLAGS="$CFLAGS -mios-version-min=$DEPLOYMENT_TARGET -fembed-bitcode"
修改为
CFLAGS="$CFLAGS -mios-version-min=$DEPLOYMENT_TARGET"

```

因为我们在上一步中仅仅编译arm64的x264,所以这里我们也仅仅编译arm64的FFmpeg.稍微修改脚本文件即可.

```
复制代码ARCHS="arm64 armv7 x86_64 i386"
修改为
ARCHS="arm64"

```

- 如果要使用avutil.h相关功能,需要更改脚本

注意: FFmpeg框架中的一个结构体命名为"AVMediaType"与苹果自带框架产生冲突,所以,我们必须修改编译脚本,使用"FFmpegAVMediaType"带替换"AVMediaType".这里需要在脚本文件中添加如下命令行,即将`AVMediaType`替换为`FFmpegAVMediaType`. 注意: $SOURCE为ffmpeg的根目录.

```
复制代码grep -rl AVMediaType ./$SOURCE | xargs sed -i .bak s@AVMediaType@FFmpegAVMediaType@g

```

编译脚本文件

```
复制代码./build-ffmpeg.sh

```

### 5. Xcode编译

- Xcode新建iOS项目

新建一个iOS工程,然后将`ViewController.m`重命名为`ViewController.mm`,因为FFmpeg中涉及C,C++混编,所以需要做此操作.

- 将FFmpeg编译好的头文件与库拉进项目中,并在主控制器测试代码,此时会有一大堆错误抛出,下面逐个解决
- 添加依赖库

> 
> 
> 
> 注意: FFmpeg源码中调用了一些iOS系统的库,所以,我们必须将依赖的库导入项目中.
> 
- 在Build Setting中禁止Bitcode
- 在Build Setting中设置头文件与库的位置

这里特别要注意,因为在大多数项目中以及FFmpeg自身源代码中,都是以以下格式来导入的头文件

## 参考文章

- [FFmpeg编译](https://link.juejin.cn/?target=http%3A%2F%2Fwww.iosxxx.com%2Fblog%2F2017-07-17-iOS%25E7%25BC%2596%25E8%25A7%25A3%25E7%25A0%2581%25E5%2585%25A5%25E9%2597%25A8%25E7%25AF%2587-FFMPEG%25E7%258E%25AF%25E5%25A2%2583%25E6%2590%25AD%25E5%25BB%25BA.html)