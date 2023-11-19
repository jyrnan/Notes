# Apple File System

🤔 it shows how to support the `Open-in-Place` feature, which allows the user to launch an app by tapping a document in Files, and then edit it directly.

也就是点击文件直接打开的模式

范例文件：

# FileManager

- 尽量使用URL方法，而不是用path：String方法
- 通过FileManager.urls 方法获得 url，再apending(component:) 来获得完整URL
- 如果有必要用 URL.path() 获得 path，也就是String
- 注意两个方法最后斜杠的区别，所以添加最后的文件名时候要注意是否添加斜杠
    
    ```swift
    print(urlForDocument.path()) //输出 /Users/jyrnan/Documents/
    print(NSHomeDirectory()) //输出 /Users/jyrnan
    ```
    
    # FileHandle