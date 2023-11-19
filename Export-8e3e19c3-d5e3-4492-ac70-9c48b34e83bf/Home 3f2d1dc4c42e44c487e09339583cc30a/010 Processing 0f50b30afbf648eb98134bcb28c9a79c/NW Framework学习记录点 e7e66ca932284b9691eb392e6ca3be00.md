# NW.Framework学习记录点

### `receiveMessage` 和 `[receive(minimumIncompleteLength:maximumLength:completion:)](https://developer.apple.com/documentation/network/nwconnection/2998572-receive)`区别

TCP is a stream-based protocol. The connection is simply two streams of bytes - send & receive. There is no concept of a "message" inherent in TCP. Your application needs to impose some framing one the stream to extract structure from it. e.g. You could send "hello\n","World\n","test\n" as three separate calls to `send` but all of that data could be sent in a single packet to the destination; The TCP stack tries to optimise network utilisation and it is more efficient to send a single, larger, packet than multiple, smaller, packets.

TCP 是一种基于流的协议。连接只是两个字节流 - 发送和接收。TCP中没有“消息”的概念。您的应用程序需要在流上强加一些框架以从中提取结构。例如，您可以将“hello\n”，“World\n”，“test\n”作为三个单独的调用发送到`send`，但所有这些数据都可以作为单个数据包发送到目的地; TCP堆栈尝试优化网络利用率，发送单个较大的数据包比发送多个较小的数据包更有效。

Similarly, a large amount of data passed to `send` might be sent as multiple packets if the amount of data exceeds the maximum transmission unit (MTU) of the network. Data sent across the Internet can even be split into smaller packets if a link with a smaller MTU is encountered. All that TCP guarantees is that the bytes sent will be delivered in the order that they were sent. There is no guarantee how many packets will arrive.

> What does this mean "If you request to receive a message on a protocol that is otherwise an unbounded byte stream, like TCP or TLS, note that this will not deliver any data until the stream is closed by the peer."?
> 

如果您要求接收像TCP或TLS这样的其他无界字节流的消息，请注意，除非对等端关闭流，否则不会传递任何数据。

It means that you will not receive any message until the server closes the TCP connection. If your server code does not close the connection after sending the response then `receiveMessage` will not complete. You can use `[receive(minimumIncompleteLength:maximumLength:completion:)](https://developer.apple.com/documentation/network/nwconnection/2998572-receive)` to receive the response in "chunks". You would then need to re-assemble these and extract your message (For example, your message might be terminated by a new line `\n` - You would look through the data for new line characters in order to split the received data into discrete messages.

Apple provides the `[NWProtocolFramer](https://developer.apple.com/documentation/network/nwprotocolframer)` protocol to let you create code to parse messages from a TCP stream. You implement code that conforms to `[NWProtocolFramerImplementation](https://developer.apple.com/documentation/network/nwprotocolframerimplementation)` and which operates on the supplied `[NWProtocolFramer.Instance](https://developer.apple.com/documentation/network/nwprotocolframer/instance)` to extract messages from the stream and encode messages back into a stream of bytes.

苹果提供`[NWProtocolFramer](<https://developer.apple.com/documentation/network/nwprotocolframer>)`协议，让您创建代码以从TCP流中解析消息。您实现符合`[NWProtocolFramerImplementation](<https://developer.apple.com/documentation/network/nwprotocolframerimplementation>)`的代码，并在提供的`[NWProtocolFramer.Instance](<https://developer.apple.com/documentation/network/nwprotocolframer/instance>)`上运行，以从流中提取消息并将消息编码回字节流。

### 另一段关于TCP/IP的传输方式的解释

TCP/IP is a *stream-based* protocol, not a *message-based* protocol. There's no guarantee that every `send()` call by one peer results in a single `recv()` call by the other peer receiving the exact data sent—it might receive the data piece-meal, split across multiple `recv()` calls, due to packet fragmentation.

You need to define your own message-based protocol on top of TCP in order to differentiate message boundaries. Then, to read a message, you continue to call `recv()` until you've read an entire message or an error occurs.

One simple way of sending a message is to prefix each message with its length. Then to read a message, you first read the length, then you read that many bytes. Here's how you might do that:

### 如何读取指定长度数据

![Untitled](NW%20Framework%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%E7%82%B9%20e7e66ca932284b9691eb392e6ca3be00/Untitled.png)

经过自己捣鼓，可以判断

- 这个Recieve的数值和MTU并没有强相关。也正如上面的范例所说：“read exactly the body length”，是可以设置成所需要的字节数量的。
- 这里设置minum和maximum两个参数，应该是为了方法的通用性：当两个参数设置成同一个树脂，也就是获取指定的字节数了。
- 如果两个参数不一致，相当于一个区间。获得这个区间内数量的data，就会呼叫这个回调，否则会等到满足条件后才呼叫。例如上面指定的字节数，只有刚好是这个字节数才会呼叫回调。

用了上面的方式，就可以实现读取指定长度header后继续读取大量body数据

# ContentContext（可以带信息？）

在NWConnection下的属性

## 2个主要属性：

```swift
/// A string description of the content, used for logging and debugging.
        final public let identifier: String
/// An array of protocol metadata to send (to inform the protocols of per-data options) or receive (to receive per-data options or statistics).
        public var protocolMetadata: [NWProtocolMetadata] { get }
```

contentContext: Content context describing the received content. This includes protocol metadata
/// that lets the caller introspect information about the received content (such as flags on a packet).

内容上下文：描述收到的内容的内容上下文。这包括协议元数据。例如`NWProtocolFramer.Message`

```swift
/// Send data on a connection. This may be called before the connection is ready,
    /// in which case the send will be enqueued until the connection is ready to send.
    /// This is an asynchronous send and the completion block can be used to
    /// determine when the send is complete. There is nothing preventing a client
    /// from issuing an excessive number of outstanding sends. To minimize memory
    /// footprint and excessive latency as a consequence of buffer bloat, it is
    /// advisable to keep a low number of outstanding sends. The completion block
    /// can be used to pace subsequent sends.
    /// - Parameter content: The data to send on the connection. May be nil if this send marks its context as complete, such
    ///   as by sending .finalMessage as the context and marking isComplete to send a write-close.
    /// - Parameter contentContext: The context associated with the content, which represents a logical message
    ///   to be sent on the connection. All content sent within a single context will
    ///   be sent as an in-order unit, up until the point that the context is marked
    ///   complete (see isComplete). Once a context is marked complete, it may be re-used
    ///   as a new logical message. Protocols like TCP that cannot send multiple
    ///   independent messages at once (serial streams) will only start processing a new
    ///   context once the prior context has been marked complete. Defaults to .defaultMessage.
    /// - Parameter isComplete: A flag indicating if the caller's sending context (logical message) is now complete.
    ///   Until a context is marked complete, content sent for other contexts may not
    ///   be sent immediately (if the protocol requires sending bytes serially, like TCP).
    ///   For datagram protocols, like UDP, isComplete indicates that the content represents
    ///   a complete datagram.
    ///   When sending using streaming protocols like TCP, isComplete can be used to mark the end
    ///   of a single message on the stream, of which there may be many. However, it can also
    ///   indicate that the connection should send a "write close" (a TCP FIN) if the sending
    ///   context is the final context on the connection. Specifically, to send a "write close",
    ///   pass .finalMessage or .defaultStream for the context (or create a custom context and
    ///   set .isFinal), and pass true for isComplete.
    /// - Parameter completion: A completion handler (.contentProcessed) to notify the caller when content has been processed by
    ///   the connection, or a marker that this data is idempotent (.idempotent) and may be sent multiple times as fast open data.
    final public func send(content: Data?, contentContext: NWConnection.ContentContext = .defaultMessage, isComplete: Bool = true, completion: NWConnection.SendCompletion)
```

# NWPathMonitor

# NWBrowser

## changHandler

回调提供两个参数

- changes：
- browserResults：

## stateHandler

# FrameProtocols的层级关系

目的就是将TCP串流的数据进行分片。

方法是通过框架在上次App和下层传输实现之间加入了一层FrameProtocol，来进行划分。

![Screenshot 2022-12-14 at 20.56.24.png](NW%20Framework%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%E7%82%B9%20e7e66ca932284b9691eb392e6ca3be00/Screenshot_2022-12-14_at_20.56.24.png)

## APP-FrameProtocols-Socket之间大致层级关系如下：

- ⬇️ app通过`send(content:contentContext:isComplete:comletion）`发送数据content,而**contentContext**里面带有符合FrameProtocol协议的message  **<Data+Context>**

---

- ⬇️ `handleOutput()` **<Data + Message>**
    - FrameProtocol在`handleOutput()`完成会根据数据创建一个header，包含真正数据的长度，以及可选message里面的信息。就是把message**封装到header**
    - 先发header后，再发送真正的**data**

---

- 分割线《=数据传输=》分割线  ： **<Header + Data>**

---

- `handleInput()`  ⬆️ **<Data + Message>**
    - 收到数据后`handleInput()` 先获得header信息，取得后续数据长度，并根据header里数据**创建message**
    - 然后往上传送相应长度的**data**，同时发送**message**

---

- app通过`revieveMessage(completion:)`来获得数据content和**contentContex**（里面包含有message）⬆️ **<Data+Context>**

## FrameProtocols内部层次更详细解析

### handleOutput()处理发出的数据

- 创建header，并通过`frame.writeOutput(data:)`发送header的data，这里这里在header的定义中直接写好序列化的方法，例如`header.encodedData`
- 把App下发的数据，紧跟上面的header通过`framer.writeOutputNoCopy(length:)`发送出去
    - 🤔我估计ap是通过`send(content:contentContext:isComplete:comletion）`这个方法来间接/直接调用`handleOutput()`方法，从而进一步调用`framer.writeOutputNoCopy(length:)`来发送数据。这部分数据应该是保存在某个缓存了。
    - 这段的实现应该是框架的代码，无从得知

### handleInput()处理收到的数据

这个方法就和上面的刚好相反

在这个方法里面分成两部分

- 通过`framer.parseInput(minimumIncompleteLength:maximumLength:)`获取Header信息，
    - `header.length`: 标志着后面的的数据body的长度
    - 有可能有其他的信息，例如type，可以用来做为后续的message：NWProtocolFramer.Message 来和body部分的data一起发送
- 调用`framer.deliverImputNoCopy(length:message:isComplete:)`方法
    - 直接向应用层发送header.length长度的字节，
    - 并随同发送message来进一步说明数据类型

# TLS连接

在TicTacToe范例里面的TLS连接应该是基于PSK的方式。

- 建立TLS连接的关键是创建tlsOptions，这里范例是基于Passcode来创建`NWProtocolTLS.Options`

```swift
// Create TLS options using a passcode to derive a preshared key.
	private static func tlsOptions(passcode: String) -> NWProtocolTLS.Options {
		let tlsOptions = NWProtocolTLS.Options()

		let authenticationKey = SymmetricKey(data: passcode.data(using: .utf8)!)
		var authenticationCode = HMAC<SHA256>.authenticationCode(for: "TicTacToe".data(using: .utf8)!, using: authenticationKey)

		let authenticationDispatchData = withUnsafeBytes(of: &authenticationCode) { (ptr: UnsafeRawBufferPointer) in
			DispatchData(bytes: ptr)
		}

		sec_protocol_options_add_pre_shared_key(tlsOptions.securityProtocolOptions,
												authenticationDispatchData as __DispatchData, //PSK，也就是后续加密的密钥
												stringToDispatchData("TicTacToe")! as __DispatchData) //PSK Identity，密钥的表示标识
		sec_protocol_options_append_tls_ciphersuite(tlsOptions.securityProtocolOptions,
													tls_ciphersuite_t(rawValue: TLS_PSK_WITH_AES_128_GCM_SHA256)!)
		return tlsOptions
	}
```

- 然后就可以创建`NWParameters`

```swift
// Create parameters with custom TLS and TCP options.
		self.init(tls: NWParameters.tlsOptions(passcode: passcode), tcp: tcpOptions)
```

- 从而进一步创建`NWConnetion`

```swift
let connection = NWConnection(to: endpoint, using: NWParameters(passcode: passcode))
```

# 创建NWConnection时候指定端口

```swift
params.requiredLocalEndpoint = NWEndpoint.hostPort(host: NWEndpoint.Host("0.0.0.0"), port: 49001)
```

# 一张流程图片

![Untitled](NW%20Framework%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%E7%82%B9%20e7e66ca932284b9691eb392e6ca3be00/Untitled%201.png)

[利用 Network.framework　在 iOS 實作簡易 HTTP 伺服器](https://www.appcoda.com.tw/network-framework-http/)

[Swift HTTP over Network framework](NW%20Framework%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%E7%82%B9%20e7e66ca932284b9691eb392e6ca3be00/Swift%20HTTP%20over%20Network%20framework%2020e199bbb8a7448995aa7221e39b611a.md)

[Intro to Network.framework Servers](http://www.alwaysrightinstitute.com/network-framework/)

[Intro to Network.framework Servers – Helge Heß – Software engineer.](../300%20Learning%2085ac233e054e457b9ea0166355f87cb8/312%20Swift%20f3f7435fa56948829718e9373255e683/Intro%20to%20Network%20framework%20Servers%20%E2%80%93%20Helge%20He%C3%9F%20%E2%80%93%20S%2081a8dca3c77849c78918c45d43520de6.md)

[一个支持心跳保持长连接的代码范例](NW%20Framework%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%E7%82%B9%20e7e66ca932284b9691eb392e6ca3be00/%E4%B8%80%E4%B8%AA%E6%94%AF%E6%8C%81%E5%BF%83%E8%B7%B3%E4%BF%9D%E6%8C%81%E9%95%BF%E8%BF%9E%E6%8E%A5%E7%9A%84%E4%BB%A3%E7%A0%81%E8%8C%83%E4%BE%8B%20fb5263bcf85d4c7fbf99ae4c2beb833b.md)

[NWConnection的另外的handler](NW%20Framework%E5%AD%A6%E4%B9%A0%E8%AE%B0%E5%BD%95%E7%82%B9%20e7e66ca932284b9691eb392e6ca3be00/NWConnection%E7%9A%84%E5%8F%A6%E5%A4%96%E7%9A%84handler%203814314145fc4736b11b96214f6be2e3.md)