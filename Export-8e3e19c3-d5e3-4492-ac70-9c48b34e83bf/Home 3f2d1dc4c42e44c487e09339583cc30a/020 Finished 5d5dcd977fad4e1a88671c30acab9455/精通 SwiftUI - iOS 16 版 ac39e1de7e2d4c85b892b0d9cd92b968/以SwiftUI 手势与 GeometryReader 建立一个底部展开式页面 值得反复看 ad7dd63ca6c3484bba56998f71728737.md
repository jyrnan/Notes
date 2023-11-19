# 以SwiftUI 手势与 GeometryReader 建立一个底部展开式页面 <值得反复看>

现在你应该对 SwiftUI 手势有了基本的了解，我们来看如何应用所学的技术来建立现实世界应用程序的常用功能。

最近“底部表”（bottom sheet ）越来越受欢迎，你可以在 Facebook 与 Uber 等知名 App 中找到这个功能，它更像是动作表（action sheet ）的加强版，底部表会从屏幕底部向上滑动，并覆盖在原始视图的上面，来提供上下文信息（contextual information ）或者可供使用者选择的其他选项。举例而言，当你将照片储存在 Instagram 的照片集中，该 App 会显示一个底部表，以选择照片集。而在 Facebook App 中，当点选贴文的“⋯”（ellipsis ）按钮，它会显示带有其他的动作项目表。Uber App 也使用底部表来显示所选择的行程价格。

底部表的大小取决于想要显示的上下文信息。在某些情况下，底部表往往较大（也称为“背幕”（backdrop ）），占据了屏幕的 80~90%。通常，使用者可以使用拖曳手势与表互动。你可以向上滑动来展开它，或者向下滑动来最小化或解除表。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-1.jpg](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-1.jpg)

图 18.1. Uber、Facebook 与Instagram 皆于App 中使用底部表

在本章中，我们将使用 SwiftUI 手势建立类似的展开式底部表（expandable bottom sheet ）。示例 App 在主视图中显示餐厅清单，当使用者点击其中一笔餐厅纪录时，该 App 会弹出一个底部表来显示餐厅的详细资讯。你可以向上滑动来展开表，而要解除表则是向下滑动，如图 18.2 所示

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-2.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-2.png)

图 18.2. 建立一个展开式底部表

### 了解初始项目

为了节省你从头建立示例App 的时间，我已经为你准备好初始项目。你可以至

[https://www.appcoda.com/resources/swiftui3/SwiftUIBottomSheetStarter.zip](https://www.appcoda.com/resources/swiftui3/SwiftUIBottomSheetStarter.zip) 下载初始项目，并解压缩文件，并开启 `SwiftUIBottomSheet.xcodeproj` 来开始。

初始项目已经提供一组餐厅图片及餐厅数据。如果你在项目导航器中查看“Model”数据夹，则应该会找到一个`Restaurant.swift` 档，此文件包含了 `Restaurant` 结构与一组示例餐厅数据。

```
struct Restaurant: Identifiable {
    var id: UUID = UUID()
    var name: String
    var type: String
    var location: String
    var phone: String
    var description: String
    var image: String
    var isVisited: Bool

    init(name: String, type: String, location: String, phone: String, description: String, image: String, isVisited: Bool) {
        self.name = name
        self.type = type
        self.location = location
        self.phone = phone
        self.description = description
        self.image = image
        self.isVisited = isVisited
    }

    init() {
        self.init(name: "", type: "", location: "", phone: "", description: "", image: "", isVisited: false)
    }
}

```

此外，我已经为你建立了主视图，它显示了一个餐厅清单。你可以开启 `ContentView. swift` 档来看一下代码，我将不会详细解释代码，因为我们已经在第 10 章实现过清单。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-3.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-3.png)

图 18.3. 清单视图

### 建立餐厅细节视图

底部表实际上包含餐厅细节与一个小横列（handlebar ），因此我们要做的第一件事是建立如图 18.4 所示的餐厅细节视图。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-4.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-4.png)

图 18.4. 具有小横列的餐厅细节视图

在随着我实现视图之前，我建议你把它当作练习，并自行建立细节视图。如您所见， 细节视图是由 UI 组件所组成，包括“图片”（Image ）、“文字”（Text ）与“滚动视图”（ScrollView ）。我们已经介绍过这些组件，所以你可以尝试自行实现细节视图。

好的，让我教你如何建立细节视图。如果你已经自行建立了细节视图，你可以参考我的实现。

细节视图的布局有点复杂，因此最好将其分成几个部分，以更容易实现：

- “横列”是一个小圆角矩形。
- 标题列包含细节视图的标题。
- 头部视图包含餐厅特色图片、餐厅名称与餐厅类型。
- 细节资讯视图包含餐厅数据，其中有地址、电话与描述。

我们将使用单独的 结构（struct ）来实现上述每个部分，以让代码更易编写。现在使用“SwiftUI View”模板建立一个新文件，并命名为`RestaurantDetailView.swift`。下面讨论的所有代码都将放到这个新文件中。

### 横列

首先是“横列”（handlebar ），它实际上是一个小圆角矩形。要建立它，我们需要做的是建立一个 `Rectangle`，并使其变成圆角。 在 `RestaurantDetailView.swift` 档中插入下列的代码：

```
struct HandleBar: View {

    var body: some View {
        Rectangle()
            .frame(width: 50, height: 5)
            .foregroundColor(Color(.systemGray5))
            .cornerRadius(10)
    }
}

```

### 标题列

接着是“标题列”（title bar ），实现很简单，因为它只是一个文字视图。我们为它建立另一个结构：

```
struct TitleBar: View {

    var body: some View {
        HStack {
            Text("Restaurant Details")
                .font(.headline)
                .foregroundColor(.primary)

            Spacer()
        }
        .padding()
    }
}

```

这里的“留白”（Spacer ）是用来将文字靠左对齐。

### 头部视图

头部视图（header view ）由一个图片视图与两个文字视图所组成。这两个文字视图会叠在图片视图上面。同样的，我们将使用单独的结构来实现头部视图：

```
struct HeaderView: View {
    let restaurant: Restaurant

    var body: some View {
        Image(restaurant.image)
            .resizable()
            .scaledToFill()
            .frame(height: 300)
            .clipped()
            .overlay(
                HStack {
                    VStack(alignment: .leading) {
                        Spacer()
                        Text(restaurant.name)
                            .foregroundColor(.white)
                            .font(.system(.largeTitle, design: .rounded))
                            .bold()

                        Text(restaurant.type)
                            .font(.system(.headline, design: .rounded))
                            .padding(5)
                            .foregroundColor(.white)
                            .background(Color.red)
                            .cornerRadius(5)

                    }
                    Spacer()
                }
                .padding()
            )
    }
}

```

由于我们需要显示餐厅数据，因此 `HeaderView` 具有 `restaurant` 属性。对于这个布局，我们建立一个图片视图，并设定它的内容模式为 `scaleToFill`，图片高度固定为“300 点”。由于我们使用 `scaleToFill` 模式，我们需要加上 `.clipped()`修饰器，以隐藏超出图片框边缘的任何内容。

对于这两个标签，我们使用 `.overlay` 修饰器来叠加两个文字视图。

### 细节资讯视图

最后是资讯视图（information view ）。如果你仔细看一下地址、电话与描述栏位，你会发现它们非常相似。地址与电话栏位在文字资讯旁都有一个图示，而描述栏位则只包含文字。

因此，建立一个能灵活处理两种栏位类型的视图不是很好吗？下列是代码片段：

```
struct DetailInfoView: View {
    let icon: String?
    let info: String

    var body: some View  {
        HStack {
            if icon != nil {
                Image(systemName: icon!)
                    .padding(.trailing, 10)
            }
            Text(info)
                .font(.system(.body, design: .rounded))

            Spacer()
        }.padding(.horizontal)
    }
}

```

`DetailInfoView`有两个参数：`icon` 与 `info`。`icon` 参数是可选用的，表示它可以有值或是 nil。

当你需要显示数据栏位及图示时，你可以像这样使用 `DetailInfoView`：

```
DetailInfoView(icon: "map", info: self.restaurant.location)

```

另外，如果你只需要显示一个文字栏位（如描述栏位），则可以像这样使用 `DetailInfoView` :

```
DetailInfoView(icon: nil, info: self.restaurant.description)

```

如你所见，通过建立一个通用视图来处理相似的布局，可以使代码更具模组化及可重用性。

### 使用 VStack 组合组件

现在，我们已经建立了所有的组件，我们可以使用 `VStack` 组合它们，如下所示：

```
struct RestaurantDetailView: View {
    let restaurant: Restaurant

    var body: some View {
        VStack {
            Spacer()

            HandleBar()

            TitleBar()

            HeaderView(restaurant: self.restaurant)

            DetailInfoView(icon: "map", info: self.restaurant.location)
                .padding(.top)
            DetailInfoView(icon: "phone", info: self.restaurant.phone)
            DetailInfoView(icon: nil, info: self.restaurant.description)
                .padding(.top)
        }
        .background(Color.white)
        .cornerRadius(10, antialiased: true)
    }
}

```

上列的代码很容易理解，我们只使用前面章节中建立过的组件，并将它们嵌入到一个垂直堆叠中。原本 `VStack` 有一个透明背景，要确保细节视图具有白色背景，我们加入 `background`修饰器并进行修改。

在测试细节视图之前，必须修改 `RestaurantDetailView_Previews` 的代码如下：

```
struct RestaurantDetailView_Previews: PreviewProvider {
    static var previews: some View {
        RestaurantDetailView(restaurant: restaurants[0])
    }
}

```

在代码中，我们传送一个示例餐厅（即 `restaurants[0]` ）进行测试。如果你正确实现， 则 Xcode 应该会在预览画布中显示相似的细节视图，如图 18.5 所示。.

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-5.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-5.png)

图 18.5. 餐厅细节视图

### 使视图可滚动

你是否注意到细节视图无法显示完整的内容呢？要解决这个问题，我们必须将内容嵌入在 `ScrollView`，以让细节视图可滚动，如下所示：

```
struct RestaurantDetailView: View {
    let restaurant: Restaurant

    var body: some View {
        VStack {
            Spacer()

            HandleBar()

            ScrollView(.vertical) {
                TitleBar()

                HeaderView(restaurant: self.restaurant)

                DetailInfoView(icon: "map", info: self.restaurant.location)
                    .padding(.top)
                DetailInfoView(icon: "phone", info: self.restaurant.phone)
                DetailInfoView(icon: nil, info: self.restaurant.description)
                    .padding(.top);
            }
            .background(Color.white)
            .cornerRadius(10, antialiased: true)
        }
    }
}

```

除了横列之外，其他视图被包裹在滚动视图中。如果你再次在预览画布中运行App， 细节视图现在可滚动了。

### 调整偏移量

底部表目前叠在原本内容的上方，不过通常只覆盖部分内容，因此我们必须调整细节视图的偏移量，使它只覆盖屏幕的一部分。为此，我们可以像这样将 `offset`修饰器加到 `VStack` 上，如下所示：

```
.offset(y: 300)

```

这会将细节视图向下移动 300 点，如果你在预览画布中测试代码，则细节视图应该会移到屏幕的下半部，如图18.6 所示。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-6.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-6.png)

图 18.6. 调整细节视图的偏移量

要使它看起来更像最终结果，你可以修改 `RestaurantDetailView_Previews` 的背景颜色如下：

```
struct RestaurantDetailView_Previews: PreviewProvider {
    static var previews: some View {
        RestaurantDetailView(restaurant: restaurants[0])
            .background(Color.black.opacity(0.3))
            .edgesIgnoringSafeArea(.all)
    }
}

```

细节视图现在看起来不错，不过有个问题是偏移量设定为固定值。由于 App 将支持多个装置或屏幕大小，因此偏移量应该能够自动调整。

我想要做的是将偏移量设定为屏幕高度的一半，那么如何才能知道装置的屏幕大小呢？ 在 SwiftUI 中，它提供一个名为“GeometryReader”的容器视图，可以让你存取父视图（parent view ）的大小与位置。因此，要取得屏幕高度，你所需要做的是使用 `GeometryReader` 包裹 `VStack`，如下所示：

```
struct RestaurantDetailView: View {
    let restaurant: Restaurant

    var body: some View {
        GeometryReader { geometry in
            VStack {
                Spacer()

                HandleBar()

                ScrollView(.vertical) {
                    TitleBar()

                    HeaderView(restaurant: self.restaurant)

                    DetailInfoView(icon: "map", info: self.restaurant.location)
                        .padding(.top)
                    DetailInfoView(icon: "phone", info: self.restaurant.phone)
                    DetailInfoView(icon: nil, info: self.restaurant.description)
                        .padding(.top)
                }
                .background(Color.white)
                .cornerRadius(10, antialiased: true)
            }
            .offset(y: geometry.size.height/2)
            .edgesIgnoringSafeArea(.all)
        }
    }
}

```

在闭包中，我们可以使用 `geometry` 参数来存取父视图的大小，这就是为什么我们设定 `offset` 修饰器如下：

```
.offset(y: geometry.size.height/2)

```

要正确计算全屏幕的尺寸，我们加入 `edgesIgnoringSafeArea`修饰器，并设定其参数为 `.all`，以完全忽略安全区域。

现在，在预览画布中再次运行这个 App。你应该会有一个底部表，而它占了屏幕大小的一半，如图 18.7 所示。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-7.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-7.png)

图 18.7. 调整细节视图的偏移量

### 弹出细节视图

现在，细节视图几乎快完成了。我们回到清单视图（即 `ContentView.swift` ），以在使用者选择餐厅时，弹出细节视图。

在 `ContentView` 结构，声明两个状态变量：

```
@State private var showDetail = false
@State private var selectedRestaurant: Restaurant?

```

`showDetail`变量指示是否显示细节视图，而 `selectedRestaurant` 变量储存使用者选择的餐厅。

如前面章节所学到的，你可以加上 `onTapGesture` 修饰器来侦测点击手势。因此，当识别到点击，我们可以切换`showDetail`的值，并修改 `selectedRestaurant` 的值如下：

```
List {
    ForEach(restaurants) { restaurant in
        BasicImageRow(restaurant: restaurant)
            .onTapGesture {
                self.showDetail = true
                self.selectedRestaurant = restaurant
            }
    }
}

```

希望细节视图（即底部表）叠在清单视图的上方。为此，要使用 `ZStack` 嵌入导航视图。而在导航视图的正下方，我们将检查细节视图是否已启用，并将其初始化，如下所示：

```
var body: some View {
    ZStack {
        NavigationView {
            List {
                ForEach(restaurants) { restaurant in
                    BasicImageRow(restaurant: restaurant)
                        .onTapGesture {
                            self.showDetail = true
                            self.selectedRestaurant = restaurant
                        }
                }
            }
            .listStyle(.plain)

            .navigationBarTitle("Restaurants")
        }

        if showDetail {
            if let selectedRestaurant = selectedRestaurant {
                RestaurantDetailView(restaurant: selectedRestaurant)
                    .transition(.move(edge: .bottom))
            }
        }
    }
}

```

我们还加入 `transition` 修饰器至细节视图，如此就可使用 `move` 转场类型。`selectedRestaurant` 属性被定义为可选属性，这表示它可以有值或是 nil。在存取属性值之前，需要检查 `selectedRestaurant` 是否有值，因此，我们使用 `if let` 来进行确认，同时提醒一下，此 `if let` 运算符只支持 iOS 14 （或以上的）版本。

现在，如果在预览画布中运行这个 App，当你选择一间餐厅时，它已经可以弹出细节视图。尽管如此，底部表的实现还尚未完成。

首先，当底部表启用时，不会阻止清单视图与使用者互动，实际上清单视图应该变暗，以表示它位于下面一层。

为了实现它，我们可以建立一个空视图，并将它放在清单视图与细节视图之间。在 `ContentView.swift` 档中插入下列的代码来建立空视图：

```
struct BlankView : View {

    var bgColor: Color

    var body: some View {
        VStack {
            Spacer()
        }
        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity)
        .background(bgColor)
        .edgesIgnoringSafeArea(.all)
    }
}

```

接下来，修改 `if` 语句如下：

```
if showDetail {

    BlankView(bgColor: .black)
            .opacity(0.5)
            .onTapGesture {
                self.showDetail = false
            }

    if let selectedRestaurant = selectedRestaurant {
        RestaurantDetailView(restaurant: selectedRestaurant)
            .transition(.move(edge: .bottom))
    }
}

```

显示细节视图时，我们在其下方放置一个空视图。空视图会填满为黑色及半透明，这将会阻止使用者与清单视图进行互动，但将它们保留在原本的内容中。我们还将点击手势识别器加到空视图，以让使用者点击空白区域时解除细节视图。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-8.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-8.png)

图 18.8. 清单视图变暗

现在运行App 并尝试修改，当你点击变暗的区域时，你可以关闭细节视图。

### 加入动画

它更接近最终成果了，不过我们还有一些事情需要注意，你是否注意到细节视图的转场没有动画呢？当我们有了 `.transition`修饰器，只有当我们将它与动画配对时，转场才会动态显示。

因此，回到 `RestaurantDetailView.swift`，并将 `.animation` 修饰器加入 `VStack`，如下所示：

```
VStack {
    ...
}
.offset(y: geometry.size.height/2)
.animation(.interpolatingSpring(stiffness: 200.0, damping: 25.0, initialVelocity: 10.0))
.edgesIgnoringSafeArea(.all)

```

在代码中，我们使用 `interpolatingSpring` 动画，刚性（stiffness ）、阻尼（damping ） 与初速度（initial velocity ）的值是可以修改的。你可以使用这些值来找到适合你的App 的最佳动画。

此外，我还想在细节视图中加入一个精巧的动画，如此当我们弹出细节视图时，它会稍微向上移动。回到`ContentView.swift`，在导航视图中加入 `.offset` 与 `.animation`修饰器：

```
NavigationView {
    List {
        ...
    }

    .navigationBarTitle("Restaurants")
}
.offset(y: showDetail ? -100 : 0)
.animation(.easeOut(duration: 0.2))

```

现在，在预览画布中再次运行这个 App。当显示细节视图时，你应该会看到一个不错的动画效果。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-9.gif](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-9.gif)

图 18.9. 加入动画至底部表

### 加入手势支持

现在，我们有了一个未完成的底部表，下一步是使它能在支持手势的情况下展开。如本章开头所述，使用者可以向上滑动视图来展开它，或是向下滑动视图来最小化。

由于你已经在前一章中学过拖曳手势的工作原理，我们会应用类似的技术来建立展开式细节视图。不过，这个展开式底部表的拖曳手势的实现会比以前更加复杂。

在 `RestaurantDetailView.swift` 中，首先定义一个枚举来表示拖曳状态：

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
        case .pressing, .dragging:
            return true
        case .inactive:
            return false
        }
    }

}

```

另外，声明一个手势状态变量来追踪拖曳，以及声明一个状态变量来储存底部表的位置偏移量：

```
@GestureState private var dragState = DragState.inactive
@State private var positionOffset: CGFloat = 0.0

```

要识别拖曳，我们可以将 `.gesture` 修饰器加到 `VStack`：

```
.gesture(DragGesture()
    .updating(self.$dragState, body: { (value, state, transaction) in
        state = .dragging(translation: value.translation)
        })
)

```

在updating 函数中，我们只使用最新的拖曳数据来修改 `dragState` 属性。

最后，像这样修改 `VStack` 的 `.offset` 修饰器来移动细节视图：

```
.offset(y: geometry.size.height/2 + self.dragState.translation.height + self.positionOffset)

```

在计算偏移量时，将考虑到拖曳的位移与位置偏移量，而不是固定值，这就是我们如何启用细节视图来支持拖曳手势的方式。

如果你在预览画布中测试细节视图，则应该能够拖曳横列来滑动视图。不过，通过拖曳内容视图来向上/ 向下滑动视图几乎是不可能的。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-10.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-10.png)

图 18.10. 拖曳细节视图的内容部分不会使视图向上滑动

为什么我们不能拖曳内容视图来展开细节视图呢？

如果你还记得的话，细节视图的内容部分被嵌入到滚动视图中。因此，我们实际上有两个手势识别器：一个内建在滚动视图中，另一个是我们加入的拖曳手势。

那么， 我们该如何解决呢？ 一种方式是禁用使用者与滚动视图的互动。你可以将 `.disabled` 修饰器加入到滚动视图，并设定其值为 `true`：

```
.disabled(true)

```

当你将这个修饰器加到 `ScrollView` 后，就可以通过拖曳内容部分来向上/ 向下滑动细节视图。

不过，接下来的问题是，当禁用滚动视图后，使用者无法与滚动视图互动，这表示使用者不能查看餐厅的全部内容，因此我们还是需要使内容部分可以滚动。

用者不能查看餐厅的全部内容，因此我们还是需要使内容部分可以滚动。

显然的，我们需要找到一种方式来控制滚动视图的启用。一个简单的解决方案是，当滚动视图在半开状态禁用它。在细节视图全部开启后，我们再次启用滚动视图。

目前实现的另一个问题是，即使使用者将视图一直滑动到状态列，细节视图也无法保持完全开启。拖曳结束后，它会反弹回半开状态。反之，即使你将细节视图向下滑动到屏幕尾端，也无法关闭细节视图。

我们该如何处理这些问题呢？

### 半开状态时的处理方式

我们先处理在半开状态时会碰到的问题。细节视图弹出时会以半开作为默认状态，这个状态下会出现以下两种情况：

1. 使用者选择向上滚动视图让它全开，不过使用者可能会向上拖曳一点，接着向下拖曳来取消动作。
2. 另外一种状况是，使用者直接向下滚动以关闭视图，此外，也有可能返回拖曳来取消关闭动作。

从这些场景中可以看出，除了追踪拖曳偏移量之外，我们还需要一些界限值（threshold） 来控制视图的开启和关闭，如图 18.11 所示。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-11.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-11.png)

图 18.11. 加入界限值来控制拖曳与视图状态

上图显示了在半开状态时，我们将要定义的界限值：

- **界限值 #1** - 当拖曳超过界限值，细节视图就会完全开启。
- **界限值 #2** - 当拖曳移动低于界限值，细节视图将会关闭。

现在，你应该知道了半开状态与全开状态的工作原理，让我们进入代码部分。首先，我们在`RestaurantDetailView.swift` 声明另一个枚举来表示这两种视图状态：

```
enum ViewState {
    case full
    case half
}

```

在 `RestaurantDetailView`中，声明另一个状态属性来储存目前的视图状态。默认上，它设定为半开状态：

```
@State private var viewState = ViewState.half

```

另外，为了解除视图本身，我们需要 `ContentView` 向我们传送其状态变量的绑定，因此声明 `isShow` 变量如下：

```
@Binding var isShow: Bool

```

> 
> 
> 
> 显然的，我们需要找到一种方法来控制滚动视图的启用，一个简单的解决方案是当它处于半开状态时，禁用滚动视图。而在细节视图完全开启后，我们再次启用滚动视图
> 

接下来，我们修正滚动视图的滚动问题。将 `.disabled` 修饰器加到 `ScrollView`，如下所示：

```
.disabled(self.viewState == .half)

```

当细节视图在半开状态，我们关闭滚动视图的使用者互动行为。

现在，我们来实现刚才讨论的界限值。正如你之前所学到的，当拖曳结束时，SwiftUI 会自动调用 `onEnded` 函数。因此，我们将处理此函数中的界限值，现在修改 `.gesture` 修饰器如下：

```
.gesture(DragGesture()
    .updating(self.$dragState, body: { (value, state, transaction) in
        state = .dragging(translation: value.translation)
        })
    .onEnded({ (value) in

        if self.viewState == .half {
            // 界限值 #1
            // 向上滑动，当它超过界限值时
            // 通过修改位置偏移量来修改视图状态为全部开启
            if value.translation.height < -geometry.size.height * 0.25 {
                self.positionOffset = -geometry.size.height/2 + 50
                self.viewState = .full
            }

            // 界限值 #2
            // 向下滑动，当它通过界限值
                      // 通过设定isShow 为 false 来关闭视图
            if value.translation.height > geometry.size.height * 0.3 {
                self.isShow = false
            }
        }

    })
)

```

我使用屏幕高度来计算界限值。举例而言，界限值 #2 设定为屏幕高度的三分之一，这只是一个示例值，你可以根据你的需要修改它。

如果你需要进一步了解代码的工作原理，请参考代码注释。

由于我们在 `RestaurantDetailView` 中加入一个绑定变量，因此必须修改 `RestaurantDetailView_Previews` 的代码：

```
struct RestaurantDetailView_Previews: PreviewProvider {
    static var previews: some View {
        RestaurantDetailView(restaurant: restaurants[0], isShow: .constant(true))
            .background(Color.black.opacity(0.3))
            .edgesIgnoringSafeArea(.all)
    }
}

```

同样的，你需要回到 `ContentView.swift`，来依照下列的内容进行修改：

```
if let selectedRestaurant = selectedRestaurant {
    RestaurantDetailView(restaurant: selectedRestaurant, isShow: $showDetail)
        .transition(.move(edge: .bottom))
}

```

经过一番努力后，是时候测试展开式细节视图了。在模拟器或预览画布中运行这个 App，你将能够向下拖曳来关闭细节视图，或向上拖曳来展开它。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-12.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-12.png)

图 18.12. 向上拖曳来完全打开细节视图

### 18.9 全开状态时的处理

现在你应该能够向下拖曳细节视图来关闭它，或将视图向上拖曳来展开。目前的功能运作正常，不过主要问题在于视图完全展开时，滚动视图会挡住到我们的拖曳手势，换句话说，你可以滚动内容，但是却无法将视图退回半开状态。

在修正这个滚动问题之前，我们先修正几个细节视图的小问题。你可能会注意到即使细节视图在完全展开的状态下，却不能滚动至描述内容的最尾端处，要解决这个问题，开启 `RestaurantDetailView.swift`，然后修改 `DetailInfoView` 来呈现餐厅的描述，如下所示：

```
DetailInfoView(icon: nil, info: self.restaurant.description)
    .padding(.top)
    .padding(.bottom, 100)

```

我们只是加上一个 `.padding` 修饰器来增加空间，这可以让你滚动完整个描述内容。

回到滚动的问题。我们如何让使用者使用拖曳手势滚动内容并可以将视图退回至半开状态呢？使用者将视图向上拖曳来揭开内容，反之，使用者想关闭细节视图时会继续将视图向下拖曳。现在，你可以向下拖曳滚动视图，不过问题是当你手指放开时，视图会弹回上面。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-13.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-13.png)

图 8.13. 在两个方向拖曳滚动视图

因为滚动视图挡住了我们加上的拖曳手势，我们必须找到另外一个方式来侦测这个“向下拖曳”手势。你已经学过如何使用 `GeometryReader` 来测量视图的大小，我们可以利用它来找到如图 8.13 所示的滚动偏移量（scroll offset）

试着将下面这段程序加在 `ScrollView` 内，并置于 `TitleBar()` 上方：

```
GeometryReader { scrollViewProxy in
    Text("\(scrollViewProxy.frame(in: .named("scrollview")).minY)")
}
.frame(height: 0)

```

这不是最后的代码，我只是想要告诉你如何使用 `GeometryReader` 来读取滚动偏移量。`GeometryReader` 的闭包中，我们可以使用 `scrollViewProxy`，调用 `frame` 函数并取得 `minY`的值来计算偏移量。在调用 `frame` 函数时，必须传递你期望的坐标空间。这里我们自己定义规范滚动视图的坐标空间。

为了定义自己的坐标空间，如下，加上 `.coordinateSpace` 修饰器至 `ScrollView`：

```
.coordinateSpace(name: "scrollview")

```

你可能想知道，为何我们必须使用自己的坐标空间，而非使用 `.global` 。请再次参考图 8.13，红色线是参考点，也就是偏移值为 0 的位置， 通过坐标空间的定义，就可以达成滚动视图的规范。

现在于模拟器中运行 App，弹出细节视图，并完全展开它。当你拖曳视图时你便可以见到这个拖曳偏移量，如图 8.14 所示，当你拖曳距离顶部列越远时，偏移量会越大。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-14.jpg](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-14.jpg)

图 8.14. 在不同方向拖曳滚动视图。

### PreferenceKey 的介绍

现在我们已经找到滚动偏移量取得方式，接下的问题是如何让它的父视图知道偏移量，以利进一步处理。 SwiftUI 框架提供了一个 `PreferenceKey` 协议，可以让你很容易地从子视图传递数据至它的父视图。

为了使用布局偏好（preference）来传递滚动偏移量，我们必须建立一个结构来遵守 `PreferenceKey`协议，如下所示，在 `RestaurantDetailView.swift` 插入以下这段程序：

```
struct ScrollOffsetKey: PreferenceKey {
    typealias Value = CGFloat

    static var defaultValue = CGFloat.zero

    static func reduce(value: inout Value, nextValue: () -> Value) {
        value += nextValue()
    }
}

```

这个协议有两个必要的实现内容，首先，你必须定义默认值，以我们这个实现而言默认值为零。第二，你必须实现 `reduce` 函数来将所有偏移量值合并成一个。

接下来，你可以修改前一节所建立的 `GeometryReader`，如下所示：

```
GeometryReader { scrollViewProxy in
    Color.clear.preference(key: ScrollOffsetKey.self, value: scrollViewProxy.frame(in: .named("scrollview")).minY)
}
.frame(height: 0)

```

程序中，我们取得滚动的偏移量，并将至储存至偏好键（preference key）。我们使用 `Color.clear` 视图来隐藏视图。

接下来的问题是如何从布局偏好取得滚动视图呢？

一个简单的方式是使用 `.onPreferenceChange` 修饰器来观察值的修改。你可以加上修饰器至 `VStack`，并置于 `.gesture` 下方，如下所示：

```
.onPreferenceChange(ScrollOffsetKey.self) { value in
    print("\(value)")
}

```

这里我们只是输出偏移量的值。在模拟器运行这个 App ，当你拖曳细节视图时，你应该会在中控台见到这个偏移量。

现在你应该对于 PreferenceKey 的运作方式有基本概念，我们完成了这个实现，细节是图可以回到半开状态。

声明一个新状态变量来持续追踪滚动偏移量：

```
@State private var scrollOffset: CGFloat = 0.0

```

接着，修改 `.onPreferenceChange` 修饰器如下：

```
.onPreferenceChange(ScrollOffsetKey.self) { value in
    if self.viewState == .full {
        self.scrollOffset = value > 0 ? value : 0

        if self.scrollOffset > 120 {
            self.positionOffset = 0
            self.scrollOffset = 0
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
               self.viewState = .half
            }
        }
    }
}

```

当偏移量有变化时，我们检查他是否超过我们的界限值（ 也就是 120 ），如果超过了，我们设定视图状态回 .half 。现在加上另一个 .offset 修饰器至 VStack：

```
.offset(y: self.scrollOffset)

```

另外加上这个 `.offset` 修饰器的目的是将细节视图向下移动。如果你在模拟器运行这个 App，你应该可以将展开后的视图退回至半开状态。

![%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-15.png](%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737/swiftui-bottom-sheet-15.png)

图 8.15. 在完全展开状态下滚动视图

### 本章小结

本章的篇幅很多，但我希望你喜欢它。在本章中，我们将利用你到目前为止所学到的知识来建立展开式底部表。当建立底部表时，我尽力记录了我解决问题的思考过程，我真的希望这可以帮助你了解我的解决方案与代码。

SwiftUI 的其中一项优势是，它鼓励你建立模组化的 UI 组件。这部分我们还没有做，但是你可能会发现通过使用我在上一章中所介绍的技术，可以很容易将这个餐厅细节视图转换为支持各种类型内容的通用底部表。

因此，请尝试自行建立通用的底部表，并将其作为你的项目中的共享组件。

在本章所准备的示例档中，有完整的项目与作业解答可以下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui3/SwiftUIBottomSheet.zip](https://www.appcoda.com/resources/swiftui3/SwiftUIBottomSheet.zip))