# swift调用GCDAsyncSocket - 知乎

网络通信，GCDAsyncSocket一个基于TCP协议的第三方库。详情见代码注释很全。

```swift
import UIKit

class SocketManage: NSObject {

    static let shared = SocketManage()

    let serverPort: UInt16 = UInt16()//添加端口
    let serveripInput:String = ""//添加IP
    var diction : NSDictionary?
    var clientSocket:GCDAsyncSocket!
    var noti: Notification?//用于接收到服务器回传数据后发送成功通知

    fileprivate var timer: Timer?

    private override init() {
        super.init()
        self.timer = Timer.scheduledTimer(timeInterval: 5, target: self, selector:  #selector(timerAction), userInfo: nil, repeats: true)
        RunLoop.main.add(timer!, forMode: RunLoopMode.commonModes)
        self.timer?.fireDate = Date.distantFuture
    }

    /**
     *计时器每秒触发事件
     **/
    func timerAction() -> Void {
        sendMessage(self.diction as? [String : Any], type: 1001)
    }

    /**
     * 销毁计时器
     **/
    func destroyTimer(){
        self.timer?.invalidate()
        self.timer = nil
    }

    /**
     * 连接服务器
     **/

    func connectServer(){

        clientSocket = GCDAsyncSocket()
        clientSocket.delegate = self
        clientSocket.delegateQueue = DispatchQueue.global()
        do {
            try clientSocket.connect(toHost: serveripInput, onPort: serverPort)
        } catch {
            print("try connect error: \(error)")
        }

    }
    /**
     * 消息数据包装
     **/

    /**
     * 字段   Length         type           boby
     * 长度   Int:4byte     Int:4byte    jsonString
     * 释义   boby长度       自定义类型       数据内容
     * 以上信息不固定和服务端提前定好解析回传数据也一样
     *  type  1001
     **/
    func sendMessage(_ serviceDic:[String:Any]?,type:Int){

        //print("type---> \(type)")

        var bodyDatas = Data()
        if serviceDic != nil{

           bodyDatas = try! JSONSerialization.data(withJSONObject: serviceDic!,options: .prettyPrinted)

        }else{
           bodyDatas.count = 4
        }

        var b:UInt32 = CFSwapInt32HostToBig(UInt32(type))
        let typeData = Data(bytes: &b, count: 4)

        print("typeData---------\(typeData.description)")

        //Length

        var len:UInt32 = CFSwapInt32HostToBig(UInt32(bodyDatas.count + 1))
        let lengthData = Data(bytes: &len, count: 4)
        var byteLengthData = Data()
        for bytesss in lengthData.reversed() {
            byteLengthData.append(bytesss)
        }

        //向服务器进行写数据
        clientSocket.write(lengthData, withTimeout: -1, tag: 0)
        clientSocket.write(typeData, withTimeout: -1, tag: 0)
        clientSocket.write(bodyDatas, withTimeout: -1, tag: 0)
    }

}

extension SocketManage:GCDAsyncSocketDelegate{

    /**
     * 连接服务器 成功
     **/
    func socket(_ sock: GCDAsyncSocket, didConnectToHost host: String, port: UInt16) -> Void {

        print("Socket 连接服务器成功")
        //需要向服务器传递的数据
        let dic:[String : Any] = ["sessionId":"87878","udid":"HelloiOS"]
        //发送数据以及之前和服务器定好的类型
        sendMessage(dic, type: 1001)
        self.diction = dic as NSDictionary
        clientSocket.readData(withTimeout: -1, tag: 0)

        //连接成功后 启动定时器 发送心跳
        timer?.fireDate = Date.distantPast
    }

    /**
     * 连接服务器 失败
     **/
    func socketDidDisconnect(_ sock: GCDAsyncSocket, withError err: Swift.Error?) -> Void {
        //print("Socket 连接服务器失败: \(String(describing: err))")
        self.timer?.fireDate = Date.distantFuture
        connectServer()
    }

    /**
     * 处理服务器发来的消息
     **/

    func socket(_ sock: GCDAsyncSocket, didRead data: Data, withTag tag: Int) -> Void {

        //print("---Data Recv---")

        var byteArr:[UInt8] = []
        var lengthData:[UInt8] = []
        var typeBytesArr:[UInt8] = []
        var bodyBytesArr:[UInt8] = []
        var i = 0

        let type:Int = 0
        var length:UInt32 = 0
        for byte in data {
            if i < 4 {
                lengthData.append(byte)
            }
            let array : [UInt8] = lengthData
            let data = Data(bytes: array)
            length = UInt32(bigEndian: data.withUnsafeBytes { $0.pointee })
            if i >= 4 && i < 8{
                typeBytesArr.append(byte)
            }

            if i >= 8 && i < length + 7{
                bodyBytesArr.append(byte)
            }
            byteArr.append(byte)
            i += 1
        }

        //print("Bytes--->\(byteArr)")
        //print("lengthData ---> \(lengthData)")

        let bodyData = Data(bytes: bodyBytesArr, count: bodyBytesArr.count)
        //将二进制流转化为json数据
        let bodyMs = JSON(bodyData)

        let bodyDic = bodyMs.dictionaryObject
        if bodyDic != nil{
            print("\(data)")
            print("body -->\(bodyMs)")
        }

        if type == 6{
            //print(" ")
        }
        //解析完数据 发送通知 对应地方接收处理
        noti = Notification(name:NSNotification.Name(rawValue: "SocketManageNotification"), object: nil, userInfo: bodyDic)
        NotificationCenter.default.post(noti!)
        //每次读完数据后，都要调用一次监听数据的方法
        clientSocket.readData(withTimeout: -1, tag: 0)
    }

}
```

[https://www.zhihu.com/api/v3/account/api/login/qrcode/YjA5ZDI0ZmItYWQ3/image](https://www.zhihu.com/api/v3/account/api/login/qrcode/YjA5ZDI0ZmItYWQ3/image)