# 如何使用 Core Data 建立 ToDo App | 精通 SwiftUI - iOS 16 版

iOS App 开发的一个常见问题是我们如何使用 Core Data 和 SwiftUI 将数据永久保存在内置数据库中。 在本章中，我们将通过构建一个 ToDo 应用程序来回答这个问题。

由于 ToDo 这个示例程序使用 List 和 Combine 来处理数据展示和共享，我假设你已经阅读了以下章节：

- 第 7 章 - 状态(State)与绑定(Binding)
- 第 10 章 - 动态列表、 ForEach 与 Identifiable 的使用方法
- 第 14 章 - 使用 Combine 与 Environment 物件进行数据分享

如果你还未阅读或忘记了Combine 和Environment Objects 是什么，请先阅读这些章节。

究竟你会将会做什么以理解 Core Data？ 我已经构建了示例App的核心部分，你并不需要从头开始构建 ToDo App。 但是，它不能永久保存数据。 更具体地说，它只能将待办事项保存在记忆体当中。 每当用户关闭App并再次启动它时，所有数据都会消失。 我们将修改这个示例 App 并将其转换为使用 Core Data 将数据永久保存到本地数据库。 图 22.1 显示了 ToDo App 的一些示例屏幕截图。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-1.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-1.png)

图 22.1. ToDo app

在我们开始修改之前，我会先介绍一下这个 Starter 项目，以便你完全理解当中的代码。

除了 Core Data，你还将会学习如何自定义切换的样式。 看看上面的截图，该核取方块实际上是 SwiftUI 的切换视图。 我将向你展示如何通过自定义 Toggle 的样式来建立这些核取方块。

本章有很多内容要讲，让我们开始吧！

### Core Data 介绍

在我们查看 ToDo App的Starter项目之前，让我向你简单介绍一下 Core Data 以及你将如何在 SwiftUI 项目中使用它。

### 什么是 Core Data?

首先，不要将 Core Data 框架与数据库混淆。 Core Data 不是数据库。 它只是一个供开发人员管理持久存储上的数据并与之交互的框架。 尽管 SQLite 数据库是 iOS 上 Core Data 的默认持久性储存器（ Persistent Storage ），但持久性储存器不限于数据库。 例如，你还可以利用 Core Data 来管理文件库文件（例如 XML）中的数据。

Core Data 框架只是将开发人员与永久存储的内部细节屏蔽开来。 以 SQLite 数据库为例， 你不需要知道如何连接到数据库，也不需要了解 SQL 来检索数据记录。 你只需要弄清楚如何使用Core Data API，例如 `NSManagedObjectContext` 和托管物件模型（Managed Object Model）。

这看起来有点复杂，是吧？不用担心。 在我们将 ToDo App从数组转换为 Core Data 之后，你就会明白我的意思。

### 使用 Core Data 模板

使用 Core Data 的最简单方式，就是建立一个新项目时启用 Core Data 选项 。你也可以试一试。 启动 Xcode 并使用 *App* 模板建立一个新项目。 将其命名为你喜欢的名称，但请确保选中 *Core Data* 选项。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-2.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-2.png)

图 22.2. 建立一个启用 Core Data 的新项目

通过使用 Core Data，Xcode 将为你生成所有必需的代码和托管物件模型。 建立项目后，你应该会看到一个名为“CoreDataTest.xcdatamodeld”的新文件。 在 Xcode 中，托管物件模型的副档名为“.xcdatamodeld”。 这是为你的项目生成的托管物件模型，也是你定义持久性物件的地方。

看一下 `Persistence.swift` 文件，这是 Xcode 生成的另一个文件。 该文件包含用于加载托管物件模型并将数据保存到持久性储存器的代码。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-3.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-3.png)

图 3. Core Data的附加代码

如果你之前使用过 UIKit 开发App，你通常会使用容器来管理数据库或其他持久性储存器中的数据。 在 SwiftUI 中，它有点不同。 我们很少直接使用这个容器。 相反，SwiftUI 将托管物件内容（Managed Object Context）注入到SwiftUI 环境中，以便任何视图都可以检索物件内容并管理数据。

现在开启 `CoreDataTestApp.swift` ， Xcode 添加了一个常数来保存 `PersistenceController` 。另外，它还加了一行代码将托管对像内容注入到环境中。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-4.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-4.png)

图 4. 将托管物件内容注入到环境中

以上所讲的，就是在启用 Core Data 选项时由 Xcode 生成的代码和文件。 如果打开`ContentView.swift`，Xcode 还会生成示例代码，用于从本地数据存储加载数据。 查看代码以了解其工作原理。 一般来说，要在本地数据库上保存和管理数据，步骤如下：

1. 利用模型编辑器（ Data Model Editor ）在数据模型 (i.e. `.xcdatamodeld`) 建立实体（ Entity ）。
2. 建立实体的相对应的托管物件（应继承自 `NSManagedObject`）。
3. 在需要保存和修改数据的视图中，使用 `@Environment` 从环境中获取托管物件内容，如下所示：   
    
    ```
    @Environment(\.managedObjectContext) var context
    
    ```
    
    然后建立托管物件并使用托管物件内容的`save`方法将物件添加到数据库中。 这是相关的代码：
    
    ```
    let task = ToDoItem(context: context)
    task.id = UUID()
    task.name = name
    task.priority = priority
    task.isComplete = isComplete
    
    ```
    
4. 至于数据检索，Apple 引入了一个名为`@FetchRequest`的属性包装器，供你从持久存储器获取数据。 以下是相关的代码：  
    
    ```
    @FetchRequest(
       entity: ToDoItem.entity(),
       sortDescriptors: [ NSSortDescriptor(keyPath: \ToDoItem.priorityNum, ascending: false) ])
    var todoItems: FetchedResults<ToDoItem>
    
    ```
    
    此属性包装器使运行读取请求变得非常简单。 你只需要指定要检索的实体物件以及数据的排序方式， 然后框架将使用环境的托管物件内容来获取数据。 最重要的是，取得数据之后，SwiftUI 会自动修改所有相关的视图。
    

这就是你在 SwiftUI 项目中使用 Core Data 的方式。 我知道你可能对某些程序感到困惑， 本节只是一个非常初步的简介。 稍后，当你修改示例程序时，我们将更详细解释这些程序。

### 了解 ToDo App 示例运作

现在你对 Core Data 有了基本的了解，让我和你一起完成这个示例App。 稍后，我们将修改此 ToDo App，让它能永久保存待办事项。 现在，如前所述，所有数据都存储在记忆体当中，并且会在App重新启动时消失。

首先，请从 [https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIToDoListStarter.zip) 下载启动项目。 解压文件并在 Xcode 中打开 `ToDoList.xcodeproj`。 选择 `ContentView.swift` 并预览 UI。 你应该会看到如图 22.5 所示的屏幕。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-5.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-5.png)

图 22.5. 预览示例App

在预览框或模拟器中运行App。 按 + 并添加待办事项。 之后，再重复添加更多项目。 然后App就会列出所有待办事项。 在待办事项的格子按一下，App就会划掉该项目。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-6.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-6.png)

图 22.6. 添加新事项

### 如何显示待办事项列表

先让我们看一下代码，以便你了解这些代码背后的运作原理。 首先，我们从模型类开始。 在 *Model* 文件夹中打开 `ToDoItem.swift`。

```
enum Priority: Int {
    case low = 0
    case normal = 1
    case high = 2
}

class ToDoItem: ObservableObject, Identifiable {
    var id = UUID()
    @Published var name: String = ""
    @Published var priority: Priority = .normal
    @Published var isComplete: Bool = false

    init(name: String, priority: Priority = .normal, isComplete: Bool = false) {
        self.name = name
        self.priority = priority
        self.isComplete = isComplete
    }
}

```

这个示例App是一个普通 ToDo App的简化版本。 每个待办事项（或任务）具有三个属性：*name*、*priority* 和 *isComplete*（即事项的状态）。 这个类别采用了`ObservableObject` 协议。 这三个属性都用`@Published` 标记，当其中的值发生任何修改时就会自动通知订阅者。 稍后，在 ContentView 的实现中，SwiftUI 会不时观察数值的变化并相应地修改视图。 例如，当 `isComplete` 的值发生变化时，它会切换核取方块。

这个类别也符合`Identifiable` 协议，这样`ToDoItem` 的每个实体（instance）都有一个独一无二的识别码。 稍后，我们将使用 `ForEach` 和 `List` 来显示待办事项。 这就是为什么我们需要采用协议并创建 `id` 属性。

现在让我们转到视图并从`ContentView.swift` 开始。 假设你已经阅读了第 10 章，你应该理解大部分代码。 内容视图包含三个主要部分，它们嵌入在一个“ZStack”中：

1. 显示所有待办事项的列表视图。
2. 没有待办事项时显示的空视图 (NoDataView)。
3. 当用户点击 + 按钮时显示的“添加新事项”视图。

先看第一个`VStack`：

```
VStack {

    HStack {
        Text("ToDo List")
            .font(.system(size: 40, weight: .black, design: .rounded))

        Spacer()

        Button(action: {
            self.showNewTask = true

        }) {
            Image(systemName: "plus.circle.fill")
                .font(.largeTitle)
                .foregroundColor(.purple)
        }
    }
    .padding()

    List {

        ForEach(todoItems) { todoItem in
            ToDoListRow(todoItem: todoItem)
        }

    }
}
.rotation3DEffect(Angle(degrees: showNewTask ? 5 : 0), axis: (x: 1, y: 0, z: 0))
.offset(y: showNewTask ? -50 : 0)
.animation(.easeOut)

```

我加了一个名为 `todoItems` 的变量来保存所有的待办事项。 这个变量标有`@State`，以便在有任何修改时修改列表。 在 List 视图中，我们使用 ForEach 以显示所有项目。

另外，我们有一个名为`ToDoListRow`的独立视图以处理列表的行：

```
struct ToDoListRow: View {

    @ObservedObject var todoItem: ToDoItem

    var body: some View {
        Toggle(isOn: self.$todoItem.isComplete) {
            HStack {
                Text(self.todoItem.name)
                    .strikethrough(self.todoItem.isComplete, color: .black)
                    .bold()
                    .animation(.default)

                Spacer()

                Circle()
                    .frame(width: 10, height: 10)
                    .foregroundColor(self.color(for: self.todoItem.priority))
            }
        }.toggleStyle(CheckboxStyle())
    }

    private func color(for priority: Priority) -> Color {
        switch priority {
        case .high: return .red
        case .normal: return .orange
        case .low: return .green
        }
    }
}

```

这个视图接受一个待办事项，它是一个`ObservableObject`。 这意味着该待办事项有任何修改时，相关的视图将自动修改UI。

对于每一行的待办事项，由三部分组成：

1. 切换（Toggle）/ 核取方块（Checkbox） - 提示事项是否完成。
2. 文字标签 - 显示事项名称
3. 点/圆圈 - 显示事项的优先次序

第二和第三部分非常简单。 至于核取方块，这个比较值得更深入的讨论。 SwiftUI 提供有一个名为`Toggle`的标准组件。 在前面的章节中，我们用它来建立一个App设定画要。 这里的切换（Toggle）更像是一个开关，可让你打开或关闭，但在 ToDo App中，我们想让切换看起来像一个核取方块。

### 自订 Toggle 的外观

与我们在第 6 章中讨论的 `Button` 类似，`Toggle` 也允许开发人员自定义其样式。 你需要做的就是实现`ToggleStyle`协议。 在项目导航器中，打开 `CheckBoxStyle.swift` 查看：

```
struct CheckboxStyle: ToggleStyle {

    func makeBody(configuration: Self.Configuration) -> some View {

        return HStack {

            Image(systemName: configuration.isOn ? "checkmark.circle.fill" : "circle")
                .resizable()
                .frame(width: 24, height: 24)
                .foregroundColor(configuration.isOn ? .purple : .gray)
                .font(.system(size: 20, weight: .bold, design: .default))
                .onTapGesture {
                    configuration.isOn.toggle()
                }

            configuration.label

        }

    }
}

```

在代码中，我们实现了`makeBody`方法，这是协议的要求。 我们加入了一个图像视图，取决于切换的状态（即`configuration.isOn`），它将显示一个选框图像或一个圆形图像。 这就是如何自定义切换样式的方法。

要使用 `CheckboxStyle`，请将 `toggleStyle` 修饰器附加到 `Toggle` 并指定核取方块样式，如下所示：

```
.toggleStyle(CheckboxStyle())

```

### 处理空列表视图

当数组中没有项目时，我们显示一个图像视图而不是一个空的列表视图。 这个并不是必须做，但是，我认为它使App看起来更好，并让使用者知道在App首次启动时该做什么。

```
// 如果没有数据，就显示一个空视图
if todoItems.count == 0 {
    NoDataView()
}

```

由于我们有一个 `ZStack` 来嵌入视图，所以很容易控制这个空视图何时显示。那就是当数组为空时显示出来。

### 显示添加事项视图

当使用者点击右上角的 + 按钮时，App 会显示“NewToDoView”。 此视图覆盖在列表视图的顶部，看起来像一个由画面底弹出来的页面。 我们还添加了一个空白视图来使列表视图变暗。

下面是代码供参考：

```
if showNewTask {
    BlankView(bgColor: .black)
        .opacity(0.5)
        .onTapGesture {
            self.showNewTask = false
        }

    NewToDoView(isShow: $showNewTask, todoItems: $todoItems, name: "", priority: .normal)
        .transition(.move(edge: .bottom))
        .animation(.interpolatingSpring(stiffness: 200.0, damping: 25.0, initialVelocity: 10.0))
}

```

### 了解添加事项视图

现在让我和你讨论一下 `NewToDoView.swift` 中的代码，该代码用于让使用者添加新任务或待办事项。 你可以参考图 22.6 或简单地打开文件进行预览，看看该视图的外观。

`NewToDoView` 接受两个绑定：*isShow* 和 *todoItems*。 `isShow` 参数控制这个 *Add New Task* 视图是否应该出现在屏幕上。 `todoItems` 变量储存了对待办事项数组的参考。 我们需要方法调用者将绑定传递给 `todoItems`，以便我们可以将新事项加进数组。

```
@Binding var isShow: Bool
@Binding var todoItems: [ToDoItem]

@State var name: String
@State var priority: Priority
@State var isEditing = false

```

在视图中，我们让用户输入事项名称并设置其优先次序（低/正常/高）。 状态变量`isEditing`标示是否进入编辑模式。 为避免软体键盘遮挡编辑画面，App将在使用者编辑文字时稍为向上移动视图。

```
TextField("Enter the task description", text: $name, onEditingChanged: { (editingChanged) in

    self.isEditing = editingChanged

})

...

.offset(y: isEditing ? -320 : 0)

```

点击 *Save* 按钮后，我们验证文字栏是否有输入文字。 如果使用者有输入文字，我们会建立一个新的 `ToDoItem` 并调用 `addTask` 方法将其附加到 `todoItems` 数组。相反，我们就什么都不做了。

```
Button(action: {

    if self.name.trimmingCharacters(in: .whitespaces) == "" {
        return
    }

    self.isShow = false
    self.addTask(name: self.name, priority: self.priority)

}) {
    Text("Save")
        .font(.system(.headline, design: .rounded))
        .frame(minWidth: 0, maxWidth: .infinity)
        .padding()
        .foregroundColor(.white)
        .background(Color.purple)
        .cornerRadius(10)
}
.padding(.bottom)

```

由于 `todoItems` 数组是一个状态变量，列表视图会自动修改并显示新事项。 这就是整个代码的运作原理。 如果你不明白 *Add task* 视图是如何显示在屏幕底部的话，请参阅第 18 章。

### 如何使用 Core Data

现在我已经为你介绍了这个ToDo项目，是时候将App转换为使用 Core Data 将待办事项存储在数据库中了。 一开始，我们曾建立了一个启用了 *Core Data* 的空白项目。 通过选中 Core Data 选框，Xcode 会自动生成 Core Data 项目的基本框架。 这一次，我将向你展示如何手动转换Xcode项目以使用 Core Data。

### 建立持久控制器 Persistent Controller

让我们首先建立一个名为`Persistence.swift`的新文件。 在项目导航器中，右键单击*Model*并使用 *Swift file* 模板。 将文件命名为`Persistence.swift`并在文件中插入以下代码：

```
import CoreData

struct PersistenceController {
    static let shared = PersistenceController()

    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "ToDoList")
        if inMemory {
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.

                /*
                Typical reasons for an error here include:
                * The parent directory does not exist, cannot be created, or disallows writing.
                * The persistent store is not accessible, due to permissions or data protection when the device is locked.
                * The device is out of space.
                * The store could not be migrated to the current model version.
                Check the error message to determine what the actual problem was.
                */
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
    }
}

```

以上的代码你应该很熟悉，因为它和Xcode生成的代码是一样的，只是容器的名字改成了*ToDo List*。

### 注入托管物件内容（Managed Object Context）

现在打开`ToDoListApp.swift` 并将托管物件内容注入到环境中。 在 `ToDoListApp` 中，建立以下变量以保存 `PersistenceController`：

```
let persistenceController = PersistenceController.shared

```

接下来，在同一个文件中，将 `environment` 修饰器附加到 `ContentView()`，如下所示：

```
ContentView()
    .environment(\.managedObjectContext, persistenceController.container.viewContext)

```

在上面的代码中，我们将托管物件内容注入到 `ContentView` 的环境中。 这使我们可以轻松地从内容视图中取得这个托管物件内容以管理数据库中的数据。

### 建立托管物件模型

接下来，我们需要手动建立托管物件模型。 在项目导航器中，右键单击 *ToDoList* 文件夹并选择 *New file...*。 选择 *Data Model* 并将文件命名为“ToDoList.xcdatamodeld”。 请确保你正确命名文件，因为它应该与`NSPersistentContainer`内的名称相同。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-7.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-7.png)

图 22.7. 选择数据模型模板

之后，选择模型文件并单击 *Add Entity* 以建立新实体。 将实体的名称从 *Entity* 修改为 *ToDoItem*。 你可以将此实体视为数据库表中的一条记录。 因此，该实体应存储“ToDoItem”的属性。 我们需要为实体添加 4 个属性，包括（见图 22.8）：

- *id* - 类型为 *UUID*
- *name* - 类型为 *String*
- *priorityNum* - 类型为 *Integer 32*
- *isComplete* - 类型为 *Boolean*

`id`、`name` 和 `isComplete` 的类型与 `ToDoItem` 类的类型完全相同。 但是为什么优先次序设置为 *Integer 32* 类型？ 如果你看一下 `ToDoItem.swift` 中的代码，你会看到 *priority* 属性是一个Enum：

```
enum Priority: Int {
    case low = 0
    case normal = 1
    case high = 2
}

```

要将这个Enum保存到数据库中，我们必须存储它的原始值，它是一个整数。 这就是我们使用 *Integer 32* 类型并将属性命名为 *priorityNum* 以避免命名冲突的原因。

在默认的情况下，Xcode 会自动生成这个 `ToDoItem` 实体的模型类。 但是，我更喜欢手动建立这个类型，以便有更好地控制。 因此，选择 `ToDoItem` 实体并打开 *Data Model Inspector*。 如果你看不到检查器，请转到菜单并选择 *View* > *Inspectors* > *Show Data Model Inspector*。 在 *Class* 部分，将 *Module* 设置为 *Current Product Module*，将 *Codegen* 设置为 *Manual/None*。 这将禁止Xcode 自动生成代码。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-8.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-8.png)

图 8. 禁用代码生成

如你所见，到目前为止我们开发的所有内容都不需要你具备数据库编程知识。 没有 SQL，没有数据库表。 你处理的所有事情都是基于对象的， 这就是 Core Data 的美妙之处。

### 定义模型类别

在 Core Data 中，每个实体都应该与一个模型类配对。 默认情况下，这个模型类是由 Xcode 生成的。 之前，我们把设置从 *code gen* 修改为 *manual*。 所以，我们需要手动建立模型类型`ToDoItem`。 切换到 `ToDoItem.swift` 并导入 *CoreData* 包：

```
import CoreData

```

像这样建立`ToDoItem`类别：

```
public class ToDoItem: NSManagedObject {
    @NSManaged public var id: UUID
    @NSManaged public var name: String
    @NSManaged public var priorityNum: Int32
    @NSManaged public var isComplete: Bool
}

extension ToDoItem: Identifiable {

    var priority: Priority {
        get {
            return Priority(rawValue: Int(priorityNum)) ?? .normal
        }

        set {
            self.priorityNum = Int32(newValue.rawValue)
        }
    }
}

```

Core Data 的模型类别应该继承自`NSManagedObject`。 每个属性都用`@NSManaged` 注释，并对应于我们之前建立的 Core Data 模型的属性。 通过使用`@NSManaged`，这告诉编译器该属性是由Core Data 处理。

在 `ToDoItem` 的原始版本中，我们有一个类型为 Enum 的 `priority` 属性。 对于 Core Data 版本，我们必须为`priority`建立一个计算属性（Computed Property）。 此计算属性将优先次序的编号转换为 Enum，反之亦然。

### 利用 @FetchRequest 取得数据

现在我们已经准备好了模型类别，让我们看看从数据库中获取记录是多么容易。 切换到`ContentView.swift`。 最初，我们有一个包含所有待办事项的数组变量，它也标有`@State`：

```
@State var todoItems: [ToDoItem] = []

```

由于我们要在数据库中存储项目，我们需要修改这行代码并从中获取数据。 Apple 引入了一个名为 `@FetchRequest` 的新属性包装器，让开发者更容易从数据库拿取数据。

将上面的代码改为 `@FetchRequest`，如下所示：

```
@FetchRequest(
    entity: ToDoItem.entity(),
    sortDescriptors: [ NSSortDescriptor(keyPath: \ToDoItem.priorityNum, ascending: false) ])
var todoItems: FetchedResults<ToDoItem>

```

回想一下，我们已经在环境中注入了托管物件内容，这个读取请求会自动利用托管物件内容以获取所需的数据。 在上面的代码中，我们指定获取 `ToDoItem` 实体以及结果的排序方式。 另外，我们想根据优先次序对各事项进行排序。

获取完成后，你将拥有一组 `ToDoItem` 托管物件，这些物件是建基于我们之前在模型层中定义的 `ToDoItem` 类别。

这就是从数据库中检索数据的方式。 而且，由于 `ToDoItem` 的属性保持不变，我们不需要为列表视图做任何代码修改，就可以直接在 ForEach 中使用所得的数据：

```
List {

    ForEach(todoItems) { todoItem in
        ToDoListRow(todoItem: todoItem)
    }

}

```

最重要的是，你可以直接传递 `todoItem`，因为它就是一个 `NSManageObject`。看一看`NSManagedObject` 的技术文， 它符合`ObservableObject`。 这就是为什么我们可以直接将 `todoItem` 传递给 `ToDoListRow`。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-9.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-9.png)

Figure 9. NSManagedObject documentation

还有一件事。 你可能还想知道我们是否需要在 `todoItems` 发生修改时手动运行 fetch 请求（例如，我们添加了一个新项目）。 这是使用`@FetchRequest` 的另一个优点。 SwiftUI 会自动管理其变动并相应地修改 UI。

### 将数据添加到持久存储

现在，让我们继续进行 Core Data 迁移并修改 `NewToDoView.swift` 的代码。 要在数据库中存新事项，首先需要从环境中获取托管物件内容：

```
@Environment(\.managedObjectContext) var context

```

由于我们不再使用数组来保存待办事项，你可以删除这行代码：

```
@Binding var todoItems: [ToDoItem]

```

接下来，让我们像这样修改 `addTask` 函数：

```
private func addTask(name: String, priority: Priority, isComplete: Bool = false) {

    let task = ToDoItem(context: context)
    task.id = UUID()
    task.name = name
    task.priority = priority
    task.isComplete = isComplete

    do {
        try context.save()
    } catch {
        print(error)
    }
}

```

要将新记录加入数据库，你需要使用托管物件内容建立一个 `ToDoItem`，然后调用 `save()` 方法来储存修改。

由于我们移除了 `todoItems` 绑定，我们需要修改预览代码：

```
struct NewToDoView_Previews: PreviewProvider {
    static var previews: some View {
        NewToDoView(isShow: .constant(true), name: "", priority: .normal)
    }
}

```

现在让我们回到`ContentView.swift`。 同样地，你应该会在`ContentView`中看到一个错误（见图 22.10）。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-10.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-10.png)

图 22.10. Xcode 向你显示 ContentView 中的错误

像这样修改代码行以修复错误：

```
NewToDoView(isShow: $showNewTask, name: "", priority: .normal)

```

我们只需删除 `todoItems` 参数。 这就是我们将示例App从使用数组存储转换为持久存储的方式。

### 修改现有项目

当你将项目标记为完成时，App应将这个修改储存在数据库中。 在 `ContentView.swift` ，找到 `ToDoListRow` 结构并加入以下变量：

```
@Environment(\.managedObjectContext) var context

```

和添加新记录类似，我们也需要先取得管理物件内容以进行修改。 在 `Toggle` 视图，加入 `onReceive` 修饰器并将其放在 `.toggleStyle(CheckboxStyle())` 之后，如下所示：

```
var body: some View {
    Toggle(isOn: self.$todoItem.isComplete) {
       .
       .
       .
    }
    .toggleStyle(CheckboxStyle())
    // 加入以下代码
    .onChange(of: todoItem, perform: { _ in
        if self.context.hasChanges {
            try? self.context.save()
        }
    })
}

```

每当切换发生修改时，`todoItem` 的 `isComplete` 属性都会修改。 但是，我们如何将它保存到数据库呢？ 回想一下 `todoItem` 已符合 `ObservableObject`，这意味着它有一个发布者来广播值的变化。

`onChange` 修饰器则监听这些变化（比如`isComplete` 的变化），并通过调用`context`的 `save()` 方法将它们保存到数据库中。

现在你可以在模拟器中运行App进行测试， 你应该能够添加新事项。 添加新事项后，它们应立即显示在列表视图中。 核取方块也应该可以用了，点一下就可以将相关事项画线。 最重要的是，所有改动现在都永久保存在装置内的数据库。 就算重新App，所有项目仍然存在。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-11.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-11.png)

图 22.11. ToDo App 现在支持 Core Data

### 从数据库中删除项目

既然我已经向你展示如何从数据库取出数据、修改和加入新项目，那么如何实现删除功能呢？ 我们将会为App添加一项功能 - 删除待办事项。

在 `ContentView` 结构中，声明一个 `context` 变量：

```
@Environment(\.managedObjectContext) var context

```

然后添加一个名为`deleteTask`的函数，如下所示：

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

此函数接收一个索引集，该集储存要删除项目的索引。 要从数据库中删除项目，你可以调用`context`的`delete` 方法并指定要删除的项目。 最后，调用 `save()` 来运行修改。

既然我们已经准备好了删除方法，那么我们应该在哪里调用它呢？ 将 `onDelete` 修饰器附加到列表视图的 `ForEach`，如下所示：

```
List {

    ForEach(todoItems) { todoItem in
        ToDoListRow(todoItem: todoItem)
    }
    .onDelete(perform: deleteTask)

}

```

`onDelete` 修饰器自动启用列表视图中的滑动删除功能。 当用户删除一个项目时，我们调用`deleteTask` 方法从数据库中删除该项目。

运行App并试试删除一个项目，按delete钮就可以将其从数据库中完全删除。

![%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-12.png](%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Core%20Data%20%E5%BB%BA%E7%AB%8B%20ToDo%20App%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%206b9d58bc4a774554941ad3d2f8590117/swiftui-core-data-12.png)

图 22.12. 删除一个项目

### 使用 SwiftUI 预览

不知你有没有留意，现在Xcode内的SwiftUI预览已经不能正常显示预览。 这是可以理解的，因为我们没有在 `ContentView_Previews` 结构中注入托管物件内容（managed object context）。 那么，我们要如何解决问题并使预览正常运作？

首先，我们需要建立一个数据暂存器（in-memory data store）并加入一些测试数据。 打开`Persistence.swift`并声明一个静态变量，如下所示：

```
static var preview: PersistenceController = {
    let result = PersistenceController(inMemory: true)
    let viewContext = result.container.viewContext

    for index in 0..<10 {
        let newItem = ToDoItem(context: viewContext)
        newItem.id = UUID()
        newItem.name = "To do item #\(index)"
        newItem.priority = .normal
        newItem.isComplete = false
    }

    do {
        try viewContext.save()
    } catch {
        let nsError = error as NSError
        fatalError("Unresolved error \(nsError), \(nsError.userInfo)")
    }

    return result
}()

```

在以上的代码，我们建立了一个 `PersistenceController` 的实体，并将 `inMemory` 参数设置为 `true`。 然后，我们添加 10 个测试待办事项并将它们保存到数据暂存器。

现在让我们切换到 `ContentView.swift` 并像这样修改预览代码：

```
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView().environment(\.managedObjectContext, PersistenceController.preview.container.viewContext)
    }
}

```

我们将数据暂存器的`context`注入到内容视图的环境中。 通过这样做，内容视图现在可以加载示例待办事项并将显示在预览画面中。

### 总结

在本章中，我们将 Todo 示例App加入永久储存数据功能。 我希望你现在了解如何在 SwiftUI 开发中使用 Core Data 并知道如何运行所有基本的 CRUD（建立、读取、修改和删除）操作。SwiftUI框架新加入的 `@FetchRequest` 属性包装器使得在数据库管理变得非常容易。

本章所讲解的示例以及最后完整的Xcode 项目：

[https://www.appcoda.com/resources/swiftui4/SwiftUIToDoList.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIToDoList.zip)

请自行下载以供你参考。