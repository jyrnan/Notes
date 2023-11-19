# 使用手势与动画建立 Tinder 风格的 UI | 精通 SwiftUI - iOS 16 版

建立一个展开式底部表不是很有趣呢？我们来继续应用我们所学到的手势，并将其应用到现实世界的项目中。我不确定你之前是否用过 Tinder App，但是在一些其他 App 中， 你可能会碰到如 Tinder 般的使用者介面。滑动动作是 Tinder UI 的设计重点，并已成为最流行的行动装置 UI 模式之一。使用者向右滑动即表示喜欢某张图片，向左滑动则表示不喜欢。

在本章中，我们要做的是建立一个具有如 Tinder 般UI 的简单 App。这个 App 向使用者显示一副旅游卡，并让他们使用滑动手势来表示喜欢/ 不喜欢一张卡片。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-1.jpg](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-1.jpg)

图 19.1. 建立如Tinder 般的使用者介面

请注意，我们将不会建立一个功能齐全的App，而是只着眼于如Tinder 般的UI 。

### 项目准备

如果你使用自己的图片，那就太棒了。不过，为了节省你准备旅游图片的时间，我已经为你建立了一个初始项目，你可以至下列网址下载：[https://www.appcoda.com/resources/swiftui4/SwiftUITinderTripStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUITinderTripStarter.zip)。这个项目已经具有一组旅游卡的照片，如图 19.2 所示。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-2.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-2.png)

图 19.2. 预载了一组旅游照片

除此之外，我已经为示例 App 准备测试数据，并建立了 Trip.swift 档来代表旅程：

```
struct Trip {
    var destination: String
    var image: String
}

#if DEBUG
var trips = [ Trip(destination: "Yosemite, USA", image: "yosemite-usa"),
              Trip(destination: "Venice, Italy", image: "venice-italy"),
              Trip(destination: "Hong Kong", image: "hong-kong"),
              Trip(destination: "Barcelona, Spain", image: "barcelona-spain"),
              Trip(destination: "Braies, Italy", image: "braies-italy"),
              Trip(destination: "Kanangra, Australia", image: "kanangra-australia"),
              Trip(destination: "Mount Currie, Canada", image: "mount-currie-canada"),
              Trip(destination: "Ohrid, Macedonia", image: "ohrid-macedonia"),
              Trip(destination: "Oia, Greece", image: "oia-greece"),
              Trip(destination: "Palawan, Philippines", image: "palawan-philippines"),
              Trip(destination: "Salerno, Italy", image: "salerno-italy"),
              Trip(destination: "Tokyo, Japan", image: "tokyo-japan"),
              Trip(destination: "West Vancouver, Canada", image: "west-vancouver-canada"),
              Trip(destination: "Singapore", image: "garden-by-bay-singapore"),
              Trip(destination: "Perhentian Islands, Malaysia", image: "perhentian-islands-malaysia")
            ]
#endif

```

假如你希望使用自己的图片与数据，则只需替换素材目录中的图片，并修改 `Trip.swift` 档。

### 建立卡片视图与菜单列

在实现滑动功能之前，我们先建立主UI，我将主屏幕分成三个部分，如图 19.3 所示：

1. 顶部菜单列（top menu bar）。
2. 顶部菜单列（top menu bar）。
3. 底部菜单列（bottom menu bar）。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-3.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-3.png)

图 19.3. 主屏幕

### 卡片视图

首先，我们建立一个卡片视图。若是你想挑战自我，我强烈建议你在这里停下来并实现它，而无须遵守本节内容，否则请继续阅读。

为了让代码更易编写，我们将在一个单独的文件中实现卡片视图。在项目导航器中，使用“SwiftUI View”模板来建立新档，并将其命名为 `CardView.swift`。

`CardView` 是设计用来显示不同的照片与标题，因此声明两个变量来储存这些数据：

```
let image: String
let title: String

```

主屏幕将显示一副卡片视图。稍后，我们将使用 `ForEach` 来逐一运行卡片视图数组并显示它们。如果你还记得`ForEach` 的用法，那么 SwiftUI 需要知道如何唯一识别数组中的每个项目。因此，我们将使 `CardView` 遵守`Identifiable` 协议，并导入一个 `id` 变量，如下所示：

```
struct CardView: View, Identifiable {
    let id = UUID()
    let image: String
    let title: String

    .
    .
    .
}

```

如果您忘记什么是 `Identifiable` 协议，则请参考第 10 章。

现在，我们继续实现卡片视图，并修改 `body` 变量如下：

```
var body: some View {
    Image(image)
        .resizable()
        .scaledToFill()
        .frame(minWidth: 0, maxWidth: .infinity)
        .cornerRadius(10)
        .padding(.horizontal, 15)
        .overlay(alignment: .bottom) {
            VStack {

                Text(title)
                    .font(.system(.headline, design: .rounded))
                    .fontWeight(.bold)
                    .padding(.horizontal, 30)
                    .padding(.vertical, 10)
                    .background(Color.white)
                    .cornerRadius(5)
            }
            .padding([.bottom], 20)
        }
}

```

卡片视图是由一张图片及一个叠在图片上方的文字组件所组成。我们设定图片为 `scaleToFill` 模式，并使用 `cornerRadius` 修饰器来为图片加上圆角。文字组件是用来显示旅程的目的地。

我们在第 5 章中深入讨论过卡片视图的类似实现。如果你不能完全了解代码，则请再次阅读该章。

你还无法预览卡片视图，因为你必须在 `CardView_Previews` 中同时提供 `image` 与 `title` 的值，因此修改`CardView_Previews` 结构如下：

```
struct CardView_Previews: PreviewProvider {
    static var previews: some View {
        CardView(image: "yosemite-usa", title: "Yosemite, USA")
    }
}

```

我只是使用素材目录中的其中一张图片来进行预览，你可以依自己的需求随意修改图片及标题。在预览画布中，你现在应该看到类似图 19.4 的卡片视图。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-4.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-4.png)

图 19.4. 预览卡片视图

### 菜单列与主 UI

准备好卡片视图后，我们可以继续实现主 UI。如前所述，主 UI 有卡片与两个菜单列， 对于这两个菜单列，我将为它们个别建立一个单独的 `struct`。

现在开启 `ContentView.swift` 并开始实现。对于顶部菜单列，建立一个新的 `struct`，如下所示：

```
struct TopBarMenu: View {
    var body: some View {
        HStack {
            Image(systemName: "line.horizontal.3")
                .font(.system(size: 30))
            Spacer()
            Image(systemName: "mappin.and.ellipse")
            .font(.system(size: 35))
            Spacer()
            Image(systemName: "heart.circle.fill")
            .font(.system(size: 30))
        }
        .padding()
    }
}

```

这三个图示使用等距的水平堆叠来排列。对于底部菜单列，实现几乎相同。在 `Content View.swift` 中插入下列的代码，以建立菜单列：

```
struct BottomBarMenu: View {
    var body: some View {
        HStack {
            Image(systemName: "xmark")
                .font(.system(size: 30))
                .foregroundColor(.black)

            Button {
                // 预定旅程
            } label: {
                Text("BOOK IT NOW")
                    .font(.system(.subheadline, design: .rounded))
                .bold()
                    .foregroundColor(.white)
                    .padding(.horizontal, 35)
                    .padding(.vertical, 15)
                    .background(Color.black)
                    .cornerRadius(10)
            }
            .padding(.horizontal, 20)

            Image(systemName: "heart")
                .font(.system(size: 30))
                .foregroundColor(.black)
        }

    }
}

```

我们不打算实现“Book Trip”功能，因此将动作区块留空。假设你了解堆叠与图片的工作原理，则其余的代码应该无需解释。

在建立主UI 之前，让我教你一个预览这两个菜单列的技巧。而在 `ContentView` 中放置这些列，来预览它们的外观及感觉，并不是强制的。

现在修改 `ContentView_Previews` 结构如下：

```
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()

        TopBarMenu()
            .previewDisplayName("TopBarMenu")

        BottomBarMenu()
            .previewDisplayName("BottomBarMenu")
    }
}

```

这里，我们使用 `Group` 来将多个组件的预览进行分组。不指定任何的预览选项（如ContentView），Xcode 会在目前模拟器上显示预览。对于 `TopBarMenu` 与 `BottomBarMenu`， 我们告诉 Xcode 在容器视图中预览布局，图19.5 可让你更加了解预览是什么样。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-5.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-5.png)

图 19.5 预览菜单列

好的，我们继续布局主UI。修改 `ContentView` 如下：

```
struct ContentView: View {
    var body: some View {
        VStack {
            TopBarMenu()

            CardView(image: "yosemite-usa", title: "Yosemite, USA")

            Spacer(minLength: 20)

            BottomBarMenu()
        }
    }
}

```

在代码中，我们只安排了使用 `VStack` 建立的 UI 组件。你的预览现在应该显示主屏幕了，如图 19.6 所示。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-6.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-6.png)

图 19.6. 预览主 UI

### 实现卡片库

在做好所有的准备之后，终于可以实现如 Tinder 般的 UI。对于之前从未用过 Tinder App 的人，让我先解释一下如 Tinder 般 UI 的工作原理。

你可以将 Tinder 般 UI 想像为一副成堆的卡片，每张卡片都显示一张照片。在我们的示例 App 中，照片是旅程的目的地。将最上面的卡片（即第一个旅程）轻微向左或向右滑动，即可揭示下一张卡片（即下一个旅程）。如果使用者放开卡片，App 就会将卡片带回原来的位置。不过，当使用者用力滑动时，他/ 她可以丢掉这张卡片，然后App 会将第二张卡片向前移动，成为最上面的卡片，如图 19.7 所示。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-7.jpg](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-7.jpg)

图 19.7. Tinder 般 UI 的工作原理

我们实现的主屏幕只包含一个卡片视图，那么我们如何实现一堆卡片视图呢？

最直截了当的方式是，使用 `ZStack` 将每个卡片视图互相堆叠，我们来试着做这个。修改 `ContentView` 结构如下：

```swift
struct ContentView: View {

    var cardViews: [CardView] = {

        var views = [CardView]()

        for trip in trips {
            views.append(CardView(image: trip.image, title: trip.destination))
        }

        return views
    }()

    var body: some View {
        VStack {
            TopBarMenu()

            ZStack {
                ForEach(cardViews) { cardView in
                    cardView
                }
            }

            Spacer(minLength: 20)

            BottomBarMenu()
        }
    }
}

```

在上面的代码中，我们初始化一个包含所有旅程的 `cardViews` 数组（其在 `Trip.swift` 档中定义）。在body 变量中，我们逐一运行所有的卡片视图，并将它们包裹在 `ZStack` 中来相互重叠。

预览画布应该会显示相同的UI，但使用另一张图片，如图19.8 所示。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-8.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-8.png)

图 19.8. 建立卡片视图库

为什么它会显示另一张图片呢？如果你引用在 `Trip.swift` 中定义的 `trips` 数组，图片是数组的最后一个元素。在`ForEach` 区块中，第一个旅程是放在卡片库的最下面，如此最后一个旅程便成为卡片库的最上面照片。

当我们实现卡片库时，实际上有两个问题：

1. .`trips` 数组的第一个旅程应该是最上面的卡片，但是现在却是最下面的卡片。
2. 我们为 15 个旅程渲染了 15 个卡片视图。如果未来有 10,000 个旅程，甚至更多时，该怎么办呢？我们是否应该为每个旅程建立一个卡片视图呢？有没有高效率的方式来实现卡片库呢？

我们先来解决卡片顺序的问题。SwiftUI 提供 `zIndex`修饰器，来指示 ZStack 中的视图顺序。zIndex 值较高的视图，位于较低值的视图之上，因此最上面的卡片应该有最大的 `zIndex` 值。

考虑到这一点，我们先在 `ContentView` 中建立以下的新函数：

```
private func isTopCard(cardView: CardView) -> Bool {

    guard let index = cardViews.firstIndex(where: { $0.id == cardView.id }) else {
        return false
    }

    return index == 0
}

```

在逐一运行卡片视图时，我们必须找到一种识别最上面卡片的方式。上面的函式带入一个卡片视图，找出其索引，并告诉你卡片视图是否位于最上面。

接下来，修改 `ZStack` 的代码区块如下：

```
ZStack {
    ForEach(cardViews) { cardView in
        cardView
            .zIndex(self.isTopCard(cardView: cardView) ? 1 : 0)
    }
}

```

我们为每个卡片视图加入了 `zIndex` 修饰器。对于最上面的卡片，我们为其指定较高的 `zIndex` 值。在预览画布中，你现在应该会看到第一个旅程的照片（即美国优胜美地国家公园）。

对于第二个问题，则更复杂些，我们的目标是确保卡片库可支持数以万计的卡片视图， 而不需耗费大量资源。

我们来更深入研究一下卡片库。我们是否真的需要为每张旅程照片初始化个别的卡片视图呢？要建立这个卡片库UI，我们只需建立两个卡片视图，并将它们互相重叠即可。

当最上面的卡片视图被丢弃时，下面的卡片视图将成为最上面的卡片；同时，我们立即使用不同的照片初始化一个新的卡片视图，并将它放在最上面的卡片后面。无论你需要在卡片库中显示多少张照片，App 永远只有两个卡片视图。不过，从使用者的角度来看，UI 是由一堆卡片所组成。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-11.jpg](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-11.jpg)

图 19.9. 我们如何使用两个卡片视图来建立卡片库

现在，你应该了解我们如何建立卡片库，我们来继续进行实现。

首先，修改 `cardViews` 数组，我们不再需要初始化所有的旅程，而只需要初始化前两个旅程。之后，当第一个旅程（即第一张卡片）被丢弃时，我们会加入另一张卡片。

```
c

```

修改代码之后，UI 看起来应该完全相同。但在底层架构中，你应该在卡片库中只看到两个卡片视图。

### 实现滑动动作

在动态建立新的卡片视图之前，我们必须先实现滑动功能。如果你忘记湍如何处理手势，请再阅读第17 章及第18 章。我们将会重新使用前面讨论的一些代码。

首先，在 `ContentView` 中定义 `DragState`枚举，它表示可能的拖曳状态：

```
enum DragState {
    case inactive
    case pressing
    case dragging(translation: CGSize)

    var translation: CGSize {
        switch self {
        case .inactive, .pressing:
            return .zero
        case .dragging(let translation):
            return translation
        }
    }

    var isDragging: Bool {
        switch self {
        case .dragging:
            return true
        case .pressing, .inactive:
            return false
        }
    }

    var isPressing: Bool {
        switch self {
        case .pressing, .dragging:
            return true
        case .inactive:
            return false
        }
    }

}

```

再一次，如果你不了解什么是枚举，则请在此处停止，并复习一下有关手势的章节。接下来，我们定义一个`@GestureState` 变量来储存拖曳状态，默认上设定为“inactive”：

```
@GestureState private var dragState = DragState.inactive

```

现在，修改 `body` 的部分如下：

```swift
var body: some View {
    VStack {
        TopBarMenu()

        ZStack {
            ForEach(cardViews) { cardView in
                cardView
                    .zIndex(self.isTopCard(cardView: cardView) ? 1 : 0)
                    .offset(x: self.dragState.translation.width, y:  self.dragState.translation.height)
                    .scaleEffect(self.dragState.isDragging ? 0.95 : 1.0)
                    .rotationEffect(Angle(degrees: Double( self.dragState.translation.width / 10)))
                    .animation(.interpolatingSpring(stiffness: 180, damping: 100), value: self.dragState.translation)
                    .gesture(LongPressGesture(minimumDuration: 0.01)
                        .sequenced(before: DragGesture())
                        .updating(self.$dragState, body: { (value, state, transaction) in
                            switch value {
                            case .first(true):
                                state = .pressing
                            case .second(true, let drag):
                                state = .dragging(translation: drag?.translation ?? .zero)
                            default:
                                break
                            }

                        })

                    )
            }
        }

        Spacer(minLength: 20)

        BottomBarMenu()
            .opacity(dragState.isDragging ? 0.0 : 1.0)
            .animation(.default, value: dragState.isDragging)
    }
}

```

基本上，我们将应用在手势章节中所学的知识来实现拖曳。`.gesture` 修饰器有两个手势识别器：长按与拖曳。当侦测到拖曳手势时，我们修改 `dragState` 变量，并储存拖曳的位移量。

`offset`、`scaleEffect`、`rotationEffect` 与 `animation` 修饰器的结合， 可建立拖曳效果。拖曳是通过修改卡片视图的 `offset`来实现。当卡片视图处于拖曳状态时，我们会使用 `scaleEffect` 将它缩小一点，并应用`rotationEffect` 修饰器将它旋转特定角度。动画设定为 `interpolatingSpring`，但你可以自由尝试其他动画。

我们还对 `BottomBarMenu` 做一些代码修改。当使用者拖曳卡片视图时，我想要隐藏底部列，因此我们应用 `.opacity` 修饰器，并且当它在拖曳状态时，设定它的值为“0”。

进行修改后，在预览画布中运行项目来测试它。你应该能够拖曳卡片并四处移动。而当你释放卡片时，卡片会回到原来的位置，如图 19.10 所示。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-13.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-13.png)

图 19.10. 拖曳卡片视图

你注意到问题了吗？当拖曳开始时，你实际上是在拖曳整个卡片库 ！假设使用者只能拖曳最上面的卡片，下面的卡片应该保持不变。而且，缩放效果应只应用于最上面的卡片。

要解决这些问题，我们需要修改 `offset`、`scaleEffect` 与 `rotationEffect` 修饰器的代码， 如此拖曳只发生在最上面的卡片视图。

```swift
ZStack {
    ForEach(cardViews) { cardView in
        cardView
            .zIndex(self.isTopCard(cardView: cardView) ? 1 : 0)
            .offset(x: self.isTopCard(cardView: cardView) ? self.dragState.translation.width : 0, y: self.isTopCard(cardView: cardView) ? self.dragState.translation.height : 0)
            .scaleEffect(self.dragState.isDragging && self.isTopCard(cardView: cardView) ? 0.95 : 1.0)
            .rotationEffect(Angle(degrees: self.isTopCard(cardView: cardView) ? Double( self.dragState.translation.width / 10) : 0))
            .animation(.interpolatingSpring(stiffness: 180, damping: 100), value: self.dragState.translation)
            .gesture(LongPressGesture(minimumDuration: 0.01)
                .sequenced(before: DragGesture())
                .updating(self.$dragState, body: { (value, state, transaction) in
                    switch value {
                    case .first(true):
                        state = .pressing
                    case .second(true, let drag):
                        state = .dragging(translation: drag?.translation ?? .zero)
                    default:
                        break
                    }

                })

            )
    }
}

```

只需要对 `offse`、`scaleEffect` 与 `rotationEffect` 修饰器进行修改，其余的代码保持不变。对于那些修饰器，我们进行额外的检查，以使效果只应用在最上面的卡片。

现在，如果你再次执 行 App，则应该看到其下方的卡片，并只能拖曳最上面的卡片。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-14.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-14.png)

图 19.11. 拖曳效果只应用在最上面的卡片

### 显示心形与 × 形图示

酷 ！拖曳现在可以运作了，不过它还没有完成。使用者应该能够向右/ 向左滑动，来丢弃最上面的卡片。而且，根据滑动的方向，卡片上应该显示一个图示（心形或 × 形）。

首先，我们在 `ContentView` 中声明一个拖曳的界限值：

```
private let dragThreshold: CGFloat = 80.0

```

当拖曳的位移超过界限值时，我们将在卡片上重叠一个图示（心形或× 形）。另外， 如果使用者释放卡片，App 会从卡片库中删除这张卡片，并建立一张新卡片，将其放置于卡片库的末尾。

要重叠图示，加入一个 `overlay` 修饰器至 `cardViews`。你可以在 `.zIndex` 修饰器下插入下列的代码：

```
.overlay {
    ZStack {
        Image(systemName: "x.circle")
            .foregroundColor(.white)
            .font(.system(size: 100))
            .opacity(self.dragState.translation.width < -self.dragThreshold && self.isTopCard(cardView: cardView) ? 1.0 : 0)

        Image(systemName: "heart.circle")
            .foregroundColor(.white)
            .font(.system(size: 100))
            .opacity(self.dragState.translation.width > self.dragThreshold  && self.isTopCard(cardView: cardView) ? 1.0 : 0.0)
    }
}

```

默认上，将不透明度设定为“0”来隐藏这两张图片。如果向右拖曳，则位移的宽度为正值，否则其为负值。依照拖曳的方向，当拖曳的位移超过界限值时，App 将显示其中一张图片。

你可以运行这个项目来快速测试一下。当你的拖曳超出界限值时，心形/× 形图示将会出现，如图 19.12 所示。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-15.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-15.png)

图 19.12. 出现心形图示

### 删除/ 插入卡片

现在，当你释放卡片时，它仍会回到原来的位置。我们如何才能删除最上面的卡片， 并同时加入一张新卡片呢？

首先，我们使用 `@State` 来标记 `cardViews` 数组，以让我们可以修改它的值，并重新修改 UI：

```
@State var cardViews: [CardView] = {

    var views = [CardView]()

    for index in 0..<2 {
        views.append(CardView(image: trips[index].image, title: trips[index].destination))
    }

    return views
}()

```

接下来，声明另一个状态变量来追踪旅程的最后一个索引。假设卡片库第一次初始化时，我们显示储存在 `trips` 数组中的前两个旅程，最后一个索引设定为 `1`。

```
@State private var lastIndex = 1

```

下面是用于删除及插入卡片视图的核心函数。定义一个名为 `moveCard` 新函数：

```
private func moveCard() {
    cardViews.removeFirst()

    self.lastIndex += 1
    let trip = trips[lastIndex % trips.count]

    let newCardView = CardView(image: trip.image, title: trip.destination)

    cardViews.append(newCardView)
}

```

这个函式先从 `cardViews` 数组中删除最上面的卡片，然后它使用后续旅程的图片来实例化一个新卡片视图。由于 `cardViews` 定义为状态属性，因此一旦数组的值修改时，SwiftUI 将再次渲染卡片视图，这就是我们如何删除最上面的卡片，并插入一张新卡片至卡片库的方式。

针对这个示例，我想要卡片库继续显示一个旅程。在 `trips` 数组的最后一张图片显示后， App 将会回到第一个元素（注意，上列代码中的模数运算子%）。

接下来，修改 `.gesture`修饰器，并插入 `.onEnded` 函式：

```
.gesture(LongPressGesture(minimumDuration: 0.01)
    .sequenced(before: DragGesture())
    .updating(self.$dragState, body: { (value, state, transaction) in
        .
        .
        .
    })
    .onEnded({ (value) in

        guard case .second(true, let drag?) = value else {
            return
        }

        if drag.translation.width < -self.dragThreshold ||
            drag.translation.width > self.dragThreshold {

            self.moveCard()
        }
    })
)

```

当拖曳手势结束时，我们检查拖曳的位移是否超过界限值，并相应调用 `moveCard()`。

现在，当你在预览画布中运行项目时，将图片向右 / 左拖曳，直到图示出现。放开拖曳，最上面的卡片应由下一张卡片取代。

![%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-16.png](%E4%BD%BF%E7%94%A8%E6%89%8B%E5%8A%BF%E4%B8%8E%E5%8A%A8%E7%94%BB%E5%BB%BA%E7%AB%8B%20Tinder%20%E9%A3%8E%E6%A0%BC%E7%9A%84%20UI%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%2046de586b7a6d4155a5b3df261b77d348/swiftui-trip-tinder-16.png)

图 19.13. 删除最上面的图片

### 微调动画 利用Transition

这个 App 几乎可以运作了，但是动画效果并不如预期。不要让卡片视图突然消失，而是卡片丢弃后应逐渐从屏幕离开。

要微调动画效果，我们将加上 `transition` 修饰器，并应用不对称转场至卡片视图。

现在建立一个 `AnyTransition` 扩展（可以加在 `ContentView.swift`最后端），并定义两个转场效果：

```
extension AnyTransition {
    static var trailingBottom: AnyTransition {
        AnyTransition.asymmetric(
            insertion: .identity,
            removal: AnyTransition.move(edge: .trailing).combined(with: .move(edge: .bottom))
        )

    }

    static var leadingBottom: AnyTransition {
        AnyTransition.asymmetric(
            insertion: .identity,
            removal: AnyTransition.move(edge: .leading).combined(with: .move(edge: .bottom))
        )
    }
}

```

之所以使用不对称转场，是因为我们只想在卡片视图被删除时，对转场设定动画。当一个新卡片视图插入卡片库时，则不应有动画。

当卡片视图向屏幕右方丢弃时，使用 `trailingBottom` 转场，而当卡片视图向屏幕左方丢弃时，则使用`leadingBottom` 转场。

接下来，声明一个包含转场类型的状态属性，默认是设定 `trailingBottom`。

```
@State private var removalTransition = AnyTransition.trailingBottom

```

现在，将 `.transition` 修饰器加到卡片视图。你可以将它放在 `.animation` 修饰器之后：

```
.transition(self.removalTransition)

```

最后，使用 `onChanged` 函式修改 `.gesture` 修饰器的代码，如下所示：

```swift
.gesture(LongPressGesture(minimumDuration: 0.01)
    .sequenced(before: DragGesture())
    .updating(self.$dragState, body: { (value, state, transaction) in
        switch value {
        case .first(true):
            state = .pressing
        case .second(true, let drag):
            state = .dragging(translation: drag?.translation ?? .zero)
        default:
            break
        }

    })
    .onChanged({ (value) in
        guard case .second(true, let drag?) = value else {
            return
        }

        if drag.translation.width < -self.dragThreshold {
            self.removalTransition = .leadingBottom
        }

        if drag.translation.width > self.dragThreshold {
            self.removalTransition = .trailingBottom
        }

    })
    .onEnded({ (value) in

        guard case .second(true, let drag?) = value else {
            return
        }

        if drag.translation.width < -self.dragThreshold ||
            drag.translation.width > self.dragThreshold {

            self.moveCard()
        }
    })

)

```

上列代码的作用是设定 `removalTransition`，转场类型是根据滑动方向来修改。现在，你可以再次运行 App 了，当丢弃卡片时，你应该会看到动画效果已改善。

### 本章小结

使用 SwiftUI，你可以轻松建立一些很酷的动画与行动装置 UI 模式，这个如 Tinder 般的 UI 就是一个例子。

我希望你能真正了解本章所介绍的内容，如此你就可以修改代码，来配合自己的项目。这是非常重要的一章，我想要记录一下我的思考过程，而不仅仅是向你提供最终的解决方案。

为了方便进一步参考，您可以至下列网址下载完整项目：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUITinderTrip.zip](https://www.appcoda.com/resources/swiftui4/SwiftUITinderTrip.zip)) 。