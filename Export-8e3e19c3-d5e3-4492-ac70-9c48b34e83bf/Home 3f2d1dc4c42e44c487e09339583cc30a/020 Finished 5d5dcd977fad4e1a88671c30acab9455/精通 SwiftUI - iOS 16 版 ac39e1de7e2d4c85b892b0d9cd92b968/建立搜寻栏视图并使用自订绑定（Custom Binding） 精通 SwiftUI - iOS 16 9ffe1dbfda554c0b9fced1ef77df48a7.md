# 建立搜寻栏视图并使用自订绑定（Custom Binding） | 精通 SwiftUI - iOS 16 版

之前，我们向你讲解了如何利用 UIKit 框架的`UISearchBar` 来实现搜索栏。 你有没有想过从头开始自己建立一个？ 如果仔细看看搜寻栏，要自己做出来并不太难。 所以，让我们在本章中尝试构建一个 SwiftUI 版本的搜寻栏。

你不仅可以学习如何建立搜寻栏视图，我们还将会和你讲解如何使用自订绑定。 之前我们讨论过绑定，但还没有向你展示如何建立自定义绑定。 当你需要为绑定（Binding）加入额外的程序逻辑时，自订绑定就特别有用。 另外，你也会学习如何在 SwiftUI 中关闭软体键盘。

图 24.1 就是我们将要建立的搜索栏，外观与 UIKit 中的 UISearchBar 非常相似。 同样地，这个搜索栏也会有个 *Cancel* 按钮，当使用者开始输入搜寻文字时就会出现。

![%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-1.png](%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-1.png)

图 24.1. 利用 SwiftUI 自己建立搜寻栏

### 实现搜寻栏 UI

我们将会修改之前的项目转为我们自己建立的搜寻栏。 因此，请首先从 [https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListUISearchBar.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListUISearchBar.zip) 下载启动项目。 编译一次以确保它有效。 该App应该向你显示一个搜索栏，但是，该栏来自 UIKit。 我们将把它转换成一个完全使用 SwiftUI 构建的搜寻栏视图。

打开`SearchBar.swift`，这是我们关注的文件。 我们将重写整段代码，但保持其名称不变。 我们仍然称它为“SearchBar”，它仍然接受搜寻文字的绑定作为参数。 对于调用者（即 ContentView），没有什么需要修改， 用法还是这样：

```
SearchBar(text: $searchText)

```

现在，让我们从 UI 开始做起。 如果你想挑战自己，请停止阅读并尝试自己实现搜寻栏 UI。 这个使用者介界非常简单。 它由一个文字栏、几个图标和取消按钮组成。

如果你不知道 UI 是如何建立的，就让我们一起写出来吧。 请将 `SearchBar.swift` 中的 `SearchBar` 结构改成这样：

```swift
struct SearchBar: View {
    @Binding var text: String

    @State private var isEditing = false

    var body: some View {
        HStack {

            TextField("Search ...", text: $text)
                .padding(7)
                .padding(.horizontal, 25)
                .background(Color(.systemGray6))
                .cornerRadius(8)
                .overlay(
                    HStack {
                        Image(systemName: "magnifyingglass")
                            .foregroundColor(.gray)
                            .frame(minWidth: 0, maxWidth: .infinity, alignment: .leading)
                            .padding(.leading, 8)

                        if isEditing {
                            Button(action: {
                                self.text = ""
                            }) {
                                Image(systemName: "multiply.circle.fill")
                                    .foregroundColor(.gray)
                                    .padding(.trailing, 8)
                            }
                        }
                    }
                )
                .padding(.horizontal, 10)
                .onTapGesture {
                    withAnimation {
                        self.isEditing = true
                    }
                }

            if isEditing {
                Button(action: {
                    self.isEditing = false
                    self.text = ""

                }) {
                    Text("Cancel")
                }
                .padding(.trailing, 10)
                .transition(.move(edge: .trailing))
            }
        }
    }
}

```

首先，我们声明了两个变量：一个是搜寻文字的绑定，另一个则用于存放搜寻状态（正在编辑与否）的变量。

我们使用 `HStack` 来布局文字栏和 *Cancel* 按钮。 对于文字栏，我们覆盖了一个放大镜图案和交叉图案（即`multiply.circle.fill`），它仅在搜寻栏处于编辑模式时显示。 *Cancel* 按钮也是如此，当使用者点击搜寻栏时才会出现。

为了预览搜寻栏，还请加入以下代码：

```
struct SearchBar_Previews: PreviewProvider {
    static var previews: some View {
        SearchBar(text: .constant(""))
    }
}

```

当加入代码后，你应该能够预览搜寻栏。 现在按*播放* 钮进行测试。 当你选择搜寻栏时，*取消* 按钮就会出现。

![%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-2.png](%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-2.png)

图 24.2. 预览搜寻栏

更重要的是，搜寻栏已经可以使用了！ 在模拟器上运行App并输入搜寻字，它就会根据显示相关的搜索结果。

![%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-3.png](%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-3.png)

图 24.3. 搜寻栏已正式运作

### 关闭键盘

如你所见，使用 SwiftUI 建立我们自己的搜寻栏并不太难。 在搜寻栏运作时，我们必须解决一个小问题。 你有否尝试点击取消按钮？ 它确实清除了搜寻文字。 然而，软键盘并没有消失。

为了解决这个问题，我们需要在 *Cancel* 按钮的 `action` 中添加一行程码：

```
// 关闭键盘
UIApplication.shared.sendAction(#selector(UIResponder.resignFirstResponder), to: nil, from: nil, for: nil)

```

在代码中，我们调用了 `sendAction` 方法来关闭键盘。 你现在可以使用模拟器运行App。 当你点击取消按钮时，它就会清除搜寻文字并关闭软体键盘。

### 使用自订绑定（Custom Binding）

SwiftUI 版本的搜寻栏已经可以正常使用了，不过我想藉此机会与你讨论自订绑定。 在 `SearchBar.swift` ，我们像这样声明搜寻文字的绑定：

```
@Binding var text: String

```

它非常适合我们当前程序的运作。 但是让我来问问你， 如果我们在读取或写入此绑定时需要添加额外的逻辑，那可以怎么办？ 例如，如何将使用者输入的文字自动变成大写？

Swift 有一个内置的功能可以将字串变为大写。 你可以使用字串的 `capitalized` 属性取得大写文字串。但 问题是我们如何修改`text`的绑定？

在这种情况下，你需要在`SearchBar.swift`中建立一个自定义绑定，如下所示：

```
private var searchText: Binding<String> {

    return Binding<String>(
        get: {
            self.text.capitalized

        }, set: {
            self.text = $0
        }
    )
}

```

在上面的代码中，我们建立了一个名为`searchText`的自定义绑定，其中包含读取和写入绑定值的闭包。 对于`get` 部分，我们通过`capitalized` 属性来自定义`text` 的绑定值。 这就是我们如何将使用者的搜寻字转成大写。 至于`set` 部分，我们不做任何修改，只将其设置为原始值。 但是，如果在设置绑定时需要添加额外的逻辑，就需要修改`set`中的程码。

题外话，你可以省略 `return` 关键字，像这样写就可以：

```
private var searchText: Binding<String> {

    Binding<String>(
        get: {
            self.text.capitalized

        }, set: {
            self.text = $0
        }
    )
}

```

以防你没有留意，这是自 Swift 5.1 就有的一项新功能，

我们仍在将 `text` 绑定传递给 `TextField`。 在此修改生效之前，我们需要再进行一个小修改。 修改`TextField`中的参数并确保将`searchText`作为绑定传递：

```
TextField("Search ...", text: searchText)

```

现在在模拟器上运行该App。 在搜寻栏输入几个词，App就会自动把每个单词的第一个字母变成大写。

![%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-4.png](%E5%BB%BA%E7%AB%8B%E6%90%9C%E5%AF%BB%E6%A0%8F%E8%A7%86%E5%9B%BE%E5%B9%B6%E4%BD%BF%E7%94%A8%E8%87%AA%E8%AE%A2%E7%BB%91%E5%AE%9A%EF%BC%88Custom%20Binding%EF%BC%89%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%209ffe1dbfda554c0b9fced1ef77df48a7/swiftui-searchbar-4.png)

图 24.4. 自动把每个单词的第一个字母变成大写

### 总结

在本章中，我们向你展示了另一种实现搜寻栏的方法。 如你所见，单是用 SwiftUI 建立一个搜寻栏并不困难。 你还学习了如何建立自定义绑定。 当你在设置或检索绑定值时需要添加额外的程序时，这就非常有用。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListSearchBarView.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListSearchBarView.zip)