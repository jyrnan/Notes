# iOS开发之网络通信（4）ios socket通信

# 专栏

[iOS开发之网络通信（1）—— 计算机网络](https://blog.csdn.net/u012078168/article/details/108663954?spm=1001.2014.3001.5501)[iOS开发之网络通信（2）—— HTTP(S)](https://blog.csdn.net/u012078168/article/details/113998630?spm=1001.2014.3001.5501)[iOS开发之网络通信（3）—— XML & JSON](https://blog.csdn.net/u012078168/article/details/114024548?spm=1001.2014.3001.5501)[iOS开发之网络通信（4）—— socket](https://blog.csdn.net/u012078168/article/details/114285199?spm=1001.2014.3001.5501)[iOS开发之网络通信（5）—— CocoaAsyncSocket](https://blog.csdn.net/u012078168/article/details/114370160?spm=1001.2014.3001.5501)[iOS开发之网络通信（6）—— AFNetworking & Alamofire](https://blog.csdn.net/u012078168/article/details/114583643?spm=1001.2014.3001.5501)

[socket](https://so.csdn.net/so/search?q=socket&spm=1001.2101.3001.7020)是一种抽象层编程接口，用于应用程序间通过网络协议进行通信。其实现方式主要有两种：TCP和UDP。关于TCP和UDP，前文有介绍了：[iOS开发之网络通信（1）—— 计算机网络](https://www.jianshu.com/p/86c0c70110f2)。
 iOS中，根据不同语言有不同的方法来创建socket连接：

1. C：CFStream。
2. OC：NSStream。
3. Swift：Stream。

OC中的NSStream和Swift中的[Stream](https://so.csdn.net/so/search?q=Stream&spm=1001.2101.3001.7020)类都是对CFStream的封装，而CFStream是基于TCP协议创建的socket连接，无法创建UDP socket连接。

iOS的另外一种原生创建socket方法 —— BSD Socket。伯克利套接字（Berkeley sockets，即BSD Socket）由伯克利大学研发，最终成为网络开发接口的标准。

socket调试工具：[ssokit](http://product.rangaofei.cn/ssokit/pages/)

# 一. Stream Socket

这种方式是基于TCP协议的socket连接。
 client端代码（swift）：

```swift
protocol StreamSocketDelegate {
    func eventCode(eventCode: Stream.Event)
    func received(data: Data)
}

class StreamSocket: NSObject, StreamDelegate {

    static let shared = StreamSocket()  // 单例
    var delegate: StreamSocketDelegate?
    var iSream: InputStream?
    var oStream: OutputStream?

    // 创建socket连接
    func create(host: String, port: UInt32) {
        print(#function)

        var readStream: Unmanaged<CFReadStream>?
        var writeStream: Unmanaged<CFWriteStream>?

        // 创建socket连接
        CFStreamCreatePairWithSocketToHost(kCFAllocatorDefault, host as CFString, port, &readStream, &writeStream)
        // 转换成 InputStream & OutputStream 对象
        self.iSream = readStream?.takeRetainedValue()
        self.oStream = writeStream?.takeRetainedValue()

        if self.iSream == nil {
            print("Erro read")
            return
        }

        if self.oStream == nil {
            print("Erro write")
            return
        }

        self.iSream!.delegate = self
        self.oStream!.delegate = self

        self.iSream!.schedule(in: RunLoop.main, forMode: RunLoop.Mode.default)
        self.oStream!.schedule(in: RunLoop.main, forMode: RunLoop.Mode.default)
        self.iSream!.open()
        self.oStream!.open()
    }

    // 关闭socket连接
    func close() {

        self.iSream?.close()
        self.oStream?.close()
    }

    // 发送数据
    func send(data: Data) {

        guard self.oStream != nil,
              self.oStream?.hasSpaceAvailable == true
        else {
            print("no space available")
            return
        }

        let bytes = [UInt8](data)
        self.oStream?.write(bytes, maxLength: data.count)
    }

    //MARK: - StreamDelegate
    func stream(_ aStream: Stream, handle eventCode: Stream.Event) {

        self.delegate?.eventCode(eventCode: eventCode)

        switch eventCode {

            case .openCompleted:        // 打开完毕
                print(#function, "openCompleted")
                break

            case .hasBytesAvailable:    // 可读
                print(#function, "hasBytesAvailable")

                let buffer = UnsafeMutablePointer<UInt8>.allocate(capacity: 4096)
                while (aStream as! InputStream).hasBytesAvailable {
                    let numberOfBytesRead = (aStream as! InputStream).read(buffer, maxLength: 4096)
                    if numberOfBytesRead < 0 { break }
                    let data = Data(bytes: buffer, count: numberOfBytesRead)
                    self.delegate?.received(data: data)
                }

                break

            case .hasSpaceAvailable:    // 可写
                print(#function, "hasSpaceAvailable")
                break

            case .errorOccurred:        // 发生错误
                print(#function, "errorOccurred", "error: \(String(describing: aStream.streamError))")
                break

            case .endEncountered:       // 结束
                print(#function, "endEncountered")
                break

            default:
                break
        }
    }
}
100101102103104105106107108109110111
```

# 二. BSD Socket

UDP是无连接的，人们习惯将监听方称为服务端，将发送端作为客户端。

- 监听方需要进行IP地址绑定操作，可选择绑定本机IP（通常是192.168.1.x）或者是任意地址IP（0.0.0.0）进行监听。
- 发送方可以选择向指定IP地址发送，或者进行广播（255.255.255.255），广播需先开启广播服务。

```objectivec
#import "UDPSocket.h"
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

static UDPSocket *_singleInstance = nil;

@implementation UDPSocket

#pragma mark - 单例

+ (instancetype)sharedInstance
{
    return [[self alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleInstance = [super allocWithZone:zone];
    });
    return _singleInstance;
}

- (instancetype)init
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleInstance = [super init];
        if (_singleInstance) {
            // 在这里初始化self的属性和方法
        }
    });
    return _singleInstance;
}

#pragma mark - 对象方法

- (void)send:(NSData *)data host:(NSString *)host port:(UInt16)port {

    struct sockaddr_in addr;
    bzero(&addr, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = inet_addr(host.UTF8String);  // 如需广播，使用htonl(INADDR_BROADCAST);，等价inet_addr(“255.255.255.255”);。只能在局域网广播，路由器不做转发。

    int socketFD = socket(AF_INET, SOCK_DGRAM, 0);    // SOCK_STREAM为TCP，SOCK_DGRAM为UDP，SOCK_RAW为IP
    if (socketFD < 0) {
        perror("socket error \n");
        return;
    }

    // 开启广播服务
    const int broadcast = 1;
    int status = setsockopt(socketFD, SOL_SOCKET, SO_BROADCAST, &broadcast, sizeof(broadcast));
    if (status < 0)
    {
        printf("Error enabling address reuse (setsockopt)");
        if ([self.delegate respondsToSelector:@selector(showMessage:)]) {
            [self.delegate showMessage:@"Error enabling address reuse (setsockopt)"];
        }
        close(socketFD);
        return;
    }

    ssize_t ret = sendto(socketFD, data.bytes, data.length, 0, (sockaddr *)&addr, sizeof(addr));
    if (ret < 0)
    {
        printf("error in sendto() function. ret=%ld \n", ret);
        if ([self.delegate respondsToSelector:@selector(showMessage:)]) {
            [self.delegate showMessage:@"error in sendto() function."];
        }
        close(socketFD);
        return;
    }
    printf("send: %s \n", data.bytes);
    if ([self.delegate respondsToSelector:@selector(showMessage:)]) {
        [self.delegate showMessage:[NSString stringWithFormat:@"send: %s", data.bytes]];
    }
}

- (void)listen:(int)port {

    int socketFD = socket(AF_INET, SOCK_DGRAM, 0);    // SOCK_STREAM为TCP，SOCK_DGRAM为UDP，SOCK_RAW为IP
    if (socketFD < 0) {
        perror("socket error \n");
        if ([self.delegate respondsToSelector:@selector(showMessage:)]) {
            [self.delegate showMessage:@"socket error \n"];
        }
        return;
    }

    struct sockaddr_in addr;
    bzero(&addr, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = htonl(INADDR_ANY);     // 等价inet_addr("0.0.0.0");

    int status = bind(socketFD, (sockaddr *)&addr, sizeof(addr));
    if (status < 0)
    {
        printf("error in bind() function \n");
        if ([self.delegate respondsToSelector:@selector(showMessage:)]) {
            [self.delegate showMessage:@"error in bind() function \n"];
        }
        close(socketFD);
        return;
    }

    char recv_msg[1024];
    while (1) {
        bzero(recv_msg, 1024);

        long byte_num = recv(socketFD, recv_msg, 1024, 0);
        recv_msg[byte_num] = '\0';
        printf("receive: %s\n", recv_msg);
        if ([self.delegate respondsToSelector:@selector(showMessage:)]) {
            [self.delegate showMessage:[NSString stringWithFormat:@"receive: %s", recv_msg]];
        }
    }
}

@end
118119120
```

### 2. TCP

**2.1 TCP Server**
 TCPServer.h

```objectivec
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

typedef NS_ENUM(NSInteger, TCPServerEvent) {
    TCPServerEvent_Error,
    TCPServerEvent_Listen,
    TCPServerEvent_Accept,
    TCPServerEvent_Send,
    TCPServerEvent_Receive
};

@protocol TCPServerDelegate <NSObject>
@optional
- (void)TCPServerEvent:(TCPServerEvent)event message:(NSString *)message;
@end

@interface TCPServer : NSObject

@property (nonatomic, weak) id<TCPServerDelegate>   delegate;
@property (nonatomic, assign) int  server_socket;
@property (nonatomic, assign) int  client_socket;
@property (nonatomic, assign) BOOL bStop;

+ (instancetype)sharedInstance;

- (void)listenWithHost:(NSString *)host port:(int)port;
- (void)stop;
- (void)send:(NSData *)data;

@end

NS_ASSUME_NONNULL_END

```

TCPServer.m

```objectivec
#import "TCPServer.h"
#include <stdio.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>

static TCPServer *_singleInstance = nil;

@implementation TCPServer

#pragma mark - 单例

+ (instancetype)sharedInstance
{
    return [[self alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleInstance = [super allocWithZone:zone];
    });
    return _singleInstance;
}

- (instancetype)init
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleInstance = [super init];
        if (_singleInstance) {
            // 在这里初始化self的属性和方法
        }
    });
    return _singleInstance;
}

#pragma mark - 对象方法

- (void)listenWithHost:(NSString *)host port:(int)port {

    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(port);
    server_addr.sin_addr.s_addr = inet_addr(host.UTF8String);

    self.server_socket = socket(AF_INET, SOCK_STREAM, 0);    // SOCK_STREAM为TCP，SOCK_DGRAM为UDP，SOCK_RAW为IP
    if (self.server_socket < 0) {
        perror("socket error \n");
        if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
            [self.delegate TCPServerEvent:TCPServerEvent_Error message:@"socket success"];
        }
        return;
    }

    // 绑定socket
    int res = bind(self.server_socket, (struct sockaddr *)&server_addr, sizeof(server_addr));
    if (res < 0) {
        perror("bind error");
        if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
            [self.delegate TCPServerEvent:TCPServerEvent_Error message:@"bind error"];
        }
        return;
    }

    // 监听
    if (listen(self.server_socket, 5) == -1) {
        perror("listen  error");
        if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
            [self.delegate TCPServerEvent:TCPServerEvent_Error message:@"listen error"];
        }
        return;
    }

    if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
        [self.delegate TCPServerEvent:TCPServerEvent_Listen message:@"listen success"];
    }

    // 接收客户端地址
    struct sockaddr_in client_addr;
    socklen_t addr_len;
    self.client_socket = accept(self.server_socket, (struct sockaddr *)&client_addr, &addr_len);
    if (self.client_socket == -1) {
        perror("accept error");
        if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
            [self.delegate TCPServerEvent:TCPServerEvent_Error message:@"accept error"];
        }
        return;
    }

    if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
        [self.delegate TCPServerEvent:TCPServerEvent_Accept message:@"accept success"];
    }

    char recv_msg[1024];
    self.bStop = NO;
    // 循环读取数据
    while (1) {
        if (self.bStop) break;
        bzero(recv_msg, 1024);
        long byte_num = recv(self.client_socket, recv_msg, 1024, 0);
        if (byte_num > 0) {
            recv_msg[byte_num] = '\0';
            printf("receive: %s\n", recv_msg);
            if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
                [self.delegate TCPServerEvent:TCPServerEvent_Receive message:[NSString stringWithFormat:@"receive: %s", recv_msg]];
            }
        }
    }
}

- (void)stop {

    self.bStop = YES;
    close(self.server_socket);
    close(self.client_socket);
}

- (void)send:(NSData *)data {

    ssize_t ret = send(self.client_socket, (char *)data.bytes, data.length, 0);
    if (ret < 0) {
        perror("send error \n");
        if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
            [self.delegate TCPServerEvent:TCPServerEvent_Error message:@"send error"];
        }
    }else {
        printf("send data: %s\n", (char *)data.bytes);
        if ([self.delegate respondsToSelector:@selector(TCPServerEvent:message:)]) {
            [self.delegate TCPServerEvent:TCPServerEvent_Send message:[NSString stringWithFormat:@"send: %s", (char *)data.bytes]];
        }
    }
}

@end

```

**2.2 TCP Client**
 TCPClient.h

```objectivec
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

typedef NS_ENUM(NSInteger, TCPClientEvent) {
    TCPClientEvent_Connected,
    TCPClientEvent_ConnectFailed,
    TCPClientEvent_SendSuccess,
    TCPClientEvent_SendError,
    TCPClientEvent_ReceiveData,
};

@protocol TCPClientDelegate <NSObject>
@optional
- (void)TCPClientEvent:(TCPClientEvent)event message:(NSString *)message;
@end

@interface TCPClient : NSObject

@property (nonatomic, weak) id<TCPClientDelegate>   delegate;
@property (nonatomic, assign) int  socketFD;
@property (nonatomic, assign) BOOL bStop;

+ (instancetype)sharedInstance;

- (void)connectWithHost:(NSString *)host port:(int)port;
- (void)disconnect;
- (void)send:(NSData *)data;

@end

NS_ASSUME_NONNULL_END

```

TCPClient.m

```objectivec
#import "TCPClient.h"
#include <stdio.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>

static TCPClient *_singleInstance = nil;

@implementation TCPClient

#pragma mark - 单例

+ (instancetype)sharedInstance
{
    return [[self alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleInstance = [super allocWithZone:zone];
    });
    return _singleInstance;
}

- (instancetype)init
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleInstance = [super init];
        if (_singleInstance) {
            // 在这里初始化self的属性和方法
        }
    });
    return _singleInstance;
}

#pragma mark - 对象方法

- (void)connectWithHost:(NSString *)host port:(int)port {

    struct sockaddr_in server_addr;
    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(port);
    server_addr.sin_addr.s_addr = inet_addr(host.UTF8String);

    self.socketFD = socket(AF_INET, SOCK_STREAM, 0);    // SOCK_STREAM为TCP，SOCK_DGRAM为UDP，SOCK_RAW为IP
    if (self.socketFD < 0) {
        perror("socket error \n");
        return;
    }

    int ret = connect(self.socketFD, (struct sockaddr *)&server_addr, sizeof(struct sockaddr_in));
    if (ret < 0) {
        perror("connect error \n");
        if ([self.delegate respondsToSelector:@selector(TCPClientEvent:message:)]) {
            [self.delegate TCPClientEvent:TCPClientEvent_ConnectFailed message:@"connect error"];
        }
        return;
    }

    printf("connect success \n");
    if ([self.delegate respondsToSelector:@selector(TCPClientEvent:message:)]) {
        [self.delegate TCPClientEvent:TCPClientEvent_Connected message:@"connect success"];
    }

    char buffer[1024];
    self.bStop = NO;
    while (1) {
        if (self.bStop) break;
        bzero(buffer, 1024);
        ssize_t byte_num = recv(self.socketFD, buffer, 1024, 0);     // 阻塞直到收到数据
        if (byte_num > 0) {
            buffer[byte_num] = '\0';
            printf("receive: %s\n", buffer);
            if ([self.delegate respondsToSelector:@selector(TCPClientEvent:message:)]) {
                [self.delegate TCPClientEvent:TCPClientEvent_ReceiveData message:[NSString stringWithFormat:@"receive: %s", buffer]];
            }
        }
    }
}

- (void)disconnect {

    self.bStop = YES;
    close(self.socketFD);
}

- (void)send:(NSData *)data {

    ssize_t ret = send(self.socketFD, (char *)data.bytes, data.length, 0);
    if (ret < 0) {
        perror("send error \n");
        if ([self.delegate respondsToSelector:@selector(TCPClientEvent:message:)]) {
            [self.delegate TCPClientEvent:TCPClientEvent_SendError message:@"send error"];
        }
    }else {
        printf("send data: %s\n", (char *)data.bytes);
        if ([self.delegate respondsToSelector:@selector(TCPClientEvent:message:)]) {
            [self.delegate TCPClientEvent:TCPClientEvent_SendSuccess message:[NSString stringWithFormat:@"send: %s", (char *)data.bytes]];
        }
    }

}

@end

```