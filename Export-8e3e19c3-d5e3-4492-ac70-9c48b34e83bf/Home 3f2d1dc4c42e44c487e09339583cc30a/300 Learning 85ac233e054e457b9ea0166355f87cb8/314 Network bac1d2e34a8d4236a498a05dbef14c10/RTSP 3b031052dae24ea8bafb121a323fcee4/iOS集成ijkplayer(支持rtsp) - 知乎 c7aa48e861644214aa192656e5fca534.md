# iOS集成ijkplayer(支持rtsp) - 知乎

[https://zhuanlan.zhihu.com/p/605507735?utm_id=0](https://zhuanlan.zhihu.com/p/605507735?utm_id=0)

![iOS%E9%9B%86%E6%88%90ijkplayer(%E6%94%AF%E6%8C%81rtsp)%20-%20%E7%9F%A5%E4%B9%8E%20c7aa48e861644214aa192656e5fca534/v2-2d7ecc334dc2cf1dcb9d3a709606bfde_1440w.jpg](iOS%E9%9B%86%E6%88%90ijkplayer(%E6%94%AF%E6%8C%81rtsp)%20-%20%E7%9F%A5%E4%B9%8E%20c7aa48e861644214aa192656e5fca534/v2-2d7ecc334dc2cf1dcb9d3a709606bfde_1440w.jpg)

## **1、前置条件**

需要Mac上已经安装Homebrew、Git、yasm

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install git
brew install yasm
```

## **2、下载ijkplayer**

在任意位置创建一个文件夹，打开终端并cd到此文件夹，然后按照下述命令进行代码的clone。 [ijkplayer官方Github地址](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttps%253A%252F%252Fgithub.com%252FBilibili%252Fijkplayer)

```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
git checkout -B latest k0.8.8
```

## **3、支持RTSP**

如果不需要支持RTSP可跳过此部分，到“编译工程”部分

1. 修改ijkplayer-ios/config下的**module-lite.sh** ，生成新的 **module.sh**

```
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-protocol=rtp"
```

找上到述行，并修改为以下内容并保存（即允许rtp、打开rtsp音视频分离器）。

```
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtp"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtsp"
```

直接删除module.sh或cd到ijkplayer-ios/config目录后，使用以下命令删除

```
rm module.sh
```

删除后重新生成module.sh

```
ln -s module-lite.sh module.sh
```

1. 修改 ijkplayer-ios/ijkmedia/ijkplayer目录下的**ff_ffplay.c**

在ff_ffplay.c文件约第336行，将packet_queue_get_or_buffering方法修改为下述内容：

```
static int packet_queue_get_or_buffering(FFPlayer *ffp, PacketQueue *q, AVPacket *pkt, int *serial, int *finished)
{
//    assert(finished);
//    if (!ffp->packet_buffering)
//        return packet_queue_get(q, pkt, 1, serial);

    while (1) {
        int new_packet = packet_queue_get(q, pkt, 0, serial);
        if (new_packet < 0)
            return -1;
        else if (new_packet == 0) {
            if (q->is_buffer_indicator && !*finished)
                ffp_toggle_buffering(ffp, 1);
            new_packet = packet_queue_get(q, pkt, 1, serial);
            if (new_packet < 0)
                return -1;
        }

        if (*finished == *serial) {
            av_packet_unref(pkt);
            continue;
        }
        else
            break;
    }

    return 1;
}
```

> 【学习地址】：
> 
> 
> [FFmpeg/WebRTC/RTMP/NDK/Android音视频流媒体高级开发](https://link.zhihu.com/?target=https%3A//ke.qq.com/course/3202131%3FflowToken%3D1042495)
> 
> **【文章福利】：免费领取更多音视频学习资料包、大厂面试题、技术视频和学习路线图，资料包括（C/C++，Linux，FFmpeg webRTC rtmp hls rtsp ffplay srs 等等）有需要的可以点击**
> 
> [1079654574](https://link.zhihu.com/?target=https%3A//jq.qq.com/%3F_wv%3D1027%26k%3DmgauMC3k)
> 
> **加群领取哦~**
> 

## **4、编译工程**

回到ijkplayer-ios路径下，逐一执行下述命令。

```
./init-ios.sh
cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```

以上命令会创建iOS工程、下载并编译ffmpeg。 可能会遇到下述错误：

1. 报错"xcrun -sdk iphoneos clang is unable to create an executable file."，运行命令行 `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/` 后重新编译

## **5、生成Framework**

打开ijkplayer-ios/ios/IJKMediaPlayer目录下的**IJKMediaPlayer.xcodeproj**，选择IJKMediaFramework，将Scheme的Build方式改为Release后，分别将设备选为模拟器和真机各编译一次。此时可见工程下的Products目录中IJKMediaFramework.framework已存在，Show in Finder并cd到文件所在目录，执行以下命令合并两个Framework

```
lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
```

## **6、集成**

将**IJKMediaFramework.framework**拖拽到你的工程，并添加以下依赖库

| AudioToolbox.framework | AVFoundation.framework | CoreGraphics.framework |
| --- | --- | --- |
| CoreMedia.framework | CoreVideo.framework | libbz2.tbd |
| libz.tbd | MediaPlayer.framework | MobileCoreServices.framework |
| OpenGLES.framework | QuartzCore.framework | UIKit.framework |
| VideoToolbox.framework | libc++.tbd |  |

调用示例如下：

```
#import <IJKMediaFramework/IJKMediaFramework.h>
@interface ViewController ()
@property(nonatomic, strong) IJKFFMoviePlayerController *player;
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    NSURL *url = [NSURL URLWithString:@"rtsp://10.1.3.146/ch5"];
    IJKFFOptions *options = [IJKFFOptions optionsByDefault];
    self.player = [[IJKFFMoviePlayerController alloc] initWithContentURL:url withOptions:options];
    [self.view addSubview:self.player.view];
    [self.player.view mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.mas_equalTo(self.view);
    }];
    [self.player prepareToPlay];
    [self.player play];
}
@end
```