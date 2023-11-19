# 卡住在 Cloning spec repo 'cocoapods' from 'https://github.com/CocoaPods/Specs.git'

# 方法1

## **发生时**

- 在第一次安装 cocoapods 后，使用 pod install 或者 pod update 的使用会更新 repo 仓库
- 有时候虽然使用了 `install --no-repo-update` 但是由于 Podfile 中有些需要 pod 的仓库的确不在本地的 repo 列表中，这样仍然需要更新本地的 repo 仓库

## **不要慌**

因为在终端看到 `Cloning spec repo 'cocoapods' from 'https://github.com/CocoaPods/Specs.git` 接着无尽的等待，心中不免急躁得经常 `control + c` 结束掉进程

## **查看进度不迷茫**

1、可以如下办法查看进度mac上找 活动监视器里的网络列表里找git-remote-https,这条即是当前的下载进度,可看到缓缓的在变动下载数据

2、何时下载完捏,这里可利用[github的api查看下项目大小](https://api.github.com/repos/CocoaPods/Specs)，这是github的api,返回一串json,是项目的相关信息.里面找size,即是项目大小

# 方法二 更换源

**[王加水](https://www.jianshu.com/u/654001fef0c6)**关注

### **1.概况**

一般是第一次安装cocoapod后, 使用pod install 或者 pod update等时候.终端显示

`Cloning spec repo 'cocoapods' from 'https://github.com/CocoaPods/Specs.git'`

就不动了,也没个提示啥的,很迷

### **2.原因**

一般来说其实是正在下载东西从github上,但是下载速度很慢

1. 可以如下办法查看进度mac上找 **活动监视器**里的**网络**列表里找**git-remote-https**,这条即是当前的下载进度,可看到缓缓的在变动下载数据
2. 何时下载完捏,这里可利用github的api查看下项目大小[https://api.github.com/repos/CocoaPods/Specs](https://links.jianshu.com/go?to=https%3A%2F%2Fapi.github.com%2Frepos%2FCocoaPods%2FSpecs)这是github的api,返回一串json,是项目的相关信息.里面找**size**,即是项目大小,我看得750M多, so 耐心些等吧

### **3.解决办法**

更换国内的镜像, 清华的不错

```objectivec
$ cd ~/.cocoapods/repos 
$ pod repo remove master
$ git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git master
```

然后记得去自己项目podfile里把source换了

`source 'https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git'`

再重新pod update但可能存在不最新的问题

# 自己的方法

直接参考方法二，手动设置成GitHub的源，这样设置过程中可以显示进度了

```objectivec
$ cd ~/.cocoapods/repos 
$ pod repo remove master
$ git clone https://github.com/CocoaPods/Specs.git
```