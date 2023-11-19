# iOS-CocoaAsyncSocket-使用教程(Swift版)

![iOS-CocoaAsyncSocket-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(Swift%E7%89%88)%20bca0f172a2ff4304a5718cb22e09fde3/1176193-c85bb9307e3500ac.jpg](iOS-CocoaAsyncSocket-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(Swift%E7%89%88)%20bca0f172a2ff4304a5718cb22e09fde3/1176193-c85bb9307e3500ac.jpg)

Socket就是对于传输层TCP/IP的封装

### 简述:

- 由于最近项目的需要,需要使用到Socket,所以就趁此整理了一下,网上对于Socket的理论知识有很多,这里就不必多说,此篇文章主要在Swift环境下使用CocoaAsyncSocket库,来实现一个非常简单的Socket连接的例子,帮助大家理解和使用.(大神勿喷呀,哈哈)

### 1.创建Swift工程,并使用Cocoapods导入CocoaAsyncSocket

![iOS-CocoaAsyncSocket-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(Swift%E7%89%88)%20bca0f172a2ff4304a5718cb22e09fde3/1176193-e36c66a1bca6af34.png](iOS-CocoaAsyncSocket-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B(Swift%E7%89%88)%20bca0f172a2ff4304a5718cb22e09fde3/1176193-e36c66a1bca6af34.png)

1.创建工程

Cocoapods:

### 2.搭建服务端和客户端UI

- 服务端和客户端的界面搭建
    
    使用Tabbar分别创建服务端界面和客户端界面
    
- 分别创建服务端和客户端ViewController并关联相关Textfield 和 Button

### 4.服务端创建Socket绑定端口,设置监听,消息发送等方法

- ServerViewController

```swift
    //
    //  ServerViewController.swift
    //  TestSocket
    //
    //  Created by YYQ on 6/21/16.
    //  Copyright © 2016 YYQ. All rights reserved.
    //

    import UIKit
    import CocoaAsyncSocket

    class ServerViewController: UIViewController {
        //端口
        @IBOutlet weak var portTF: UITextField!

        //消息
        @IBOutlet weak var msgTF: UITextField!

        //信息显示
        @IBOutlet weak var infoTV: UITextView!

        //服务端和客户端的socket引用
        var serverSocket: GCDAsyncSocket?
        var clientSocket: GCDAsyncSocket?

        override func viewDidLoad() {
            super.viewDidLoad()

        }

        //对InfoTextView添加提示内容
        func addText(text: String) {
            infoTV.text = infoTV.text.stringByAppendingFormat("%@\n", text)
        }

        ///监听
        @IBAction func listeningAct(sender: AnyObject) {

            serverSocket = GCDAsyncSocket(delegate: self, delegateQueue: dispatch_get_main_queue())

            do {
                try serverSocket?.acceptOnPort(UInt16(portTF.text!)!)
                addText("监听成功")
            }catch _ {
                addText("监听失败")
            }

        }

        ///发送
        @IBAction func sendAct(sender: AnyObject) {
            let data = msgTF.text?.dataUsingEncoding(NSUTF8StringEncoding)
            //向客户端写入信息   Timeout设置为 -1 则不会超时, tag作为两边一样的标示
            clientSocket?.writeData(data, withTimeout: -1, tag: 0)
        }

    }

    extension ServerViewController: GCDAsyncSocketDelegate {

        //当接收到新的Socket连接时执行
        func socket(sock: GCDAsyncSocket!, didAcceptNewSocket newSocket: GCDAsyncSocket!) {

            addText("连接成功")
            addText("连接地址" + newSocket.connectedHost)
            addText("端口号" + String(newSocket.connectedPort))
            clientSocket = newSocket

            //第一次开始读取Data
            clientSocket!.readDataWithTimeout(-1, tag: 0)
        }

        func socket(sock: GCDAsyncSocket!, didReadData data: NSData!, withTag tag: Int) {
            let message = String(data: data,encoding: NSUTF8StringEncoding)
            addText(message!)
            //再次准备读取Data,以此来循环读取Data
            sock.readDataWithTimeout(-1, tag: 0)
        }

    }```
```

### 5.客户端创建Socket,设置连接,消息发送,读取,断开连接等方法

- ClientViewController

```swift

    import UIKit
    import CocoaAsyncSocket

    class ClientViewController: UIViewController {

        //IP地址
        @IBOutlet weak var ipTF: UITextField!

        //端口
        @IBOutlet weak var portTF: UITextField!

        //消息
        @IBOutlet weak var msgTF: UITextField!

        //信息显示
        @IBOutlet weak var infoTV: UITextView!

        var socket: GCDAsyncSocket?

        override func viewDidLoad() {
            super.viewDidLoad()

        }

        func addText(text: String) {
            infoTV.text = infoTV.text.stringByAppendingFormat("%@\n", text)
        }

        //连接
        @IBAction func connectionAct(sender: AnyObject) {
            socket = GCDAsyncSocket(delegate: self, delegateQueue: dispatch_get_main_queue())

            do {
                try socket?.connectToHost(ipTF.text, onPort: UInt16(portTF.text!)!)
                addText("连接成功")
            }catch _ {
                addText("连接失败")
            }
        }

        //断开
        @IBAction func disconnectAct(sender: AnyObject) {
            socket?.disconnect()
            addText("断开连接")
        }

        //发送
        @IBAction func sendMsgAct(sender: AnyObject) {
            socket?.writeData(msgTF.text?.dataUsingEncoding(NSUTF8StringEncoding), withTimeout: -1, tag: 0)
        }

    }

    extension ClientViewController: GCDAsyncSocketDelegate {

        func socket(sock: GCDAsyncSocket!, didConnectToHost host: String!, port: UInt16) {
            addText("连接服务器" + host)
            self.socket?.readDataWithTimeout(-1, tag: 0)
        }

        func socket(sock: GCDAsyncSocket!, didReadData data: NSData!, withTag tag: Int) {
            let msg = String(data: data, encoding: NSUTF8StringEncoding)
            addText(msg!)
            socket?.readDataWithTimeout(-1, tag: 0)
        }

    }
```

|服务端| 客户端 |
 |-----|:----:|----|

|||

### 结语:

CocoaAsyncSocket的使用还是非常直观清晰的,这里只是简单的抛砖引玉,要更深入理解Socket,如果有时间的话,我觉得还是研究下底层的socket实现比较好,大家有什么问题欢迎提出来,一起共勉~

如果本文对你有用，非常感谢能给个赞噢~~

- 推荐 ：

iOS进阶技术群：1080059793，讨论技术， 大家一起交流学习成长

申请即送： 各类资料免费领取，包括 BAT大厂面试题、数据结构、底层进阶、图形视觉、音视频、架构设计、逆向安防、RxSwift、flutter