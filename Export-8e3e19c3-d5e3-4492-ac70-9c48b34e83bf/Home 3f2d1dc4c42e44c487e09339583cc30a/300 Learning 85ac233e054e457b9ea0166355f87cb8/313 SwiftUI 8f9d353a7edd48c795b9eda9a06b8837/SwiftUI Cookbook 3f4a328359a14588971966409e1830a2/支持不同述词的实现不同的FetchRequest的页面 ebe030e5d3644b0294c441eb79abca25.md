# 支持不同述词的实现不同的FetchRequest的页面

<aside>
💡 创建View

- 通过Binding实现传入String，实现每次Binding修改，View自动更新
- 在init中实现FetchRequest属性赋值，每次View更新都会通过init重新生成FetchRequest
    - 通过Binding.wrappedValue获取传入数据
    - 创建Predicate
    - 创建FetchRequest
- 利用计算属性获取FetchRequest的wrappedValue，这就是搜索结果
</aside>

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

我们声明了一个名为 `fetchRequest` 的变量来存放读取请求，并声明另一个名为 `todoItems` 的变量来储存读取的结果。 

读取的结果实际上是从读取请求的 `wrappedValue` 属性中所获得的。

<aside>
💡 `@MainActor **public** **var** wrappedValue: FetchedResults<Result> { **get** }`

</aside>

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