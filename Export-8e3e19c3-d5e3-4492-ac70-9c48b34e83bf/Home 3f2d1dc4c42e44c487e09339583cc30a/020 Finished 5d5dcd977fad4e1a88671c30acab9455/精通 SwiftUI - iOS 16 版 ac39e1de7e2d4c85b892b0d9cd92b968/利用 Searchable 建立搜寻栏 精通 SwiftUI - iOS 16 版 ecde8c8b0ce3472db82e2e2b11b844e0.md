# 利用 Searchable 建立搜寻栏 | 精通 SwiftUI - iOS 16 版

在 iOS 15 之前，SwiftUI 没有内置修饰器在List`视图中加入搜寻功能。开发者必须建立自己的解决方案。 在前面的章节中，我已经介绍了两个自建搜寻栏的方法。随着 iOS 15 推出，SwiftUI 框架为 List 视图带来了一个名为` `searchable` 的新修饰器，让你更轻易建立搜寻栏。

在本章中，你将学习使用`searchable`，待会你就会受到这个修饰器有多方便。

### Searchable 的基本用法

为了讲解 `Searchable` 的用法，请从以下网址下载示例项目：[https://www.appcoda.com/resources/swiftui4/SwiftUISearchableStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUISearchableStarter.zip).

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-1.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-1.png)

图 37.1. 示例项目

示例的初始项目已经准备好了一个列表视图来显示一组文章。 之后，我们会加入一个用于过滤文章的搜寻栏。 要将搜寻栏添加到列表视图，你需要做的就是声明一个状态变量（例如`searchText`）来存放搜寻文字并将`searchable`修饰器附加到`NavigationStack`，如下所示：

```
struct ContentView: View {

    @State var articles = sampleArticles
    @State private var searchText = ""

    var body: some View {
        NavigationStack {
            .
            .
            .
        }
        .searchable(text: $searchText)
    }
}

```

SwiftUI 会自动为你加入搜寻栏，并将其放在导航栏标题下方。 如果找不到搜寻栏，请尝试运行App并向下拖动列表视图。 这样搜寻框就应该出现。

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-2.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-2.png)

图 37.2. searchable 修饰器自动为你加入搜寻栏

在默认情况下，它显示 *Search* 作为占位符。 如果你想修改它，你可以像这样编写`.searchable`修饰器并使用你指定的占位符：

```
.searchable(text: $searchText, prompt: "Search articles...")

```

### 搜寻栏位置

`.searchable` 修饰器有一个 `placement` 参数，用于指定搜寻栏的位置。 在默认的情况下，它设置为`.automatic`。 就 iPhone 而言，搜寻栏会显示于导航栏标题下方。 当你向上滚动列表视图时，搜寻栏就会自动隐藏。

如果想永久显示搜寻栏，可以修改 `.searchable` 修饰器并提供 `placement` 参数，如下所示：

```
.searchable(text: $searchText, placement: .navigationBarDrawer(displayMode: .always))

```

到目前为止，我们将 `.searchable` 修饰器附加到导航视图。 你实际上可以将它附加到 `List` 视图并在 iPhone 上实现相同的结果。

话虽如此，在 iPadOS 上使用 split view 时，`.searchable` 修饰器的摆放位置会直接影响搜寻栏的显示位置。 再看一下示例代码：

```
NavigationSplitView {
    List {
        ForEach(articles) { article in
            ArticleRow(article: article)
        }

        .listRowSeparator(.hidden)

    }
    .listStyle(.plain)

    .navigationTitle("AppCoda")
} detail: {
    Text("Article details")
}
.searchable(text: $searchText, placement: .navigationBarDrawer(displayMode: .always))

```

像往常一样，我们将 `.searchable` 修饰器附加到导航视图。 如果你在 iPad 上运行示例App，搜寻栏会显示在拆分视图的侧边栏上。

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-3.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-3.png)

图 37.3. 在 iPad 上显示拆分视图的搜索栏

如果你想将搜寻栏放在详细视图中，那应怎样做？ 你可以尝试在 `.navigationTitle` 修饰器的正上方加入以下代码：

```
Text("Article details")
    .searchable(text: $searchText)

```

这样，iPadOS 将在详细视图的右上角显示一个额外的搜寻栏。

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-4.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-4.png)

图 37.4. 在详细视图的右上角显示一个额外的搜寻栏

同样地，你可以通过调整 `placement` 参数的值来进一步修改搜寻栏的位置。 这是一个例子：

```
.searchable(text: $searchText, placement: .navigationBarDrawer)

```

通过将 `placement` 参数设置为 `.navigationBarDrawer`，iPadOS 就会将搜寻栏置于导航栏标题下方。

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-5.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-5.png)

图 37.5. 利用 .navigationBarDrawer 改变搜寻栏位置

### 如何运行搜寻并显示搜索结果

搜寻数据的方法有很多种。 你可以建立一个计算属性（computed property）来实时过滤数据。 又或者，你可以附加 `.onChange` 修饰器以监测使用者输入的搜寻文字，如有修改就立刻启动搜寻。 试试修改 `NavigationStack` 的代码，如下所示：

```
NavigationStack {
    List {
        ForEach(articles) { article in
            ArticleRow(article: article)
        }

        .listRowSeparator(.hidden)

    }
    .listStyle(.plain)

    .navigationTitle("AppCoda")
}
.searchable(text: $searchText)
.onChange(of: searchText) { searchText in

    if !searchText.isEmpty {
        articles = sampleArticles.filter { $0.title.contains(searchText) }
    } else {
        articles = sampleArticles
    }
}

```

每当使用者输入搜寻字时，App都会运行 `.onChange` 修饰器。 然后我们使用 `filter` 方法实时进行搜寻。Xcode 预览无法正常搜索，请在模拟器上测试搜索功能。

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-6.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-6.png)

图 37.6. 运行搜寻

### 添加搜寻建议

`.searchable` 修饰器允许你添加搜寻建议列表，以显示一些常用的搜索词或搜寻历史。 例如，你可以像这样建立可点击的搜寻建议：

```
.searchable(text: $searchText) {
    Text("SwiftUI").searchCompletion("SwiftUI")
    Text("iOS 15").searchCompletion("iOS 15")
}

```

当使用者进行搜索时，搜寻栏就会显示两个常用的搜寻建议。 使用者可以键入搜寻关键字或点击搜寻建议来运行搜索。

![%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-7.png](%E5%88%A9%E7%94%A8%20Searchable%20%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ecde8c8b0ce3472db82e2e2b11b844e0/searchable-7.png)

图 37.7. 显示搜寻建议

在 iOS 16 上，Apple 引入了一个名为 `.searchSuggestions` 的修饰符，用于添加搜索建议。 以上同一段代码可以写成这样：

```
.searchable(text: $searchText)
.searchSuggestions {
    Text("SwiftUI").searchCompletion("SwiftUI")
    Text("iOS 15").searchCompletion("iOS 15")
}

```

### 总结

`.searchable` 修饰器简化了搜寻栏的实现方法。在一般情况下，我们不再需要自己开发搜寻栏。 缺点是此功能仅适用于 iOS 15（或更高版本）。 如果你正在构建一个需要支持旧版本 iOS 的应用程序，你仍然需要建立自己的搜寻栏。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUISearchable.zip](https://www.appcoda.com/resources/swiftui4/SwiftUISearchable.zip)