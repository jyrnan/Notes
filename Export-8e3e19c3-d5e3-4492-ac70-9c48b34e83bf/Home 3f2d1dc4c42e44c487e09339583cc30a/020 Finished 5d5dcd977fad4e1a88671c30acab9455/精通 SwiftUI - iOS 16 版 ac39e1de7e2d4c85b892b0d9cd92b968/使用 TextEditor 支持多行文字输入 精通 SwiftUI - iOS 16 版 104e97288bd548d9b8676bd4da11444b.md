# 使用 TextEditor 支持多行文字输入 | 精通 SwiftUI - iOS 16 版

SwiftUI 的第一个版本（随 iOS 13 推出）没有提供用于支持多行文字输入的文本视图（Text View）。 如要使用的话，就需要从 UIKit 框架中借用 `UITextView` ，并通过采用 `UIViewRepresentable` 协议使其可用于你的 SwiftUI 项目。 自 iOS 14 开始，Apple 为 SwiftUI 框架引入了一个名为`TextEditor`的新组件。 这个 `TextEditor` 使开发者能够在App中显示和编辑多行文字。 在 iOS 16 中，Apple 进一步改进了内置的 `TextField` 以支持多行输入。

在本章中，我们将详细讲解如何使用 `TextEditor` 以支持多行文字输入。

### 如何使用 TextEditor

`TextEditor` 的使用方法并不复杂，可以说是非常容易。 你只需要一个状态变量来存放输入文字。 然后，在视图的`body`中建立 `TextEditor` 就可以了。下面是示例代码：

```
struct ContentView: View {
    @State private var inputText = ""

    var body: some View {
        TextEditor(text: $inputText)
    }
}

```

要使用 `TextEditor`，你需要传一个 `inputText` 的绑定，以便状态变量可以储存使用者的文字输入。

你可以像任何 SwiftUI 视图一样客制编辑器的样式。举例，以下代码可以修改字体类型并调整文本编辑器的行距：

```
TextEditor(text: $inputText)
    .font(.title)
    .lineSpacing(20)
    .autocapitalization(.words)
    .disableAutocorrection(true)
    .padding()

```

另外，你也可以选择启用/停用自动大写和自动更正功能。

![%E4%BD%BF%E7%94%A8%20TextEditor%20%E6%94%AF%E6%8C%81%E5%A4%9A%E8%A1%8C%E6%96%87%E5%AD%97%E8%BE%93%E5%85%A5%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20104e97288bd548d9b8676bd4da11444b/swiftui-texteditor-1.png](%E4%BD%BF%E7%94%A8%20TextEditor%20%E6%94%AF%E6%8C%81%E5%A4%9A%E8%A1%8C%E6%96%87%E5%AD%97%E8%BE%93%E5%85%A5%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20104e97288bd548d9b8676bd4da11444b/swiftui-texteditor-1.png)

图 32.1. 使用 TextEditor

### 使用 onChange() 修饰器侦测文字输入

在 UIKit 框架， `UITextView`与 `UITextViewDelegate` 协议共同运作以处理文字输入。 那么，`TextEditor` 呢？ 我们如何侦测使用者输入的文字并对其进一步的处理？

SwiftUI 提供了一个 `onChange()` 修饰器，可以附加到 `TextEditor` 或任何其他视图。 假设你正在使用 `TextEditor` 建立一个笔记App，并且需要实时显示总字数。你可以将 `onChange()` 修饰器附加到 `TextEditor`，如下所示：

```
struct ContentView: View {
    @State private var inputText = ""
    @State private var wordCount: Int = 0

    var body: some View {
        ZStack(alignment: .topTrailing) {
            TextEditor(text: $inputText)
                .font(.body)
                .padding()
                .padding(.top, 20)
                .onChange(of: inputText) { value in
                    let words = inputText.split { $0 == " " || $0.isNewline }
                    self.wordCount = words.count
                }

            Text("\(wordCount) words")
                .font(.headline)
                .foregroundColor(.secondary)
                .padding(.trailing)
        }
    }
}

```

在上面的代码中，我们声明了一个 state 属性来储存字数。 另外，我们在 `onChange()` 修饰器中指定监测 `inputText` 的变化。 每当使用者输入文字时， `onChange()` 修饰符内的代码就会被运行。 在闭包中，我们计算 `inputText` 中的单词总数并相应地修改 `wordCount` 变量。

如果你在模拟器中运行App，应该会看到一个纯文字编辑器。试试输入几字，编辑器就会实时显示字数。

![%E4%BD%BF%E7%94%A8%20TextEditor%20%E6%94%AF%E6%8C%81%E5%A4%9A%E8%A1%8C%E6%96%87%E5%AD%97%E8%BE%93%E5%85%A5%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20104e97288bd548d9b8676bd4da11444b/swiftui-texteditor-2.png](%E4%BD%BF%E7%94%A8%20TextEditor%20%E6%94%AF%E6%8C%81%E5%A4%9A%E8%A1%8C%E6%96%87%E5%AD%97%E8%BE%93%E5%85%A5%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20104e97288bd548d9b8676bd4da11444b/swiftui-texteditor-2.png)

图 32.2. 利用 onChange() 侦测文字输入并显示字数

### 支持多行输入、可扩展的文字栏

在 iOS 16 之前，内置的 `TextField` 只能支持单行文字输入。 现在，Apple 大大改进了 `TextField` 视图，允许使用者输入多行文字。 更好的是，你现在可以使用一个名为 `axis` 的新参数来告诉 iOS 是否应该扩展文字栏。

例如，文字栏最初显示单行输入。 当使用者输入文字时，文字栏便会自动扩展以支持多行输入。 以下是示例代码：

```swift
struct TextFieldDemo: View {
    @State private var comment = ""

    var body: some View {
        TextField("Comment", text: $comment, prompt: Text("Please input your comment"), axis: .vertical)
            .padding()
            .background(Color.green.opacity(0.2))
            .cornerRadius(5.0)
            .padding()

    }
}

```

`axis` 参数的值可以是 `.vertical` 或 `.horizontal`。 在上面的代码中，我们将值设置为`.vertical`。 在这种情况下，文字栏会垂直扩展以支持多行输入。 如果设置为 `.horizontal`，文本字段将水平扩展并保持为单行文本字段。

![%E4%BD%BF%E7%94%A8%20TextEditor%20%E6%94%AF%E6%8C%81%E5%A4%9A%E8%A1%8C%E6%96%87%E5%AD%97%E8%BE%93%E5%85%A5%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20104e97288bd548d9b8676bd4da11444b/swiftui-texteditor-3.png](%E4%BD%BF%E7%94%A8%20TextEditor%20%E6%94%AF%E6%8C%81%E5%A4%9A%E8%A1%8C%E6%96%87%E5%AD%97%E8%BE%93%E5%85%A5%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20104e97288bd548d9b8676bd4da11444b/swiftui-texteditor-3.png)

图 32.3. TextField 视图支持多行输入

通过配对 `lineLimit` 修饰器，你可以修改文字栏的初始大小。 比方说，如果要显示一个三行文字，可以附加 `lineLimit` 修饰器并将值设置为 `3`：

```
TextField("Comment", text: $comment, prompt: Text("Please input your comment"), axis: .vertical)
    .lineLimit(3)

```

你也可以提供行限制的范围。 以下是一个例子：

```
TextField("Comment", text: $comment, prompt: Text("Please input your comment"), axis: .vertical)
    .lineLimit(3...5)

```

在这种情况下，文字栏就不会扩展超过 5 行。

### 总结

自 SwiftUI 首次发布以来，`TextEditor` 一直是最受期待的 UI 组件之一。 你现在可以使用这个原生组件来处理多行输入。 随着 iOS 16 的发布，你还可以使用 `TextField` 来灵活控制文字输入的空间。 自动扩展功能使你可以轻松创建足够灵活的文本字段以接受单行或多行输入。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUITextEditor.zip](https://www.appcoda.com/resources/swiftui4/SwiftUITextEditor.zip)