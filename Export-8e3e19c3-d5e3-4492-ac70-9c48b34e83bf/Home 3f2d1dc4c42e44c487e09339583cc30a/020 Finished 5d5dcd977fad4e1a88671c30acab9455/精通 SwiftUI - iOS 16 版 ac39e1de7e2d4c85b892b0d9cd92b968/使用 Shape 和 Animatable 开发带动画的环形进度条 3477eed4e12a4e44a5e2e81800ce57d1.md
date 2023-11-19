# 使用 Shape 和 Animatable 开发带动画的环形进度条

iPhone 内置的“活动”App 使用三个环形进度条来显示你的*移动*、*锻炼*和*站立*的进度。 这种进度条又被称为活动环。 如果你未曾使用过“活动”App或者你不知道什么是活动环，请查看图 30.1。 自 Apple Watch 使用环形进度条后，这设计渐渐成为流行的 UI 模式。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-1.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-1.png)

图 30.1. 环形进度条

在本章中，我们将深入讲解环形进度条并使用 SwiftUI 构建一个类似的活动环。 我们的目标不仅仅是创建一个静态活动环，而是带有动画的。图 30.2 示范了最终完成品的动画效果。或者你可以在 [https://link.appcoda.com/progressring](https://link.appcoda.com/) 上查看示范。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-2.gif](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-2.gif)

图 30.2. 带动画的环形进度条

### 创建新项目

让我们创建一个新项目来构建这个环形进度指示器。 像往常一样，请使用 *App* 模板， 将其命名为 *SwiftUIProgressRing* 或你喜欢的任何名称。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-3.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-3.png)

图 30.3. 使用 App 模板创建新项目

为了更好地整理我们的代码，使用 SwiftUI 视图模板建立一个新文件并将其命名为`ProgressRingView.swift`。 完成后，Xcode 应该使用自动加入以下代码：

```
import SwiftUI

struct ProgressRingView: View {
    var body: some View {
        Text("Hello, World!")
    }
}

struct ProgressRingView_Previews: PreviewProvider {
    static var previews: some View {
        ProgressRingView()
    }
}

```

### 分析活动环的实现

在我们深入实现之前，请再次查看图 30.1 和图 30.2。 你应该会发现，一个活动环实际上是由两个或多个圆形进度条组成的。 所以，我们需要构建一个圆形进度条视图，可以灵活地显示指定的百分比值，并允许使用者调整进度条的宽度和颜色。

例如，如果你告诉条形视图以红色显示 60% 的进度并将其宽度设置为 250 点。 循环进度视图应显示如下内容：

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-4.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-4.png)

图 30.4. 圆形进度条例子

通过构建圆形进度条视图，开发活动环就变得非常容易。 例如，我们可以在图 30.4 所示的上面叠加另一个尺寸更大、颜色不同的圆形进度条，就成为一个活动环。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-5.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-5.png)

图 5. 活动环示范

这就是我们将如何构建活动环的方式。 现在让我们正式开始开发圆形进度条！

### 准备颜色扩展

如前所述，我们将要实现的圆形进度条，而这进度条可以灵活地支持多种颜色和渐变。 为了这个示范，我们将使用 `Color` 扩展来准备一组预定颜色。 在项目导航器中，右键单击 *SwiftUIProgressRing* 并选择 *New file...*， 选择 *Swift 文件* 模板并将文件命名为 `Color+Ext.swift`。 将文件内容替换为以下代码：

```
import SwiftUI

extension Color {

    public init(red: Int, green: Int, blue: Int, opacity: Double = 1.0) {
        let redValue = Double(red) / 255.0
        let greenValue = Double(green) / 255.0
        let blueValue = Double(blue) / 255.0

        self.init(red: redValue, green: greenValue, blue: blueValue, opacity: opacity)
    }

    public static let lightRed = Color(red: 231, green: 76, blue: 60)
    public static let darkRed = Color(red: 192, green: 57, blue: 43)
    public static let lightGreen = Color(red: 46, green: 204, blue: 113)
    public static let darkGreen = Color(red: 39, green: 174, blue: 96)
    public static let lightPurple = Color(red: 155, green: 89, blue: 182)
    public static let darkPurple = Color(red: 142, green: 68, blue: 173)
    public static let lightBlue = Color(red: 52, green: 152, blue: 219)
    public static let darkBlue = Color(red: 41, green: 128, blue: 185)
    public static let lightYellow = Color(red: 241, green: 196, blue: 15)
    public static let darkYellow = Color(red: 243, green: 156, blue: 18)
    public static let lightOrange = Color(red: 230, green: 126, blue: 34)
    public static let darkOrange = Color(red: 211, green: 84, blue: 0)
    public static let purpleBg = Color(red: 69, green: 51, blue: 201)
}

```

在上面的代码中，我们建立了一个 `init` 方法，它提供 `red`、`green` 和 `blue` 的参数。 这使得使用 RGB 颜色代码初始化 Color 实体变得更加容易。 所有颜色均来自平面调色板 ([https://flatuicolors.com/palette/defo)，](https://flatuicolors.com/palette/defo)%EF%BC%8C) 如果你喜欢使用其他颜色，你可以简单地修改颜色值。

### 实现圆形进度条

参考图 30.4，圆形进度条实际上由两个圆圈组成：下方为灰色的完整圆圈，上方为渐变色的另一个部分（或完整）圆圈。 因此，为了实现进度条，我们需要一个`ZStack`来覆盖两个视图：

1. 灰色的圆形视图
2. 位于 #1 上层的渐变色环形

现在打开 `ProgressRingView.swift` 并声明以下变量：

```
var thickness: CGFloat = 30.0
var width: CGFloat = 250.0

```

由于这个圆形进度条应该支持各种大小，上面的变量都有一个默认值。 顾名思义，`thickness` 变量控制进度条的粗细，而 `width` 变量储存圆的直径。

你可以使用内置的 `Circle` 视图创建圆形视图，如下所示：

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-6.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-6.png)

图 30.6. 绘制圆形视图

我们使用 `stroke` 修饰器来绘制灰色圆圈的轮廓。 如图所示，`thickness` 属性用于控制轮廓的宽度， `width` 属性是圆的直径。 我故意突出框架，以便你可以看到厚度和宽度。

接下来，我们将实现环形。 创建这种环形的一种方法是使用`Circle`。 我们已经在第 8 章讨论过画圆。这一次，让我向你展示另一种实现方式。 我们将使用`Shape`协议来创建一个客制化的环形。

在同一文件中，插入以下代码：

```
struct RingShape: Shape {
    var progress: Double = 0.0
    var thickness: CGFloat = 30.0

    func path(in rect: CGRect) -> Path {

        var path = Path()

        path.addArc(center: CGPoint(x: rect.width / 2.0, y: rect.height / 2.0),
                    radius: min(rect.width, rect.height) / 2.0,
                    startAngle: .degrees(0),
                    endAngle: .degrees(360 * progress), clockwise: false)

        return path.strokedPath(.init(lineWidth: thickness, lineCap: .round))
    }
}

```

我们通过采用`Shape`协议创建了一个`RingShape`结构， 在结构中声明了两个属性。 `progress` 属性允许用户指定进度百分比，手 `thickness` 属性，类似于 `ProgressRingView` 中的属性，可让你控制环的宽度。

要绘制圆环，我们使用 `addArc` 方法，然后使用 `strokedPath`。 弧的半径可以通过将框架的宽度（或高度）除以 2 来计算。初始角度当前设定为零度。 而结束角度（ending angle），我们就将 360 乘以进度值来计算。 例如，如果我们将 `progress` 设定为 0.5，就会绘制一个半环（从 0 到 180 度）。

要使用 `RingShape`，你可以像这样修改 `body` 变量：

```
ZStack {
    Circle()
        .stroke(Color(.systemGray6), lineWidth: thickness)

    RingShape(progress: 0.5, thickness: thickness)
 }
.frame(width: width, height: width, alignment: .center)

```

进行修改后，你应该会在灰色圆圈的顶部看到部分环形覆盖。 请注意，由于我们将 `strokedPath` 的 `lineCap` 参数设为 `.round`，所以它的两端都是圆帽形。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-7.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-7.png)

图 30.7. 显示 RingShape

除了环的颜色，你可能还会注意到我们需要调整的一些代码。 圆弧的起点与图 30.4 是不同的。要解决此问题，你需要将 `startAngle` 从0修改为 -90。

在 `RingShape` 中声明以下属性：

```
var startAngle: Double = -90.0

```

然后像这样修改 `addArc` 方法：

```
path.addArc(center: CGPoint(x: rect.width / 2.0, y: rect.height / 2.0),
            radius: min(rect.width, rect.height) / 2.0,
            startAngle: .degrees(startAngle),
            endAngle: .degrees(360 * progress + startAngle), clockwise: false)

```

我们将 `startAngle` 参数修改为 `-90` 度数。 另外，还需要修改 `endAngle` 参数，因为初始角度已修改。 修改后，圆弧现在就逆时针旋转 90 度。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-8.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-8.png)

图 30.8。 改变初始角度后的局部环

### 添加渐变色

现在你有了一个可以显示不同的进度的环，如果每条进度条都能显示渐变颜色，不就更好吗？ SwiftUI 提供了三种类型的渐变，包括线性渐变、角度渐变和径向渐变。 Apple 使用角度渐变来为进度条加进渐变效果。

这是一个使用 `AngularGradient` 的示例：

```
AngularGradient(gradient: Gradient(colors: [.darkPurple, .lightYellow]), center: .center, startAngle: .degrees(0), endAngle: .degrees(180))

```

角度渐变是随着角度的变化做出不同的渐变颜色。 在上面的代码中，我们将渐变从 0 度渲染到 180 度。 图 30.9 显示了两种不同角度的结果。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-9.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-9.png)

图 30.9. 具有不同开始和结束角度的渐变效果

由于环形的初始角度设定为 -90 度，我们将像这样应用角度渐变（假设进度设置为 0.5）：

```
AngularGradient(gradient: Gradient(colors: [.darkPurple, .lightYellow]), center: .center, startAngle: .degrees(startAngle), endAngle: .degrees(360 * 0.5 + startAngle))

```

现在让我们修改代码为`RingShape`加入渐变效果。 首先，在 `ProgressRingView` 中声明以下属性：

```
var gradient = Gradient(colors: [.darkPurple, .lightYellow])
var startAngle = -90.0

```

然后通过附加 `.fill` 修饰器，为 `RingShape` 填入渐变色，如下所示：

```
RingShape(progress: 0.5, thickness: thickness)
    .fill(AngularGradient(gradient: gradient, center: .center, startAngle: .degrees(startAngle), endAngle: .degrees(360 * 0.5 + startAngle)))

```

完成修改后，圆形进度条就即时显示渐变效果。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-10.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-10.png)

图 30.10. 带有渐变的圆形进度条

### 改变进度

进度百分比现在固定为 0.5。 不用多说，你也知道我们需要为此建立一个变量以使其可以随时调整。 在 `ProgressRingView` 中，声明一个名为 `progress` 的变量，如下所示：

```
@Binding var progress: Double

```

我们开发的 `ProgressRingView` 是能够让使用者控制进度百分比。 因此，进度是应该由使用者提供。 这就是为什么 `progress` 被标记为绑定变量的原因。

要使用该变量，我们可以相应地修改以下代码：

```
RingShape(progress: progress, thickness: thickness)
    .fill(AngularGradient(gradient: gradient, center: .center, startAngle: .degrees(startAngle), endAngle: .degrees(360 * progress + startAngle)))

```

Xcode 现在应该在 `ProgressRingView_Previews` 中显示错误，因为我们必须将 `ProgressRingView` 传给 `progress` 参数。 因此，像这样修改`ProgressRingView_Previews`：

```
struct ProgressRingView_Previews: PreviewProvider {
    static var previews: some View {
        ProgressRingView(progress: .constant(0.5))
            .previewDisplayName("ProgressRingView (50%)")
        ProgressRingView(progress: .constant(0.9))
            .previewDisplayName("ProgressRingView (90%)")
    }
}

```

我想预览两个不同的进度值的最终结果，所以就创建了两个`ProgressRingView`实体。 现在我们就可以轻松地同时查看两个结果。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-11.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-11.png)

图 30.11. 同时预览两个不同的进度条

### 使用 Animatable 为环形设置动画

圆形进度条看起来做得不错，那就让我们试一试创建一个如图 30.12 所示的范列。该图有三个用于调整进度的按钮。 当任何一个按钮被点击时，进度条会逐渐增加（或减少）到指定的百分比。 例如，当前进度设置为 0。当点击“50%”按钮时，进度条会从 0% 逐渐上升到 50%。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-12.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-12.png)

图 12. 圆形进度条示范

现在让我们切换到 `ContentView.swift` 来创建这个示范App。 首先，声明一个状态变量来存放进度，如下所示：

```
@State var progress = 0.0

```

然后在 `body` 变量中插入以下代码来建立 UI：

```
VStack {
    ProgressRingView(progress: $progress)

    HStack {
        Group {
            Text("0%")
                .font(.system(.headline, design: .rounded))
                .onTapGesture {
                    self.progress = 0.0
                }

            Text("50%")
                .font(.system(.headline, design: .rounded))
                .onTapGesture {
                    self.progress = 0.5
                }

            Text("100%")
                .font(.system(.headline, design: .rounded))
                .onTapGesture {
                    self.progress = 1.0
                }
        }
        .padding()
        .background(Color(.systemGray6))
        .clipShape(RoundedRectangle(cornerRadius: 15.0, style: .continuous))
        .padding()
    }
    .padding()
}

```

在预览画布中，你应该有如下图所示的内容。 进度条只显示下方的灰色圆圈，主要原因是进度值默认为零。 单击 *Play* 按钮运行App，尝试点击不同的按钮以查看进度条如何变化。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-13.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-13.png)

图 30.13. 示范App UI

Does it work up to your expections? I think not. When you tap the 50% button, the progress bar instantly fills half of the ring without any animation. This isn't what we expect.

App 的运作是否符合你的期望？ 我想不是。 当你点击 50% 按钮时，进度条会立即填满圆的一半，而没有带任何动画。 这当然不是我们所期望的效果。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-14.gif](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-14.gif)

图 30.14. 进度条没有带任何动画

我想你可能知道为什么视图没有动画， 因为我们还没有将 `.animation` 修饰器附加到环形。 切换至 `ProgressRingView.swift` 并将 `.animation` 修饰器附加到 `ProgressRingView` 的 `ZStack`。 你可以在 `.frame` 修饰器之后插入代码：

```
.animation(.easeInOut(duration: 1.0), value: progress)

```

好的，看来我们已经找到了解决方案。 让我们回到 `ContentView.swift` 并再次测试App。 试试再按任何按钮看看效果。

你的结果是什么？ 我们之间的改动有效吗？

不幸的是，圆环仍然没有为进度变化设置动画，但渐变色的变化现在已有带动画了。

原因是什么？

在解决这个问题之前，让我进一步解释一下 `.animation` 修饰器是如何运作的。 在 `.animation` 修饰器的 [官方文件](https://developer.apple.com/documentation/swiftui/view/animation(_:)) 中，它提到 *该修饰器可为所有可以动画化的值（animatable values）自动加入动画*。 这里的关键字是 *animatable*。 当你在视图上使用 `.animation` 修饰器时，SwiftUI 会自动为对视图的可动画属性进行动画处理。

SwiftUI 带有一个名为`Animatable`的协议。 对于支持动画的视图，你可以采用协议并提供 `animatableData` 属性。 这个属性告诉 SwiftUI 视图有哪些数据可以动画化。

在第 9 章中，我介绍了 SwiftUI 动画的基础知识。 你可以使用 `.scaleEffect` 轻松为视图的大小变化或使用 `.offset` 动画位置变化加入动画。 可能对于你来说，所有这些动画都是自动运行的。 但在这些动画背后，Apple 的工程师实际上采用了`Animatablew`协议，并为`CGSize`和`CGPoint`提供了动画数据。

那么，为什么 `RingShape` 不能将进度动画化呢？

`RingShape` 结构符合 `Shape` 协议。 如果你查看它的API文件，`Shape` 采用了 `Animatable` 协议并提供了默认实现（default implementation）。 然而，`animatableData` 属性的默认实现是返回 `EmptyAnimatableData` 的实体（instance），这意味着没有动画数据。 这就是为什么 `ProgressRingView` 的进度变化没有带动画。

要解决此问题并使进度可动画化，你需要做的就是覆载（override）原本的实现并提供可动画化的值。 回到 `RingShape` 结构，在 `path` 函数之前加入以下代码：

```
var animatableData: Double {
    get { progress }
    set { progress = newValue }
}

```

代码非常简单， 我们只是告诉 SwiftUI 为 `progress` 值设置动画。就是这样！

现在回到 `ContentView.swift` 再进行另一个测试， 这次修改进度就可以见到动画。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-15.gif](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-15.gif)

图 30.15. 进度条的变化带有动画

### 进度条的 100% 问题

有了动画后，这个圆形进度条的用户体验变得更好了。 但是，你可能会注意到一个小问题。 当百分比设定为 100% 时，圆弧变成一个完整的圆，遮盖圆帽（round cap）。 为了突出圆弧的结束位置，最好添加带有阴影的圆帽，如图 30.1 中的活动环。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-16.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-16.png)

图 30.16. 盖上红色圆帽

问题是如何计算这个小圆的位置或圆弧的终点位置？ 这需要一些数学知识。 图 30.17 显示了我们如何计算小圆圈的位置。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-17.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-17.png)

图 30.17. 计算这个小圆的位置

现在就让我们创建这个小圆圈。 我称这个视图为 `RingTip` 并在 `ProgressRingView.swift` 文件中实现它，如下所示：

```
struct RingTip: Shape {
    var progress: Double = 0.0
    var startAngle: Double = -90.0
    var ringRadius: Double

    private var position: CGPoint {
        let angle = 360 * progress + startAngle
        let angleInRadian = angle * .pi / 180

        return CGPoint(x: ringRadius * cos(angleInRadian), y: ringRadius * sin(angleInRadian))
    }

    var animatableData: Double {
        get { progress }
        set { progress = newValue }
    }

    func path(in rect: CGRect) -> Path {
        var path = Path()

        guard progress > 0.0 else {
            return path
        }

        let frame = CGRect(x: position.x, y: position.y, width: rect.size.width, height: rect.size.height)

        path.addRoundedRect(in: frame, cornerSize: frame.size)

        return path
    }

}

```

`RingTip` 结构接受三个参数：`progress`、`startAngle` 和 `ringRadius` 用于计算圆的位置。 一旦我们确定了位置，就可以使用 `addRoundedRect` 绘制圆的路径。

现在回到 `ProgressRingView` 并声明以下计算属性来计算环的半径：

```
private var radius: Double {
    Double(width / 2)
}

```

接下来，在 `ZStack` 中的 `RingShape` 之后插入以下代码来创建 `RingTip`：

```
RingTip(progress: progress, startAngle: startAngle, ringRadius: radius)
    .frame(width: thickness, height: thickness)
    .foregroundColor(progress > 0.96 ? gradient.stops[1].color : Color.clear)

```

我们通过传当前进度、初始角度和圆环的半径来建立`RingTip`。 前景色设置为结束渐变色。 你可能想知道为什么我们只在进度大于 0.96 时才显示渐变色。 看看图 30.18，你就会明白我为什么会做出这个决定。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-18.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-18.png)

图 30.18. 仅当进度大于 0.96 时才需要覆盖圆圈

在 `ZStack` 中添加 `RingTip` 后，运行App试一试。 按 100% 按钮， 进度条现在应该有一个圆顶。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-19.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-19.png)

图 30.19. 在环末端覆盖一个小圆圈

你已经构建了一个非常漂亮的圆形进度条， 但是我们还要在圆弧末端添加点阴影。 在 SwiftUI 中，你可以简单地附加 `.shadow` 修饰器来添加阴影。 就这个App，我们可以将修饰器附加到 `RingTip`。 最困难的部分是我们需要弄清楚要在哪里添加阴影。

计算阴影位置与计算环尖的方式非常相似。 因此，在 `ProgressRingView.swift` ，加入一个用于计算环尖端位置的函数：

```
private func ringTipPosition(progress: Double) -> CGPoint {
    let angle = 360 * progress + startAngle
    let angleInRadian = angle * .pi / 180

    return CGPoint(x: radius * cos(angleInRadian), y: radius * sin(angleInRadian))
}

```

然后添加一个新的计算属性来计算环尖端的阴影偏移值（Shadow offset），如下所示：

```
private var ringTipShadowOffset: CGPoint {
    let shadowPosition = ringTipPosition(progress: progress + 0.01)
    let circlePosition = ringTipPosition(progress: progress)

    return CGPoint(x: shadowPosition.x - circlePosition.x, y: shadowPosition.y - circlePosition.y)
}

```

在当前进度上加上 0.01，我们可以计算出阴影位置。 这只是我提供的计算阴影位置的解决方案， 试试自己想一下，你或许能会找出一个更好的替代解决方案。

我们可以将 `.shadow` 修饰器附加到 `RingTip`：

```
.shadow(color: progress > 0.96 ? Color.black.opacity(0.15) : Color.clear, radius: 2, x: ringTipShadowOffset.x, y: ringTipShadowOffset.y)

```

我只是想添加一个浅色的阴影，所以将`opacity`设定为0.15。 如果你喜欢较暗色的阴影，请增加`opacity`值（例如 1.0）。 修改代码后，当进度大于 0.96，你应该会在环的末尾看到一个阴影。 你也可以尝试将进度值设置为大于 1.0 的值，然后看看进度条的外观。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-20.png](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-20.png)

图 30.20. 环形末端现在有一个阴影

### 练习

现在你已经创建了一个圆形进度条，是时候进行练习了。 你的任务是利用你已构建的内容并创建一个活动环。 另外，还需要提供四个按钮来调整活动环，如图 30.21 所示。

![%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-21.gif](%E4%BD%BF%E7%94%A8%20Shape%20%E5%92%8C%20Animatable%20%E5%BC%80%E5%8F%91%E5%B8%A6%E5%8A%A8%E7%94%BB%E7%9A%84%E7%8E%AF%E5%BD%A2%E8%BF%9B%E5%BA%A6%E6%9D%A1%203477eed4e12a4e44a5e2e81800ce57d1/swiftui-progressring-21.gif)

图 30.21. 活动环示范

### 总结

通过构建一个活动环，我们在本章中介绍了许多 SwiftUI 功能。 你现在应该知道如何使用 `Shape` 以及如何使用 Animatable 协议为不同形状设置动画。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- 示例程序 （[https://www.appcoda.com/resources/swiftui4/SwiftUIProgressRing.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIProgressRing.zip)）
- 练习答案（[https://www.appcoda.com/resources/swiftui4/SwiftUIProgressRingExercise.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIProgressRingExercise.zip)）