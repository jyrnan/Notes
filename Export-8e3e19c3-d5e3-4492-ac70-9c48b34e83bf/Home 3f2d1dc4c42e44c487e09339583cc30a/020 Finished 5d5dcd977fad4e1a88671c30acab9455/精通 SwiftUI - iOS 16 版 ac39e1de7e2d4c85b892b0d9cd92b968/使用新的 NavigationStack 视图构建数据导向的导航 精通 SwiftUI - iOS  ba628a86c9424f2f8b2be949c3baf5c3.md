# 使用新的 NavigationStack 视图构建数据导向的导航 | 精通 SwiftUI - iOS 16 版

在 iOS 开发中，导航视图 (Navigation View) 绝对是我们最常用的组件。在 SwiftUI 刚推出的时侯，就已经有一个 `NavigationView` 视图，让开发者可以构建基于导航的使用者界面。随着 iOS 16 的发布，Apple 弃用了旧的导航视图，并引入了一个新视图 `NavigationStack` 来呈现堆叠视图。更重要的是，开发者现在可以利用这个新视图来构建数据导向 (data driven) 的导航。

### 旧的 Navigation Views 的操作

在 iOS 16 之前，我们会用 `NavigationView` 和 `NavigationLink` 如此构建导航介面：

```
NavigationView {
    NavigationLink {
        Text("Destination")
    } label: {
        Text("Tap me")
    }
}

```

以上代码会构建出一个基本的导航介面，当中有一个 *Tap me* 按钮。当我们点击按钮，App 就会导航到下一级，来显示目标视图。

![%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-1.png](%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-1.png)

图 46.1. 基本的导航介面

### 使用 NavigationStack

从 iOS 16 开始，我们可以用新的 `NavigationStack` 来取代 `NavigationView`。我们可以完全保留 `NavigationLink`，都会得到相同的结果。

```
NavigationStack {
    NavigationLink {
        Text("Destination")
    } label: {
        Text("Tap me")
    }
}

```

我们也可以这样编写代码：

```
NavigationStack {
    NavigationLink("Tap me") {
        Text("Destination")
    }
}

```

要显示一个数据项目的列表，通常我们会使用导航视图来构建一个 master-detail flow。来看看以下的示例：

```
struct ContentView: View {
    private var bgColors: [Color] = [ .indigo, .yellow, .green, .orange, .brown ]

    var body: some View {

        NavigationStack {
            List(bgColors, id: \.self) { bgColor in
                NavigationLink {
                    bgColor
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } label: {
                    Text(bgColor.description)
                }

            }
            .listStyle(.plain)

            .navigationTitle("Color")
        }

    }
}

```

以上的代码会建立一个导航视图，来显示构建一个 master-detail 流程。当使用者选择了一个项目，App 就会导航到 detail 视图，并显示 color 视图。

![%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-2.png](%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-2.png)

图 46.2. 一个 master-detail 导航视图

### 建基于数值的 Navigation Links

`NavigationStack` 引入了一个新修饰符 `navigationDestination`，用来把目标视图与呈现的数据类型连系。我们可以这样重新编写上一节的代码：

```
NavigationStack {
    List(bgColors, id: \.self) { bgColor in

        NavigationLink(value: bgColor) {
            Text(bgColor.description)
        }

    }
    .listStyle(.plain)

    .navigationDestination(for: Color.self) { color in
        color
            .frame(maxWidth: .infinity, maxHeight: .infinity)
    }

    .navigationTitle("Color")
}

```

我们还是会使用 `NavigationLinks` 来显示数据列表，并实现导航功能；不同之处是每个 `NavigationLink` 都连系着一个数值。更重要的是，我们添加了新的 `navigationDestination` 修饰符来捕捉数值的变化。当使用者选择特定的 link 时，`navigationDestination` 修饰符就会显示相应带有 `Color` 类型数据的目标视图。

如果我们在预览中测试这个 App，你会发现它的操作方式与之前完全相同。但是，内部的实现已经使用了新的 `navigationDestination` 修饰符。

### 多个 Navigation Destination 修饰器

我们可以定义多于一个 `navigationDestination` 修饰符，来处理不同类型的 Navigation Link。在之前的示例中，我们只有一个 `navigationDestination` 修饰符来处理 `Color` 类型。现在，让我们为 `String` 类型设置另一组 Navigation Link：

```
List(systemImages, id: \.self) { systemImage in

    NavigationLink(value: systemImage) {
        Text(systemImage.description)
    }

}
.listStyle(.plain)

```

`systemImages` 变量会储存系统图像名称的数组 (array)。

```
private var systemImages: [String] = [ "trash", "cloud", "bolt" ]

```

在这个示例中，我们有两个类型的 Navigation Link，一个是 `Color`，另一个是 `String` 类型。让我们把另一个 `navigationDestination` 修饰器嵌入到堆叠中，来处理 `String` 类型的导航：

```
.navigationDestination(for: String.self) { systemImage in
    Image(systemName: systemImage)
        .font(.system(size: 100.0))
}

```

现在，如果使用者点击其中一个系统图像名称，就会导航到另一个视图，来显示系统图像。

![%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-3.png](%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-3.png)

图 46.3. Navigation Destination 修饰器

### 了解 Navigation 状态

与旧的 `NavigationView` 不同，新的 `NavigationStack` 让我们可以轻松地追踪导航的状态。`NavigationStack` 视图有另一个 initialization 方法，它有一个 `path` 参数 (parameter)，就是堆叠导航状态的 binding：

```
init(
    path: Binding<Data>,
    root: () -> Root
) where Data : MutableCollection, Data : RandomAccessCollection, Data : RangeReplaceableCollection, Data.Element : Hashable

```

如果我们想储存或管理导航状态，可以创建一个状态变量。以下是示例代码：

```
struct ContentView: View {
    private var bgColors: [Color] = [ .indigo, .yellow, .green, .orange, .brown ]

    @State private var path: [Color] = []

    var body: some View {

        NavigationStack(path: $path) {
            List(bgColors, id: \.self) { bgColor in

                NavigationLink(value: bgColor) {
                    Text(bgColor.description)
                }

            }
            .listStyle(.plain)

            .navigationDestination(for: Color.self) { color in
                VStack {
                    Text("\(path.count), \(path.description)")
                        .font(.headline)

                    HStack {
                        ForEach(path, id: \.self) { color in
                            color
                                .frame(maxWidth: .infinity, maxHeight: .infinity)
                        }

                    }

                    List(bgColors, id: \.self) { bgColor in

                        NavigationLink(value: bgColor) {
                            Text(bgColor.description)
                        }

                    }
                    .listStyle(.plain)

                }
            }

            .navigationTitle("Color")

        }

    }
}

```

这段代码与之前的示例有点相似，我们添加了一个 `path` 状态变量，它是 `Color` 的数组，用来储存导航状态。在 `NavigationStack` 进行 initialization 时，我们会传递它的 binding 来管理堆叠。当导航堆叠的状态发生变化时，`path` 变量的数值就会被自动修改。

我修改了导航目的地，它会显示使用者选择的颜色，和另一个颜色列表以供使用者选择。

![%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-4.png](%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-4.png)

图 46.4. 使用 path 变量

在上面的代码中，我们有一行代码来显示 path 的内容：

```
Text("\(path.count), \(path.description)")

```

`count` 属性 (property) 是指堆叠的 level，而 description 就是当前的颜色。举个例子，我们首先选择颜色 *indigo*，然后选择 *yellow*。在这个情况下，`count` 的数值就应该是 2*，*代表导航堆叠有 2 个 level。

我们可以利用这个 `path` 变量，来以编程方式控制堆叠的导航。举个例子，我们可以添加一个按钮，让使用者直接跳转到堆叠的 root level，以下是示例代码：

```
Button {
    path = .init()
} label: {
    Text("Back to Main")
}
.buttonStyle(.borderedProminent)
.controlSize(.large)

```

我们可以重置 `path` 变量的数值，来指示导航堆叠返回 root level。

大家可能已经知道，我们可以操纵 `path` 变量的数值，来控制导航堆叠的状态。举个例子，我们可以向 `path` 变量添加三种颜色，如此一来，当 `ContentView` 出现时，App 就会自动导航 3 个 level：

```
NavigationStack(path: $path) {
  .
  .
  .
}
.onAppear {
    path.append(.indigo)
    path.append(.yellow)
    path.append(.green)
}

```

你可以试着运行 App，就会看到 App 自动导航了 3 层。如此一来，我们就可以以编程方式控制导航状态，并处理 deep linking。

![%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-5.png](%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84%20NavigationStack%20%E8%A7%86%E5%9B%BE%E6%9E%84%E5%BB%BA%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%90%91%E7%9A%84%E5%AF%BC%E8%88%AA%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%20%20ba628a86c9424f2f8b2be949c3baf5c3/swiftui-navigationstack-5.png)

图 46.5. App 自动导航了 3 层

### 总结

iOS 16 推出的新 `NavigationStack`，可以让开发者轻松地构建数据导向的导航 UI。如果你的 App 不需要支持旧版本的 iOS，就可以利用这个新组件，来处理 deep linking 和复杂的 user flow。