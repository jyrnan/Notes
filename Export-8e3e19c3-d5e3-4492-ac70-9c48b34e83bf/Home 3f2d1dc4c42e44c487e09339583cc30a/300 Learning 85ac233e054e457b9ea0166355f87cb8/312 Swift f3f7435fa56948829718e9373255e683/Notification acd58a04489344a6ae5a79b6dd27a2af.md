# Notification

创建通知中心
设置监听方法
设置通知的名字

```swift
NotificationCenter.default.addObserver(self, selector: #selector(test), name: NSNotification.Name(rawValue:"isTest"), object: nil)
```

实现通知监听方法

```swift
@objc func test(nofi : Notification){
        let str = nofi.userInfo!["post"]
        print(String(describing: str!) + "this notifi")
    }
```

点击发送通知

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        NotificationCenter.default.post(name: NSNotification.Name("isTest"), object: self, userInfo: ["post":"NewTest"])
    }
```

最后要记得移除通知

```swift
deinit {
        /// 移除通知
        NotificationCenter.default.removeObserver(self)
    }
```