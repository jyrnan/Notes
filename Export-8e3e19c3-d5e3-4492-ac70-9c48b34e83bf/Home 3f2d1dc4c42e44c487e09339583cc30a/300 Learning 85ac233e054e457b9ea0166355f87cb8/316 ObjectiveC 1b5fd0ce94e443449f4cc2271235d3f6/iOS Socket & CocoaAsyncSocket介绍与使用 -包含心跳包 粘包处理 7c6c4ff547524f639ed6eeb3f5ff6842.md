# iOS | Socket & CocoaAsyncSocket介绍与使用 -包含心跳包 粘包处理

### 介绍

网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个套接字`socket`。
 实际上`socke`t是对`TCP/IP协议`的封装，它的出现只是使得程序员更方便地使用TCP/IP协议栈而已。socket本身并不是协议，它是应用层与TCP/IP协议族通信的中间软件抽象层，是一组调用接口（TCP/IP网络的API函数）

### TCP/IP

`TCP/IP（Transmission Control Protocol/Internet Protocol）`即传输控制协议/网间协议，是一个工业标准的协议集，它是为广域网（WANs）设计的。`UDP（User Data Protocol，用户数据报协议）`是与TCP相对应的协议，它是属于TCP/IP协议族中的一种。**TCP/IP协议族**包括运输层、网络层、链路层。`socket`是一个接口，在用户进程与TCP/IP协议之间充当中间人，完成TCP/IP协议的书写，关系如下图：

socket-tcp/ip.png

### Socket 工作流程

我们来看下 Socket 是如何进行工作以及连接的

working process.png

通过上面图片，可以明白，服务器端先初始化Socket，然后与端口绑定(bind)，对端口进行监听(listen)，调用accept阻塞，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器(connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。

了解到 Socket 的概念以及工作原理，socket 的原生接口本文不做详细使用说明了，本文主要对 `CocoaAsyncSocket`进行介绍与使用

### CocoaAsyncSocket介绍

**[CocoaAsyncSocket](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbiehanson%2FCocoaAsyncSocket)**为Mac和iOS提供了易于使用和功能强大的异步套接字库，主要包含两个类:

- **GCDAsyncSocket** 用GCD搭建的基于TCP/IP协议的socket网络库
- **GCDAsyncUdpSocket** 用GCD搭建的基于UDP/IP协议的socket网络库.

本文主要介绍 `GCDAsyncSocket`的使用，他是一个TCP库，建在Grand Central Dispatch上面的

### 客户端

1. 初始化

```objectivec
@property (nonatomic, strong) GCDAsyncSocket *clientSocket;

 self.clientSocket = [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_main_queue()];

```

需要`delegate`和`delegate_queue`才能使GCDAsyncSocket调用您的委托方法。提供`delegateQueue`是一个新概念。`delegateQueue`要求必须是一个串行队列，使得委托方法在delegateQueue队列中执行。

2.连接服务器

```objectivec
NSError *err = nil;
if (![self.clientSocket connectToHost:@"ip地址" onPort: "端口号" error:&err])  //异步！
{
    //  如果有错误，很可能是"已经连接"或"没有委托集"
    NSLog(@"I goofed: %@", err);
}

```

连接方法是异步的。这意味着当您调用connect方法时，它们会启动后台操作以连接到所需的主机/端口。

### 3.发送数据

```
//  发送数据
- (void)sendData:(NSData *)data{
     // -1表示超时时间无限大
     // tag:消息标记
     [self.clientSocket writeData:data withTimeout:-1 tag:0];
}

```

通过调用`writeData: withTimeout: tag:`方法，即可发送数据给服务器。

### 4.委托方法

4.1连接成功会是执行的委托方法

```objectivec
// socket连接成功会执行该方法
- (void)socket:(GCDAsyncSocket*)sock didConnectToHost:(NSString*)host port:(UInt16)port{
    NSLog(@"--连接成功--");
    [sock readDataWithTimeout:-1 tag:0];
}

```

4.2读取到了服务端数据委托方法

```
// 收到服务器发送的数据会执行该方法
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag{
    NSString *serverStr = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"服务端回包了--回包内容--%@---长度%lu",serverStr,(unsigned long)data.length);
    [sock readDataWithTimeout:-1 tag:0];
}

```

4.3断开连接委托方法

```
// 断开连接会调取该方法
- (void)socketDidDisconnect:(GCDAsyncSocket*)sock withError:(NSError*)err{
    NSLog(@"--断开连接--");
    //  sokect断开连接时,需要清空代理和客户端本身的socket.
    self.clientSocket.delegate = nil;
    self.clientSocket = nil;
}

```

以上几个委托方法，使我们比较使用比较频繁的委托代理方法。

### 5 心跳包

**心跳包**就是在`客户端`和`服务器`间定时通知对方自己状态的一个自己定义的命令，按照一定的时间间隔发送，类似于心跳
 用来判断对方（设备，进程或其它网元）是否正常运行，采用定时发送简单的通讯包，如果在指定时间段内未收到对方响应，则判断对方已经离线。

```objectivec
@property(nonatomic, strong) NSTimer *heartbeatTimer;

- (void)beginSendHeartbeat{
    // 创建心跳定制器
    self.heartbeatTimer = [NSTimer scheduledTimerWithTimeInterval:5.0 target:self selector:@selector(sendHeartbeat:) userInfo:nil repeats:YES];
    [[NSRunLoop mainRunLoop] addTimer:self.heartbeatTimer forMode:NSRunLoopCommonModes];
}

- (void)sendHeartbeat:(NSTimer *)timer {
    if (timer != nil) {
        char heartbeat[4] = {0xab,0xcd,0x00,0x00}; // 心跳字节，和服务器协商
        NSData *heartbeatData = [NSData dataWithBytes:&heartbeat length:sizeof(heartbeat)];
        [self.clientSocket writeData:heartbeatData withTimeout:-1 tag:0];
    }
}

```

### 服务端

`GCDAsyncSocket`还允许您创建服务器，并接受传入的连接; 服务端使用基本和客户类似，只不过需要开启端口进行监听客户端连接。

### 1. 初始化

```
@property(nonatomic, strong) GCDAsyncSocket *serverSocket;

// 初始化服务端socket
self.serverSocket = [[GCDAsyncSocket alloc]initWithDelegate:self delegateQueue:dispatch_get_main_queue()];

```

### 2.开放服务端的指定端口.

```
 //  开放服务端的指定端口.
    NSError *error = nil;
    BOOL result = [self.serverSocket acceptOnPort:port error:&error];
    if (result) {
        NSLog(@"端口开启成功,并监听客户端请求连接...");
    }else {
        NSLog(@"端口开启失...");
    }

```

### 3.发送数据给客户端

```
//  发送数据，和客户端使用一致
- (void)sendData:(NSData *)data{
     // -1表示超时时间无限大
     // tag:消息标记
    [self.serverSocket writeData:data withTimeout:-1 tag:0];
}

```

### 4.委托方法

4.1 监听到新的客户端socket连接委托

```
/* 存储所有连接的客户端 socket*/
@property(nonatomic, strong) NSMutableArray *arrayClient;

//  监听到新的客户端socket连接，会执行该方法
- (void)socket:(GCDAsyncSocket *)serveSock didAcceptNewSocket:(GCDAsyncSocket *) newSocket{

    NSLog(@"%@ IP: %@: %hu 客户端请求连接...",newSocket,newSocket.connectedHost,newSocket.localPort);
    // 将客户端socket保存起来
    if (![self.arrayClient containsObject:newSocket]) {
        [self.arrayClient addObject:newSocket];
    }
    [newSocket readDataWithTimeout:-1 tag:0];
}

```

监听到客户端连接，将客户端 socket 保存起来，因为服务器可能会收到很多客户端连接。

4.2 读取客户端数据

```
//  读取客户端发送的数据
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag {
  //  记录客户端心跳
    char heartbeat[4] = {0xab,0xcd,0x00,0x00}; // 心跳
    NSData *heartbeatData = [NSData dataWithBytes:&heartbeat length:sizeof(heartbeat)];
    if ([data isEqualToData:heartbeatData]) {
        NSLog(@"*************心跳**************");
        self.heartbeatDateDict[sock.connectedHost] = [NSDate date];
    }
    [sock readDataWithTimeout:-1 tag:0];
}

```

4.3断开连接

```
//  断开连接
- (void)socketDidDisconnect:(GCDAsyncSocket *)sock withError:(NSError *)err{
    NSLog(@"断开连接");
}

```

### 5.监听心跳包

```
// 子线程用于监听心跳包
@property(nonatomic, strong) NSThread *checkThread;
// 记录每个心跳缓存
@property (nonatomic, strong) NSMutableDictionary *heartbeatDateDict;
// 初始化子线程，并启动
self.checkThread = [[NSThread alloc]initWithTarget:sharedInstance selector:@selector(checkClientOnline) object:nil];
[self.checkThread start];

#pragma checkTimeThread

//  这里设置10检查一次 数组里所有的客户端socket 最后一次通讯时间,这样的话会有周期差（最多差10s），可以设置为1s检查一次，这样频率快
//  开启线程 启动runloop 循环检测客户端socket最新time
- (void)checkClientOnline{

    @autoreleasepool {
        [NSTimer scheduledTimerWithTimeInterval:10 target:self selector:@selector(repeatCheckClinetOnline) userInfo:nil repeats:YES];
        [[NSRunLoop currentRunLoop] run];
    }
}

//  移除 超过心跳时差的 client
- (void)repeatCheckClinetOnline{

    if (self.arrayClient.count == 0) {
        return;
    }
    NSDate *date = [NSDate date];
    for (GCDAsyncSocket *socket in self.arrayClient ) {
        if ([date timeIntervalSinceDate:self.heartbeatDateDict[socket.connectedHost]]>10) {
            [self.arrayClient removeObject:socket];
        }
    }
}

```

> 
> 
> 
> CocoaAsyncSocket的读写操作、粘包处理思路可以看下一篇： [Socket & CocoaAsyncSocket 读/写操作以及粘包处理](https://www.jianshu.com/p/1f87d8ba157d)
> 

CocoaAsyncSocket github地址:[https://github.com/robbiehanson/CocoaAsyncSocket](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbiehanson%2FCocoaAsyncSocket)

参考链接:

[https://github.com/robbiehanson/CocoaAsyncSocket/wiki/Intro_GCDAsyncSocket](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frobbiehanson%2FCocoaAsyncSocket%2Fwiki%2FIntro_GCDAsyncSocket)[https://www.jianshu.com/p/321bc95d077f](https://www.jianshu.com/p/321bc95d077f)

# **iOS | Socket & CocoaAsyncSocket 读/写操作以及粘包处理**

> 本文主要介绍 CocoaAsyncSocket的 读/写 操作，以及如何处理TCP粘包的问题，CocoaAsyncSocket基本使用的请查看上一篇文章Socket & CocoaAsyncSocket介绍与使用
> 

### **排队 读/写 操作**

`CocoaAsyncSocket`库的最佳功能之一是“排队 读/写 操作”。

- `写操作`： 套接字未连接，但我还是可以开始写它，库将排队我的所有写操作，在套接字连接后，它将自动开始执行我的写操作！

```objectivec
NSError * err = nil ;
NSError *err = nil;
if (![self.clientSocket connectToHost:@"ip地址" onPort: "端口号" error:&err])  //异步！
{
    NSLog(@"I goofed: %@", err);
}

//此时套接字未连接。
//但我还是可以开始写它！
//库将排队我的所有写操作，
//在套接字连接后，它将自动开始执行我的写操作！
   [socket writeData: data1 withTimeout: - 1  tag: 1 ];
   [socket writeData: data2 withTimeout: - 1  tag: 2 ];
```

- `排队读`: 我们可以通过长度获取到相应长度的数据，可以很好解决粘包问题

`[socket readDataToLength: datalength withTimeout: -1  tag: 0];`

### **Tag参数了解**

CocoaAsyncSocket中的`tag参数`是不通过套接字发送的，也不是从套接字读取的。tag参数只需通过各种委托方法回显给您。它旨在帮助简化委托方法中的代码。

```objectivec
  [socket writeData: data1 withTimeout: - 1  tag: 1 ];
  [socket writeData: data2 withTimeout: - 1  tag: 2 ];
//  当我们发送数据时候使用 tag 标记后，发送后可以在委托方法中根据 tag 看到那条数据已经发送出去了。
- （void）socket：（GCDAsyncSocket *）sock didWriteDataWithTag :( long）tag{
    if（tag == 1）
         NSLog（@"First request sent "）;
    else  if（tag == 2）
         NSLog（@"Second request sent "）;
}
```

在读取时数据时，tag 也很有帮助：

```objectivec
[socket readDataWithTimeout:-1 tag:0];
// 读取 tag 与上面方法的 tag 值是一一对应的。
#define TAG_WELCOME 10
#define TAG_CAPABILITIES 11
#define TAG_MSG 12
- （void）socket：（GCDAsyncSocket *）sender didReadData :( NSData *）data withTag :( long）tag{
    if（tag == TAG_WELCOME）
    {
        //  忽略欢迎信息
    }
    else  if（tag == TAG_CAPABILITIES）
    {
        [self  processCapabilities:data];
    }
    else if (tag == TAG_MSG)
    {
        [self  processMessage: data];
    }
}
```

###