# 建立像 Apple Wallet App 的动画和转场效果 | 精通 SwiftUI - iOS 16 版

你使用过 Apple 电子钱包 App 吗？我相信应该有，在上一章中，我们建立了一个如 Tinder UI 的简单 App，而本章要做的是建立一个动态 UI，类似于你在电子钱包 App 中看到的 UI。当你在电子钱包 App 中长按信用卡时，则可使用拖曳手势来重新排列卡片。如果你没有使用过这个 App，请开启电子钱包快速浏览一下，或者你可以访问这个URL （[https://link.appcoda.com/swiftui-wallet](https://link.appcoda.com/swiftui-wallet)）来了解我们将建立的动画。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-1.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-1.png)

图 20.1. 建立如电子钱包的动画与视图转场

在电子钱包 App 中，点击其中一张信用卡，就会弹出交易历史纪录。我们还建立一个类似的动画，以让你更了解视图转场与水平滚动视图。

### 项目准备

为了让你专注于学习动画与视图转场，你可以从初始项目开始([https://www.appcoda.com/resources/swiftui4/SwiftUIWalletStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIWalletStarter.zip))。这个初始项目已经绑定了所需的信用卡图片，并且带有内建的交易历史纪录视图，如果想要使用自己的图片，请在素材目录中替换它们，如图20.2 所示。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-2.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-2.png)

图 20.2 初始项目绑定了信用卡图片

在项目导航器中，你应该会发现一些 `.swift` 档：

- **Transaction.swift** - `Transaction` 结构代表电子钱包 App中的交易。每一笔交易有一个唯一的 ID、交易商、金额、日期与图示。除了 `Transaction` 结构之外，作为示范之用，我们还声明一个测试交易的数组。
- **Card.swift** - 这个文件包含了 `Card` 的结构。`Card` 表示信用卡的数据，包含卡号、类型、有效日期、图片与客户姓名，除此之外，你可以在文件中找到一个测试信用卡的数组。需要注意的一点是，卡片图片中不包含任何的个人资讯，而只包含卡片品牌（例如： Visa ）。稍后，我们将为信用卡建立一个视图。
- **TransactionHistoryView.swift** - 这是图 20.1中显示的交易历史纪录，初始项目已经带有交易历史纪录视图的实现。我们在水平滚动视图中显示交易，你之前已经使用过垂直滚动视图，而建立水平视图的技巧是在滚动视图初始化期间传送 `.horizontal` 值。请看一下图 20.3 或 Swift 档来了解详细资讯 。
- **ContentView.swift** - 这是 Xcode产生的默认 SwiftUI 视图。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-3.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-3.png)

图 20.3. 使用.horizontal 建立水平滚动视图

### 建立卡片视图

如上一节所述，所有的卡片图片皆不包含任何个人资讯与卡号。再次开启素材目录， 并看一下图片，每张卡片图片只具有卡片标志。我们将很快建立一个卡片视图，来布局个人资讯与卡号，如图 20.4 所示。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-4.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-4.png)

图 20.4. 示例卡片

要建立卡片视图，则在项目导航器中，右键点选 `View` 群组，然后建立一个新文件。选取“SwiftUI View”模板，文件名称命名为 `CardView.swift`。接下来，修改代码如下：

```
struct CardView: View {
    var card: Card

    var body: some View {
        Image(card.image)
        .resizable()
        .scaledToFit()
            .overlay(

                VStack(alignment: .leading) {
                    Text(card.number)
                        .bold()
                    HStack {
                        Text(card.name)
                            .bold()
                        Text("Valid Thru")
                            .font(.footnote)
                        Text(card.expiryDate)
                            .font(.footnote)
                    }
                }
                .foregroundColor(.white)
                .padding(.leading, 25)
                .padding(.bottom, 20)

            , alignment: .bottomLeading)
            .shadow(color: .gray, radius: 1.0, x: 0.0, y: 1.0)

    }
}

```

我们声明一个 `card` 属性来带入卡片数据。为了在卡片图片上显示个人数据与卡号，我们使用 `overlay` 修饰器，并以垂直堆叠视图与水平堆叠视图来布局文字组件。

要预览卡片，则修改 `CardView_Previews`结构如下：

```
struct CardView_Previews: PreviewProvider {
    static var previews: some View {
        ForEach(testCards) { card in
            CardView(card: card).previewDisplayName(card.type.rawValue)
        }
    }
}

```

`testCards` 变量在 `Card.swift` 中定义，因此我们使用 `ForEach` 来逐一运行卡片，并调用 `previewDisplayName` 来设定预览的名称。Xcode 将如图 20.5 所示布局卡片。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-5.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-5.png)

图 20.5. 预览卡片视图

### 建立电子钱包视图与卡片库

我们现在已经实现了卡片视图，让我们开始建立电子钱包视图。如果你忘记电子钱包视图的外观，请看一下图 20.6。在进行手势与动画之前，我们将先布局卡片库。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-6.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-6.png)

图 20.6 钱包视图

在项目导航器中，你应该会看到 `ContentView.swift` 档。删除它，然后右键点选 `View` 数据夹，以建立一个新文件。在对话方块中，选取“SwiftUI View”作为模板，并将文件命名为 `WalletView.swift`。

如果你预览 `WalletView` 或在模拟器中运行 App，Xcode 应该会显示一个错误，因为 `ContentView` 设定为初始视图，并且其被删除了。要修正这个错误，则开启 `SwiftUIWallet App.swift`，并将 `WindowGroup` 中下列这行代码：

```
ContentView()

```

修改为：

```
WalletView()

```

切换回 `WalletView.swift`。当你修改后，将可修正编译错误，现在我们继续布局电子钱包视图。首先，我们从标题列开始，在 `WalletView.swift` 档中，为标题列插入一个新结构：

```
struct TopNavBar: View {

    var body: some View {
        HStack {
            Text("Wallet")
                .font(.system(.largeTitle, design: .rounded))
                .fontWeight(.heavy)

            Spacer()

            Image(systemName: "plus.circle.fill")
                .font(.system(.title))
        }
        .padding(.horizontal)
        .padding(.top, 20)
    }
}

```

代码非常简单，我们使用水平堆叠来布局标题与加号图片。

接下来，是针对卡片库。首先，在 `WalletView` 结构中，为信用卡数组声明一个属性：

```
var cards: [Card] = testCards

```

为了示范，我们只将默认值设定为 `Card.swift` 档中定义的 `testCards`。要布局电子钱包视图，我们同时使用`VStack` 与 `ZStack`，修改 `body` 变量如下：

```
var body: some View {
   VStack {

        TopNavBar()
            .padding(.bottom)

        Spacer()

        ZStack {
            ForEach(cards) { card in
                CardView(card: card)
                    .padding(.horizontal, 35)
            }
        }

        Spacer()
    }
}

```

如果你在模拟器中运行这个 App 或直接预览 UI，则应该只看到卡片库中的最后一张卡片，如图 20.7 所示。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-7.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-7.png)

图 20.7. 尝试显示卡片库

目前的实现有两个问题：

1. *卡片现在彼此重叠了* - 我们需要找出一种方式来展开一副卡片。
2. *Discover卡应该是最后一张卡片* - 在 ZStack中，项目彼此堆叠在一起。放入 ZStack的第一个项目成为最低层，而最后一个项目成为最高层。如果你看一下 `Card.swift` 中的 `testCards` 数组，第一张卡片是 Visa 卡，最后一张卡片是 Discover 卡。

那么，我们要如何修正这个问题呢？对于第一个问题，我们可以使用 `offset`修饰器来展开一副卡片。而对于第二个问题，我们显然可以修改每个 `CardView` 的 `zIndex`，以改变卡片的顺序。图 20.8 说明了这个解决方案是如何工作的。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-8.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-8.png)

图 20.8. 了解zIndex 与偏移量

我们先讨论一下 z-index。每张卡片的 z-index 是其在 `cards` 数组中索引的负值，如此最后一个项目拥有数组索引的最大值，也将会有最小的 z-index。对于实际的实现，我们将建立一个单独的函数来处理 z-index 的计算。在`WalletView` 中，插入下列的代码：

```
private func zIndex(for card: Card) -> Double {
    guard let cardIndex = index(for: card) else {
        return 0.0
    }

    return -Double(cardIndex)
}

private func index(for card: Card) -> Int? {
    guard let index = cards.firstIndex(where: { $0.id == card.id }) else {
        return nil
    }

    return index
}

```

这两个函数可以一起找出给定卡片的正确 z-index。要计算正确的 z-index，我们首先需要取得 `cards` 数组中卡片的索引值，`index(for:)` 函数是为了找出给定卡片的数组索引值而设计的。当我们有了索引值后，就可以将其变成负值，这就是 `zIndex(for:)` 函数的作用。

现在，你可以将 `zIndex` 修饰器加到 `CardView`，如下所示：

```
CardView(card: card)
    .padding(.horizontal, 35)
    .zIndex(self.zIndex(for: card))

```

当你修改后，Visa 卡片应该移到卡片库的最上方。

接下来，我们修正第一个问题来展开卡片，每张卡片都应偏移一定的垂直距离。而这个距离是使用卡片的索引值来计算的。假设我们将默认的垂直偏移量设定为 50 点，最后一张卡片将会位移 200 点（50×4）。

现在，你应该了解我们将如何展开卡片了，我们来编写代码。在 `WalletView` 中声明默认的垂直偏移量：

```
private static let cardOffset: CGFloat = 50.0

```

接下来，建立一个名为 `offset(for:)`的新函数，用于计算给定卡片的垂直偏移量：

```
private func offset(for card: Card) -> CGSize {

    guard let cardIndex = index(for: card) else {
        return CGSize()
    }

    return CGSize(width: 0, height: -50 * CGFloat(cardIndex))
}

```

最后，将 `offset` 修饰器加入到 `CardView`：

```
CardView(card: card)
    .padding(.horizontal, 35)
    .offset(self.offset(for: card))
    .zIndex(self.zIndex(for: card))

```

这就是我们使用 `offset` 修饰器来展开卡片的方式。若是一切正确，你应该会看到如图 20.9 所示的预览。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-9.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-9.png)

图 20.9. 展开卡片

### 加入滑入动画

我们现在已经完成了电子钱包视图的布局，是时候加入一些动画了，我要加入的第一个动画是滑入动画。当第一次开启 App 时，每张卡片都从屏幕最左侧滑入，你可能认为这个动画是不必要的，但是我想藉此机会教你如何建立动画以及开启 App 时的视图转场。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-10.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-10.png)

图 20.10 滑入动画

首先，我们需要一种触发过渡动画的方法， 先在 `CardView` 的开头声明一个状态变量：

```
@State private var isCardPresented = false

```

此变量指示卡片是否应显示在屏幕上。 在默认的情况下，它设置为`false`。 稍后，我们将此值设置为 `true` 以启动视图转换。

每张卡片都是一个视图。要实现如图 20.10 所示的动画，我们需要将 `transition` 与 `animation` 修饰器加到`CardView` 上，如下所示：

```
CardView(card: card)
    .offset(self.offset(for: card))
    .padding(.horizontal, 35)
    .zIndex(self.zIndex(for: card))
    .transition(AnyTransition.slide.combined(with: .move(edge: .leading)).combined(with: .opacity))
    .animation(self.transitionAnimation(for: card), value: isCardPresented)

```

对于转场，我们将默认的滑动转场与移动转场结合在一起。如前所述，若是没有 `animation` 修饰器，则转场将不会动画化，这就是为何我们还要加入 `animation` 修饰器的缘故。由于每张卡片都有自己的动画，我们建立一个名为 `transitionAnimation(for:)` 函数来计算动画。插入下列的代码来建立函数：

```
private func transitionAnimation(for card: Card) -> Animation {
    var delay = 0.0

    if let index = index(for: card) {
        delay = Double(cards.count - index) * 0.1
    }

    return Animation.spring(response: 0.1, dampingFraction: 0.8, blendDuration: 0.02).delay(delay)
}

```

事实上，所有的卡片都有相似的动画（即弹簧动画），差别在于“延迟”（delay ）。卡片库的最后一张卡片将先出现，因此延迟值应该最小。下面是我们如何计算每张卡片的延迟的公式，索引值越小，延迟越长。

```
delay = Double(cards.count - index) * 0.1

```

哪我们要如何在 App 启动的时候触发转场？诀窍就是为每一个卡视图加入`id` 修饰器。

```
CardView(card: card)
    .
    .
    .
    .id(isCardPresented)
    .
    .animation(self.transitionAnimation(for: card), value: isCardPresented)

```

`id` 的值设定为 `isCardPresented` 。之后，加入 `onAppear` 修饰器，并将它加到 `ZStack`：

```
.onAppear {
    isCardPresented.toggle()
}

```

当 `ZStack` 出现时，我们将 `isCardPresented` 的值从 `false` 修改为 `true`，这将触发卡片的视图动画。应用修改后，在预览画布中点墼“Play”按钮，来进行测试。

修改后，点击 *Play* 按钮在模拟器中测试App，App启动时就会呈现动画。

### 处理点击手势及显示交易纪录

当使用者点击卡片时，App 会向上移动所选的卡片，并且显示历史交易纪录。对于其他没有选到的卡片，它们会被移出屏幕。

要实现这个功能，我们还需要两个状态变量。在 `WalletView` 中声明这些变量：

```
@State var isCardPressed = false
@State var selectedCard: Card?

```

`isCardPressed` 变量指示是否选择卡片，而 `selectedCard` 变量储存使用者选择的卡片。

```
.gesture(
    TapGesture()
        .onEnded({ _ in
            withAnimation(.easeOut(duration: 0.15).delay(0.1)) {
                self.isCardPressed.toggle()
                self.selectedCard = self.isCardPressed ? card : nil
            }
        })
)

```

要处理点击手势，我们可以将上述的 `gesture` 修饰器加到 `CardView`，并使用内建的 `TapGesture` 来捕捉点击事件。在代码区块中，我们只需切换`isCardPressed`的状态，并将目前的卡片设定为 `selectedCard` 变量。

要将所选的卡片（及其下方的卡片）向上移动，并让其余的卡片移出屏幕的话，则修改 `offset(for:)` 函数如下：

```
private func offset(for card: Card) -> CGSize {

    guard let cardIndex = index(for: card) else {
        return CGSize()
    }

    if isCardPressed {
        guard let selectedCard = self.selectedCard,
            let selectedCardIndex = index(for: selectedCard) else {
                return .zero
        }

        if cardIndex >= selectedCardIndex {
            return .zero
        }

        let offset = CGSize(width: 0, height: 1400)

        return offset
    }

    return CGSize(width: 0, height: -50 * CGFloat(cardIndex))
}

```

我们加入了一个 `if` 语句来检查卡片是否被选中。如果给定的卡片是使用者选择的卡片， 则我们将偏移量设定为`.zero`。对于所选卡片正下方的那些卡片，我们也将向上移动， 这就是为什么我们将偏移量设定为 `.zero`。而其余的卡片，我们将它们移出屏幕，因此垂直偏移量设定为 `1400 点`。

现在，我们已经准备好编写用于弹出交易历史纪录视图的代码。正如一开始所述， 初始项目已经提供这个交易历史纪录视图。因此，你不需要自己建立它。

藉由 `isCardPressed` 状态变量，我们可以使用它来确定是否显示交易历史纪录视图。在 `Spacer()` 前面插入下列代码：

```
if isCardPressed {
    TransactionHistoryView(transactions: testTransactions)
        .padding(.top, 10)
        .transition(.move(edge: .bottom))
}

```

在上列的代码中，我们设定转场为 `.move`，以从屏幕底部带入视图，你可以依照自己的喜好来随意修改它。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-11.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-11.png)

图 20.11 显示交易历史纪录

### 使用拖曳手势重新排列卡片

现在，来到本章的核心部分，我们来看如何让使用者以拖曳手势重新排列卡片库。首先，我详细描述此功能的工作原理：

1. 要开始拖曳动作，使用者必须长按卡片。简单的点击将只会弹出交易历史纪录。
2. 当使用者成功按住卡片后，App会将卡片向上移动一点。这是我们想要给使用者的一种回馈，告诉使用者已经准备好可任意拖曳卡片了。
3. 当使用者拖曳卡片时，使用者应该能在卡片库中移动它。
4. 使用者在某个位置放开卡片后，App会修改卡片库中的所有卡片位置。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-12.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-12.png)

图 20.12. 使用拖曳手势在卡片库上移动卡片

### 处理长按与拖曳手势

现在你应该了解我们将要做什么，我们来继续实现。如果你忘记我们如何使用 SwiftUI 处理手势，则请回头阅读第 17 章，该章已经讨论了我们将使用的大多数技术。

首先，在 `WalletView.swift` 中插入下列的代码，来建立 `DragState` 枚举，以使我们可以轻松追踪拖曳状态：

```
enum DragState {
    case inactive
    case pressing(index: Int? = nil)
    case dragging(index: Int? = nil, translation: CGSize)

    var index: Int? {
        switch self {
        case .pressing(let index), .dragging(let index, _):
            return index
        case .inactive:
            return nil
        }
    }
    var translation: CGSize {
        switch self {
        case .inactive, .pressing:
            return .zero
        case .dragging(_, let translation):
            return translation
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

    var isDragging: Bool {
        switch self {
        case .dragging:
            return true
        case .inactive, .pressing:
            return false
        }
    }
}

```

接下来，在 `WalletView` 中声明一个状态变量，来持续追踪拖曳状态：

```
@GestureState private var dragState = DragState.inactive

```

如果你之前阅读过第 17 章，那么你应该已经知道如何侦测长按与拖曳手势。然而，这次有点不同，我们需要同时处理点击手势、拖曳与长按手势。而且，如果侦测到长按手势，则 App 应该忽略点击手势。

现在修改 `CardView` 的 `gesture` 修饰器如下：

```
.gesture(
    TapGesture()
        .onEnded({ _ in
            withAnimation(.easeOut(duration: 0.15).delay(0.1)) {
                self.isCardPressed.toggle()
                self.selectedCard = self.isCardPressed ? card : nil
            }
        })
        .exclusively(before: LongPressGesture(minimumDuration: 0.05)
        .sequenced(before: DragGesture())
        .updating(self.$dragState, body: { (value, state, transaction) in
            switch value {
            case .first(true):
                state = .pressing(index: self.index(for: card))
            case .second(true, let drag):
                state = .dragging(index: self.index(for: card), translation: drag?.translation ?? .zero)
            default:
                break
            }

        })
        .onEnded({ (value) in

            guard case .second(true, let drag?) = value else {
                return
            }

            // 重新排列卡片
        })

    )
)

```

SwiftUI 让你可专门组合多种手势。在上列的代码中，我们告诉 SwiftUI 捕捉点击手势或长按手势，换句话说，当侦测到点击手势时，SwiftUI 将忽略长按手势。

<aside>
💡 上面代码的意思是：
如果是单点就执行单点，所以就执行.onEnded方法
如果按下不放，就变成了长按，因此单点的方法.onEnded就不执行。**长按和后面的drag变成组合手势**：分别处理长按（上移20）的和拖动（跟随）的状态

</aside>

点击手势的代码与我们之前编写的代码完全相同，而拖曳手势是排列在长按手势之后。在 `updating` 函数中，我们将拖曳的状态、转场与卡片的索引值设定为之前定义的 `dragState` 变量。我将不会像第 17 章那样详细解释代码。

在拖曳卡片之前，你必须修改 `offset(for:)` 函数如下：

```
private func offset(for card: Card) -> CGSize {

    guard let cardIndex = index(for: card) else {
        return CGSize()
    }

    if isCardPressed {
        guard let selectedCard = self.selectedCard,
            let selectedCardIndex = index(for: selectedCard) else {
                return .zero
        }

        if cardIndex >= selectedCardIndex {
            return .zero
        }

        let offset = CGSize(width: 0, height: 1400)

        return offset
    }

    // Handle dragging
    var pressedOffset = CGSize.zero
    var dragOffsetY: CGFloat = 0.0

    if let draggingIndex = dragState.index,
        cardIndex == draggingIndex {
        pressedOffset.height = dragState.isPressing ? -20 : 0

        switch dragState.translation.width {
        case let width where width < -10: pressedOffset.width = -20
        case let width where width > 10: pressedOffset.width = 20
        default: break
        }

        dragOffsetY = dragState.translation.height
    }

    return CGSize(width: 0 + pressedOffset.width, height: -50 * CGFloat(cardIndex) + pressedOffset.height + dragOffsetY)
}

```

我们加入一段代码区块来处理拖曳。请谨记，只有选定的卡片是可拖曳的。因此， 在修改偏移量之前，我们需要检查给定的卡片是否为使用者拖曳的卡片。

之前，我们将卡片索引值储存在 `dragState` 变量中，因此我们可轻松比较给定的卡片索引值与储存在 `dragState` 中的卡片索引值，以找出拖曳的卡片。

对于拖曳的卡片，我们在水平与垂直方向上都加入了额外的偏移量。

现在，你可以运行App 来进行测试，长按卡片并任意拖曳，如图20.13 所示。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-13.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-13.png)

图 20.13. 拖曳卡片

现在，你应该可以拖曳卡片，不过卡片的 z-index 并没有相应做修改，例如：如果你拖曳Visa 卡，它总是停留在卡片库的最上层，我们通过修改 `zIndex(for:)` 函数来修正它：

```
private func zIndex(for card: Card) -> Double {
    guard let cardIndex = index(for: card) else {
        return 0.0
    }

    // 卡片的默认 z-index 设定为卡片索引值的负值，
    // 因此第一张卡片具有最大的 z-index
    let defaultZIndex = -Double(cardIndex)

    // 如果它是拖曳的卡片
    if let draggingIndex = dragState.index,
        cardIndex == draggingIndex {
        // 我们根据位移的高度来计算新的 z-index
        return defaultZIndex + Double(dragState.translation.height/Self.cardOffset)
    }

    // 否则我们回传默认的 z-index
    return defaultZIndex
}

```

默认的 z-index 仍设定为卡片索引值的负值。对于拖曳的卡片，当使用者在卡片库上拖曳时，我们需要计算一个新的 z-index。修改后的 z-index 是根据位移的高度与卡片的默认偏移量（即 50 点）来计算。

运行 App 并尝试再次拖曳 Visa 卡片。现在，当你拖曳卡片时，z-index 会不断修改。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-14.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-14.png)

图 20.14. 将 Visa 卡移到后面

### 修改卡片库

当你放开卡片时，它现在会回到原来的位置，那么我们如何在拖曳之后重新排序卡片的位置呢？

这里的技巧是修改 `cards`数组的项目，以触发 UI 修改。首先，我们需要将 `cards` 变量标记为状态变量，如下所示：

```
@State var cards: [Card] = testCards

```

接下来，我们建立另一个新函数来重新排列卡片：

```
private func rearrangeCards(with card: Card, dragOffset: CGSize) {
    guard let draggingCardIndex = index(for: card) else {
        return
    }

    var newIndex = draggingCardIndex + Int(-dragOffset.height / Self.cardOffset)
    newIndex = newIndex >= cards.count ? cards.count - 1 : newIndex
    newIndex = newIndex < 0 ? 0 : newIndex

    let removedCard = cards.remove(at: draggingCardIndex)
    cards.insert(removedCard, at: newIndex)

}

```

当你将卡片拖曳到相邻的卡片上时，一旦拖曳的位移量大于默认的偏移量，我们便需要修改 z-index。图 20.15 显示了拖曳的预期行为。

![%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-15.png](%E5%BB%BA%E7%AB%8B%E5%83%8F%20Apple%20Wallet%20App%20%E7%9A%84%E5%8A%A8%E7%94%BB%E5%92%8C%E8%BD%AC%E5%9C%BA%E6%95%88%E6%9E%9C%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%200d21980fbbf948a191c3847139a3633a/swiftui-wallet-15.png)

图 20.15. 在相邻卡片之间拖曳万事达卡

这是我们计算修改后的 z-index 的公式：

```
var newIndex = draggingCardIndex + Int(-dragOffset.height / Self.cardOffset)

```

一旦我们有了修改后的索引值，最后一步就是通过移除拖曳的卡片，并将其插入新位置，以修改在 `cards` 数组中的项目。由于 `cards` 数组现在是一个状态变量，因此 SwiftUI 修改卡片库且自动渲染动画。

最后，在“// 重新排列卡片”的下面插入下列这行代码来调用函数：

```
withAnimation(.spring()) {
    self.rearrangeCards(with: card, dragOffset: drag.translation)
}

```

之后，你可以运行 App 来测试它。你已经建立了如电子钱包般的动画。

### 本章小结

阅读完本章后，我希望你对于 SwiftUI 动画与视图转场有更深入的了解。如果你将 SwiftUI 与原来的UIKit 框架进行比较，你会发现 SwiftUI 让“使用动画”变得非常容易。你还记得如何为使用者放开拖曳的卡片来渲染卡片动画吗？你需要做的是修改状态变量， 而 SwiftUI 就会处理这些繁重的工作，这就是 SwiftUI 的力量。

为了方便进一步参考，您可以至下列网址下载完整项目：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIWallet.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIWallet.zip))