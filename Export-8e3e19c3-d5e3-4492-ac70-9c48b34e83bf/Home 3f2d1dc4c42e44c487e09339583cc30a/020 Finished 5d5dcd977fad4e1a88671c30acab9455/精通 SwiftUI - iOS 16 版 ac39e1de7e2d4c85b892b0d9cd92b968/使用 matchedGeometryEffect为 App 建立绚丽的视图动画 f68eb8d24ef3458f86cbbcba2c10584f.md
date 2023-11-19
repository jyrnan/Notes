# 使用 matchedGeometryEffect为 App 建立绚丽的视图动画

在 iOS 14 中，Apple 为 SwiftUI 框架引入了很多新功能，像是 LazyVGrid 以及 LazyHGrid。其中 `matchedGeometryEffect` 非常引人注目，这个功能让开发者只需要几行代码，就能够创造绚丽的视图动画。SwiftUI 框架已经让开发者可以简单地使用动画来呈现视图的变化，而 `matchedGeometryEffect` 修饰器 (modifier) 更将视图动画 (view animations) 的实现提升到另一个境界。

对所有手机 App 来说，我们经常需要在多个视图之间转换，因此一个令人喜欢的视图转换绝对可以提升整体的使用者体验。有了 `matchedGeometryEffect`，你只需要描述两个视图的外观，修饰器就会自动计算两个视图的差异，并且自动为大小和位置的变化加上动画。

可能你会觉得十分困惑，但别担心，介绍完整个示例 App 之后，你就会明白我在说什么了。

### 重温 SwiftUI 动画

在我们开始介绍 `matchedGeometryEffect` 之前，让我们先来看一下如何使用 SwiftUI 来实现动画。下面的图片显示了一个视图开始和结束的状态。当你点击左边的圆形视图，它应该会变大且往上移动；相反地，当你点击右边的视图时，它就会回到原本的大小和位置。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-1.png](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-1.png)

图 33.1. 圆形视图开始状态 (左), 圆形视图结束状态 (右)

要实现这个可点击的圆形视图非常简单。开启一个新的 SwiftUI 项目后，如此修改 `ContentView` 结构：

```
struct ContentView: View {

    @State private var expand = false

    var body: some View {
        Circle()
            .fill(Color.green)
            .frame(width: expand ? 300 : 150, height: expand ? 300 : 150)
            .offset(y: expand ? -200 : 0)
            .animation(.default, value: expand)
            .onTapGesture {
                self.expand.toggle()
            }
    }
}

```

我们用一个状态变量 `expand`，来记录 `Circle` 视图目前的状态。当状态改变时，我们会通过 `.frame` 和 `.offset` 这两个修饰器，来修改视图框 (frame) 的大小和位移 (offset)。如果在预览画面中运行这个 App ，你应该可以在点击圆形视图时看到动画效果。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-2.gif](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-2.gif)

图 33.2. 圆形视图的动画效果

### 了解 matchedGeometryEffect 修饰器

那么到底什么是 `matchedGeometryEffect` 呢？这个功能如何简化实现视图动画的步骤？让我们再来看一下第一张图片、以及圆形视图动画的代码。我们需要找出开始及结束状态时的确切数值差异。在这个例子当中，就是视图框的大小及位移。

有了 `matchedGeometryEffect` 修饰器，你不再需要找出两个状态之间的差异了。你只需要描述两个视图：*一个是开始的状态，而另一个是结束的状态*，`matchedGeometryEffect` 会自动添加在两个视图之间大小和位置的差异。

要使用 `matchedGeometryEffect` 建立跟之前一样的动画效果，你需要先声明一个命名空间 (namespace) 变量：

```
@Namespace private var shapeTransition

```

然后，如此重写 `body` 的部分：

```
var body: some View {
    if expand {

        // Final State
        Circle()
            .fill(Color.green)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(width: 300, height: 300)
            .offset(y: -200)
            .onTapGesture {
                withAnimation(.easeIn) {
                    expand.toggle()
                }
            }

    } else {

        // Start State
        Circle()
            .fill(Color.green)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(width: 150, height: 150)
            .offset(y: 0)
            .onTapGesture {
                withAnimation(.easeIn) {
                    expand.toggle()
                }
            }
    }
}

```

在这段代码中，我们建立了两个圆形视图，一个表示开始状态，而另一个表示结束状态。当它一开始被初始化之后，我们会得到一个 `Circle` 视图，位置置中以及宽度为 150 点。当 `expand` 状态变量从 `false` 变成 `true` 时， App 会显示另一个 `Circle` 视图，其位置是从中间往上 200 点以及宽度为 300 点。

对于两个 `Circle` 视图，我们都添加了 `matchedGeometryEffect` 修饰器，并且指定了相同的 ID 和命名空间。通过这样的设定，SwiftUI 可以计算两个视图之间大小和位置的差异，并且添加视图转换。随着后续加上的 `animation` 修饰器，SwiftUI 框架会自动动画化视图转换。

ID 以及命名空间的用途，是用来标记属于同一个转换的视图，所以两个 `Circle` 视图会使用相同的 ID 和命名空间。

以上我们介绍了如何使用 `matchedGeometryEffect`，来实现两个视图之间的动画转换效果。如果你有使用过Keynote 的 Magic Move，这个新的修饰器跟 Magic Move非常类似。我建议你使用 iPhone 模拟器来运行这个 App，以便测试这个动画效果。在我写这篇文章时， Xcode 中有一个 bug，导致我们无法在预览画面测试这个动画效果。

### 从圆形变为圆角长方形

现在，让我们来试试实现另一个视图转换动画。这一次，我们会将一个圆形变化成为一个圆角长方形 (rounded rectangle)。圆形位置于屏幕的上端，而圆角长方形则位于屏幕的底端。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-3.png](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-3.png)

图 33.3. 圆形视图的开始状态 (左), 圆角长方形的结束状态 (右)

我们可以使用刚刚学到的技巧，准备两个视图：一个圆形视图和一个圆角长方形视图，`matchedGeometryEffect` 修饰器就会处理视图转换的部分。现在，让我们将 `body` 变量的 `ContentView` 结构改成这样：

```
VStack {
    if expand {

        // Rounded Rectangle
        Spacer()

        RoundedRectangle(cornerRadius: 50.0)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(minWidth: 0, maxWidth: .infinity, maxHeight: 300)
            .padding()
            .foregroundColor(Color(.systemGreen))
            .onTapGesture {
                withAnimation {
                    expand.toggle()
                }
            }

    } else {

        // Circle
        RoundedRectangle(cornerRadius: 50.0)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(width: 100, height: 100)
            .foregroundColor(Color(.systemOrange))
            .onTapGesture {
                withAnimation {
                    expand.toggle()
                }
            }

        Spacer()
    }
}

```

我们同样使用 `expand` 状态变量，来切换圆形视图和圆角长方形视图。这段代码跟之前的示例非常相似，只是我们在这边加上了 `VStack` 和 `Spacer` 来定位视图。或许你会问，为什么要使用 `RoundedRectangle` 来建立圆形物件呢？主要原因是这样可以让视图转换就会更顺畅。

在这两个视图中，我们都加上了 `matchedGeometryEffect` 修饰器，并且指定了一样的 ID 以及命名空间，我们要做的事就完成了。修饰器会自动比较两个视图的差异，并且使用动画呈现这个改变。如果你在预览画面或是 iPhone 模拟器上运行这个 App，会看到视图在圆形和圆角长方形之间完美的转换。这就是 `matchedGeometryEffect` 的威力。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-4.gif](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-4.gif)

图 33.4. 从圆形变为圆角长方形

不过，你可能注意到这个修饰器无法运行改变颜色的动画。没错，`matchedGeometryEffect` 只能用来处理位置和大小的变化。

### 练习一

让我们来做一个小小的练习，来测试你对 `matchedGeometryEffect` 的了解。你的任务是要建立如下图的动画视图转换。开始时，它是一个橘色的圆形视图，圆形视图被点击后，就会转换成全屏幕的背景图。你可以在项目档中找到完整代码。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-5.png](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-5.png)

图 33.5. 圆形视图转换成全屏幕的背景图

### 使用动画转换来交换两个视图

现在你应该对 `matchedGeometryEffect` 有了基础的认识，让我们继续来看看它如何帮助我们建立一些绚丽的动画。在这个示例中，我们会交换两个圆形视图的位置，并且套用修饰器来建立顺畅的视图转换。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-6.gif](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-6.gif)

图 33.6. 交换两个视图的位置

我们会使用一个状态变量，来储存交换的状态，并建立一个命名空间变量给 `matchedGeometryEffect` 来使用。让我们在 `ContentView` 声明这些参数：

```
@State private var swap = false

@Namespace private var dotTransition

```

橘色圆形默认位在屏幕的左边，而绿色圆形则在屏幕的右边。当使用者点击任意一个圆形图案，就会触发互换的动画。使用 `matchedGeometryEffect` 时，你不用了解交换动画是如何达成的。要建立视图转换，你只需要做到以下事情：

1. 建立橘色和绿色圆形交换之前的版面配置
2. 建立两个圆形在交换之后的版面配置

如果你要将版面配置转换成代码，你可以如此编写 `body` 变量：

```
if swap {

    // After swap
    // Green dot on the left, Orange dot on the right

    HStack {
        Circle()
            .fill(Color.green)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "greenCircle", in: dotTransition)

        Spacer()

        Circle()
            .fill(Color.orange)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "orangeCircle", in: dotTransition)
    }
    .frame(width: 100)
    .onTapGesture {
        withAnimation {
            swap.toggle()
        }
    }

} else {

    // Start state
    // Orange dot on the left, Green dot on the right

    HStack {
        Circle()
            .fill(Color.orange)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "orangeCircle", in: dotTransition)

        Spacer()

        Circle()
            .fill(Color.green)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "greenCircle", in: dotTransition)
    }
    .frame(width: 100)
    .onTapGesture {
        withAnimation {
            swap.toggle()
        }
    }
}

```

我们使用 `HStack` 来将两个圆形配置成水平排列，并且利用 `Spacer` 在两个圆形中间建立一些空间。当 `swap` 变量设定成 `true` 之后，绿色圆形会被放在橘色圆形的左边。相反地，绿色圆形则会被放在橘色圆形的右边。

如你所见，我们只需要描述不同状态下圆形视图的配置，`matchedGeometryEffect` 就会处理余下的事情。我们在每个 `Circle` 视图加上修饰器，不过这次有一点不同，因为我们有两个不同的 `Circle`视图需要配置，我们使用了两个不同的 ID 来建立 `matchedGeometryEffect` 修饰器。我们将橘色圆形的识别名称设定为 `orangeCircle`，而绿色圆形则是设定为 `greenCircle`。

现在，如果你在模拟器运行这个 App，应该可以在点击任何一个圆形时看到互换的动画。

### 练习二

在刚刚的练习中，我们在两个圆形上使用了 `matchedGeometryEffect`，并交换它们的位置。这个练习会使用到一样的技巧，不过这次是要使用两张图片。图33.7展示了这个示例，点击交换按钮时，这个 App 就会用漂亮的动画来交换这两张图片。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-7.gif](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-7.gif)

图 33.7. 交换两张图片

你可以随意使用自己的图片。我在示例中使用了这些 Unsplash.com 的免费图片：

- [https://unsplash.com/photos/pMW4jzELQCw](https://unsplash.com/photos/pMW4jzELQCw)
- [https://unsplash.com/photos/PM4Vu1B0gxk](https://unsplash.com/photos/PM4Vu1B0gxk)

### 建立一个英雄动画效果（Hero Animation）

除了从一种形状转换到另一种形状之外，你还可以使用 `matchedGeometryEffect` 修改器来创建基本的英雄动画。 给你一个例子，图 33.8 显示了一个图像和一段文字的堆栈视图。 点击视图时，图像和文字都会展开以占据全屏幕。 这种类型的动画通常被称为英雄动画（Hero Animation）。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-8.png](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-8.png)

图 33.8. 将堆栈视图扩展到全屏幕

同样地，我们可以应用 `matchedGeometryEffect` 技巧来创建这种类型的过场动画。 如果参考图 33.8，视图转换中有两个视图：

1. 一个是显示较小图像和文章节录的视图。
2. 另一个是扩展为全屏幕的视图，显示精选照片和全文。

首先，声明一个状态变量来控制视图模式的状态：

```
@State private var showDetail = false

```

当 `showDetail` 设定为 false 时，App就会显示具有较小图像的文章视图。 如果为 true，则将显示整个画面的文章视图。 要使用 `matchedGeometryEffect` 修饰器，我们必须声明一个`namespace`变量：

```
@Namespace private var articleTransition

```

接下来，像这样修改 `body` 变量：

```
// Display an article view with smaller image
if !showDetail {
    VStack {
        Spacer()

        VStack {
            Image("latte")
                .resizable()
                .scaledToFill()
                .frame(minWidth: 0, maxWidth: .infinity)
                .frame(height: 200)
                .matchedGeometryEffect(id: "image", in: articleTransition)
                .cornerRadius(10)
                .padding()
                .onTapGesture {
                    withAnimation(.interactiveSpring(response: 0.5, dampingFraction: 0.8, blendDuration: 0.2)) {
                        showDetail.toggle()
                    }
                }

            Text("The Watertower is a full-service restaurant/cafe located in the Sweet Auburn District of Atlanta.")
                .matchedGeometryEffect(id: "text", in: articleTransition)
                .padding(.horizontal)
        }
    }
}

// Display the article view in a full screen
if showDetail {

    ScrollView {
        VStack {
            Image("latte")
                .resizable()
                .scaledToFill()
                .frame(minWidth: 0, maxWidth: .infinity)
                .frame(height: 400)
                .clipped()
                .matchedGeometryEffect(id: "image", in: articleTransition)
                .onTapGesture {
                     withAnimation(.interactiveSpring(response: 0.5, dampingFraction: 0.8, blendDuration: 0.4)) {
                         showDetail.toggle()
                     }
                }

            Text("The Watertower is a full-service restaurant/cafe located in the Sweet Auburn District of Atlanta. The restaurant features a full menu of moderately priced \"comfort\" food influenced by African and French cooking traditions, but based upon time honored recipes from around the world. The cafe section of The Watertower features a coffeehouse with a dessert bar, magazines, and space for live performers.\n\nThe Watertower will be owned and operated by The Watertower LLC, a Georgia limited liability corporation managed by David N. Patton IV, a resident of the Empowerment Zone. The members of the LLC are David N. Patton IV (80%) and the Historic District Development Corporation (20%).\n\nThis business plan offers financial institutions an opportunity to review our vision and strategic focus. It also provides a step-by-step plan for the business start-up, establishing favorable sales numbers, gross margin, and profitability.\n\nThis plan includes chapters on the company, products and services, market focus, action plans and forecasts, management team, and financial plan.")
                .matchedGeometryEffect(id: "text", in: articleTransition)
                .animation(nil, value: showDetail)
                .padding(.all, 20)

            Spacer()
        }

    }
    .edgesIgnoringSafeArea(.all)
}

```

在上面的代码中，我们将视图布局为不同的状态。 当 `showDetail` 设为 `false` 时，我们使用 `VStack` 来布局文章的图片和节录。 图像的高度设定为 200 点以使其变小。

文章视图的布局在全屏幕模式下非常相似， 主要分别在于 `VStack` 视图嵌入在 `ScrollView` 中以使内容可滚动。 图像的高度设定为 400 点，这样图像就会变大一点。 为了将图像和文字视图扩展出屏幕安全区域以外，我们将 `.edgesIgnoringSafeArea` 修饰器加到滚动视图并将其值设置为 `.all`。

由于我们在过场效果有两个不同的视图，因此我们在 `matchedGeometryEffect` 修饰器要使用两个不同的 ID。 对于图像，我们将 ID 设定为`image`：

```
.matchedGeometryEffect(id: "image", in: articleTransition)

```

另一方面，我们将文字视图的 ID 设定为 `text`：

```
.matchedGeometryEffect(id: "text", in: articleTransition)

```

此外，我们为文字和图像视图使用了两种不同的动画。 图像视图使用 `.interactiveSpring` 动画，至于文字视图，我们使用 `.easeOut` 动画。

现在，可以在模拟器中运行App并试一下。 当你点击图像视图时，App会显示漂亮的动画并以全屏幕显示文章。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-9.gif](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-9.gif)

图 33.9. 英雄动画效果

### 在视图之间传递@Namespace

参考前面的例子，我们可以将两个不同的堆栈视图分成为子视图让代码整理得更好。 但问题是我们如何在视图之间传递`@Namespace` 变量， 让我们可以怎样做。

首先，按住command键并单击第一个堆栈视图的`VStack`。 从菜单中选择 *Extract Subview* 并将子视图命名为`ArticleExcerptView`。

![%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-10.png](%E4%BD%BF%E7%94%A8%20matchedGeometryEffect%E4%B8%BA%20App%20%E5%BB%BA%E7%AB%8B%E7%BB%9A%E4%B8%BD%E7%9A%84%E8%A7%86%E5%9B%BE%E5%8A%A8%E7%94%BB%20f68eb8d24ef3458f86cbbcba2c10584f/swiftui-matchedgeometryeffect-10.png)

图 33.10. 将堆栈视图分成为子视图

你应该会在 `ArticleExcerptView` 结构中看到很多错误，说缺少namespace 和 `showDetail` 变量。 要改正 `showDetail` 变量的错误，你可以在 `ArticleExcerptView` 中声明一个绑定，如下所示：

```
@Binding var showDetail: Bool

```

要从另一个视图接受namespace，诀窍是声明一个具有 `Namespace.ID` 类型的变量，如下所示：

```
var articleTransition: Namespace.ID

```

现在应该可以修复 `ArticleExcerptView` 中的所有错误。 回到 `ContentView` 并将 `ArticleExcerptView()` 替换为：

```
ArticleExcerptView(showDetail: $showDetail, articleTransition: articleTransition)

```

我们将绑定传给`showDetail`，并将namespace变量传递给子视图。 这就是你在不同视图之间共享namespace的方式。 重复以上的程序将 `ScrollView` 抽出到另一个子视图中，并将子视图命名为`ArticleDetailView`。

另外，你需要在 `ArticleDetailView` 中声明以下变量和绑定以解决所有错误：

```
@Binding var showDetail: Bool

var articleTransition: Namespace.ID

```

你还要像这样修改 `ArticleDetailView()` ：

```
ArticleDetailView(showDetail: $showDetail, articleTransition: articleTransition)

```

在所有修改之后，`ContentView` 结构现在被简化如下：

```
struct ContentView: View {

    @State private var showDetail = false

    @Namespace private var articleTransition

    var body: some View {

        // Display an article view with smaller image
        if !showDetail {
            ArticleExcerptView(showDetail: $showDetail, articleTransition: articleTransition)
        }

        // Display the article view in a full screen
        if showDetail {
            ArticleDetailView(showDetail: $showDetail, articleTransition: articleTransition)
        }

    }
}

```

App的运作和之前一样，但代码现在就更容易阅读。

### 总结

引进 `matchedGeometryEffect` 修饰器之后，视图动画的实现提升到了另一个层次。你可以用更少的代码来创造漂亮的视图动画。即便你是 SwiftUI 的新手，你也可以从这个新修饰器中得益，并且让你的 App 更棒。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUIMatchedGeometry.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIMatchedGeometry.zip)