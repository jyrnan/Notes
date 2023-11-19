# 滑动删除、内容菜单与动作列表 | 精通 SwiftUI - iOS 16 版

在前面的章节中，你已经学会如何使用清单来显示数据列。而在本章中，我们将会更深入一点，了解如何让使用者和清单视图进行互动，包括：

- 使用滑动删除一列。
- 点击一列来启用动作表（action sheet）。
- 长按一列来弹出内容菜单（context menu）。

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-1.jpg](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-1.jpg)

图 16.1. 滑动删除（左图）、内容菜单与动作表（右图)

参考图 16.1，我相信你应该非常熟悉“滑动删除”与“动作表”，这两个 UI 元素在 iOS 中已经存在多年，而内容菜单则是在 iOS 13 新导入，尽管它看起来类似 3D Touch 的预览（peek ）与弹出（pop ）。对于使用内容菜单实现的任何视图（例如：按钮），每当使用者在视图上做压力触控（force touch ）时，iOS 会弹出一个弹出菜单。对于开发者而言，配置菜单中显示的动作项目是你的责任。

虽然本章的重点是“与清单的互动”，但是我所介绍的技巧也可以应用于其他的UI 控制组件（例如：按钮）。

### 准备初始项目

我们开始建立这个项目，而我们将以之前建立的清单 App 为基础，来建立一个互动式清单。你可以至

[https://www.appcoda.com/resources/swiftui4/SwiftUIActionSheetStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIActionSheetStarter.zip) 来下载项目，下载后开启项目，并检查预览，它应该会显示一个包含文字与图片的简单清单，如图 16.2 所示。稍后，我们将在这个示例 App 中加入滑动删除功能、一个动作表与一个内容菜单。

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-2.png](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-2.png)

图 16.2. 初始项目会显示一个简单的清单视图

如果你的眼睛够敏锐的话，你可能会发现初始项目使用 `ForEach` 来实现该清单。为什么我将它修改回 `ForEach`，而不是传送数据集合至 `List` 呢？主要原因是我将介绍的 `onDelete`处理器只能使用 `ForEach`。

### 实现滑动删除

假设你已经准备好初始项目了，我们开始实现“滑动删除”（swipe to delete ）功能，我已经简要提过 `onDelete` 处理器。要对清单中的所有列启用“滑动删除”功能，你只需要将这个处理器加到所有列的数据即可。因此，修改 `List` 如下：

```
List {
    ForEach(restaurants) { restaurant in
        BasicImageRow(restaurant: restaurant)
    }
    .onDelete { (indexSet) in
        self.restaurants.remove(atOffsets: indexSet)
    }
}
.listStyle(.plain)

```

在 `onDelete` 的闭包中，它将传送一个 `indexSet`，其储存要删除的列的索引，然后我们使用 `indexSet` 调用`remove` 方法，以删除在 `restaurants` 数组中的特定项目。

在“滑动删除”功能可以运作之前，还一件事要做。每当使用者从清单中删除一列时， UI 应该相应修改。正如前面章节所讨论的，SwiftUI 有一个非常强大的功能来管理应用程序的状态。在我们的代码中，当使用者选择删除一笔纪录时，`restaurants` 数组的值将更考，我们必须要求 SwiftUI 监控属性，并在属性值修改时修改UI。

为此，插入 `@State` 关键字至 `restaurants` 变量：

```
@State var restaurants = [ ... ]

```

当你修改后，你应该能够在预览试试这个删除功能。向左滑动任一列来显示出“删除”（Delete ）按钮，如图 16.3 所示。点击它，该列将从清单中删除。顺带一提，你是否注意到删除该列时的精美动画呢？你不需要撰写任何额外的代码，这个动画是由 SwiftUI 自动产生。很酷，对吧？

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-3.png](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-3.png)

图 16.3. 从清单中删除项目

如果你之前使用过 UIKit 编写相同的功能，我相信你会对 SwiftUI 感到惊讶。只需要几行代码与一个关键字，你便已实现了“滑动删除”功能。

### 建立内容菜单

接下来，我们来讨论内容菜单（Context Menu）。如前所述，内容菜单类似于 3D Touch 的预览（peek ）及弹出（pop ），有个明显的差别在于，这个功能适用于所有运行 iOS 13 与之后版本的装置，即使该装置不支持 3D Touch 也可以。要弹出内容菜单，人们需要使用长按的手势，而若是装置使用 3D Touch，则使用压力触控。

SwiftUI 使得内容菜单的实现变得非常简单，你只需要将contextMenu 容器加到视图， 并设定它的菜单项目即可。

在我们的示例 App 中，我们想在人们长按任何一列时触发内容菜单。而在菜单中，它提供了“删除”（Delete ）与“最爱”（Favorite ）等两个动作按钮，供使用者选择。当选择后，“删除”（Delete ）按钮将从清单中删除该列，“最爱”（Favorite ）按钮将以星号标记所选的列。

要在内容菜单中显示这两个项目，我们可以将 `contextMenu` 加到清单中的每一列，如下所示：

```
List {
    ForEach(restaurants) { restaurant in
        BasicImageRow(restaurant: restaurant)
            .contextMenu {

                Button(action: {
                    // 删除所选的餐厅
                }) {
                    HStack {
                        Text("Delete")
                        Image(systemName: "trash")
                    }
                }

                Button(action: {
                    // 将所选的餐厅标记为最爱
                }) {
                    HStack {
                        Text("Favorite")
                        Image(systemName: "star")
                    }
                }
            }
    }
    .onDelete { (indexSet) in
        self.restaurants.remove(atOffsets: indexSet)
    }
}
.listStyle(.plain)

```

现在，我们还没有实现任何按钮动作。不过，若你试用 App，则当你长按其中一列时， 这个 App 会弹出内容菜单，如图 16.4 所示。

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-4.png](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-4.png)

图 16.4. 从清单中删除项目

现在，我们继续实现删除动作。与 `onDelete` 处理器不同的是，`contextMenu` 不会给我们所选餐厅的索引。要找出已选餐厅的索引，则需要做一些工作。在 `ContentView` 建立一个新函数：

```
private func delete(item restaurant: Restaurant) {
    if let index = self.restaurants.firstIndex(where: { $0.id == restaurant.id }) {
        self.restaurants.remove(at: index)
    }
}

```

这个 `delete` 函数接受一个 =restaurant 物件，并在 `restaurants` 数组中搜寻它的索引。要找出索引，我们调用`firstIndex` 函数，并指定搜寻条件，这个函数会逐一运行数组，并将给定的餐厅 id 与数组中的 id 进行比对，如果有匹配的话，`firstIndex` 函式会回传给定餐厅的索引。当我们有了索引后，我们就可以通过调用 `remove(at:)` 从 `restaurants` 数组中删除餐厅。

接下来，在“// 删除所选的餐厅”下面插入下列的代码：

```
self.delete(item: restaurant)

```

当使用者选择“删除”（Delete ）按钮时，我们只调用delete 函式。现在，你已经可以测试App 了。在画布中点选“播放”（Play ）按钮，来运行该App，长按其中一列来弹出内容菜单，接着选择“删除”（Delete ），你应会看到所选的餐厅从清单中删除了。

我们继续“最爱”（Favorite ）按钮的实现。当这个按钮被选中时， App 将在所选的餐厅中放置一颗星。要实现这个功能，我们必须先修改 Restaurant 结构，并加入一个名为“isFavorite”的新属性，如下所示：

```
struct Restaurant: Identifiable {
    var id = UUID()
    var name: String
    var image: String
    var isFavorite: Bool = false
}

```

这个 `isFavorite` 属性指示餐厅是否标记为最爱，默认是设定为 `false`。

与“删除”功能类似，我们将会在 `ContentView`中建立一个单独的函数，来设定最爱的餐厅。插入下列代码来建立新函数：

```
private func setFavorite(item restaurant: Restaurant) {
    if let index = self.restaurants.firstIndex(where: { $0.id == restaurant.id }) {
        self.restaurants[index].isFavorite.toggle()
    }
}

```

这段代码与 `delete` 函式的代码很相似。我们首先找出给定餐厅的索引，当我们有了索引后，我们修改它的`isFavorite` 属性值。这里，我们调用 `toggle` 函数来切换这个值，例如：若 `isFavorite` 的原始值设定为`false`，则在调用 `toggle()` 后将变为 `true`。

接下来，我们必须处理列的 UI。当餐厅的 isFavorite 属性设定为 `true`时，列应该显示一个星号。修改`BasicImageRow` 结构如下：

```
struct BasicImageRow: View {
    var restaurant: Restaurant

    var body: some View {
        HStack {
            Image(restaurant.image)
                .resizable()
                .frame(width: 40, height: 40)
                .cornerRadius(5)
            Text(restaurant.name)

            if restaurant.isFavorite {
                Spacer()

                Image(systemName: "star.fill")
                    .foregroundColor(.yellow)
            }
        }
    }
}

```

在上列的代码中，我们只在 `HStack` 中加入一个代码片段。若给定餐厅的 `isFavorite` 属性设定为 `true`，我们加入一个留白与一个系统图片至该列。

这就是我们如何实现“最爱”功能的方式。最后在“// 将所选的餐厅标记为最爱”下面插入下列这行代码来调用setFavorite 函数：

```
self.setFavorite(item: restaurant)

```

现在该进行测试了。在画布中运行 App，长按其中一列（例如：Petite Oyster ），然后选择“最爱”（Favorite ），你应该会看到该列末尾出现一个星号，如图 16.5 所示。

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-5.png](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-5.png)

图 16.5. 使用“最爱”的功能

### 使用动作表

以上为如何实现内容菜单的方式。最后，我们来看如何在 SwiftUI 中建立一个动作表。我们将要建立的动作表提供了与内容菜单一样的相同选项，如果你忘记动作表的外观，请再次参考图 16.1。

SwiftUI 框架有一个 `ActionSheet` 视图，可以让你建立动作表。基本上，你可以建立动作列表如下：

```
ActionSheet(title: Text("What do you want to do"), message: nil, buttons: [.default(Text("Delete"))]

```

你使用标题及选项信息初始化一个动作表，`buttons` 参数接受一个按钮的数组。在示例代码中，它提供标题为“Delete”的默认按钮。

要启用动作表，可将 `actionSheet` 修饰器加到按钮或任何视图上来触发动作表。如果你研究 SwiftUI 的文件，则有两个弹出动作表的方式。

你可以使用 `isPresented` 参数来控制动作表的外观：

```
func actionSheet(isPresented: Binding<Bool>, content: () -> ActionSheet) -> some View

```

或者使用 Optional 绑定：

```
func actionSheet<T>(item: Binding<T?>, content: (T) -> ActionSheet) -> some View where T : Identifiable

```

我们将使用这两个方法来显示动作表，你将可了解何时使用哪个方法。

于第一种方法，我们需要一个布林变量来表示动作的状态，以及一个 `Restaurant` 类型的变量来储存所选的餐厅。因此，在 `ContentView`中声明这两个变量：

```
@State private var showActionSheet = false

@State private var selectedRestaurant: Restaurant?

```

默认上，`showActionSheet` 变量设定为 `false`，这表示动作表不显示。当使用者选取一列时，我们会将这个变量切换为 `true`。顾名思义，`selectedRestaurant` 变量是设计用来存放所选的餐厅。这两个变量都有 `@State` 关键字，因为我们想要 SwiftUI 监控它们的变化，并相应修改UI。

接下来，加入 `onTapGesture` 与 `actionSheet` 修饰器至 `List` 视图，如下所示：

```
List {
    ForEach(restaurants) { restaurant in
        BasicImageRow(restaurant: restaurant)
            .contextMenu {

                ...
            }
            .onTapGesture {
                self.showActionSheet.toggle()
                self.selectedRestaurant = restaurant
            }
            .actionSheet(isPresented: self.$showActionSheet) {

                ActionSheet(title: Text("What do you want to do"), message: nil, buttons: [

                    .default(Text("Mark as Favorite"), action: {
                        if let selectedRestaurant = self.selectedRestaurant {
                            self.setFavorite(item: selectedRestaurant)
                        }
                    }),

                    .destructive(Text("Delete"), action: {
                        if let selectedRestaurant = self.selectedRestaurant {
                            self.delete(item: selectedRestaurant)
                        }
                    }),

                    .cancel()
                ])
            }
    }
    .onDelete { (indexSet) in
        self.restaurants.remove(atOffsets: indexSet)
    }
}

```

加到每列的 `onTapGesture`修饰器， 是用于侦测使用者的触控。当点击一列后， `onTapGesture` 中的代码区块将会运行。这里，我们切换 `showActionSheet` 变量，并设定 `selectedRestaurant`。

之前， 我已经解释过 `actionSheet` 修饰器的用法。在上列的代码中， 我们使用 `showActionSheet` 的绑定来传送 `isPresented` 参数，当 `showActionSheet` 设定为`true`时， 则会运行该代码区块。我们建立一个具有“标记为最爱”（Mark as Favorite ）、“删除”（Delete ）与“取消”（Cancel ）等三个按钮的`ActionSheet`，而动作表有三种按钮类型， 包括“默认”（default ）、“破坏性”（destructive）与“取消”（cancel ）。对于一般动作， 动作表通常使用默认按钮类型；破坏性按钮与默认按钮非常相似，但是字型颜色设定为“红色”，以表示为一些破坏性的动作（例如：删除）；“取消”按钮是一个特别的类型， 用于解除动作表。

对于“标记为最爱”（Mark as Favorite ）按钮，我们将其建立为默认按钮。在action 闭包中，我们调用 `setFavorite` 函式来加入星星。对于“删除”（Delete ）按钮，则建立为破坏性按钮，其与内容菜单的“删除”（Delete ）按钮类似，我们调用 `delete` 函式来删除所选餐厅。

如果你已正确进行修改，则在清单视图中点击其中一列时应可弹出动作表。当选择“删除”（Delete）按钮，将会删除该列。若是你选择“标记为最爱”（Mark as Favorite）选项， 则将使用黄色星星标记该列，如图16.6 所示

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-6.png](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-6.png)

图 16.6. 点击一列来显示动作表

一切都运作得很好，不过我承诺过带你了解使用 `actionSheet` 修饰器的第二种方法。我们已经介绍过第一种方法，它依据布林值（即 `showActionSheet` ）来指示是否应显示动作表。

第二种方法是通过一个 Optional Identifiable 绑定来触发动作表：

```
func actionSheet<T>(item: Binding<T?>, content: (T) -> ActionSheet) -> some View where T : Identifiable

```

以白话来说，这表示当你传送的项目有值时，将会显示动作。对于我们的示例， `selectedRestaurant` 变量是一个 Optional，并遵守 `Identifiable` 协议。要使用第二种方法，你只需要将 `selectedRestaurant` 绑定传送至`actionSheet` 修饰器，如下所示：

```
.actionSheet(item: self.$selectedRestaurant) { restaurant in

    ActionSheet(title: Text("What do you want to do"), message: nil, buttons: [

        .default(Text("Mark as Favorite"), action: {
            self.setFavorite(item: restaurant)
        }),

        .destructive(Text("Delete"), action: {
            self.delete(item: restaurant)
        }),

        .cancel()
    ])
}

```

如果 `selectedRestaurant` 有一个值，App 将会弹出动作表。从闭包的参数中，你可以取得所选的餐厅，并运行对应的操作。当你使用这个方法，则不再需要 `shownActionSheet` 布林变量，你可以从代码删除它：

```
@State private var showActionSheet = false

```

另外，在 `tapGesture` 修饰器中，移除下列这行切换 `showActionSheet` 变量的代码：

```
self.showActionSheet.toggle()

```

再次测试 App，动作表看起来还是一样，但你是以不同的方法来实现动作表。

### 作业

现在你应该对如何建立一个内容菜单有了概念，我们来做一个作业，以测试你对内容的了解程度。你的任务是在内容菜单中加入“打卡”（ Check-in ）项目。当使用者选择该选项时，App 将会在所选餐厅加入一个打卡符号，你可以参考图 16.7 的UI 示例。对于这个示例，我使用了名为 `checkmark.seal.fill` 的系统图片作为打卡符号，但你可以使用自己的图片。

在参考解答之前，请花点时间来练习这个作业，祝你玩得愉快 ！

![%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-7.png](%E6%BB%91%E5%8A%A8%E5%88%A0%E9%99%A4%E3%80%81%E5%86%85%E5%AE%B9%E8%8F%9C%E5%8D%95%E4%B8%8E%E5%8A%A8%E4%BD%9C%E5%88%97%E8%A1%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20fc5b43788c6c4f06a937ba4c82fef61a/swiftui-actionsheet-7.png)

图 16.7. 加入“打卡”功能

在本章所准备的示例档中，有完整的项目与作业解答可以下载：

- 示例项目与作业解答 ([https://www.appcoda.com/resources/swiftui4/SwiftUIActionSheet.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIActionSheet.zip))