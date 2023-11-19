# 使用 Combine 与 Environment 物件进行数据分享 | 精通 SwiftUI - iOS 16 版

在第 13 章中，你学到了使用 Form 组件来布局表单。不过，目前表单还没有功能，不论你选择哪个选项，清单视图都不会反映使用者偏好而有任何改变，这也是我们将在本章中讨论与实现的内容。我们将继续开发设定画面，并依照使用者的个人偏好修改餐厅清单， 使 App 的功能完善。

具体而言，我们将在后面的小节讨论下列主题：

1. 如何使用枚举（enum）来组织代码。
2. 如何使用 UserDefaults 来永久储存使用者偏好。
3. 如何使用 Combine 与 @EnvironmentObject 来共享数据。

如果你还没有完成第 13 章作业，我鼓励你花点时间练习。不过，如果你等不及要阅读本章内容，你可以至下列网址下载示例项目：[https://www.appcoda.com/resources/swiftui4/SwiftUIForm.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIForm.zip) 。

### 使用枚举重构代码

我们目前使用一个数组来储存“Display Order”的三个选项，它虽然能够正常运作，不过还有一个更好的方式可以改善代码。

> 枚举为一组相关的值定义一般类型，并使你在代码中以类型安全的方式使用这些值。
> 
> - Apple 的官方文件 ([https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html))

由于这组固定值是和“Display Order”有关，因此我们可以使用枚举（ `Enum` ）来存放它们，每个情况（case ）指定一个整数值，如下所示：

```
enum DisplayOrderType: Int, CaseIterable {
    case alphabetical = 0
    case favoriteFirst = 1
    case checkInFirst = 2

    init(type: Int) {
        switch type {
        case 0: self = .alphabetical
        case 1: self = .favoriteFirst
        case 2: self = .checkInFirst
        default: self = .alphabetical
        }
    }

    var text: String {
        switch self {
        case .alphabetical: return "Alphabetical"
        case .favoriteFirst: return "Show Favorite First"
        case .checkInFirst: return "Show Check-in First"
        }
    }
}

```

使用枚举的优点是我们可在代码中以类型安全的方式使用这些值。最重要的是， Swift 中的 `Enum` 本身就是一级类型，这表示你可以建立实例方法来提供与值相关的附加功能。稍后，我们将会加入一个处理筛选的功能。同时，我们建立一个名为 `SettingStore. swift` 的新 Swift 档来储存 Enum，如图 14.1 所示，你可以在项目导航处的 `SwiftUIForm` 数据夹点击右键，并选取 “ New File... ”来建立这个文件。

![%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-1.png](%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-1.png)

图 14.1 建立一个新的 Swift 档

建立 `SettingStore.swift` 之后，将上列的代码片段插入文件中。接下来，回到 `Setting View.swift`，我们将修改代码来使用 `DisplayOrder` 枚举，而不是使用 `displayOrders` 数组。

首先，从 `SettingView` 删除下列这行代码：

```
private var displayOrders = [ "Alphabetical", "Show Favorite First", "Show Check-in First"]

```

接下来，修改 `selectedOrder` 的默认值为 `DisplayOrderType.alphabetical`，如下所示：

```
@State private var selectedOrder = DisplayOrderType.alphabetical

```

这里，我们默认显示顺序为依字母排列（alphabetical ），如果你与先前的值（即“0”） 进行比较，转换为使用枚举后，代码更易于阅读了。接下来，你还需要在“Sort Preference”区块修改代码，具体而言，我们修改`ForEach` 回圈中的代码：

```
Section(header: Text("SORT PREFERENCE")) {
    Picker(selection: $selectedOrder, label: Text("Display order")) {
        ForEach(DisplayOrderType.allCases, id: \.self) {
            orderType in
            Text(orderType.text)
        }
    }
}

```

由于我们在 `DisplayOrder` 枚举中采用 `CaseIterable` 协议，因此我们可存取 `allCases` 属性（该属性包含所有枚举情况的数组）来找出所有的显示顺序。

现在，你可以再次测试设定画面，它应该可正常运作且外观相同，不过底层代码更易于管理及阅读了。

### 在 UserDefaults 储存使用者偏好

目前，App 还不能永久储存使用者偏好。每当你重新启动这个 App 时，设定画面都会重置为默认设定。

有多种储存设定的方式。要储存少量数据（如 iOS 的使用者设定），内建的默认数据库是方案之一。此默认系统让App 以键值对的形式来储存使用者偏好。要和这个默认数据库互动，你可以使用一个名为 `UserDefaults`的可程序化介面（programmatic interface ）。

在 `SettingStore.swift` 档中，我们将会建立一个 `SettingStore` 类别，以提供一些方便的方法来储存及载入使用者偏好，在 `SettingStore.swift` 中插入下列的代码片段：

```
final class SettingStore {

    init() {
        UserDefaults.standard.register(defaults: [
            "view.preferences.showCheckInOnly" : false,
            "view.preferences.displayOrder" : 0,
            "view.preferences.maxPriceLevel" : 5
        ])
    }

    var showCheckInOnly: Bool = UserDefaults.standard.bool(forKey: "view.preferences.showCheckInOnly") {
        didSet {
            UserDefaults.standard.set(showCheckInOnly, forKey: "view.preferences.showCheckInOnly")
        }
    }

    var displayOrder: DisplayOrderType = DisplayOrderType(type: UserDefaults.standard.integer(forKey: "view.preferences.displayOrder")) {
        didSet {
            UserDefaults.standard.set(displayOrder.rawValue, forKey: "view.preferences.displayOrder")
        }
    }

    var maxPriceLevel: Int = UserDefaults.standard.integer(forKey: "view.preferences.maxPriceLevel") {
        didSet {
            UserDefaults.standard.set(maxPriceLevel, forKey: "view.preferences.maxPriceLevel")
        }
    }

}

```

我来简短解释一下代码，在 `init`方法中，我们使用一些默认值来初始化默认系统。如果数据库中找不到使用者偏好，才会使用这些值。

如前所述，你可以使用 `UserDefaults`，以键值对的形式储存设定。在上列的代码中， 我们为此目的声明了三个属性，以特定的键，从默认系统中载入对应的值，在didSet中，我们使用 `UserDefaults` 的 `set` 方法，来将值储存在使用者默认，这三个属性前面都标注 `@Published`，因此当值修改后，它会通知订阅者。

`settingStore` 准备好后，我们切换到 `SettingView.swift` 档来实现“Save”操作。首先， 在 `SettingView` 中为 `SettingStore` 声明一个属性。

```
var settingStore: SettingStore

```

而“储存”（Save ）按钮的程序修改如下：

至于 *Save* 按钮，你可以将 *Save* 按钮的代码（在 `ToolbarItem(placement: .navigationBarTrailing)` 块中）并将之改为：

```
Button {
    self.settingStore.showCheckInOnly = self.showCheckInOnly
    self.settingStore.displayOrder = self.selectedOrder
    self.settingStore.maxPriceLevel = self.maxPriceLevel
    dismiss()

} label: {
    Text("Save")
        .foregroundColor(.primary)
}

```

我们插入三行代码来储存使用者偏好。要在带入设定视图时载入偏好，你可以加入一个 `onAppear`修饰器至`NavigationView`，如下列所示：

```
.onAppear {
    self.selectedOrder = self.settingStore.displayOrder
    self.showCheckInOnly = self.settingStore.showCheckInOnly
    self.maxPriceLevel = self.settingStore.maxPriceLevel
}

```

当视图出现时，`onAppear` 修饰器会被调用，因此我们在它的闭包中从默认系统载入使用者设定。

在测试其变化之前，你必须修改 `SettingView_Previews`，如下所示：

```
struct SettingView_Previews: PreviewProvider {
    static var previews: some View {
        SettingView(settingStore: SettingStore())
    }
}

```

现在，切换至 `ContentView.swift`，并声明 `settingStore` 属性：

```
var settingStore: SettingStore

```

并修改 `sheet` 修饰器如下：

```
.sheet(isPresented: $showSettings) {
    SettingView(settingStore: self.settingStore)
}

```

最后，修改 `ContentView_Previews` 如下：

```
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView(settingStore: SettingStore())
    }
}

```

我们只是初始化一个 `SettingStore`，并将其传送给 `SettingView`。这是必需的，因为我们已经在 `SettingView` 中加入 `settingStore` 属性。

如果你现在编译并运行该 App，Xcode 将显示一个错误。在 App 可正常运作之前，我们还需要做一个修改。

![%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-2.png](%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-2.png)

图 14.2. SwiftUIFormApp.swift 中出现错误

至 `SwiftUIFormApp.swift` 并加入下列属性来建立一个 `SettingStore` 实例：

```
var settingStore = SettingStore()

```

接着，将这行代码修改为下列代码，来修正错误：

```
ContentView(settingStore: settingStore)

```

现在，你应该能够运行 App，并进行设定了。当你储存设定之后，它将永久储存在本地默认系统中。你可尝试停止 App，然后再次启动它，储存的设定应已载入至设定画面中，如图 14.3 所示。

![%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-3.png](%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-3.png)

图 14.3. 设定画面应该已载入你的使用者偏好

### 使用 @EnvironmentObject 在视图间共享数据

现在，使用者偏好已经储存在本地默认系统中，但是清单视图并没有依照使用者设定来修改。同样的，有多种方式可以解决这个问题。

我们概括说明一下目前的情形，当使用者在设定画面中点击“Save”按钮，我们储存所选的选项至本地默认系统中，然后关闭设定画面，App 将带使用者回到清单视图。因此， 我们指示清单视图来重新载入设定，或者清单视图能够监控默认系统的修改，并触发清单的修改。

随着 SwiftUI 的推出，Apple 还发布了一个名为“Combine”的新框架，根据Apple 的说法，这个框架提供一个声明式 API 来随着时间推移处理值。在此示例的内容中，Combine 让你轻松监控单一物件，并取得修改通知。与SwiftUI 一起使用时，我们甚至可不撰写一行代码，就触发视图修改，一切都由 SwiftUI 与 Combine 在幕后处理。

那么，清单视图如何知道使用者偏好已被修改，并触发修改呢？

我来介绍三个关键字：

1. **@EnvironmentObject** - 以技术而言，这就是一个属性包裹器（ property wrapper ），不过，你可将此关键字视为一个特殊的标记，当你声明一个属性为环境物件时，SwiftUI 会监控该属性的值，并在有任何改变时，使对应的视图无效。@EnvironmentObject 的运作方式与 @State 几乎相同，不过属性被声明为环境物件时， 整个 App 中的所有视图皆可存取它。举例而言，如果你的App 有很多共享相同数据（例如：使用者设定）的视图，则环境物件在这种情况下可运作得很好，你不需要在视图间传送属性，就可以自动存取它。
2. **ObservableObject** - 这是一个 Combine框架的协议。当你声明一个属性为环境物件时， 该属性的类型必须实现此协议。回到我们的问题：我们如何让清单视图知道使用者偏好已经修改？通过实现此协议，这个物件可以作为一个发布者，发出修改后的值。而那些监控值的订阅者将会收到通知。
3. **@Published** - 这是一个与 `ObservableObject` 一起使用的属性包裹器。当一个属性以 `@Publisher` 为前缀时，这表示发布者应该在值发生修改时通知所有订阅者。

我知道这有点令人困惑。不过，当我们看完代码后，你将会更加了解。

我们从 `SettingStore.swift` 开始。设定视图与清单视图需要监控使用者偏好的变化，因此 `SettingStore` 应该实现 `ObservableObject` 协议，并宣布 `defaults` 属性的修改。在 `Setting Store.swift` 档的开始处，我们必须先汇入 Combine 框架：

```
import Combine

```

`SettingStore` 类别应该采用 `ObservableObject` 协议。修改类别声明，如下所示：

```
final class SettingStore: ObservableObject {

```

接下来，如下，在所有的属性的前面插入 `@Published` 标注：

```
@Published var showCheckInOnly: Bool = UserDefaults.standard.bool(forKey: "view.preferences.showCheckInOnly") {
    didSet {
        UserDefaults.standard.set(showCheckInOnly, forKey: "view.preferences.showCheckInOnly")
    }
}

@Published var displayOrder: DisplayOrderType = DisplayOrderType(type: UserDefaults.standard.integer(forKey: "view.preferences.displayOrder")) {
    didSet {
        UserDefaults.standard.set(displayOrder.rawValue, forKey: "view.preferences.displayOrder")
    }
}

@Published var maxPriceLevel: Int = UserDefaults.standard.integer(forKey: "view.preferences.maxPriceLevel") {
    didSet {
        UserDefaults.standard.set(maxPriceLevel, forKey: "view.preferences.maxPriceLevel")
    }
}

```

藉由使用 `@Published` 属性包裹器，发布者将在属性的值发生变化时通知订阅者（例如：`displayOrder` 的修改）。

如你所见，使用 Combine 通知修改的值非常容易。实际上，我们还没有编写任何新代码，只有采用所需的协议，并插入一个标记。

现在，我们切换至 `SettingView.swift`。`settingStore` 现在应该声明为环境物件，以让我们可以与其他视图共享数据。修改 `settingStore` 变量，如下所示：

```
@EnvironmentObject var settingStore: SettingStore

```

你不需要修改和“Save”按钮有关的代码。不过，当你设定一个新值至设定储存区时（例如：修改`showCheckInOnly`，从 true 改为 false ），此修改将会发布，并让所有订阅者知道。

由于此修改，我们需要修改 `SettingView_Previews` 为下列内容：

```
struct SettingView_Previews: PreviewProvider {
    static var previews: some View {
        SettingView().environmentObject(SettingStore())
    }
}

```

这里，我们将 `SettingStore` 的实例注入至环境中，以进行预览。

好的，发布方已经完成，那么订阅者呢？我们要如何监控 `defaults` 的变化，并相应修改UI 呢？

在这个示例项目中，清单视图是订阅方，它需要监控设定储存区的变化，并重新渲染清单视图，以反映使用者的设定。现在开启 `ContentView.swift` 来做一些修改。和我们刚才所做的操作类似，`settingStore` 现在应该声明为一个环境物件：

```
@EnvironmentObject var settingStore: SettingStore

```

由于这个修改，因此应要修改 `sheet` 修饰器中的代码，以获取此环境物件：

```
.sheet(isPresented: $showSettings) {
    SettingView().environmentObject(self.settingStore)
}

```

另外，为了测试的目的，预览代码应要相应修改，以注入环境物件：

```
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView().environmentObject(SettingStore())
    }
}

```

最后，开启`SwiftUIFormApp.swift` ，并修改WindowGroup 内的代码，如下所示：

```
struct SwiftUIFormApp: App {

    var settingStore = SettingStore()

    var body: some Scene {
        WindowGroup {
            ContentView().environmentObject(settingStore)
        }
    }
}

```

这里，我们调用 `environmentObject` 方法，将设定储存区注入至环境。现在，设定储存区的实例可用于 App 内的所有视图。换句话说，设定与清单视图皆可自动存取它了。

### 实现筛选选项

现在，我们已经实现了一个可以让所有视图存取的通用设定储存区。最棒的是，只要设定储存区中有任何修改，它会自动通知监控修改的视图。尽管你看不出任何的视觉差异，但是当你修改设定画面的选项时，设定储存区会将修改通知至清单视图。

我们最终任务是实现筛选与排序选项，以只显示和使用者偏好相配的餐厅。我们从实现下列两个筛选选项来开始：

- 只显示打卡过的餐厅（Show Check-in Only）。
- 显示低于某个价位级别的餐厅（Show restaurants below a certain price level）。

在 `ContentView.swift` 中，我们将建立一个名为 `showShowItem` 的新函数来处理筛选：

```
private func shouldShowItem(restaurant: Restaurant) -> Bool {
    return (!self.settingStore.showCheckInOnly || restaurant.isCheckIn) && (restaurant.priceLevel <= self.settingStore.maxPriceLevel)
}

```

该函数带入一个餐厅物件，并告诉调用者是否应该显示餐厅。在上列的代码中，我们检查“Show Check-in Only”选项是否被选取，并检验指定餐厅的价位级别。

接下来，使用 `if` 语句包裹 `BasicImageRow`，如下所示：

```
if self.shouldShowItem(restaurant: restaurant) {
        BasicImageRow(restaurant: restaurant)
            .contextMenu {

                 ...

            }
}

```

这里，我们首先调用刚才实现的 `shouldShowItem` 函数，来检查是否应该显示餐厅。

现在按 Play 在模拟器运行 App 并快速测试。在设定画面中，设定“Show Check-in Only”选项为“ON”， 并配置价位级别选项，以显示价位级别为 3（即$$$ ）或以下的餐厅，如图 14.4 所示。当你点击“Save”按钮后，清单视图应会自动修改（使用动画），并显示筛选后的纪录。

![%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-4.png](%E4%BD%BF%E7%94%A8%20Combine%20%E4%B8%8E%20Environment%20%E7%89%A9%E4%BB%B6%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%88%86%E4%BA%AB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2080f8bcfd54ed4ce480a249c8958e7529/swiftui-formdata-4.png)

图 14.4. 当你修改筛选偏好时，清单视图现在会修改其项目

### 实现排序选项

现在，我们已经完成筛选选项的实现，我们来继续处理排序选项。在 Swift 中，你可以使用 sort(by:) 方法对一个序列中的元素排序。当使用此方法时，你需要提供一个述词（predicate ）给它，在第一个元素应排在第二个元素之前，该述词会回传 `true`。

举例而言，要将 `restaurants` 数组依字母排序，则可以使用 `sort(by:)`方法，如下所示：

```
restaurants.sorted(by: { $0.name < $1.name })

```

这里，$0 是第一个元素，$1 是第二个元素。在这个例子中，名称为“Upstate”的餐厅大于名称为“Homei”的餐厅，因此“Homei”将依顺序放在“Upstate”的前面。

反之，如果你想要以字母降幂来排序餐厅，你可以编写代码如下：

```
restaurants.sorted(by: { $0.name > $1.name })

```

我们如何排序数组来显示“check-in”优先，或显示“favorite”优先呢？我们可以使用相同的方法，但是提供不同的述词，如下所示：

```
restaurants.sorted(by: { $0.isFavorite && !$1.isFavorite })
restaurants.sorted(by: { $0.isCheckIn && !$1.isCheckIn })

```

为了更加组织代码，我们可以将这些述词放在 `DisplayOrder` 枚举中。至 SettingStore.swift 档，于 DisplayOrderType 中加入一个新函数，如下所示：

```
func predicate() -> ((Restaurant, Restaurant) -> Bool) {
    switch self {
    case .alphabetical: return { $0.name < $1.name }
    case .favoriteFirst: return { $0.isFavorite && !$1.isFavorite }
    case .checkInFirst: return { $0.isCheckIn && !$1.isCheckIn }
    }
}

```

此函数仅回传对应显示顺序的述词（即一个闭包）。现在，我们准备进行最后的修改。回到 `ContentView.swift`，并将 `ForEach` 叙述从：

```
ForEach(restaurants) {
  ...
}

```

修改为：

```
ForEach(restaurants.sorted(by: self.settingStore.displayOrder.predicate())) {
  ...
}

```

如此，你可以测试 App，并修改排序偏好。当你修改排序选项时，这个清单视图将得到通知，并相应地重新排序餐厅。

### 下一章的主题

你知道 SwiftUI 与 Combine 可帮助我们撰写出更好的代码吗？在本章最后两节中，我们并没有撰写很多的代码来实现筛选及排序选项。Combine 处理事件处理的繁重工作， 将它与 SwiftUI 搭配使用时，它的功能更加强大，并节省你开发实现来监控物件的状态变化与触发 UI 修改的时间。一切几乎是自动的，并由这两个新框架来负责。

在下一章中，我们将会继续通过“建立注册画面”来探索 Combine 。你将进一步了解 Combine 如何帮助你写出更简洁与模组化的代码。

在本章所准备的示例档中，有完整的项目可供下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIFormData.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIFormData.zip))