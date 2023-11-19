# 利用Group来对同一个View实现相同的Modifier

### 群组视图内容

### 群组视图内容

如果你再次阅读一下代码，所有的 `CardViews` 是以 `.frame` 修饰器来限制其宽度为 300 点，是否有其他简化的方式，并移除重复的代码呢？SwiftUI 框架提供了开发者群组视图（`Group` view）的功能，可以将相关内容群组起来。更重要的是，你可以将修饰器加至群组，所有嵌入群组内的视图皆能够同步做产生效果。

举例而言，你可以将 `HStack` 内的程序重写如下来完成同样的结果：

```swift
HStack {
    Group {
        CardView(image: "swiftui-button", category: "SwiftUI", heading: "Drawing a Border with Rounded Corners", author: "Simon Ng")
        CardView(image: "macos-programming", category: "macOS", heading: "Building a Simple Editing App", author: "Gabriel Theodoropoulos")
        CardView(image: "flutter-app", category: "Flutter", heading: "Building a Complex Layout with Flutter", author: "Lawrence Tan")
        CardView(image: "natural-language-api", category: "iOS", heading: "What's New in Natural Language API", author: "Sai Kambampati")
    }
    .frame(width: 300)
}

```