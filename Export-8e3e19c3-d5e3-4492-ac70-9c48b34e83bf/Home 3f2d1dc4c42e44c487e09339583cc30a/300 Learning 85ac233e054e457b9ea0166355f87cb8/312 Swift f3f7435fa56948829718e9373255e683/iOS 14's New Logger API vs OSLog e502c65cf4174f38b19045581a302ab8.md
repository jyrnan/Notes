# iOS 14's New Logger API vs. OSLog

### At WWDC 20, Apple announced a new unified logging API to gather, process log messages, and help debug unexpected behavior

Photo by [William Hook](https://unsplash.com/@williamtm?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/mobile-phone?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

# Let’s Start OSLog vs Logger

We will go through in short with migration from existing `print` logging to the new Apple Logging API.

Then we will revisit the `OSLog` API and check differences between existing old `OSLog` and new iOS 14 `Logger` APIs.

To start off, let’s convert a simple print statement from this, which we use often in our project to log messages:

```swift
print(“Start network call”)
```

to this:

```
let log = OSLog.init(subsystem: "com.mandy.loggingdemo", category: "main")
os_log("Start network call", log: log)
```

Remember to use `os_log` API for this; you have to write `import os.log` statement. The unified logging system has been Apple’s standard for logging since iOS 10. (It was [announced at WWDC 2016](https://developer.apple.com/videos/play/wwdc2016/721/).)

You can consider **subsystem** as a unique bundle id of your app and **category** as app modules.

- subsystem： 包名
- category： 类名

## iOS14 new Logger

Now, in iOS 14, Apple has introduced a new `Logger` API.

 To use this API, we have to write **same** `import os.log` statement, but the syntax is quite different to `os_log` API.

**同样的引用方式，但是语法不一样了**

```swift
let logger = Logger.init(subsystem: "com.mandy.loggingdemo", category: "main")
logger.debug("Start network call")
```

You can view the `os_log` and `Logger` logs in Xcode console in debug mode and also in the Mac Console app.

The Console app is very powerful; I would recommend using it as you can filter the values using **subsystem** and **category** and debug your app more extensively.

It also helps when you want to debug issues of device and export the **logarchive**. The archive file opens in Console and helps you determine the issue. Below is an example in which we have logged iPhone 11 Pro and filtered the simple log with subsystem *com.mandy.loggingdemo.*

Both `os_log` and `Logger` API have most of the same benefits compared to `print` statements. In this article we will mainly focus on their usage and the differences between the two APIs. I have also added some good references at the end of article to walk you through `os_log` and `Logger` API. Let’s deep dive into it.

# Benefits of Unified Logging API

- Low performance overhead;
- String interpolation (starting from iOS 14, `os_log` also supports this feature);
- Different logging type (debug, info, warning, error, fault, notice) for different usage;
- Persistent and efficient logging — we can debug even the rare case scenarios bugs from devices and fix them;
- Many formatting and expressive options with no cost at run time; and
- Visibility options.

Let’s look at some key differences in action.

# String Interpolation and Formatting Strings

`os_log` accept formatted strings prior to iOS 14:

```
let count = 0
let log = OSLog.init(subsystem: "com.mandy.loggingdemo", category: "main")
os_log("Start %d network call", log: log, type: OSLogType.debug, count)
```

***Note**: All string interpolation methods are also available in* `os_log`, *starting from iOS 14 onwards.*

<aside>
💡 新的logger支持string语法，更加swift化

</aside>

The new `Logger` API supports string interpolation out of box, more like Swift-style compared to old C- style formatting. Some examples are

- Numeric data type such as Int and Double;
- Objective-C objects with description; and
- Any type conforming to `CustomStringConvertible`.

```swift
let count = 0
let logger = Logger.init(subsystem: “com.mandy.loggingdemo”, category: “main”)
logger.debug(“Start \(count) network call”)
```

# Different Logging Types 不同类型

Both `os_log` and `Logger` support different logging types. Below are some examples of types and their purposes.

![Untitled](iOS%2014's%20New%20Logger%20API%20vs%20OSLog%20e502c65cf4174f38b19045581a302ab8/Untitled.png)

<aside>
💡 类型的语法区别就是旧的当参数，而新的直接用不同的方法

</aside>

### Prior to iOS 14,

we used `os_log` to declare it with `type` parameter. It was `OSLogType` enum:

```swift
let count = 0
let log = OSLog.init(subsystem: "com.mandy.loggingdemo", category: "main")
os_log("Start %d network call", log: log, type: OSLogType.debug, count)
```

### From iOS 14 onward,

Apple has made the Logger API more developer-friendly, direct, and simple to promote its usage.

![Untitled](iOS%2014's%20New%20Logger%20API%20vs%20OSLog%20e502c65cf4174f38b19045581a302ab8/Untitled%201.png)

```swift
let count = 0
let logger = Logger.init(subsystem: “com.mandy.loggingdemo”, category: “main”)
logger.debug(“Start \(count) network call”)
```

While we are on this topic, let’s look at the different properties of these logging types, which will come in handy when using them in your app.

## Persistent 持久化方式

These log messages are stored in disks based on their type. 

- Debug is not persisted,  不保存
- info is only persisted during `log collect` option,  部分保存
- and the rest are persisted until storage limit. 其他都保存
- 

![Untitled](iOS%2014's%20New%20Logger%20API%20vs%20OSLog%20e502c65cf4174f38b19045581a302ab8/Untitled%202.png)

## Performance

![Untitled](iOS%2014's%20New%20Logger%20API%20vs%20OSLog%20e502c65cf4174f38b19045581a302ab8/Untitled%203.png)

## Visibility 可见度

考虑隐私信息的保存方式

As messages are logged, even when app is deployed, they can be viewed if a person has physical access to the device. Therefore, it’s the developer’s responsibility to not mark this sensitive info public.

In `os_log` we can set visibility with formatting options like the one below. We have set `%{private}s` as private in the same way we can set this as public. By default in `os_log`, dynamic values are private.

```
let log = OSLog.init(subsystem: “com.mandy.loggingdemo”, category: “main”)
 os_log(“Bank account number %{private}s, “, log: log, type: .info, 00011112222)
```

### Logger iOS14

Let’s look in `Logger`. This API gives us some more customization options apart from visibility option. By default in `Logger`, dynamic values are private.

We can set the value as public or private with the help of privacy parameters like `os_log`.

```
logger.log(“Bank account number \(XXXXXXXX, privacy: .private)”)
```

We can also mask the sensitive info:

# Log Archives

Consider a scenario in which you want to debug some rare-case scenario error or bug that doesn’t occur on your desktop computer or while debugging. Let say your QA has faced some unique issue, and he doesn’t know the exact steps to reproduce it.

We can connect his device, export the log archive of your app from the time when the error occurred, and open in Console to debug this scenario, provided we have added appropriate logging in our app.

## Command to export the archive

```
log collect --device --start '2020-06-22 9:41:00' --output yourapp.logarchive
```

Open this archive in Console and filter using subsystem and category to find the issue.

I hope this article will help you understand the power of Logger API and helps you realize that now is the time we should switch our projects from traditional logging mechanisms to new Apple APIs for better and reliable debugging.

# References and Good Reads