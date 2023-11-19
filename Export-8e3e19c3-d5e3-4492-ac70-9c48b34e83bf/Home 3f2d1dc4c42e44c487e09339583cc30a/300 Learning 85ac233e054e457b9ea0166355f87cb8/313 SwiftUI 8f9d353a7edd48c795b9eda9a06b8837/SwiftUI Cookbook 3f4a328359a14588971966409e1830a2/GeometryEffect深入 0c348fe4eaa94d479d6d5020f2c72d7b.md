# GeometryEffect深入

# isSource

有些情况下动画的两个参与者可能同时出现在画面当中，这样会导致报错。

通过这个参数决定了View是从哪一个变成哪一个。（另一方法是想办法去掉一个）

`isSource`: false变成true。或者说false 去match `isSource:true`

![Untitled](GeometryEffect%E6%B7%B1%E5%85%A5%200c348fe4eaa94d479d6d5020f2c72d7b/Untitled.png)

# properties

决定了变化的内容是什么。包含了三个选项：

- frame 默认值，包含了下面两个的内容
- position
- size

# anchor

控制变形的锚点位置。例如，位置

[MatchedGeometryEffect in SwiftUI](https://www.youtube.com/watch?v=xGNR7tvDE0Q)