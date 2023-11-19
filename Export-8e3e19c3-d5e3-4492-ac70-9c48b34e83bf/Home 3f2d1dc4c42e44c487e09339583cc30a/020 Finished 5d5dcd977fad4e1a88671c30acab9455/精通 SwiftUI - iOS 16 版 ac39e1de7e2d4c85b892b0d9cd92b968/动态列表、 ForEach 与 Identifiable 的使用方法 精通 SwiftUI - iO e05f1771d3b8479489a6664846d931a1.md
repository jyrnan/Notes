# 动态列表、 ForEach 与 Identifiable 的使用方法 | 精通 SwiftUI - iOS 16 版

在 UIKit 中，表格视图是 iOS 中最常见的 UI 控制组件之一。如果你之前使用过 UIKit 开发 App，则你应该知道可使用表格视图显示数据清单。这个 UI 控制组件在以内容为主的App 很常见，例如：报纸 App。图 10.1 展示了一些清单 / 表格视图，你可在诸如 Instagram、Twitter、Airbnb 与 Apple News 等流行应用程序中找到这些视图。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-1.jpg](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-1.jpg)

图 10.1. 清单视图示例

在 SwiftUI 中，我们使用 `List` 代替表格视图来显示数据列。如果你之前使用过 `UIKit` 建立表格视图，你应该知道实现一个简单的表格视图，就需要花一点工夫。而若是要建立自订 Cell 布局的表格视图，则要做的工作更多。SwiftUI 简化了整个过程，只需几行代码，你就能以表格形式来陈列数据。即使你需要自订列的布局，也只需要极少的工夫即可办到。

仍是觉得困惑吗？待会你就能明白我的意思。

在本章中，我们将从一个简单的清单来开始。当你了解这些基础知识，我将教你如何以更复杂的布局来陈列数据清单，如图 10.2 所示。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-2.jpg](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-2.jpg)

图 10.2. 建立一个简单且多样化的清单

### 建立一个简单的清单

我们从简单的清单来开始。首先，开启 Xcode，并使用“App”模板建立一个新项目。在下一个画面中，设定项目名称为“SwiftUIList”（或你喜欢的任何名称），并填入所有必填的值，只需确保在“Interface”选项中选取“SwiftUI”。

Xcode 应会在 `ContentView.swift` 档中产生一些代码。现在修改代码如下：

```
struct ContentView: View {
    var body: some View {
        List {
            Text("Item 1")
            Text("Item 2")
            Text("Item 3")
            Text("Item 4")
        }
    }
}

```

以上是建立一个简单的清单或表格所需要的代码。当你将文字视图嵌入 `List` 中时，清单视图会以列显示数据。这里，每一列显示具不同叙述的文字视图。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-3.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-3.png)

图 10.3. 建立一个简单的清单

相同的代码片段可以使用 `ForEach` 来编写，如下所示：

```
struct ContentView: View {
    var body: some View {
        List {
            ForEach(1...4, id: \.self) { index in
                Text("Item \(index)")
            }
        }
    }
}

```

由于这些文字视图非常相似，因此你可在 SwiftUI 中使用 `ForEach` 回圈来建立视图。

> 从已识别的底层集合中，依照需求计算视图的一种结构。
> 
> - Apple 官方文件 ([https://developer.apple.com/documentation/swiftui/foreach](https://developer.apple.com/documentation/swiftui/foreach))

你可以提供 `ForEach` 一组数据集合或一个范围。不过，你必须要注意的是，你需要告诉 `ForEach` 如何识别集合中的每个项目，参数 `id` 的目的在于此，为什么 `ForEach` 需要识别项目的唯一性呢？ SwiftUI 功能强大，可在部分或全部集合内的项目修改时自动修改 UI， 因此当修改或删除项目时，需要一个识别码来识别该项目。

在上面的代码中，我们传送给 `ForEach` 一个范围的值来逐一运行。该识别码设定为其值（即 1、2、3、4），`index` 参数储存回圈的目前值，例如：它从“1”这个值开始， `index` 参数的值则为“1”。

在闭包中，即是渲染视图所需的代码，这里也是我们需要建立的文字视图，其叙述将依据回圈中的 `index` 值而变化。如此，你就可以在清单中建立四个不同标题的项目。

我再教你一种技巧，相同的代码片段也可以进一步重写如下：

```
struct ContentView: View {
    var body: some View {
        List {
            ForEach(1...4, id: \.self) {
                Text("Item \($0)")
            }
        }
    }
}

```

你可以省略 `index` 参数，并使用参数名称缩写 `$0`，它表示闭包的第一个参数。

我们进一步将代码重写得更简单些，你可将数据集合直接传送到 `List` 视图，代码如下：

```
struct ContentView: View {
    var body: some View {
        List(1...4, id: \.self) {
            Text("Item \($0)")
        }
    }
}

```

如你所见，只需两行代码，即可建立一个简单的清单 / 表格。

### 建立具文字及图片的清单视图

现在你已经知道如何建立一个简单的清单，接着我们来看如何使用更多样化的布局。在大多数的情况下，清单视图的项目皆会包含文字与图片，而你该如何实现呢？如果你知道 `Image`、`Text`、`VStack` 与 `HStack` 的用法的话，你应该对如何建立它有概念了。

如果你阅读过《[iOS App 程序开发实务心法](https://www.appcoda.com.tw/swift)》一书，你应该非常熟悉这个示例。我们以此为例来看使用 SwiftUI 建立相同的表格有多么容易，如图 10.4 所示。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-4.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-4.png)

图 10.4 显示餐厅列的简单表格视图

要使用 UIKit 建立表格，你需要建立一个表格视图或表格视图控制器，然后自订 Prototype Cell。另外，你还必须编写表格视图数据来源的代码，以提供数据。建立一个表格UI 需要很多的步骤，现在我们来看如何在 SwiftUI 中实现相同的表格视图。

首先，我们至下列网址来下载图片素材：[https://www.appcoda.com/resources/swiftui/SwiftUISimpleTableImages.zip](https://www.appcoda.com/resources/swiftui/SwiftUISimpleTableImages.zip) 然后将 zip 档解压缩，并把所有图片汇入素材目录，如图 10.5 所示。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-5.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-5.png)

图 10.5. 汇入图片至素材目录

现在，切换到 `ContentView.swift` 来编写 UI 的代码。首先，我们在 `ContentView` 中声明两个数组，这些数组是用来储存餐厅名称与图片。下列是完整的代码：

```
struct ContentView: View {

    var restaurantNames = ["Cafe Deadend", "Homei", "Teakha", "Cafe Loisl", "Petite Oyster", "For Kee Restaurant", "Po's Atelier", "Bourke Street Bakery", "Haigh's Chocolate", "Palomino Espresso", "Upstate", "Traif", "Graham Avenue Meats And Deli", "Waffle & Wolf", "Five Leaves", "Cafe Lore", "Confessional", "Barrafina", "Donostia", "Royal Oak", "CASK Pub and Kitchen"]

    var restaurantImages = ["cafedeadend", "homei", "teakha", "cafeloisl", "petiteoyster", "forkeerestaurant", "posatelier", "bourkestreetbakery", "haighschocolate", "palominoespresso", "upstate", "traif", "grahamavenuemeats", "wafflewolf", "fiveleaves", "cafelore", "confessional", "barrafina", "donostia", "royaloak", "caskpubkitchen"]

    var body: some View {
        List(1...4, id: \.self) {
            Text("Item \($0)")
        }
    }
}

```

两个数组具有相同项目数，`restaurantNames` 数组储存餐厅名称，`restaurantImages` 变量储存你刚汇入的图片名称。要建立如图 10.4 所示的清单视图，你只需要修改 `body` 变量如下： :

```
var body: some View {
    List(restaurantNames.indices, id: \.self) { index in
        HStack {
            Image(self.restaurantImages[index])
                .resizable()
                .frame(width: 40, height: 40)
                .cornerRadius(5)
            Text(self.restaurantNames[index])
        }
    }
}

```

我们做了一些修改，首先是 `List` 视图，我们传送餐厅名称的范围（即 `restaurantNames. indices` ），而不是一个固定的范围。例如：`restaurantNames` 数组有 21 个项目，范围是从 0 至 20。

在闭包中，代码会修改，以建立列的布局，我将不会深入探讨细节，因为如果你对堆叠视图已经完全了解，那么代码一看便明白了。为了修改`List` 视图的样式，我们附加了`listStyle` 修饰器并将样式设置为`plain`。

就是这样，使用不到 10 行的代码，我们已经建立了一个自订布局的清单（或表格）。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-6.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-6.png)

图 10.6. 自订列布局的清单视图

### 使用数据集合

如前所述，`List` 可以带入一个范围或一个数据集合。你已经学过如何处理范围，我们来看如何将 `List` 与餐厅数据的数组一起使用。

我们将建立一个 `Restaurant` 结构来加以组织数据，而不是将餐厅数据储存在两个单独的数组中。这个结构有两个属性：“name”与“image”。在 `ContentView.swift` 档的最后面， 插入下列代码：

```
struct Restaurant {
    var name: String
    var image: String
}

```

使用这个结构，我们可以将 `restaurantNames` 与 `restaurantImages` 数组合并为一个数组。现在删除`restaurantNames` 与 `restaurantImages` 变量，并以 `ContentView`中的这个变量来代替：

```
var restaurants = [ Restaurant(name: "Cafe Deadend", image: "cafedeadend"),
               Restaurant(name: "Homei", image: "homei"),
               Restaurant(name: "Teakha", image: "teakha"),
               Restaurant(name: "Cafe Loisl", image: "cafeloisl"),
               Restaurant(name: "Petite Oyster", image: "petiteoyster"),
               Restaurant(name: "For Kee Restaurant", image: "forkeerestaurant"),
               Restaurant(name: "Po's Atelier", image: "posatelier"),
               Restaurant(name: "Bourke Street Bakery", image: "bourkestreetbakery"),
               Restaurant(name: "Haigh's Chocolate", image: "haighschocolate"),
               Restaurant(name: "Palomino Espresso", image: "palominoespresso"),
               Restaurant(name: "Upstate", image: "upstate"),
               Restaurant(name: "Traif", image: "traif"),
               Restaurant(name: "Graham Avenue Meats And Deli", image: "grahamavenuemeats"),
               Restaurant(name: "Waffle & Wolf", image: "wafflewolf"),
               Restaurant(name: "Five Leaves", image: "fiveleaves"),
               Restaurant(name: "Cafe Lore", image: "cafelore"),
               Restaurant(name: "Confessional", image: "confessional"),
               Restaurant(name: "Barrafina", image: "barrafina"),
               Restaurant(name: "Donostia", image: "donostia"),
               Restaurant(name: "Royal Oak", image: "royaloak"),
               Restaurant(name: "CASK Pub and Kitchen", image: "caskpubkitchen")
]

```

如果你是 Swift 新手，这里做个解释，数组的每一个项目表示一笔特定餐厅的纪录。当你进行修改后，你将会在Xcode 中见到一个错误，其指出遗失了`restaurantNames` 变量， 这很正常，因为我们刚才删除这个变量了。

现在修改 `body` 变量如下：

```
var body: some View {
    List(restaurants, id: \.name) { restaurant in
        HStack {
            Image(restaurant.image)
                .resizable()
                .frame(width: 40, height: 40)
                .cornerRadius(5)
            Text(restaurant.name)
        }
    }
    .listStyle(.plain)
}

```

看一下我们传入 `List` 的参数，我们没有传送范围，而是传送 `restaurants` 数组，并告诉 `List` 使用其 `name` 属性作为识别码。`List` 将逐一运行数组，并让我们知道它在闭包中正在处理的目前餐厅。因此，在闭包中，我们指示清单如何显示餐厅列，这里我们只在 `HStack` 中同时显示餐厅图片与餐厅名称。

一切都没有改变，UI 仍然相同，不过底层代码已经修改为利用 `List` 与数据集合了。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-7.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-7.png)

图 10.7. 与图 10.6 的 UI 相同

### 遵守 Identifiable 协议

为了让你更了解 `List` 内 `id` 参数的用途，我们对 `restaurants` 数组做一个小修改。由于我们使用餐厅名称作为识别码，因此我们来看当有两笔数据具有相同的餐厅名称时会发生什么事？现在将 `restaurants` 数组中的“Upstate”修改为“Homei”，如下所示：

```
Restaurant(name: "Homei", image: "upstate")

```

请注意，我们只修改“name”属性值，图片仍保持为 `upstate`，如图 10.8 所示。再次检查预览画布，看看你得到了什么。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-8.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-8.png)

图 10.8. 两间餐厅的名称相同

你有看到图 10.8 中的问题吗？我们现在有两笔名称是“Homei”的纪录。你可能希望第二笔“Homei”纪录会显示 upstate 图片，但实际上，iOS 会以相同的文字与图片来渲染这两笔纪录。在代码中，我们告诉 `List`使用餐厅名称作为唯一的识别码，当两间餐厅的名称相同时，iOS 会将这两间餐厅视为同一餐厅，因此它重用相同的视图，并渲染相同的图片。

那么，你该如何修正这个问题呢？

其实非常简单，你应该给每间餐厅一个唯一的识别码，而不是使用名称作为 ID。现在修改 `Restaurant` 结构如下：

```
struct Restaurant {
    var id = UUID()
    var name: String
    var image: String
}

```

在上列的代码中，我们加入了 `id` 属性，并以唯一识别码来初始化它。这个 `UUID()`函数的作用是产生通用唯一的随机识别码，UUID 是由128 位元数所组成，因此理论上要同时产生两个相同识别码的机率几乎为零。

现在，每间餐厅应该皆有一个唯一的 ID，但是在修正这个错误之前，我们还需要再做一个修改。对于 `List`，将 `id` 参数的值从 `\.name`改为 `\.id`：

```
List(restaurants, id: \.id)

```

这告诉 `List` 视图使用餐厅的 `id` 属性作为唯一的识别码。再看一遍预览，第二笔的“Homei”纪录应该显示它自己的图片了，如图 10.9 所示。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-9.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-9.png)

图 10.9. 错误已被修正而可显示正确的图片

通过让 `Restaurant` 结构遵守 `Identifiable` 协议，我们可进一步简化代码。这个协议只有一个要求，就是实现协议的类型应该具备某种 `id` 作为唯一识别码。现在修改 `Restauran`t 来实现 `Identifiable` 协议，如下所示：

```
struct Restaurant: Identifiable {
    var id = UUID()
    var name: String
    var image: String
}

```

由于 `Restaurant` 已经提供了唯一的 `id` 属性，因此符合协议的要求。

那么，这里实现 `Identifiable` 协议的目的是什么呢？它可以进一步节省一些代码，当 `Restaurant` 结构遵守 `Identifiable` 协议时，你可不使用 `id` 参数来初始化这个 `List`，修改后的清单视图代码，如下所示：

```
List(restaurants) { restaurant in
    HStack {
        Image(restaurant.image)
            .resizable()
            .frame(width: 40, height: 40)
            .cornerRadius(5)
        Text(restaurant.name)
    }
    .listStyle(.plain)
}

```

这就是使用 `List` 显示数据集合的方式。

### 重构代码

代码运作正常，不过将代码重构，让它变得更好，始终是一件好事。你已经学过如何取出视图，现在我们将取出 `HStack` 至一个单独的结构中。按住键不放， 并点击 `HStack`， 选择“Extract subview”来取出代码， 并将结构重新命名为 `BasicImageRow`。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-10.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-10.png)

图 10.10. 取出子视图

当你修改后，Xcode 会立即显示一个错误。由于取出的子视图没有 `restaurant` 属性，因此像这样修改`BasicImageRow` 结构，以声明 `restaurant` 属性：

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
        }
    }
}

```

接着，修改 `List` 视图来传送 `restaurant` 参数：

```
List(restaurants) { restaurant in
    BasicImageRow(restaurant: restaurant)
}

```

现在一切都应该正常运作。这个清单视图渲染后看起来仍然相同，不过底层的代码更具易读性与组织性，而且更容易修改代码。例如：你将列建立为其他布局，如下所示：

```
struct FullImageRow: View {
    var restaurant: Restaurant

    var body: some View {
        ZStack {
            Image(restaurant.image)
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(height: 200)
                .cornerRadius(10)
                .overlay(
                    Rectangle()
                        .foregroundColor(.black)
                        .cornerRadius(10)
                        .opacity(0.2)
                )

            Text(restaurant.name)
                .font(.system(.title, design: .rounded))
                .fontWeight(.black)
                .foregroundColor(.white)
        }
    }
}

```

这个列布局是用于显示更大的餐厅图片，并将餐厅名称叠在上面。由于我们已经重构代码，因此非常容易修改 App，来使用新的布局，你只需要在 `List` 闭包中，将 `BasicImageRow` 改成 `FullImageRow` 即可：

```
List(restaurants) { restaurant in
    FullImageRow(restaurant: restaurant)
}

```

修改一行代码后，这个 App 会立即切换至另一个布局，如图 10.11 所示。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-11.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-11.png)

图 10.11. 修改列布局

你可以进一步混合列布局， 以建立更有趣的 UI。举例而言， 新的设计是使用 `FullImageRow` 来显示前两列的数据，其余的列则利用 `BasicImageRow`，如图 10.12 所示。你可以修改 `List` 如下：

```
List {
    ForEach(restaurants.indices, id: \.self) { index in
        if (0...1).contains(index) {
            FullImageRow(restaurant: self.restaurants[index])
        } else {
            BasicImageRow(restaurant: self.restaurants[index])
        }
    }
}
.listStyle(.plain)

```

由于我们需要检索列索引，因此我们向 `List` 传送餐厅数据的范围。在闭包中，我们检查 `index` 的值来决定要使用哪一种列布局。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-12.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-12.png)

图 10.12. 使用两种不同的列布局来建立清单视图

### 修改行分隔符的颜色

从 iOS 15 开始，Apple 为开发人员提供了自定义列表视图外观的选项。 要修改行分隔符的色调颜色，可以使用 `listRowSeparatorTint` 修饰符，如下所示：

```
List(restaurants) { restaurant in
    ForEach(restaurants.indices, id: \.self) { index in
        if (0...1).contains(index) {
            FullImageRow(restaurant: self.restaurants[index])
        } else {
            BasicImageRow(restaurant: self.restaurants[index])
        }
    }
    .listRowSeparatorTint(.green)
}
.listStyle(.plain)

```

在以上的代码中，我们将行分隔符的颜色修改为绿色。

### 隐藏列表分隔线

iOS 15 引入了其中一个最受期待的 List 功能。 你终于可以使用 `listRowSeparator` 修饰器并将其值设置为 `.hidden` 以隐藏分隔线。以下是一个示例：

```
List {
    ForEach(restaurants.indices) { index in
        if (0...1).contains(index) {
            FullImageRow(restaurant: self.restaurants[index])
        } else {
            BasicImageRow(restaurant: self.restaurants[index])
        }
    }

    .listRowSeparator(.hidden)
}
.listStyle(.plain)

```

`listRowSeparator` 修饰器应该嵌入在 `List` 视图中。 要使列表分隔线再次出现，你可以将修饰器的值设置为`.visible`。 又或者你可以简单地删除 `listRowSeparator` 修饰器。

如果你想对列表分隔线进行更精细的控制，可以使用`.listRowSeparator`修饰器内的 `edges`参数。 比如说，如果你想让分隔线保持在列表视图的顶部，你可以这样写代码：

```
.listRowSeparator(.hidden, edges: .bottom)

```

### 自定义滚动区域的背景

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-13.png](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-13.png)

图 10.13. 修改可滚动区域的颜色

在 iOS 16 中，你可以自定义列表视图的可滚动区域的颜色。 只需将 `scrollContentBackground` 修饰符附加到 `List` 视图并将其设置为您喜欢的颜色。 这是一个例子：

```
List(restaurants) { restaurant in
    .
    .
    .
}
.scrollContentBackground(Color.yellow)

```

除了使用纯色之外，你还可以使用图像作为背景。 修改这样的代码试试：

```
List(restaurants) { restaurant in
    .
    .
    .
}
.background {
    Image("homei")
        .resizable()
        .scaledToFill()
        .clipped()
}
.scrollContentBackground(Color.clear)

```

我们使用`background`修饰符来设置背景图片， 然后我们将 `scrollContentBackground` 修饰符设置为 `Color.clear` 以使可滚动区域透明。

### 作业

在进入下一章之前，自我挑战一下，建立如图 10.13 所示的清单视图，它看起来有些复杂，但是若你完全了解我在本章所教过的内容，则你应该能够建立这个 UI。请花点时间来练习这个作业，我保证你会学习到很多。

为了节省你寻找图片的时间，你可以至下列网站：[https://www.appcoda.com/resources/swiftui/SwiftUIArticleImages.zip](https://www.appcoda.com/resources/swiftui/SwiftUIArticleImages.zip) 。

![%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-13.jpg](%E5%8A%A8%E6%80%81%E5%88%97%E8%A1%A8%E3%80%81%20ForEach%20%E4%B8%8E%20Identifiable%20%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iO%20e05f1771d3b8479489a6664846d931a1/swiftui-list-13.jpg)

图 10.13. 建立一个多样化列布局的清单视图

在本章所准备的示例档中，有完整的项目与作业解答可以下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIList.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIList.zip))
- 作业解答 ([https://www.appcoda.com/resources/swiftui4/SwiftUIListExercise.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIListExercise.zip))