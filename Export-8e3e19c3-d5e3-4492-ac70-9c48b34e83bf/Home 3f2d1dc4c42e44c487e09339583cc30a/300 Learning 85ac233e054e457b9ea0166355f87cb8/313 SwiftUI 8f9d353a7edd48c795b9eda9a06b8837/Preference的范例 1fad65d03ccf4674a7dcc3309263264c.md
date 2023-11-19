# Preference的范例

这样操作以后，TextView的实际尺寸通过GeometryReader传递，利用preferenceKey从下往上传递回@State width变量，从而再利用frame()来显性设置高度采用实际宽度，这样就可以实现TextView的实际ViewSize是一个正方形，不会产生部分重叠（最后图片）。

![Untitled](Preference%E7%9A%84%E8%8C%83%E4%BE%8B%201fad65d03ccf4674a7dcc3309263264c/Untitled.png)

## 利用PreferenceKey可以将数值从下往上传递

先定义一个PreferenceKey的结构

> **因为其采用泛型，所以其类型很关键?**🤔
> 

```swift
struct WidthKey: PreferenceKey {
    static let defaultValue: CGFloat? = nil
    
    static func reduce(value: inout CGFloat?, nextValue: () -> CGFloat?) {
        value = value ?? nextValue()
    }
    
}
```

# 范例应用

先定义一个状态变量用来控制View的状态，在PreferenceKey改变后会写入到这个变量

```swift
@State **var** width: CGFloat? = **nil**
```

在视图中利用PreferenceKey获取相应数值并写入到状态变量从而实现从下往上传递数据

```swift
Text("hello")
    .padding(10)
    .foregroundColor(.white)
    .background(
        ///此处利用GeometryReader获取TextView所给予的实际空间
        GeometryReader {proxy in
            ///利用一个Color.clear来生成一个Key，并将获得的TextView宽度赋予数值到Key 中
            Color.clear.preference(key: WidthKey.self, value: proxy.size.width)
        }
    )
///观察Key数值改变，执行闭包内容，赋值到@State变量中
    .onPreferenceChange(WidthKey.self, perform: { value in
        self.width = value
    })
///获取@State变量来设置宽度
    .frame(width: width, height: width
           , alignment: .center)
    .background(Circle())
```

如果不采用这种方式会出现如下图的情况

![Untitled](Preference%E7%9A%84%E8%8C%83%E4%BE%8B%201fad65d03ccf4674a7dcc3309263264c/Untitled%201.png)