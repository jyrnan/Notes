# <iOS开发>之CocoaAsyncSocket使用 - 粘包处理以及时间延迟测试.

### 本文介绍了CocoaAsyncSocket库中GCDAsyncSocket类的使用、粘包处理以及时间延迟测试.

### 一.CocoaAsyncSocket介绍

CocoaAsyncSocket中主要包含两个类:

**1.GCDAsyncSocket.**

> 用GCD搭建的基于TCP/IP协议的socket网络库 GCDAsyncSocket is a TCP/IP socket networking library built atop Grand Central Dispatch. -- 引自CocoaAsyncSocket.
> 

**2.GCDAsyncUdpSocket.**

> 用GCD搭建的基于UDP/IP协议的socket网络库. GCDAsyncUdpSocket is a UDP/IP socket networking library built atop Grand Central Dispatch..-- 引自CocoaAsyncSocket.
> 

### 二.下载CocoaAsyncSocket

- 首先,需要到[这里](https://link.jianshu.com/?t=https://github.com/robbiehanson/CocoaAsyncSocket)下载CocoaAsyncSocket.
- 下载后可以看到文件所在位置.

![iOS%E5%BC%80%E5%8F%91%20%E4%B9%8BCocoaAsyncSocket%E4%BD%BF%E7%94%A8%20-%20%E7%B2%98%E5%8C%85%E5%A4%84%E7%90%86%E4%BB%A5%E5%8F%8A%E6%97%B6%E9%97%B4%E5%BB%B6%E8%BF%9F%E6%B5%8B%E8%AF%95%20e1684027e89d4ab2b40874a082828b43/3284707-680f66e017b5023b.png](iOS%E5%BC%80%E5%8F%91%20%E4%B9%8BCocoaAsyncSocket%E4%BD%BF%E7%94%A8%20-%20%E7%B2%98%E5%8C%85%E5%A4%84%E7%90%86%E4%BB%A5%E5%8F%8A%E6%97%B6%E9%97%B4%E5%BB%B6%E8%BF%9F%E6%B5%8B%E8%AF%95%20e1684027e89d4ab2b40874a082828b43/3284707-680f66e017b5023b.png)

文件路径

- 这里只要拷贝以下两个文件到项目中.

### 三.客户端

因为,大部分项目已经有服务端socket,所以,先讲解客户端创建过程.

### 步骤:

1.继承`GCDAsyncSocketDelegate`协议.

2.声明属性

```objectivec
// 客户端socket
@property (strong, nonatomic) GCDAsyncSocket *clientSocket;

```

3.创建socket并指定代理对象为self,代理队列必须为主队列.

```objectivec
self.clientSocket = [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_main_queue()];

```

4.连接指定主机的对应端口.

```objectivec
NSError *error = nil;
self.connected = [self.clientSocket connectToHost:self.addressTF.text onPort:[self.portTF.text integerValue] viaInterface:nil withTimeout:-1 error:&error];

```

5.成功连接主机对应端口号.

```objectivec
- (void)socket:(GCDAsyncSocket *)sock didConnectToHost:(NSString *)host port:(uint16_t)port
{
//    NSLog(@"连接主机对应端口%@", sock);
    [self showMessageWithStr:@"链接成功"];
    [self showMessageWithStr:[NSString stringWithFormat:@"服务器IP: %@-------端口: %d", host,port]];

    // 连接成功开启定时器
    [self addTimer];
    // 连接后,可读取服务端的数据
    [self.clientSocket readDataWithTimeout:- 1 tag:0];
    self.connected = YES;
}

```

注意:
 *The host parameter will be an IP address, not a DNS name. -- *引自GCDAsyncSocket
 连接的主机为IP地址,并非DNS名称.

6.发送数据给服务端

```objectivec
// 发送数据
- (IBAction)sendMessageAction:(id)sender
{
    NSData *data = [self.messageTextF.text dataUsingEncoding:NSUTF8StringEncoding];
    // withTimeout -1 : 无穷大,一直等
    // tag : 消息标记
    [self.clientSocket writeData:data withTimeout:- 1 tag:0];
}

```

注意:
 发送数据主要通过`- (void)writeData:(NSData *)data withTimeout:(NSTimeInterval)timeout tag:(long)tag`写入数据的.

7.读取服务端数据

```objectivec
/**
 读取数据

 @param sock 客户端socket
 @param data 读取到的数据
 @param tag 本次读取的标记
 */
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag
{
    NSString *text = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    [self showMessageWithStr:text];
    // 读取到服务端数据值后,能再次读取
    [self.clientSocket readDataWithTimeout:- 1 tag:0];
}

```

注意:
 有的人写好代码,而且第一次能够读取到数据,之后,再也接收不到数据.那是因为,在读取到数据的代理方法中,需要再次调用`[self.clientSocket readDataWithTimeout:- 1 tag:0];`方法,框架本身就是这么设计的.

8.客户端socket断开连接.

```objectivec
/**
 客户端socket断开

 @param sock 客户端socket
 @param err 错误描述
 */
- (void)socketDidDisconnect:(GCDAsyncSocket *)sock withError:(NSError *)err
{
    [self showMessageWithStr:@"断开连接"];
    self.clientSocket.delegate = nil;
    self.clientSocket = nil;
    self.connected = NO;
    [self.connectTimer invalidate];
}

```

注意:
 sokect断开连接时,需要清空代理和客户端本身的socket.

```objectivec
self.clientSocket.delegate = nil;
self.clientSocket = nil;

```

9.建立心跳连接.

```objectivec
 // 计时器
@property (nonatomic, strong) NSTimer *connectTimer;

// 添加定时器
- (void)addTimer
{
     // 长连接定时器
    self.connectTimer = [NSTimer scheduledTimerWithTimeInterval:5.0 target:self selector:@selector(longConnectToSocket) userInfo:nil repeats:YES];
     // 把定时器添加到当前运行循环,并且调为通用模式
    [[NSRunLoop currentRunLoop] addTimer:self.connectTimer forMode:NSRunLoopCommonModes];
}

// 心跳连接
- (void)longConnectToSocket
{
    // 发送固定格式的数据,指令@"longConnect"
    float version = [[UIDevice currentDevice] systemVersion].floatValue;
    NSString *longConnect = [NSString stringWithFormat:@"123%f",version];

    NSData  *data = [longConnect dataUsingEncoding:NSUTF8StringEncoding];

    [self.clientSocket writeData:data withTimeout:- 1 tag:0];
}

```

注意:
 心跳连接中发送给服务端的数据只是作为测试代码,根据你们公司需求,或者和后台商定好心跳包的数据以及发送心跳的时间间隔.因为这个项目的服务端socket也是我写的,所以,我自定义心跳包协议.客户端发送心跳包,服务端也需要有对应的心跳检测,以此检测客户端是否在线.

### 四.服务端

### 步骤:

1.继承`GCDAsyncSocketDelegate`协议.

2.声明属性

```objectivec
// 服务端socket(开放端口,监听客户端socket的连接)
@property (strong, nonatomic) GCDAsyncSocket *serverSocket;

```

3.创建socket并指定代理对象为self,代理队列必须为主队列.

```objectivec
// 初始化服务端socket
self.serverSocket = [[GCDAsyncSocket alloc]initWithDelegate:self delegateQueue:dispatch_get_main_queue()];

```

4.开放服务端的指定端口.

```objectivec
BOOL result = [self.serverSocket acceptOnPort:[self.portF.text integerValue] error:&error];

```

5.连接上新的客户端socket

```objectivec
// 连接上新的客户端socket
- (void)socket:(GCDAsyncSocket *)sock didAcceptNewSocket:(nonnull GCDAsyncSocket *)newSocket
{
    // 保存客户端的socket
    [self.clientSockets addObject: newSocket];
    // 添加定时器
    [self addTimer];

    [self showMessageWithStr:@"链接成功"];
    [self showMessageWithStr:[NSString stringWithFormat:@"客户端的地址: %@ -------端口: %d", newSocket.connectedHost, newSocket.connectedPort]];

    [newSocket readDataWithTimeout:- 1 tag:0];
}

```

6.发送数据给客户端

```objectivec
// socket是保存的客户端socket, 表示给这个客户端socket发送消息
- (IBAction)sendMessage:(id)sender
{
    if(self.clientSockets == nil) return;
    NSData *data = [self.messageTextF.text dataUsingEncoding:NSUTF8StringEncoding];
    // withTimeout -1 : 无穷大,一直等
    // tag : 消息标记
    [self.clientSockets enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        [obj writeData:data withTimeout:-1 tag:0];
    }];
}

```

7.读取客户端的数据

```objectivec
/**
 读取客户端发送的数据

 @param sock 客户端的Socket
 @param data 客户端发送的数据
 @param tag 当前读取的标记
 */
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag
{
    NSString *text = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    [self showMessageWithStr:text];

    // 第一次读取到的数据直接添加
    if (self.clientPhoneTimeDicts.count == 0)
    {
        [self.clientPhoneTimeDicts setObject:[self getCurrentTime] forKey:text];
    }
    else
    {
        // 键相同,直接覆盖,值改变
        [self.clientPhoneTimeDicts enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
            [self.clientPhoneTimeDicts setObject:[self getCurrentTime] forKey:text];
        }];
    }

    [sock readDataWithTimeout:- 1 tag:0];
}

```

8.建立检测心跳连接.

```objectivec
// 检测心跳计时器
@property (nonatomic, strong) NSTimer *checkTimer;

// 添加计时器
- (void)addTimer
{
    // 长连接定时器
    self.checkTimer = [NSTimer scheduledTimerWithTimeInterval:10.0 target:self selector:@selector(checkLongConnect) userInfo:nil repeats:YES];
    // 把定时器添加到当前运行循环,并且调为通用模式
    [[NSRunLoop currentRunLoop] addTimer:self.checkTimer forMode:NSRunLoopCommonModes];
}

// 检测心跳
- (void)checkLongConnect
{
    [self.clientPhoneTimeDicts enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        // 获取当前时间
        NSString *currentTimeStr = [self getCurrentTime];
        // 延迟超过10秒判断断开
        if (([currentTimeStr doubleValue] - [obj doubleValue]) > 10.0)
        {
            [self showMessageWithStr:[NSString stringWithFormat:@"%@已经断开,连接时差%f",key,[currentTimeStr doubleValue] - [obj doubleValue]]];
            [self showMessageWithStr:[NSString stringWithFormat:@"移除%@",key]];
            [self.clientPhoneTimeDicts removeObjectForKey:key];
        }
        else
        {
            [self showMessageWithStr:[NSString stringWithFormat:@"%@处于连接状态,连接时差%f",key,[currentTimeStr doubleValue] - [obj doubleValue]]];
        }
    }];
}

```

心跳检测方法只提供部分思路:
 1.懒加载一个`可变字典`,字典的`键`作为`客户端的标识`.如:`客户端标识`为`13123456789`.

2.在`- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag`方法中,将`读取到的数据`或者`数据中的部分字符串`作为键.字典的`值`为`系统当前时间`.服务端第一次读取数据时,字典中没有数据,所以,直接添加到可变字典中,之后每次读取数据时,都用字典的`setObject: forKey:`方法添加字典,若`存储的键相同`,即`客户端标识相同`,键会被覆盖,再使用`系统的当前时间`作为值.

3.在`- (void)checkLongConnect`中,获取此时的`当前时间`,遍历字典,将每个键的值和当前时间进行比较即可.判断的延迟时间可以写8秒.时间自定.之后,再根据自己的需求进行后续处理.

### 五.数据粘包处理.

1.粘包情况.
 例如:包数据为:`abcd`.

| 完整型 | abcd | abcdabcd | abcdabcdabcd |
| --- | --- | --- | --- |
| 多余型 | abcdab | cdabcdab | cdabcdabcdab |
| 不完整型 | ab | cda | bcdabc |

2.粘包解决思路.

- 思路1: 发送方将数据包加上`包头`和`包尾`,`包头、包体以及包尾`用`字典`形式包装成`json字符串`,接收方,通过解析获取`json字符串`中的包体,便可进行进一步处理. 例如:

```
{
// head:包头,body:包体,end:包尾
 NSDictionary *dict = @{
               @"head" : @"phoneNum",
               @"body" : @(13133334444),
               @"end" : @(11)};
               }

```

- 
    
    思路2: 添加前缀.和包内容拼接成同一个字符串.
    
    例如:当发送数据是`13133334444`,如果出现粘包情况只属于`完整型`: `13133334444` `1313333444413133334444` `131333344441313333444413133334444`... 可以将`ab`作为前缀.则接收到的数据出现的粘包情况: `ab13133334444` `ab13133334444ab13133334444` `ab13133334444ab13133334444ab13133334444`... 使用`componentsSeparatedByString:`方法,以ab为分隔符,将每个包内容存入数组中,再取对应数组中的数据操作即可.
    
- 
    
    思路3: 如果最终要得到的数据的长度是个`固定长度`,用一个`字符串`作为`缓冲池`,每次收到数据,都用`字符串`拼接对应数据,每当`字符串的长度`和`固定长度`相同时,便得到一个`完整数据`,处理完这个数据并`清空字符串`,再进行下一轮的`字符拼接`.
    
    例如:处理上面的`不完整型`.创建一个`长度是4`的`tempData`字符串`作为`数据缓冲池.`第1次`收到数据,数据是:`ab`,`tempData`拼接上`ab`,`tempData`中只能再存储2个字符,`第2次`收到数据,将`数据长度`和`2`进行比较,第2次的数据是:`cda`,截取前两位字符,即`cd`,`tempData`继续拼接`cd`,此时,`tempData`为`abcd`,就是我们想要的数据,我们可以处理这个数据,处理之后并清空`tempData`,将第2次`收到数据`的`剩余数据`,即`cda`中的`a`,再与`tempData`拼接.之后,再进行类似操作.
    
- 核心代码

```objectivec
/**
 处理数据粘包

 @param readData 读取到的数据
 */
  - (void)dealStickPackageWithData:(NSString *)readData
{
    // 缓冲池还需要存储的数据个数
    NSInteger tempCount;

    if (readData.length > 0)
    {
        // 还差tempLength个数填满缓冲池
        tempCount = 4 - self.tempData.length;
        if (readData.length <= tempCount)
        {
            self.tempData = [self.tempData stringByAppendingString:readData];

            if (self.tempData.length == 4)
            {
                [self.mutArr addObject:self.tempData];
                self.tempData = @"";
            }
        }
        else
        {
            // 下一次的数据个数比要填满缓冲池的数据个数多,一定能拼接成完整数据,剩余的继续
            self.tempData = [self.tempData stringByAppendingString:[readData substringToIndex:tempCount]];
            [self.mutArr addObject:self.tempData];
            self.tempData = @"";

            // 多余的再执行一次方法
            [self dealStickPackageWithData:[readData substringFromIndex:tempCount]];
        }
    }
}

```

- 调用

```objectivec
// 存储处理后的每次返回数据
@property (nonatomic, strong) NSMutableArray *mutArr;
// 数据缓冲池
@property (nonatomic, copy) NSString *tempData;

  /** 第四次测试 -- 混合型**/
    self.mutArr = nil;
    /*
     第1次 : abc
     第2次 : da
     第3次 : bcdabcd
     第4次 : abcdabcd
     第5次 : abcdabcdab
     */
    // 数组中的数据代表每次接收的数据
    NSArray *testArr4 = [NSArray arrayWithObjects:@"abc",@"da",@"bcdabcd",@"abcdabcd",@"abcdabcdab", nil];
    self.tempData = @"";
    for (NSInteger i = 0; i < testArr4.count; i++)
    {
        [self dealStickPackageWithData:testArr4[i]];
    }
    NSLog(@"testArr4 = %@",self.mutArr);

```

- 结果:

```
2017-06-09 00:49:12.932976+0800 StickPackageDealDemo[10063:3430118] testArr4 = (
    abcd,
    abcd,
    abcd,
    abcd,
    abcd,
    abcd,
    abcd
)

```

数据粘包处理Demo在文末.

### 六.测试.

1.测试配置.
 测试时,两端需要处于同一WiFi下.客户端中的IP地址为服务端的IP地址,具体信息进入Wifi设置中查看.

IP和端口描述

2.测试所需环境.
 将客户端程序安装在每个客户端,让一台服务端测试机和一台客户端测试机连接mac并运行,这两台测试机可以看到打印结果,所有由服务端发送到客户端的数据,通过客户端再回传给服务端,在服务端看打印结果.

当年的图

3.进行延迟差测试.`延迟差`即服务端发送数据到`第一台客户端`和服务端发送数据到`最后一台客户端`的时间差.根据服务端发送数据给不同数量的客户端进行测试.而且,发送数据时,是随机发送.

延迟差测试结果:

由图所知,延迟差在200毫秒以内的比例基本保持在99%以上.所以符合开发需求(延迟在200毫秒以内).

4.单次信息收发测试.
 让服务端给每个客户端随机发送`200`次数据.并计算服务端发送数据到某一客户端,完整的`一次收发时间`情况.

单次信息收发测试结果:

由图所知,一次收发时间基本在95%以上,这个时间会根据网络状态和数据包大小波动.不过,可以直观看到数据从服务端到客户端的时间.

### CSDN

### 个人博客

### GitHub

[数据粘包处理Demo](https://github.com/CherishJoyBy/BYStickPackageDealDemo)[CocoaAsyncSocket客户端Demo](https://github.com/CherishJoyBy/BYSocketClientDemo)[CocoaAsyncSocket服务端Demo](https://github.com/CherishJoyBy/BYSocketServerDemo)[CocoaAsyncSocket客户端Demo(含粘包解决和测试)](https://github.com/CherishJoyBy/BYSocketClientTestDemo)[CocoaAsyncSocket服务端Demo(含粘包解决和测试)](https://github.com/CherishJoyBy/BYSocketServerTestDemo)

### 被以下专题收入，发现更多相似内容