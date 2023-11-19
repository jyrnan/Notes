# 自定义ButtonStyle来对Button外观和行为进行自定义

[使用 ButtonStyle 设计按钮](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c.md)

<aside>
💡 自定义ButtonStyle总结就是，
一个方法：`makeBody(configuration: )`
三个属性：

- `. label` 属性，你可以应用修饰器来修改按钮的样式。
- `.isPressed`属性，可以根据这个属性来动态调整返回的View
- `.role`属性，只读属性，用于判断
</aside>

# 定义方法

SwiftUI 提供了一个名为 `ButtonStyle`的协议，可以让你建立自己的按钮样式。

**该协议要求我们提供接受 `configuration` 参数的 makeBody 函数的实现，configuration 参数多个属性，我们就是根据这些属性进行调整，返回一个新的View**

- . label 属性，你可以应用修饰器来修改按钮的样式。
- .isPressed属性，可以根据这个属性来动态调整返回的View
- .role属性
    
    ![Untitled](%E8%87%AA%E5%AE%9A%E4%B9%89ButtonStyle%E6%9D%A5%E5%AF%B9Button%E5%A4%96%E8%A7%82%E5%92%8C%E8%A1%8C%E4%B8%BA%E8%BF%9B%E8%A1%8C%E8%87%AA%E5%AE%9A%E4%B9%89%20fdbfc1ac12c645b4a7eb4f073f14e9d5/Untitled.png)
    

## label属性

SwiftUI 提供了一个名为 `ButtonStyle`的协议，可以让你建立自己的按钮样式。要为我们的按钮建立一个可以重复使用的样式， 则我们建立一个名为 `Gradient BackgroundStyle` 的新结构，该结构遵守 ButtonStyle 协议。在 `struct ContentPreview_Previews`上方，插入下列这段代码：

```swift
struct GradientBackgroundStyle: ButtonStyle {

    func makeBody(configuration: Self.Configuration) -> some View {
        configuration.label
            .frame(minWidth: 0, maxWidth: .infinity)
            .padding()
            .foregroundColor(.white)
            .background(LinearGradient(gradient: Gradient(colors: [Color("DarkGreen"), Color("LightGreen")]), startPoint: .leading, endPoint: .trailing))
            .cornerRadius(40)
            .padding(.horizontal, 20)
    }
}

```

## isPressed属性

你也可以通过 configuration 的 `isPressed 属性`，来判断按钮是否有被按下，这可以让你在使用者按下它时改变按钮的样式。

举例而言，我们想要制作一个当使用者按下去时会变小一点的按钮，你可以加入一行程序，如图 6.23 所示。

![../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-23.png](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/SwiftUI%20%E6%8C%89%E9%92%AE%E4%B8%8E%E6%B8%90%E5%B1%82%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20e846a7e5dd64491ea44caef057045c2c/swiftui-buttons-23.png)

## role属性

这是一个let属性，所以不能改变，只能对其进行读取，判断来进行一些操作

```swift
struct MyButtonStyle: ButtonStyle {
  func makeBody(configuration: Configuration) -> some View {
   
    if configuration.role == .destructive {
      configuration.label
        .rotationEffect(configuration.isPressed ? Angle(degrees: 90) : Angle(degrees: 0))
    } else {
      configuration.label
    }
  }
}
```

# 使用方法

```swift
.buttonStyle(GradientBackgroundStyle())
```