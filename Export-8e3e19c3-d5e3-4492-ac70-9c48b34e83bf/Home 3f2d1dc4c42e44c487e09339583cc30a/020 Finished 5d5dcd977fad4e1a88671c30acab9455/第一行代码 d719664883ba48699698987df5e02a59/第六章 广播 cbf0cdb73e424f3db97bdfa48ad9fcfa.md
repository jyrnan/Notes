# 第六章 广播

# 6.1 广播机制简介

- 标准广播
- 有序广播

# 6.2 接受系统广播

## 6.2.1 动态注册监听时间变化

注册 BroadcastReceiver的方式一般有两种:

- 在代码中注册： 动态注册
    
    动态注册的BroadcastReceiver可以自由地控制注册与注销，在灵活性方面有很大的优势。但是它存在着一个缺点，即必须在程序启动之后才能接收广播，因为注册的逻辑是写在 onCreate()方法中的。
    
- AndroidManifest.xml中注册：静态注册。
    
    那么有没有什么办法可以让程序在未启动的情况下也能接收广播呢? 这就需要使用静态注册的方式了。
    

# 6.3 发送自定义广播

### 发送顺序广播

如何设定BroadcastReceiver的先后顺序呢?当然是在注册的时候进行设定了，修改 AndroidManifest.xml中的代码，通过`android:priority`属性给BroadcastReceiver设置了优先级来决定传播顺序

```swift
<receiver
            android:name=".MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter android:priority="100">
                <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
            </intent-filter>
</receiver>
```

在Receiver类中代码方法来决定是否中断

```swift
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "received in MyBroadcastReceiver",
                  Toast.LENGTH_SHORT).show()
        abortBroadcast() // 中断传播
} }
```