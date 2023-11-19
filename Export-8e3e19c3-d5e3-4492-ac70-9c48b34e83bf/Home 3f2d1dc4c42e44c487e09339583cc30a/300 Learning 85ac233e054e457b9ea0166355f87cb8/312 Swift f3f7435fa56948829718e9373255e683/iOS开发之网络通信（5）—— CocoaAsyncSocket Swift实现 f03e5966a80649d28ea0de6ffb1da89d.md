# iOS开发之网络通信（5）—— CocoaAsyncSocket Swift实现

# 专栏

[iOS开发之网络通信（1）—— 计算机网络](https://blog.csdn.net/u012078168/article/details/108663954?spm=1001.2014.3001.5501)[iOS开发之网络通信（2）—— HTTP(S)](https://blog.csdn.net/u012078168/article/details/113998630?spm=1001.2014.3001.5501)[iOS开发之网络通信（3）—— XML & JSON](https://blog.csdn.net/u012078168/article/details/114024548?spm=1001.2014.3001.5501)[iOS开发之网络通信（4）—— socket](https://blog.csdn.net/u012078168/article/details/114285199?spm=1001.2014.3001.5501)[iOS开发之网络通信（5）—— CocoaAsyncSocket](https://blog.csdn.net/u012078168/article/details/114370160?spm=1001.2014.3001.5501)[iOS开发之网络通信（6）—— AFNetworking & Alamofire](https://blog.csdn.net/u012078168/article/details/114583643?spm=1001.2014.3001.5501)

[CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)是一个异步[socket](https://so.csdn.net/so/search?q=socket&spm=1001.2101.3001.7020)库，提供了macOS、iOS和tvOS版本，其底层基于BSD socket。
 本文简单介绍Swift下TCP和[UDP](https://so.csdn.net/so/search?q=UDP&spm=1001.2101.3001.7020)用法。

# 1. UDP

```swift
// 创建socket
self.udpSocket = GCDAsyncUdpSocket(delegate: self, delegateQueue: .global())
do {
    // 绑定端口
    try self.udpSocket?.bind(toPort: UInt16(port))
    // 启用广播
    try self.udpSocket?.enableBroadcast(true)
    // 开始接收数据
    try self.udpSocket?.beginReceiving()
} catch {
    // 出错，关闭socket
    self.udpSocket?.close()
    self.udpSocket = nil
    print("socket error")
}

// 发送数据：可定向发送（如192.168.1.x）或者广播（255.255.255.255）
self.udpSocket?.send(messageData, toHost: host, port: port, withTimeout: -1, tag: 0)

//MARK: - GCDAsyncUdpSocketDelegate
// 发送成功
func udpSocket(_ sock: GCDAsyncUdpSocket, didSendDataWithTag tag: Int) {

    DispatchQueue.main.async {
        print("send: \(self.messageTF.text!), tag: \(tag)")
        self.showMessage("send: \(self.messageTF.text!), tag: \(tag)")
    }
}

// 发送失败
func udpSocket(_ sock: GCDAsyncUdpSocket, didNotSendDataWithTag tag: Int, dueToError error: Error?) {

    print("didNotSendDataWithTag: \(tag), error: \(String(describing: error))")
    showMessage("didNotSendDataWithTag, error: \(String(describing: error))")
}

// 收到数据
func udpSocket(_ sock: GCDAsyncUdpSocket, didReceive data: Data, fromAddress address: Data, withFilterContext filterContext: Any?) {

    if let dataStr = String.init(data: data, encoding: .utf8) {
        print("receive: " + dataStr)
        showMessage("receive: " + dataStr)
    }
}

```

# 2. TCP Server

```swift
// 创建socket
func setup() {
    self.tcpServer = GCDAsyncSocket(delegate: self, delegateQueue: .global())
    self.tcpClients = [GCDAsyncSocket?]()
}

// 监听
func listen(port: UInt16) {
    do {
        // 检测有无客户端连接
        try self.tcpServer?.accept(onPort: port)
    } catch {
        print("accept error")
    }
}

// 关闭
func close() {
    // 断开所有客户端的连接
    guard self.tcpClients != nil,
          self.tcpClients!.count > 0 else {
        break
    }
    for tcpClient in self.tcpClients! {
        tcpClient?.disconnect()
    }
    // 断开服务端连接
    self.tcpServer?.disconnect()
}

// 发送数据
func send(data: Data) {
    guard self.tcpClients != nil,
          self.tcpClients!.count > 0 else {
        return
    }
    for tcpClient in self.tcpClients! {
        tcpClient?.write(data, withTimeout: -1, tag: 0)
    }
}

//MARK: - GCDAsyncSocketDelegate

// 检测到客户端socket
func socket(_ sock: GCDAsyncSocket, didAcceptNewSocket newSocket: GCDAsyncSocket) {

    self.tcpClients?.append(newSocket)
    newSocket.readData(withTimeout: -1, tag: 0)
}

// 收到数据
func socket(_ sock: GCDAsyncSocket, didRead data: Data, withTag tag: Int) {

    DispatchQueue.main.async {
        if let message = String.init(data: data, encoding: .utf8) {
            self.showMessage(message: "receive: " + message)
            print("receive: " + message)
        }
    }

    sock.readData(withTimeout: -1, tag: 0)
}

// 发送成功
func socket(_ sock: GCDAsyncSocket, didWriteDataWithTag tag: Int) {

    DispatchQueue.main.async {
        if let message = self.messageTF.text {
            self.showMessage(message: "send: " + message)
            print("send: " + message)
        }
    }
}

// 出错
func socketDidDisconnect(_ sock: GCDAsyncSocket, withError err: Error?) {

    DispatchQueue.main.async {
        let message = "error: " + String(describing: err)
        self.showMessage(message: message)
        print(message)
    }
}

```

# 3. TCP Client

```swift
// 创建socket
func setup() {
    self.tcpClient = GCDAsyncSocket(delegate: self, delegateQueue: .global())
}

// 连接
func connect(host: String, port: UInt16) {
    do {
        try self.tcpClient?.connect(toHost: host, onPort: port)
    } catch {
        print("connect error")
        return
    }
}

// 关闭连接
func disconnect() {
    self.tcpClient?.disconnect()
}

// 发送数据
func send(data: Data) {
    self.tcpClient?.write(data, withTimeout: -1, tag: 0)
}

//MARK: - GCDAsyncSocketDelegate

// 连接成功
func socket(_ sock: GCDAsyncSocket, didConnectToHost host: String, port: UInt16) {

    sock.readData(withTimeout: -1, tag: 0)

    DispatchQueue.main.async {
        self.showMessage(message: "connected")
    }
}

// 收到数据
func socket(_ sock: GCDAsyncSocket, didRead data: Data, withTag tag: Int) {

    DispatchQueue.main.async {
        if let message = String.init(data: data, encoding: .utf8) {
            self.showMessage(message: "receive: " + message)
            print("receive: " + message)
        }
    }

    sock.readData(withTimeout: -1, tag: 0)
}

// 发送成功
func socket(_ sock: GCDAsyncSocket, didWriteDataWithTag tag: Int) {

    DispatchQueue.main.async {
        if let message = self.messageTF.text {
            self.showMessage(message: "send: " + message)
            print("send: " + message)
        }
    }
}

// 出错
func socketDidDisconnect(_ sock: GCDAsyncSocket, withError err: Error?) {

    DispatchQueue.main.async {
        let message = "error: " + String(describing: err)
        self.showMessage(message: message)
        print(message)
    }
}

```