# 利用 UIViewRepresentable 整合UIKit 组件 | 精通 SwiftUI - iOS 16 版

开发者一般都会问两个有关 SwiftUI 的常见问题。 首先是如何在SwiftUI项目使用 Core Data， 而另一个常见问题是如何在 SwiftUI 项目中使用 UIKit 视图。 在本章中，我们将通过在 Todo App 建立搜索栏以学习如何将UIKit内的 UISearchBar 整合至 SwiftUI 项目。

如果你是 UIKit 的新手，UISearchBar 是UIKit框架的一个内置组件，它允许开发者为数据搜索呈现一个搜索栏。 图 23.1 显示了 iOS 中的标准搜索栏。 然而，在SwiftUI 刚推出时，它并没有附带这个标准的 UI 组件。 要在 SwiftUI 项目（例如我们的 ToDo App）加入搜索栏，其中一种方法就是使用 UIKit 中的`UISearchBar` 组件。

那么，我们如何在 SwiftUI 中整合 UIKit 视图或控制器？

为了向后兼容，Apple 在 iOS SDK 中引入了几个新协议，即 UIViewRepresentable 和 UIViewControllerRepresentable。 使用这些协议，你可以包装 UIKit 视图（或视图控制器）并使其可用于你的 SwiftUI 项目。

为了了解它是如何工作的，我们将为Todo App加入搜索功能 。 我们将在App标题正下方添加一个搜索栏，让用户输入搜索词来过滤待办事项。

![%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-1.png](%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-1.png)

图 1. 标准搜索栏

首先，请在 [https://www.appcoda.com/resources/swiftui4/SwiftUIToDoList.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIToDoList.zip) 下载 ToDo 项目。 我们将在 ToDoList 项目之上再作修改加入搜索功能。 如果你还没有阅读第 22 章，我建议你先阅读。 这将帮助你更理解我们将在下面讨论的主题，特别是如果你没有 Core Data 的实现经验。

### 了解 UIViewRepresentable 运作

要在 SwiftUI 中使用 UIKit 视图，你需要使用 UIViewRepresentable 协议包装视图。 基本上，你只需要在 SwiftUI 中建立一个 `struct`，它采用协议来建立和管理一个 `UIView` 对象。 这是 UIKit 视图自订包装器（custom wrapper）的框架：

```
struct CustomView: UIViewRepresentable {

    func makeUIView(context: Context) -> some UIView {
        // Return the UIView object
    }

    func updateUIView(_ uiView: some UIView, context: Context) {
        // Update the view
    }
}

```

在实际应用中，你将`some UIView` 替换为你想要包装的 UIKit 视图。 比方说，我们想在 UIKit 中使用`UISearchBar`。 代码可以这样写：

```
struct SearchBar: UIViewRepresentable {

    func makeUIView(context: Context) -> UISearchBar {

        return UISearchBar()
    }

    func updateUIView(_ uiView: UISearchBar, context: Context) {

        // Update the view
    }
}

```

在`makeUIView` 方法中，我们回一个`UISearchBar` 的实体（instance）。 这就是如何将 UIKit 视图整合至 SwiftUI。 要使用 `SearchBar`，你可以像对待任何 SwiftUI 视图一样。以下是一个个例子：

```
struct ContentView: View {
    var body: some View {
        SearchBar()
    }
}

```

### 添加搜索栏

现在回到 *ToDoList* 项目，我们会为App加入搜索栏。 首先，我们将为搜索栏创建一个新文件。 在项目导航器中，右键单击 *View* 文件夹并选择 *New File....* 选择 *SwiftUI View* 模板并将文件命名为`SearchBar.swift`。

将内容替换为以下代码：

```
import SwiftUI

struct SearchBar: UIViewRepresentable {

    @Binding var text: String

    func makeUIView(context: Context) -> UISearchBar {

        let searchBar = UISearchBar()

        searchBar.searchBarStyle = .minimal
        searchBar.autocapitalizationType = .none
        searchBar.placeholder = "Search..."

        return searchBar
    }

    func updateUIView(_ uiView: UISearchBar, context: Context) {

        uiView.text = text
    }
}

struct SearchBar_Previews: PreviewProvider {
    static var previews: some View {
        SearchBar(text: .constant(""))
    }
}

```

该代码与上一节中显示的代码类似，但有以下区别：

1. 当建立`UISearchBar`时，我们并没有使用默认外观，而是使用最简单（`.minimal`）的样式，并停用自动大写和修改其占位符的文字。
2. 我们添加了一个绑定来存放使用者于搜寻栏位中的搜寻文字。 `makeUIView` 方法负责建立和初始化视图，而 `updateUIView` 方法则负责修改 UIKit 视图的状态。 每当 SwiftUI 中的状态发生变化时，框架都会自动调用 `updateUIView` 方法来修改视图的配置。 在这种情况下，每当你在 SwiftUI 中修改搜索文字时，都会调用该方法并修改`UISearchBar`的`text`。

现在切换到`ContentView.swift`。 声明一个状态变量来存放搜索文字：

```
@State private var searchText = ""

```

要显示搜索栏，请在 `List` 之前加入以下代码：

```
SearchBar(text: $searchText)
    .padding(.top, -20)

```

`SearchBar` 就像任何其他 SwiftUI 视图一样，你可以应用`.padding`等修饰器来调整布局。 如果你在模拟器中运行该App，你应该会看到一个搜索栏，但它尚未起能正常运作。

![%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-2.png](%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-2.png)

图 2. 加入搜索栏

### 处理搜索文字

如你所见，在 SwiftUI App中显示 UIKit 视图并不是一件很复杂的事。 虽然这样说，要样搜索栏运作就是另一回事。 目前，你可以在搜索字段中输入，但App尚未能运行查询。 这个App应该在使用者输入搜索文字时即时搜索待办事项。

那么，我们如何知道用户正在输入搜索文字呢？

搜索栏有一个名为`UISearchBarDelegate`的配套协议。 该协议提供了多种管理搜索文字的方法。 特别是，每当用户修改搜索文字时，都会调用以下方法：

```
optional func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String)

```

为了使搜索栏运作，我们必须采用`UISearchBarDelegate` 协议。如你没有任何UIKit开发经验， 这就是复杂的地方。

到目前为止，我们只讨论了 `UIViewRepresentable` 协议中的几个方法。 如果你需要在 UIKit 中使用委托并与 SwiftUI 沟通，则必须实现 `makeCoordinator` 方法并提供一个 `Coordinator` 实体（instance）。 这个`Coordinator` 充当了 UIView 的委托和 SwiftUI 之间的桥梁。 让我们看一下代码，这样你就会明白它的含义。

在 `Search Bar` 结构（`Search Bar.swift`），建立一个 `Coordinator` 类别并实现 `makeCoordinator` 方法，如下所示：

```
func makeCoordinator() -> Coordinator {
    Coordinator($text)
}

class Coordinator: NSObject, UISearchBarDelegate {
    @Binding var text: String

    init(_ text: Binding<String>) {
        self._text = text
    }

    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {

        searchBar.showsCancelButton = true
        text = searchText

        print("textDidChange: \(text)")
    }
}

```

`makeCoordinator` 方法只回一个 `Coordinator` 的实体。 `Coordinator`采用`UISearchBarDelegate` 协议并实现`searchBar(_:textDidChange:)` 方法。 如前所述，每次用户修改搜寻文字时都会调用此方法。 因此，当有修改时，我们通过修改 `text` 绑定将其传回 SwiftUI。 我特意在方法里加了一个`print`语句，方便以后我们测试app的时候可以看到变化。

现在我们有一个采用了 `UISearchBarDelegate` 协议的 Coordinator，我们需要再做一个修改。 在 `makeUIView` 方法中，加入以下一行代码，将协调器指定给搜索栏：

```
searchBar.delegate = context.coordinator

```

就是这样！ 再次运行App并在输入搜寻文字。 你应该会在控制台中看到`textDidChange:`的信息。

![%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-3.png](%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-3.png)

图 3. 控制台在你输入搜寻文字时即时显示信息

### 处理取消按钮

你是否点击了*取消*按钮？ 如果你尝试过，就会知道它又是不能正常运作。 要解决这个问题，我们必须在`Coordinator`中实现以下方法：

```
func searchBarCancelButtonClicked(_ searchBar: UISearchBar) {
    text = ""
    searchBar.resignFirstResponder()
    searchBar.showsCancelButton = false
    searchBar.endEditing(true)
}

func searchBarShouldBeginEditing(_ searchBar: UISearchBar) -> Bool {
    searchBar.showsCancelButton = true

    return true
}

```

单击取消按钮时触发第一种方法。 在代码中，我们调用 `resignFirstResponder()` 来关闭键盘并告诉搜索栏结束编辑。 第二种方法确保在使用者点击搜寻文字时出现 *Cancel* 按钮。

你可以在模拟器中运行App来试试这个修改。 在编辑时，点击 *Cancel* 按钮应该会自动关闭软体键盘。

### 搜寻文字

我们现在可以检索搜寻文字并处理了取消按钮。 可惜，搜索栏仍然无法正常使用。 这就是我们将在本节中讲解的内容。 对于这个App，有几种方法可以运行搜索：

1. 使用 `filter` 函数对 `todoItems` 运行搜索
2. 通过指定述词（predicate ）并利用 `FetchRequest` 运行搜寻

基本上第一种方法对于这个App来说已经足够，因为 `todoItems` 与储存在数据库中的待办事项同步。 但我还是想向你示范如何使用`FetchRequest`来运行搜寻。 因此，我们将一齐研究这两种方法。

### 使用 `filter` 函数

在 Swift 中，你可以使用 `filter` 函数以回圈来重复查询一个集合，然后回传一个内含符合搜寻条件项目的新数组。 以下是一个例子：

```
todoItems.filter({ $0.name.contains("Buy") })

```

`filter` 函数内的其中一个参数是给调用者以闭包形式指定过滤条件。 举例，以上的代码就会搜出所有包含“Buy”字名称的项目。

To implement the search, we can replace the `ForEach` loop of the `List` like this:为了实现搜寻功能，我们可以像这样修改 `List` 的 `ForEach` 回圈：

```
ForEach(todoItems.filter({ searchText.isEmpty ? true : $0.name.contains(searchText) })) { todoItem in
    ToDoListRow(todoItem: todoItem)
}
.onDelete(perform: deleteTask)

```

在 `filter` 函数的闭包中，我们首先检查搜寻文字是否有值。 如果没有，我们简单地回 `true`。这意味所有待办事项都会被加入新数组中。 相反，就只回包含搜寻字串的待办事项。

酷 ！你可以准备启动你的 App 来测试搜寻功能。 输入搜寻文字，App就会搜寻相关记录。

![%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-4.png](%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-4.png)

图 4. 搜寻待办事项

### 使用`FetchRequest`来运行搜寻

刚刚讨论的搜寻方法是从已取得的结果运行搜寻。 另一种方法则是直接使用 Core Data 运行搜寻。 当我们从数据库中获取数据时，我们明确指定要过虑的待办事项。

`@FetchRequest` 属性包装器允许你传递一个我们之前没有讨论过的述词（predicate）。通过提供述词，你就可指定过滤条件。

以下是一个例子：

```
@FetchRequest(
    entity: ToDoItem.entity(),
    sortDescriptors: [ NSSortDescriptor(keyPath: \ToDoItem.priorityNum, ascending: false) ],
    predicate: NSPredicate(format: "name CONTAINS[c] %@", "Buy")
)

```

通过提供述词，获取请求（fetch request）就只会搜出包含“Buy”这个搜索词的待办事项。 `CONTAINS` 后面的 `[c]` 表示搜寻不区分大小写。 如果你想试一试，请确保将 `ForEach` 恢复为原始代码（没有过滤功能）。 然后，用上面的代码修改`@FetchRequest`。

假设你添加了一些待办事项而其中有一些名称带有“Buy”这个字，你应该只会看到搜索词为“Buy”的待办事项。

![%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-5.png](%E5%88%A9%E7%94%A8%20UIViewRepresentable%20%E6%95%B4%E5%90%88UIKit%20%E7%BB%84%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%206f5bdcaa762b483f941f33e38722902f/swiftui-uikit-5.png)

图 5. 搜索词为“Buy”的待办事项

看起来很简单吧？ 但是当你需要建立一个带有动态述词的 fetch 请求时，就没有那么简单了。 一旦你使用特定述词初始化读取请求（fetch request），你就无法修改它。 排序描述符（sort descriptor）也是如此。

那么，我们如何构建一个支持不同述词的读取的请求呢？

诀窍是不要使用`@FetchRequest` 属性包装器。 相反，我们手动建立读取请求。 为了做到这一点，我们将建立一个名为`FilteredList`的独立视图，它接受搜寻文字作为参数。 这个 `FilteredList` 负责建立读取请求，搜索相关的待办事项，并将它们呈现在列表视图中。

在`ContentView.swift`中，加入以下代码来建立`FilteredList`：

```swift
struct FilteredList: View {

    @Environment(\.managedObjectContext) var context

    @Binding var searchText: String

    var fetchRequest: FetchRequest<ToDoItem>
    var todoItems: FetchedResults<ToDoItem> {
        fetchRequest.wrappedValue
    }

    init(_ searchText: Binding<String>) {
        self._searchText = searchText

        let predicate = searchText.wrappedValue.isEmpty ? NSPredicate(value: true) : NSPredicate(format: "name CONTAINS[c] %@", searchText.wrappedValue)

        self.fetchRequest = FetchRequest(entity: ToDoItem.entity(),
                                         sortDescriptors: [ NSSortDescriptor(keyPath: \ToDoItem.priorityNum, ascending: false) ],
                                         predicate: predicate,
                                         animation: .default)
    }

    var body: some View {

        ZStack {
            List {

                ForEach(todoItems) { todoItem in
                    ToDoListRow(todoItem: todoItem)
                }
                .onDelete(perform: deleteTask)

            }
            .listStyle(.plain)

            if todoItems.count == 0 {
                NoDataView()
            }
        }

    }

    private func deleteTask(indexSet: IndexSet) {
        for index in indexSet {
            let itemToDelete = todoItems[index]
            context.delete(itemToDelete)
        }

        do {
            try context.save()
        } catch {
            print(error)
        }
    }
}

```

看一下 `body` 和 `deleteTask`。 两者都和以前完全一样。 我们只是将相关代码转移至`FilteredList`。主要是修改了 `init` 方法和读取请求。

我们声明了一个名为 `fetchRequest` 的变量来存放读取请求，并声明另一个名为 `todoItems` 的变量来储存读取的结果。 读取的结果实际上是从读取请求的 `wrappedValue` 属性中所获得的。

现在让我们深入研究 `init` 方法。 这个自订的 `init` 方法接受搜寻文字作为参数。 更明确一点说，它应是搜寻文字的绑定。 这个也是我们需要建立自订 `init` 的原因，让我们能根据不同的搜寻文字建立不同的读取请求。

`init` 方法的第一行是储存搜寻文字的绑定。 要指出绑定，请如下所示使用下划线：

```
self._searchText = searchText

```

接下来，我们检查搜寻文字是否为空并相应地构建述词：

```
let predicate = searchText.wrappedValue.isEmpty ? NSPredicate(value: true) : NSPredicate(format: "name CONTAINS[c] %@", searchText.wrappedValue)

```

当准备好述词后，我们就可以像以下代码建立读取请求：

```
self.fetchRequest = FetchRequest(entity: ToDoItem.entity(),
                                 sortDescriptors: [ NSSortDescriptor(keyPath: \ToDoItem.priorityNum, ascending: false) ],
                                 predicate: predicate,
                                 animation: .default)

```

如你所见，其用法与 `@FetchRequest` 属性包装器的用法非常相似。

酷！ 我们现在有一个 `FilteredList` 可以处理具有不同述词的读取请求。 现在让我们修改 `ContentView` 结构以使用这个新的 `FilteredList`。

由于我们已将读取请求移至`FilteredList`，因此我们可以删除以下变量：

```
@Environment(\.managedObjectContext) var context

@FetchRequest(
    entity: ToDoItem.entity(),
    sortDescriptors: [ NSSortDescriptor(keyPath: \ToDoItem.priorityNum, ascending: false) ],
    predicate: NSPredicate(format: "name CONTAINS[c] %@", "buy")
)
var todoItems: FetchedResults<ToDoItem>

```

跟着，将下面的代码：

```
List {

    ForEach(todoItems.filter({ searchText.isEmpty ? true : $0.name.contains(searchText) })) { todoItem in
        ToDoListRow(todoItem: todoItem)
    }
    .onDelete(perform: deleteTask)

}

```

改为:

```
FilteredList($searchText)

```

这里我们使用`FilteredList` 来渲染列表视图。 我们传递`searchText` 的绑定来运行搜寻。 由于`searchText` 是一个状态变量，任何有关搜寻文字的修改都会触发`FilteredList` 的修改。 实际上，当使用者输入搜寻文字时，App会建立一个不同的述词以取得一组新的待办事项。

接下来，删除以下代码，因为它已在`FilteredList`中：

```
// If there is no data, show an empty view
if todoItems.count == 0 {
    NoDataView()
}

```

最后，删除`ContentView`中的`deleteTask`方法，因为它已迁移到了`FilteredList`：

```
private func deleteTask(indexSet: IndexSet) {
    for index in indexSet {
        let itemToDelete = todoItems[index]
        context.delete(itemToDelete)
    }

    DispatchQueue.main.async {
        do {
            try context.save()
        } catch {
            print(error)
        }
    }
}

```

现在你已准备好进行测试！ 如果你已正确修改所有代码，App会在你输入搜寻文字时过滤待办事项。

### 总结

在本章中，你学习如何使用 `UIViewRepresentable` 协议将 UIKit 视图整合至 SwiftUI 项目。 虽然 SwiftUI 仍然很新并且没有附带所有标准 UI 组件，但这种向后兼容性允许你利用旧框架使用任何视图。

我们还讲解了两种实现数据搜寻的方法， 你现在应该知道如何使用 `filter` 函数并了解如何建立动态读取请求。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListUISearchBar.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListUISearchBar.zip)