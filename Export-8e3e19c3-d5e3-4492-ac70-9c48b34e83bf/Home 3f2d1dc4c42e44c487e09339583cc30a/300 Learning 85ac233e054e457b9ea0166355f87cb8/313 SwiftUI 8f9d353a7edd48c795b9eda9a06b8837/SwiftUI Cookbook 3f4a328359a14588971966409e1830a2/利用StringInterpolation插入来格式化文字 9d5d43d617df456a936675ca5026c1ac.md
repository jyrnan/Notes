# 利用StringInterpolation插入来格式化文字

这个方法的主要方式是可以在一个Text视图的内容里插入更多的Text视图，并对其进行格式化来实现更丰富的视觉效果

## Text直接插入

```swift
Section("Multiple text views") {
   Text("\(Text("**Summer**").foregroundColor(.orange)) is a great time to visit **Vancouver**. Make sure you check out \(Text("[Grouse mountain](https://grousemountain.com)"))")
                        .tint(.green)
```

![Untitled](%E5%88%A9%E7%94%A8StringInterpolation%E6%8F%92%E5%85%A5%E6%9D%A5%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%96%87%E5%AD%97%209d5d43d617df456a936675ca5026c1ac/Untitled.png)

## Text中插入SFSymbol Image

```swift
LabeledContent("String Interpolation"){
    Text("Paint with \(Image(systemName: "paintpalette.fill").symbolRenderingMode(.multicolor))")
                    }
```

![Untitled](%E5%88%A9%E7%94%A8StringInterpolation%E6%8F%92%E5%85%A5%E6%9D%A5%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%96%87%E5%AD%97%209d5d43d617df456a936675ca5026c1ac/Untitled%201.png)

### 💡这里需要关注两个用法：

- labeledContent可以用在Form里面显示类似于Label的效果，标签+内容
- SFSymbol可以通过`.symbolRenderingMode`来实现多种色彩的显示模式

## 利用潜入的更多用法

```swift
let palette = Image(systemName: "paintpalette.fill").symbolRenderingMode(.multicolor)
let introString = Text("Paint with").foregroundColor(.red)
let largePalette = Text(palette).font(.largeTitle)

//结合嵌套的用法
LabeledContent("String Interpolation with style"){
                        Text("Paint with \(Text(palette).font(.largeTitle))")
                    }
LabeledContent("Even better") {
                        Text("\(introString) \(largePalette)")
                    }
```

效果：

![Untitled](%E5%88%A9%E7%94%A8StringInterpolation%E6%8F%92%E5%85%A5%E6%9D%A5%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%96%87%E5%AD%97%209d5d43d617df456a936675ca5026c1ac/Untitled%202.png)