# 绘制边框 boarder() or stroke()

Boarder是给一个普通的View加上边框，Stroke是个一个Shape的路径着色

- SwiftUI gives us a dedicated **border()** modifier to draw borders around views.
- 但是如果需要的是带圆角的边框则需要用**overlay()**来实现

SwiftUI gives us both **stroke()** and **strokeBorder()** modifiers for drawing borders around shapes, and they have subtly different behavior:但是他们都只能用于shape，而不能用于Text或Image。二者也有区别，后者是全部在图形内绘制其宽度，而前者有一半宽度是在图形外绘制

- SwiftUI’s stroke can be configured with dash and dash phase options that gives us very fine-grained control over how the line is drawn. For example, we could draw a box with a dashed stroke like this:还可以画虚线。

![struct Contentyiew View .png](%E7%BB%98%E5%88%B6%E8%BE%B9%E6%A1%86%20boarder()%20or%20stroke()%2056aacd38bfbf47769a0f4d618fbe9fd2/struct_Contentyiew_View_.png)