# 在SwiftUI预览中显示Core Data数据

### 方法一  通过直接修改PersistenceController的类型全局变量

不知你有没有留意，现在Xcode内的SwiftUI预览已经不能正常显示预览。 这是可以理解的，因为我们没有在 `ContentView_Previews` 结构中注入托管物件内容（managed object context）。 那么，我们要如何解决问题并使预览正常运作？

首先，我们需要建立一个数据暂存器（in-memory data store）并加入一些测试数据。 打开`Persistence.swift`并声明一个静态变量，如下所示：

```swift
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

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView().environment(\.managedObjectContext, PersistenceController.preview.container.viewContext)
    }
}

```

我们将数据暂存器的`context`注入到内容视图的环境中。 通过这样做，内容视图现在可以加载示例待办事项并将显示在预览画面中。

### 方法二 在Preview视图处添加

即时预览功能是 SwiftUI 其中一个最受欢迎的功能。 但是，如果要与 Core Data 整合，则需要一些额外的代码才能使预览成功运作。 这是预览的代码：

```swift
struct PaymentFormView_Previews: PreviewProvider {
    static var previews: some View {

        let context = PersistenceController.shared.container.viewContext

        let testTrans = PaymentActivity(context: context)
        testTrans.paymentId = UUID()
        testTrans.name = ""
        testTrans.amount = 0.0
        testTrans.date = .today
        testTrans.type = .expense

        return Group {
            PaymentFormView(payment: testTrans)
            PaymentFormView(payment: testTrans)
                .preferredColorScheme(.dark)
                .previewDisplayName("Payment Form View (Dark)")

            FormTextField(name: "NAME", placeHolder: "Enter your payment", value: .constant("")).previewLayout(.sizeThatFits)
                .previewDisplayName("Form Text Field")

            ValidationErrorText(text: "Please enter the payment name").previewLayout(.sizeThatFits)
                .previewDisplayName("Validation Error")

        }
    }
}

```

要使用`PaymentFormView`，您必须提供一个`PaymentActivity`物件。 由于`PaymentActivity`是一个托管物件（managed object），我们需要从`PersistenceController`中读取`context`来创建一个。 一旦我们创建了 `PaymentActivity` 物件，就可以用它来预览 `PaymentFormView`。