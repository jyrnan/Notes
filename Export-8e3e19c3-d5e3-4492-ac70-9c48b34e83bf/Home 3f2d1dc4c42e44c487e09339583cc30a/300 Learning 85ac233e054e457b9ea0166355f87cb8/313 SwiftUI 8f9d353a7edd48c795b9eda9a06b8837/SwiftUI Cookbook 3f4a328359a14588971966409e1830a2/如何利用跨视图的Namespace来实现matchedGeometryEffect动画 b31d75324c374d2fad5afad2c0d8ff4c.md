# 如何利用跨视图的Namespace来实现matchedGeometryEffect动画

<aside>
💡 - 通过`Namespace.ID` 来传递NameSpace
- .matchedGeometryEffect作用的位置

</aside>

# 通过通过`Namespace.ID` 来传递

matchedGeometryEffect需要一个关键参数： Namespace，但是如果动画的两个元素分别处于不同的视图，则需要将NameSpace参数传递过去，以保证它们能在同一个NameSpace，反方是通过`Namespace.ID` 来传递

```swift
/// 父视图定义
@Namespace var transition

///子视图定义
var transition: Namespace.ID

///传递参加
SubViewA(transition: transition)
```

# .matchedGeometryEffect作用的位置

**作用的位置不同，效果是不一样的。**

<aside>
💡 目前看起来应该是放在.frame之前了
原因是主要都是通过frame变化来实现动画[properties](GeometryEffect%E6%B7%B1%E5%85%A5%200c348fe4eaa94d479d6d5020f2c72d7b.md)

</aside>

应当尽量靠近真正产生变化的View，例如下面实例中，.matchedGeometryEffect作用的位置必须是尽量靠近RoundedRectangle(cornerRadius: 64)， 这样形状的变化才会明显。

如果放置在.frame(maxHeight: .infinity, alignment: .bottom)后面，则是针对整个 frame来产生作用了，所以动画变化的基础就不一样了。

完整代码：

```swift
import SwiftUI

struct GeometryEffectView: View {
  @Namespace var transition
  @State var show: Bool = true
    var body: some View {
      ZStack {
        if show {
          SubViewA(transition: transition)
            
        }
        if !show {
          SubViewB(transition: transition)
            
        }
        Button("exchange") {
          withAnimation {
            show.toggle()
          }
          
        }
      }
    }
}

struct GeometryEffectView_Previews: PreviewProvider {
    static var previews: some View {
        GeometryEffectView()
    }
}

struct SubViewA: View {
  var transition: Namespace.ID
  var body: some View {
    RoundedRectangle(cornerRadius: 64)
      .fill(Color.green)
      .matchedGeometryEffect(id: "icon", in: transition)
      .frame(width: 128, height: 128)
      .frame(maxHeight: .infinity, alignment: .top)
      
  }
}
struct SubViewB: View {
  var transition: Namespace.ID
  var body: some View {
    RoundedRectangle(cornerRadius: 16)
      .fill(Color.green)
      .matchedGeometryEffect(id: "icon", in: transition)
      .frame(width: 200, height: 200)
      .frame(maxHeight: .infinity, alignment: .bottom)
      //.matchedGeometryEffect(id: "icon", in: transition) 不能放这里，否则就是针对整个frame来变换，动画效果就不明显了。
  }
}
```

[视觉效果](%E5%A6%82%E4%BD%95%E5%88%A9%E7%94%A8%E8%B7%A8%E8%A7%86%E5%9B%BE%E7%9A%84Namespace%E6%9D%A5%E5%AE%9E%E7%8E%B0matchedGeometryEffect%E5%8A%A8%E7%94%BB%20b31d75324c374d2fad5afad2c0d8ff4c/Simulator_Screen_Recording_-_iPhone_14_Pro_Max_-_2023-06-11_at_13.31.32.mp4)

视觉效果

# 传递Namespace在Preview中实现方法

```swift
@Namespace static var siteNS
```

![Untitled](%E5%A6%82%E4%BD%95%E5%88%A9%E7%94%A8%E8%B7%A8%E8%A7%86%E5%9B%BE%E7%9A%84Namespace%E6%9D%A5%E5%AE%9E%E7%8E%B0matchedGeometryEffect%E5%8A%A8%E7%94%BB%20b31d75324c374d2fad5afad2c0d8ff4c/Untitled.png)