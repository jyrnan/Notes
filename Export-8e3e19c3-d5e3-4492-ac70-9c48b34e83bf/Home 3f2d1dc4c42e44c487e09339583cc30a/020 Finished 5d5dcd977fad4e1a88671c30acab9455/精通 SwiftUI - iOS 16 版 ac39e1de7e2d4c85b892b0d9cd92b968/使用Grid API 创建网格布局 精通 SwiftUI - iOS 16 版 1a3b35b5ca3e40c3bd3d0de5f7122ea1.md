# 使用Grid API 创建网格布局 | 精通 SwiftUI - iOS 16 版

SwiftUI 4.0 引入了一个新的 `Grid` API 来组成基于网格的布局。 你可以使用 `VStack` 和 `HStack` 安排相同的布局。 然而，`Grid` 视图使其变得容易得多。 `Grid` 视图是一种容器视图，它以二维布局排列其他视图。

### 基本用法

让我们从一个简单的网格开始。 要创建一个 2x2 网格，你可以编写如下代码：

```
struct ContentView: View {
    var body: some View {
        Grid {
            GridRow {
                Color.purple
                Color.orange
            }

            GridRow {
                Color.green
                Color.yellow
            }
        }
    }
}

```

假设你已经在 Xcode 中创建了一个 SwiftUI 项目，你应该会看到一个 2x2 的网格，其中填上了不同的颜色。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-1.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-1.png)

图 44.1. 一个简单的网格（Grid）

要创建 3x3 网格，你只需添加另一个 `GridRow`。 并且，对于每个`GridRow`，再插入一个子视图。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-2.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-2.png)

图 44.2. 一个 3x3 的网格

### 比较网格视图和堆栈视图

你可能想知道为什么我们需要使用 `Grid` API。 我们其实可以使用 `HStack` 和 `VStack`创建相同的网格 UI。 那么，苹果为什么要引入这个新的 API？ 对！你可以使用堆栈视图构建相同的网格。 然而，有一个主要的区别使得 `Grid` 视图在构建网格布局时更受欢迎。

让我们使用下面的代码创建另一个 2x2 网格：

```
struct ContentView: View {
    var body: some View {
        Grid {
            GridRow {
                Image(systemName: "trash")
                    .font(.system(size: 100))
                Image(systemName: "trash")
                    .font(.system(size: 50))
            }

            GridRow {
                Color.green
                Color.yellow
            }
        }
    }
}

```

这次，第一行显示两个系统图像，第二行显示彩色视图。 您的预览应显示如下所示的网格。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-3.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-3.png)

图 44.3. 2x2 网格以显示图像和颜色视图

要使用嵌套的`VStack`和`HStack`构建相同的网格布局，我们可以编写如下代码：

```
VStack {
    HStack {
        Image(systemName: "trash")
            .font(.system(size: 100))
            .frame(minWidth: 0, maxWidth: .infinity)
        Image(systemName: "trash")
            .font(.system(size: 50))
            .frame(minWidth: 0, maxWidth: .infinity)
    }

    HStack {
        Color.green
        Color.yellow
    }
}

```

在默认的情况下，`Grid` 视图的每一列在一行中占据相等的空间。 但对于 `HStack`，我们必须附加 `frame` 修饰器以使每张图像在行中占据相同的空间。

`Grid` 视图只是让创建网格视图变得更容易，并提供了几个修改器来自定义网格布局。

### 使用 gridCellColumns 合并单元格

创建网格视图时，你可能希望在特定行中显示单个单元格，而其他行继续显示多个单元格。 `gridCellColumns` 修饰器专为你合并单元格而设计。 这是一个例子：

```
Grid {
    GridRow {
        Image(systemName: "trash")
            .font(.system(size: 100))
        Image(systemName: "trash")
            .font(.system(size: 50))
    }

    GridRow {
        Color.purple
            .overlay {
                Image(systemName: "magazine.fill")
                    .font(.system(size: 100))
                    .foregroundColor(.white)
            }
            .gridCellColumns(2)
    }
}

```

第二行只有一个单元格。 我们附加 `gridCellColumn` 修饰器并将其值设置为 `2` 以合并单元格。 如果你不使用修饰器，就会看到一个空白单元格。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-4.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-4.png)

图 44.4. 合并单元格

### 添加空白单元格

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-5.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-5.png)

图 44.5. 3x3 网格视图

现在让我们像这样创建另一个 3x3 网格视图：

```
struct ContentView: View {
    var body: some View {
        Grid {
            GridRow {
                IconView(name: "cloud")
                IconView(name: "cloud")
                IconView(name: "cloud")
            }

            GridRow {
                IconView(name: "cloud")
                IconView(name: "cloud")
                IconView(name: "cloud")
            }

            GridRow {
                IconView(name: "cloud")
                IconView(name: "cloud")
                IconView(name: "cloud")
            }
        }
    }
}

struct IconView: View {
    var name: String = "trash"

    var body: some View {
        Image(systemName: name)
            .frame(width: 100, height: 100)
            .background(in: Rectangle())
            .backgroundStyle(.purple)
            .foregroundStyle(.white.shadow(.drop(radius: 1, y: 3.0)))
            .font(.system(size: 50))
    }
}

```

如果我们想为网格中心的单元格显示一个空白视图怎么办？ 为此，你可以使用以下代码添加一个空白单元格：

```
Color.clear
    .gridCellUnsizedAxes([.horizontal, .vertical])

```

如果用上面的代码替换中心单元格，你将在网格中心看到一个空白单元格。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-6.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-6.png)

图 44.6. 加入空白格

`gridCellUnsizedAxes` 修饰器可防止空白单元格占用的空间超过行或列中其他单元格所需的空间。 如果省略修饰器，就会出现如图 7 所示的网格布局。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-7.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-7.png)

图 44.7. 省略 gridCellUnsizedAxes 修饰器

### 调整单元格间距

要控制单元格之间的垂直和水平间距，你可以在建立 `Grid` 视图时使用 `horizontalSpacing` 和 `verticalSpacing` 参数：

```
Grid(horizontalSpacing: 0, verticalSpacing: 0) {
  .
  .
  .
}

```

如果需要在行之间添加一些间距，可以将 `padding` 修饰器附加到 `GridRow`。 图 8 显示了一个示例。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-8.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-8.png)

图 44.8. 调整间距

### 控制单元对齐

`Grid` 视图有一个名为 `alignment` 的参数，供你配置单元格的默认对齐方式。 例如，此处将对齐方式设置为 .bottom ：

```
Grid(alignment: .bottom, horizontalSpacing: 0, verticalSpacing: 0) {
  .
  .
  .
}

```

如果你修改了代码，黑框应与单元格底部就会对齐。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-9.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-9.png)

图 44.9. 设置对齐方式

要修改默认对齐设置，单元格本身可以附加 `gridCellAnchor` 修饰器来修改对齐方式。 图 44.10 显示了一个示例。

![%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-10.png](%E4%BD%BF%E7%94%A8Grid%20API%20%E5%88%9B%E5%BB%BA%E7%BD%91%E6%A0%BC%E5%B8%83%E5%B1%80%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%201a3b35b5ca3e40c3bd3d0de5f7122ea1/swiftui-grid-10.png)

图 44.10. 修改默认对齐设置

### 总结

`Grid` API 为开发人员提供了构建基于网格的布局的更多选项。 你可以使用`HStack`和`VStack`来构建类似的布局。 但是 `Grid` 视图可以为你节省大量代码并使你的代码更具可读性。 如果你的App只需要支持最新版本的 iOS，可以尝试使用 `Grid` API 来构建一些有趣的布局。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- [https://www.appcoda.com/resources/swiftui4/SwiftUIGrid.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIGrid.zip)