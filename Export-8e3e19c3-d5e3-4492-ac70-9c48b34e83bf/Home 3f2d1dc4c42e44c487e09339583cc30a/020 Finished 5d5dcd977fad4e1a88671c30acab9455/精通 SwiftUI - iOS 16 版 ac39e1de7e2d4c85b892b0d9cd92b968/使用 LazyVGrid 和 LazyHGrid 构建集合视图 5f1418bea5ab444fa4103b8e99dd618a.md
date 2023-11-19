# 使用 LazyVGrid 和 LazyHGrid 构建集合视图

SwiftUI 最初的版本没有原生集合视图 (collection view)。你可以自己构建一个解决方案，或是使用[第三方程序库](https://github.com/apptekstudios/ASCollectionView)。在2020年的 WWDC 中，Apple 为 SwiftUI 框架引入了许多新功能，其中一个就是实现 Grid 视图。SwiftUI 现在为开发者提供两个新的 UI 组件： LazyVGrid 和 LazyHGrid，一个用于创建垂直 Grid，另一种用于创建水平 Grid。正如 Apple 所说，“Lazy” 一词意思是 Grid 视图在需要时才会创建项目。

在这章中，我会教你如何创建水平和垂直视图。LazyVGrid 和 LazyHGrid 都设计得十分灵活，因此开发者可以轻松创建各种类型的 Grid 布局。我们还会深入了解如何修改 Grid 项目的大小，以实现不同的布局（如图29.1）。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-1.jpg](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-1.jpg)

图 29.1. 集合视图示例

### SwiftUI Grid 布局的基础

我们可以依照以下步骤，创建水平或垂直的网格(Grid) 布局：

1. 准备要显示在网格中的原始数据。例如，以下是我们将在示例 App 中显示的一组 SF Symbol： 
    
    ```
    private var symbols = ["keyboard", "hifispeaker.fill", "printer.fill", "tv.fill", "desktopcomputer", "headphones", "tv.music.note", "mic", "plus.bubble", "video"]
    
    ```
    
2. 创建一个 `GridItem` 数组，来描述 Grid 的外观。比方说，Grid 需要有多少列？以下是描述一个 3 列 (column) 网格的示例代码： 
    
    ```
    private var threeColumnGrid = [GridItem(.flexible()), GridItem(.flexible()), GridItem(.flexible())]
    
    ```
    
3. 使用 `LazyVGrid` 和 `ScrollView` 布局 Grid。可以参考以下这是示例代码： 
    
    ```
    ScrollView {
        LazyVGrid(columns: threeColumnGrid) {
            // Display the item
        }
    }
    
    ```
    
4. 如果你想要构建水平 Grid，就如此使用 `LazyHGrid`： 
    
    ```
    ScrollView(.horizontal) {
        LazyHGrid(rows: threeColumnGrid) {
            // Display the item
        }
    }
    
    ```
    

### 使用 LazyVGrid 来创建垂直 Grid

对 Grid 布局有一些基本了解之后，让我们开始编写代码吧！我们将从构建一个 3 列的网格(Grid)开始。打开 Xcode 并使用 *App* 模板创建一个新项目。 请确保选择 *SwiftUI* 作为界面选项。 将项目命名为 *SwiftUIGridLayout* 或你喜欢的名称。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-2.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-2.png)

图 29.2. 使用 App 模板创建一个新项目

之后，在 `ContentView.swift` 内，声明以下变量 (variable)：

```
private var symbols = ["keyboard", "hifispeaker.fill", "printer.fill", "tv.fill", "desktopcomputer", "headphones", "tv.music.note", "mic", "plus.bubble", "video"]

private var colors: [Color] = [.yellow, .purple, .green]

private var gridItemLayout = [GridItem(.flexible()), GridItem(.flexible()), GridItem(.flexible())]

```

我们将要在一个 3 列网格中显示一组 SF Symbol。如此修改 `body` 变量来显示网格：

```
var body: some View {
    ScrollView {
        LazyVGrid(columns: gridItemLayout, spacing: 20) {
            ForEach((0...9999), id: \.self) {
                Image(systemName: symbols[$0 % symbols.count])
                    .font(.system(size: 30))
                    .frame(width: 50, height: 50)
                    .background(colors[$0 % colors.count])
                    .cornerRadius(10)
            }
        }
    }
}

```

我们使用 `LazyVGrid`，并告诉垂直 Grid 使用 3 列布局。我们还指定行与行之间有 20 point 的间距。在代码中，我们有一个 `ForEach` 回圈 (loop)，来显示总共 10,000 个图像视图。如果有正确地跟随步骤，你就会在预览中看到一个三列 Grid。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-3.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-3.png)

图 29.3. 一个三列 Grid

我们成功创建了一个三列的垂直 Grid 了！现在，图像框架 (frame) 的大小固定为 50 x 50 point。你可以如此修改框架修饰符 (modifier)：

```
.frame(minWidth: 0, maxWidth: .infinity, minHeight: 50)

```

然后，图像的宽度就会扩大到列的宽度（如图29.4）。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-4.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-4.png)

图 29.4. 修改网格项目的框架大小

请注意，列和行之间有一个空格。 有时，你可能想要建立一个没有任何空格的网格。 要怎么才能做到这一点？ 行之间的间距由 `LazyVGrid` 的 `spacing` 参数控制。 我们已将其值设置为20点。 你可以简单地将其修改为“0”，这样行之间就没有空格。

网格项之间的间距由在 `gridItemLayout` 中的 `GridItem` 来控制。 你可以通过将值传给 `spacing` 参数来设置项目之间的间距。 因此，要删除行之间的空白地方，你可以像这样初始化 `gridLayout` 变量：

```swift
private var gridItemLayout = [GridItem(.flexible(), spacing: 0), GridItem(.flexible(), spacing: 0), GridItem(.flexible(), spacing: 0)]

```

对于每个“GridItem”，我们指定为零间距。 为简单起见，上面的代码可以改写成这样：

```swift
private var gridItemLayout = Array(repeating: GridItem(.flexible(), spacing: 0), count: 3)

```

如果你已修改相关代码，你的预览画布应该会显示一个没有任何间距的网格视图。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-5.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-5.png)

Figure 5. Removing the spacing between columns and rows

## 利用 GridItem 修改 Grid 布局 (Flexible/ Fixed/ Adaptive)

让我们进一步看看 `GridItem`。你可以使用 `GridItem` 实例在 `LazyHGrid` 和 `LazyVGrid` 视图中配置项目的布局。在前文中，我们定义了一个有三个 `GridItem` 实例的数组，而每个实例都使用尺寸类型 (Size Type) `.flexible()`。Flexible 尺寸类型让我们可以创建三个大小相等的列。如果想要一个 6 列的 Grid，则可以如此创建 `GridItem` 数组：

```
private var sixColumnGrid: [GridItem] = Array(repeating: .init(.flexible()), count: 6)

```

`.flexible()` 只是其中一个用于控制 Grid 布局的尺寸类型。

如果要在一行中放置尽可能多的项目，你可以使用如下的 `*adaptive*` 尺寸类型：

```
private var gridItemLayout = [GridItem(.adaptive(minimum: 50))]

```

*Adaptive* 尺寸类型要求你指定项目的最小尺寸 (minimum size)。在上面的代码中，我们设定了每个 Grid 项目的最小尺寸为 50。如果你像这样修改 `gridItemLayout` 变量，就可以达到以下的 Grid 布局。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-6.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-6.png)

图 29.6. 使用 adaptive 尺寸类型

我们使用了 `.adaptive(minimum: 50)`，指示 `LazyVGrid` 在一行中填满尽可能多的图像，而每个项目的最小尺寸为 50 point。

*注意：我使用 iPhone 13 Pro 作为模拟器。 如果你使用其他屏幕尺寸不同的 iOS 模拟器，你可能会获得不同的结果。*

除了 `.flexible` 和 `.adaptive` 之外，如果想创建固定宽度的列，我们可以使用 `.fixed`。例如，我们想将图像分为两列，第一列的宽度为 100 point，第二列的宽度为 150 point，我们可以这样编写代码：

```
private var gridItemLayout = [GridItem(.fixed(100)), GridItem(.fixed(150))]

```

同样地，如果你像这样修改 `gridItemLayout` 变量，就会出现尺寸不同的两列 Grid。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-7.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-7.png)

图 29.7. 具有固定大小项目的网格

你可以混合使用不同的尺寸类型，来创建更复杂的 Grid 布局。例如，你可以定义一个 `.fixed` 的`GridItem`，然后定义一个 `.adaptive` 的GridItem，像这样：

```
private var gridItemLayout = [GridItem(.fixed(150)), GridItem(.adaptive(minimum: 50))]

```

在这个情况下，`LazyVGrid` 将创建一个固定尺寸的列，其宽度为 100 point。然后，它会让尽可能多的项目填满剩余空间。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-8.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-8.png)

图 29.8. 混合使用不同的尺寸类型

### 使用 LazyHGrid 来创建水平 Grid

现在，你已经创建了垂直 Grid，`LazyHGrid` 可以很容易地将垂直 Grid 转换为水平 Grid。水平 Grid 的用法与 `LazyVGrid` 几乎完全相同，只是我们会将其嵌入到水平滚动视图中。此外，`LazyHGrid`的参数是 `rows` 而不是 `columns`。

因此，我们只需要重写几行代码，就可以将 Grid 视图从垂直方向转换为水平方向：

```
ScrollView(.horizontal) {
    LazyHGrid(rows: gridItemLayout, spacing: 20) {
        ForEach((0...9999), id: \.self) {
            Image(systemName: symbols[$0 % symbols.count])
                .font(.system(size: 30))
                .frame(minWidth: 0, maxWidth: .infinity, minHeight: 50, maxHeight: .infinity)
                .background(colors[$0 % colors.count])
                .cornerRadius(10)
        }
    }
}

```

在预览中运行示例或在模拟器上进行测试，你应该会看到一个水平 Grid。最棒的是，此 Grid 视图可以自动支持 iPhone 和 iPad。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-9.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-9.png)

Figure 9. Creating a horizontal grid with LazyHGrid

### 在不同的网格布局之间切换

现在你已经对 LazyVGrid 和 LazyHGrid 有所认识，让我们创建一些更复杂的东西。 想像一下，你要构建一个显示咖啡照片集合的App。 在App中，使用者可以随时改变 Grid 的列数。在默认的情况下，它会在一列中显示所有照片。 使用者可以点击 *Grid* 按钮将列表视图修改为具有 2 列的网格视图。 再次点击相同的按钮以进行 3 列布局，然后是 4 列布局。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-10.jpg](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-10.jpg)

图 29.10. 改变 Grid 的列数

要开发这个示例App，请先创建一个新的 Xcode 项目。 今次也是选择 *App* 模板并将项目命名为 *SwiftUIPhotoGrid*。 接下来，在 [https://www.appcoda.com/resources/swiftui/coffeeimages.zip](https://www.appcoda.com/resources/swiftui/coffeeimages.zip) 下载图像包。 解压缩图像并将它们添加到`Assets`。

在建立网格视图之前，我们将为照片集合创建数据模型。 在项目导航器中，右键单击 *SwiftUIGridView* 并选择 *New file...* 以加入新文件。 选择 *Swift File* 模板并将文件命名为 *Photo.swift*。

在 `Photo.swift` 文件中插入以下代码以建立 `Photo` 结构：

```
struct Photo: Identifiable {
    var id = UUID()
    var name: String
}

let samplePhotos = (1...20).map { Photo(name: "coffee-\($0)") }

```

图像包中有 20 张咖啡照片，因此我们初始化了一个包含 20 个`Photo`实体的数组。 准备好数据模型后，让我们切换到 `ContentView.swift` 来构建网格。

首先，声明一个 `gridLayout` 变量来定义我们首选的网格布局：

```
@State var gridLayout: [GridItem] = [ GridItem() ]

```

在默认情况下，我们希望显示一个列表视图。 除了使用 `List`，你实际上还可以使用 `LazyVGrid` 来构建列表视图。 我们通过使用一个网格项定义 `gridLayout` 来做到这一点。 通过告诉 `LazyVGrid` 使用单列网格布局，它将像列表视图一样排列项目。 在 `body` 中插入以下代码以创建网格视图：

```
NavigationStack {
    ScrollView {
        LazyVGrid(columns: gridLayout, alignment: .center, spacing: 10) {

            ForEach(samplePhotos.indices, id: \.self) { index in

                Image(samplePhotos[index].name)
                    .resizable()
                    .scaledToFill()
                    .frame(minWidth: 0, maxWidth: .infinity)
                    .frame(height: 200)
                    .cornerRadius(10)
                    .shadow(color: Color.primary.opacity(0.3), radius: 1)

            }
        }
        .padding(.all, 10)
    }

    .navigationTitle("Coffee Feed")
}

```

我们使用 `LazyVGrid` 创建一个垂直网格，行间距为 10 点。 Grid用于显示咖啡照片，因此我们使用 `ForEach` 取得所有 `samplePhotos` 项目。 我们将网格嵌入到滚动视图中，使其可滚动并用导航视图包装。 修改代码后，你应该会在预览画布中看到照片列表。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-11.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-11.png)

图 29.11. 使用 LazyVGrid 创建列表视图

现在我们需要一个按钮让用户在不同的布局之间切换。 我们将按钮添加到导航栏。自 iOS 14 ，Apple 加入了一个名为`.toolbar`的新修饰器，用于填充导航栏中的项目。 在 `.navigationTitle` 之后，插入以下代码：

```
.toolbar {
    ToolbarItem(placement: .navigationBarTrailing) {
        Button {
            self.gridLayout = Array(repeating: .init(.flexible()), count: self.gridLayout.count % 4 + 1)
        } label: {
            Image(systemName: "square.grid.2x2")
                .font(.title)
                .foregroundColor(.primary)
        }
    }
}

```

在上面的代码中，我们修改了`gridLayout`变量并初始化了`GridItem`的数组。 假设当前项目数为 1，我们将创建一个包含两个 `GridItem` 的数组以修改为 2 列网格。 由于我们将 `gridLayout` 标记为状态变量，因此每次修改变量时，SwiftUI 都会修改网格视图。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-12.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-12.png)

图 29.12. 添加用于切换网格布局的按钮

你可以运行该App进行快速测试。 点击网格按钮将切换到另一个网格布局。

我们有几件事需要改进。 首先，对于具有两列或更多列的网格，应将网格项的高度调整为 100 点。 使用 `height` 参数修改 `.frame` 修饰器，如下所示：

```
.frame(height: gridLayout.count == 1 ? 200 : 100)

```

其次，当你从一种网格布局切换到另一种时，SwiftUI 只会简单地重绘网格视图。 如果我们在布局修改之间添加一个漂亮的过场动画是不是会更好呢？ 为此，你只需添加一行代码。 在 `.padding(.all, 10)` 之后插入以下代码：

```
.animation(.interactiveSpring(), value: gridLayout.count)

```

这就是 SwiftUI 的强大之处。 通过告诉 SwiftUI 你想要动画变化，框架会处理剩下的事情，而使用者就会看到漂亮的过场动画。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-13.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-13.png)

图 29.13. SwiftUI 自动设置过场动画

### 使用多个网格构建网格布局

你不仅限于在你的App中使用单个 `LazyVGrid` 或 `LazyHGrid`。 通过组合多个`LazyVGrid`，你可以构建一些有趣的布局。 看一下图 29.14。我们将创建这种网格布局， 网格显示咖啡馆照片列表。 在每张咖啡馆照片下，它显示了一份咖啡照片列表。 当设备处于横向时，咖啡厅照片和咖啡照片列表将并排排列。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-14.jpg](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-14.jpg)

图 29.14. 使用两个网格构建复杂的网格布局

让我们回到 Xcode 项目，首先创建数据模型。 你之前下载的图像包带有一组咖啡馆照片。 因此，创建一个新的 Swift 文件并将其命名为 *Cafe.swift*。 在文件中，加入以下代码：

```
struct Cafe: Identifiable {
    var id = UUID()
    var image: String
    var coffeePhotos: [Photo] = []
}

let sampleCafes: [Cafe] = {

    var cafes = (1...18).map { Cafe(image: "cafe-\($0)") }

    for index in cafes.indices {
        let randomNumber = Int.random(in: (2...12))
        cafes[index].coffeePhotos = (1...randomNumber).map { Photo(name: "coffee-\($0)") }
    }

    return cafes
}()

```

`Cafe` 结构是不言自明的。 它有一个用于存储咖啡照片的`image` 属性和用于存储咖啡照片列表的`coffeePhotos` 属性。 在上面的代码中，我们还创建了一个`Cafe` 数组。 对于每个咖啡馆，我们随机挑选一些咖啡照片。 如果你有其他喜欢的图像，请随时修改代码。

让我们创建一个新文件来实现这个网格视图。 右键单击 *SwiftUIPhotoGrid* 并选择 *New File...*。 选择 *SwiftUI View* 模板并将文件命名为 *MultiGridView*。

与前面的实现类似，让我们声明了一个 `gridLayout` 变量来储存当前的网格布局：

```
@State var gridLayout = [ GridItem() ]

```

在默认的情况下，我们的网格使用一个 `GridItem` 进行初始化。 接下来，在 `body` 中加入以下代码以创建具有单列的垂直网格：

```
NavigationView {
    ScrollView {
        LazyVGrid(columns: gridLayout, alignment: .center, spacing: 10) {

            ForEach(sampleCafes) { cafe in
                Image(cafe.image)
                    .resizable()
                    .scaledToFill()
                    .frame(minWidth: 0, maxWidth: .infinity)
                    .frame(maxHeight: 150)
                    .cornerRadius(10)
                    .shadow(color: Color.primary.opacity(0.3), radius: 1)
            }

        }
        .padding(.all, 10)
        .animation(.interactiveSpring(), value: gridLayout.count)
    }
    .navigationTitle("Coffee Feed")
}

```

相信我不需要再次重复讲解这个代码，因为它与我们之前编写的代码几乎相同。 如果跟着做，你应该会看到一个列表视图，其中显示了咖啡馆照片的集合。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-15.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-15.png)

图 29.15. 咖啡馆照片列表

### 添加额外的网格（Grid）

要如何在每张照片下显示另一个网格？ 你需要做的就是在 `ForEach` 循环中添加另一个 `LazyVGrid`。 在 `Image` 视图之后加入以下代码：

```
LazyVGrid(columns: [GridItem(.adaptive(minimum: 50))]) {
    ForEach(cafe.coffeePhotos) { photo in
        Image(photo.name)
            .resizable()
            .scaledToFill()
            .frame(minWidth: 0, maxWidth: .infinity)
            .frame(height: 50)
            .cornerRadius(10)
    }
}
.frame(minHeight: 0, maxHeight: .infinity, alignment: .top)
.animation(.easeIn, value: gridLayout.count)

```

我们为照片创建另一个垂直网格（Grid）。 通过使用 *adaptive* 尺寸类型，此网格将在一行中尽量填满最多的照片。 修改代码后，App UI 就会如图 29.16 所示。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-16.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-16.png)

图 29.16. 添加额外的网格显示图片

如果你喜欢并排排列咖啡馆和咖啡的照片，你可以像这样修改 `gridLayout` 变量：

```
@State var gridLayout = [ GridItem(.adaptive(minimum: 100)), GridItem(.flexible()) ]

```

当你修改了 `gridLayout` 变量，你的预览将随即修改为并以另一种形式显示咖啡馆和咖啡的照片。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-17.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-17.png)

图 29.17. 并排排列咖啡馆和咖啡的照片

### 处理横向方向

要在 Xcode 预览中测试横向模式，你可以选择 *Device Settings* 选项并启用 *Orientation* 选项。 将其设置为 *Landscape (Left)* 或 *Landscape (Right)*。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-18.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-18.png)

图 29.18. 横向模式下的 App UI

另外，你也可以在模拟器上运行App，然后将模拟器转为横向试一试。

但在运行App之前，你需要在 `SwiftUIGridLayoutApp.swift` 中做一个小的改动。 由于我们已经创建了一个新文件来实现这个多网格，所以将 `WindowGroup` 中的 `ContentView()`视图 修改为 `MultiGridView()`，如下所示：

```
struct SwiftUIPhotoGridApp: App {
    var body: some Scene {
        WindowGroup {
            MultiGridView()
        }
    }
}

```

现在你可以在 iPhone 模拟器中运行该App。 在直向上，App的UI没有任何问题，就像预览时一样。 但是，如果你通过 command + 左（或右）键来旋转模拟器，网格布局看起来就不像预期的那样。 我们期望的是它看起来应该与直向模式时差不多。

为了解决这个问题，我们可以调整自适应网格项的最小宽度，并在iPhone处于横向时使其更宽一些。 问题是如何检测iPhone的旋转方向变化？

在 SwiftUI 中，每个视图都带有一组环境变量。 你可以通过这组环境变量取得iPhone的营幕方向，如下所示：

```
@Environment(\.horizontalSizeClass) var horizontalSizeClass: UserInterfaceSizeClass?
@Environment(\.verticalSizeClass) var verticalSizeClass: UserInterfaceSizeClass?

```

`@Environment` 属性包装器允许你读取环境值。 在上面的代码中，我们告诉 SwiftUI 要同时读取水平和垂直尺寸类别，并侦测它们的变化。 换句话说，只要iPhone的营幕方向改变，App就会收到通知。

如果你还没有这样做，请确保将上面的代码插入到 `SwiftUIPhotoGrid` 中。

下一个问题是我们如何取得通知并作出相对应的行动？ SwiftUI 提供了一个名为 `.onChange()` 的修饰器。 你可以将此修饰器附加到任何视图以侦视任何状态修改。 在这种情况下，我们可以像以代码将修饰器附加到 `NavigationView`：

```
.onChange(of: verticalSizeClass) { value in
    self.gridLayout = [ GridItem(.adaptive(minimum:  verticalSizeClass == .compact  ? 100 : 250)), GridItem(.flexible()) ]
}

```

我们侦测 `horizontalSizeClass` 和 `verticalSizeClass` 变量的任何变化。 每当有改变时，我们都会用新的网格配置修改 `gridLayout` 变量。 iPhone 在横向上具有 *compact* 高度。 因此，如果 `verticalSizeClass` 的值等于 `.compact`，我们将网格项的最小大小修改为 250 点。

现在再次在 iPhone 模拟器上运行App。 当你将iPhone向横转动时，它就会并排显示咖啡馆照片和咖啡照片。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-19.png](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-19.png)

图 29.19. 横向形式的 App UI

### 练习

我有几个简单练习给你。 首先，App UI 在 iPad 上看起来不是太好。 请修改一下代码并解决下图的问题，使其仅显示两列：一列用于咖啡馆照片，另一列用于咖啡照片。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-21.jpg](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-21.jpg)

图 29.21. 在iPad上的UI

下一个练习会稍为复杂一点，以下是你要做的事项：

1. **iPhone 和 iPad 的不同默认网格布局** - 当首次运行App时，它会在iPhone直向模式下显示单个列网格。 但当 iPad 和 iPhone 在横向模式，App就会以 2 列网格显示咖啡馆照片。
2. **显示/隐藏咖啡照片** - 在导航栏中添加一个新按钮，用于显示/隐藏咖啡照片。 在默认的情况下，App仅显示咖啡馆照片列表。 点击此按钮时，就会显示咖啡照片网格。
3. **建立另一个用于切换网格布局的按钮** - 添加另一个条形按钮，使用者可以用于切换一列和两列的网格布局。

![%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-22.jpg](%E4%BD%BF%E7%94%A8%20LazyVGrid%20%E5%92%8C%20LazyHGrid%20%E6%9E%84%E5%BB%BA%E9%9B%86%E5%90%88%E8%A7%86%E5%9B%BE%205f1418bea5ab444fa4103b8e99dd618a/swiftui-gridlayout-22.jpg)

图 29.22. 加强 App 在 iPhone 和 iPad 上的功能

为了帮助你更理解这个练习，请去 [https://link.appcoda.com/multigrid-demo](https://link.appcoda.com/multigrid-demo) 观看此动画示范。

### 总结

第一个版本的 SwiftUI 框架缺少了的集合视图，新版本终于解决了这个问题。 苹果公司加入了 `LazyVGrid` 和 `LazyHGrid` 让开发者可以用几行代码就能建立不同类型的网格布局。 本章只是简略示范了这两个新 UI 组件的运作， 我鼓励你尝试不同的 `GridItem` 配置，看看可以实现哪些网格布局。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- [https://www.appcoda.com/resources/swiftui4/SwiftUIGridLayout.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIGridLayout.zip)
- [https://www.appcoda.com/resources/swiftui4/SwiftUIPhotoGrid.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIPhotoGrid.zip)