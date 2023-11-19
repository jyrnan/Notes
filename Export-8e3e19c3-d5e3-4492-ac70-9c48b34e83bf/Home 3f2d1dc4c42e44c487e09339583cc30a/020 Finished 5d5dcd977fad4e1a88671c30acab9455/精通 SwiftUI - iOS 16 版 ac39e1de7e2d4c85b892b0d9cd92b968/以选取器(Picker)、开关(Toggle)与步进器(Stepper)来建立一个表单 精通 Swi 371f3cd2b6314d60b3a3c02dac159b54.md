# 以选取器(Picker)、开关(Toggle)与步进器(Stepper)来建立一个表单 | 精通 SwiftUI - iOS 16 版

行动 App 运用表单与使用者互动，并从使用者请求所需的数据。每天使用 iPhone 时， 你很可能碰到行动表单。举例而言，行事历 App 可能显示表单，让你填写新行程的资讯， 或者购物 App 显示表单，要求你提供购物与付款资讯。作为一个使用者，我不否认我讨厌填写表单，但是作为开发者，这些表单可帮助我们与使用者互动，并请求资讯来完成某些操作。开发一个表单，绝对是你需要掌握的基本技能。

在 SwiftUI 框架中，有一个名为“Form”的特别UI 控制组件。使用这个新控制组件， 你可以轻松建立表单。我将教你如何使用 Form 组件来建立表单。在建立表单时，你也将学习如何使用常见的控制组件，例如：选择器（picker ）、切换（toggle ）与步进器（stepper ）。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-1.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-1.png)

图 13.1. 建立一个设定画面

那么，我们准备要做什么项目呢？以图 13.1 为例，我们将为前面章节所制作的餐厅 App 来建立其设定画面，该画面提供使用者设定“排序”与“筛选”的偏好选项。这种类型的画面在真实的项目中很常见，当你了解其如何运作，你将可在你的 App 项目中建立你自己的表单。

在本章中，我将着重在实现表单布局，你将了解如何使用“Form”组件来布局设定画面，我们也将实现选择器来选择“排序”的偏好，并建立一个切换与一个步进器，以表示“筛选”的偏好。当你了解如何布局表单后，在下一章中，我将教你如何依照使用者的偏好来修改清单，以使 App 的功能完善。你将学会如何储存使用者的偏好、分享视图间的数据，并以 `@EnvironmentObject` 来监控数据的修改。

### 准备初始项目

为了节省你从头再建一次餐厅清单的时间，我已经建好一个初始项目。首先，至 [https://www.appcoda.com/resources/swiftui4/SwiftUIFormStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIFormStarter.zip)下载初始项目，下载之后，以 Xcode 开启 `SwiftUIForm.xcodeproj` 档，在画布上预览 `ContentView.swift`，你将看到熟悉的 UI，除此之外，还加入更多的餐厅细节资讯，如图 13.2 所示。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-2.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-2.png)

图 13.2. 餐厅清单视图

现在，`Restaurant` 结构具有三个属性：“type”、“phone”与“priceLevel”。我觉得“type”与“phone”的意思本身很清楚了，不必另外说明，而“priceLevel”则是储存了范围为 1 至 5 的整数，以反映该餐厅的平均价位。restaurants 数组已预填一些范本数据，为了之后的测试，将一些范本餐厅的 `isFavorite` 与 `isCheckIn`设定为“true”，这就是为何你会在预览中看到一些打卡与最爱符号的原因。

### **建立表单**UI

如前所述，SwiftUI 提供一个名为“Form”的 UI 组件来建立表单 UI。输入数据时，它是一个用于存放及分组控制组件的容器（例如：切换）。与其向你解释用法，不如我们直接进入实现，在此过程中，你将了解如何使用该组件。

由于我们将建立一个单独的设定画面，因此为表单建立一个新文件。在项目导航器中，在“SwiftUIList”数据夹按右键，并选取“New File....”，如图 13.3 所示。接下来，选择使用“SwiftUI View”作为模板，并将文件命名为“SettingView.swift”。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-3.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-3.png)

图 13.3. 建立一个新 SwiftUI 档

现在，我们来开始建立表单。以下列代码替代 `SettingView`：

```
struct SettingView: View {
    var body: some View {
        NavigationStack {
            Form {
                Section(header: Text("SORT PREFERENCE")) {
                    Text("Display Order")
                }

                Section(header: Text("FILTER PREFERENCE")) {
                    Text("Filters")
                }
            }

            .navigationBarTitle("Settings")
        }
    }
}

```

要布局表单，你只需要使用 Form 容器，在其中建立所需要的 UI 组件。在上列的代码中，我们建立两个区块：“Sort Preference”与“Filter Preference”。每个区块都有一个文字视图，你的画布应会显示如图 13.4 所示的预览。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-4.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-4.png)

图 13.4. 建立一个包含两个区块的简易表单

### 建立选择器视图

在显示表单时，你一定希望保护某些资讯。如果只显示一个文字组件是无用的。在实际的表单中，我们使用三种类型的 UI 控制组件来进行使用者输入，包括选择器视图、切换与步进器。先从“排序”偏好来开始，我们将实现一个选择器视图。

对于“排序”偏好，可让使用者选择餐厅清单的显示顺序，其中我们提供三个选项供使用者选择：

1. Alphabetically（依字母顺序）。
2. Show Favorite First（最爱优先）。
3. Show Check-in First（打卡优先）。

`Picker` 控制组件非常适合处理此类输入。首先，你如何在代码中表示上述的选项呢？ 你可能考虑使用一个数组来存放这些选项。好的，我们在 `SettingView` 中声明一个名为 `displayOrders` 的数组：

```
private var displayOrders = [ "Alphabetical", "Show Favorite First", "Show Check-in First"]

```

要使用选择器，你还需要声明一个状态变量来储存使用者所选的选项。在 `SettingView` 中声明变量如下：

```
@State private var selectedOrder = 0

```

这里的“0”表示 `displayOrders` 的第一个项目。现在以下列代码替代“SORT PREFERENCE”区块：

```
Section(header: Text("SORT PREFERENCE")) {
    Picker(selection: $selectedOrder, label: Text("Display order")) {
        ForEach(0 ..< displayOrders.count, id: \.self) {
            Text(self.displayOrders[$0])
        }
    }
}

```

这是在 SwiftUI 中建立选择器容器的方式。你必须提供两个值，包括所选的绑定（即 `$selectedOrder`）以及描述选项用途的文字标签。在闭包中，以 `Text` 的形式显示可用选项。

在画布中，你应该会看到“Display Order”（显示顺序）设定为“Alphabetical”，如图13.5 所示，这是因为`selectedOrder` 默认为“0”。如果你点选“Play”按钮来测试视图， 点击该选项会将带你到下一个画面，其画面显示了所有的可用选项，你可以选择任何一个选项（例如：Show Favorite First ）进行测试。当你回到设定画面，“Display Order”会出现你刚才的选择，这便是 `@State` 关键字的强大之处，它自动监控变化，并帮助你储存选择的状态。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-5.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-5.png)

图 13.5. 对“Display Order”选项使用选择器视图

### 使用切换开关

接下来，我们进入设定筛选偏好的输入。首先，我们实现一个“切换”（或开关）来启用/ 禁用“Show Check-in Only”的筛选。“切换”有两个状态：“ON”或“OFF”，这对于提示使用者在两个互斥选项中选择特别有用。

使用 SwiftUI 建立一个切换开关非常简单，与 `Picker` 类似，我们必须声明一个状态变量来储存“切换”的目前设定。因此，在 `SettingView` 声明下列的变量：

```
@State private var showCheckInOnly = false

```

接着，修改“FILTER PREFERENCE”区块如下：

```
Section(header: Text("FILTER PREFERENCE")) {
    Toggle(isOn: $showCheckInOnly) {
        Text("Show Check-in Only")
    }
}

```

你使用 `Toggle` 建立一个切换开关，并传送“切换”的目前状态。在闭包中，你显示切换的描述，这里我们只使用一个 `Text` 视图。

上面是实现切换所需的代码，画布应该会在“Filter Preference”区块下显示一个切换开关，如图13.6 所示。如果你运行这个 App，你可以在 ON 与 OFF 状态之间切换。同样的，这个状态变量 `showCheckInOnly` 将会持续追踪使用者的选择。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-6.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-6.png)

图 13.6. 显示一个切换开关

### 使用步进器

设定表单中最后一个 UI 控制组件是“步进器”。再次参考图 13.1，使用者可以通过设定价位级别来筛选餐厅， 每间餐厅都有一个价位指示器，范围在1 至 5 之间。使用者可调整价位级别，以缩小清单视图中显示的餐厅数量。

在设定表单中，我们将实现一个步进器，供使用者调整设定。基本上，iOS 中的步进器显示了一个“+”和“-”的按钮，来运行递增及递减的动作。

要在 SwiftUI 中实现步进器，我们首先需要一个状态变量来存放步进器的目前值。在本例中，这个变量储存使用者选择的价位级别。在 `SettingView` 中声明状态变量，如下所示：

```
@State private var maxPriceLevel = 5

```

我们默认 `maxPriceLevel` 为“5”，现在 FILTER PREFERENCE 的区块修改如下：

```
Section(header: Text("FILTER PREFERENCE")) {
    Toggle(isOn: $showCheckInOnly) {
        Text("Show Check-in Only")
    }

    Stepper(onIncrement: {
        self.maxPriceLevel += 1

        if self.maxPriceLevel > 5 {
            self.maxPriceLevel = 5
        }
    }, onDecrement: {
        self.maxPriceLevel -= 1

        if self.maxPriceLevel < 1 {
            self.maxPriceLevel = 1
        }
    }) {
        Text("Show \(String(repeating: "$", count: maxPriceLevel)) or below")
    }
}

```

你可通过初始化一个 `Stepper`组件来建立一个步进器。对于 `onIncrement` 参数，你指定点选“+”按钮时要运行的动作。在代码中，我们只将 `maxPriceLevel` 增加“1”。反之， 点选“-”按钮时将运行 `onDecrement` 参数中指定的代码。

由于价位级别在 1 至 5 的范围之间，我们运行检查来确保 `maxPriceLevel`的值介于 1 至 5 之间。在闭包中，我们显示筛选偏好的文字描述。这里的最高价位是以美元符号表示， 如图 13.7 所示。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-7.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-7.png)

图 13.7. 实现一个步进器

要测试步进器，则点选“Play”按钮来运行 App。当你点选 `+`/ `-` 按钮时，将会调整 $ 符号的数量。

### 显示表单

现在你已经完成了表单UI，下一步是显示表单给使用者。以本示例来说，我们将以强制回应视图的形式来显示此表单。在内容视图中，我们将会在导航列加入一个“Setting”按钮，以触发设定视图。

切换至 `ContentView.swift`，我假设你已经阅读过强制回应视图一章，因此我将不再深入解释代码。首先，我们需要一个变量来追踪强制回应视图状态（即显示或不显示）。插入下列这行代码来声明状态变量：

```
@State private var showSettings: Bool = false

```

接下来，将下列的修饰器插入至 `NavigationStack` 中（加在`navigationTitle`之后）：

```
.toolbar {
    ToolbarItem(placement: .navigationBarTrailing) {
        Button(action: {
            self.showSettings = true
        }, label: {
            Image(systemName: "gear").font(.title2)
                .foregroundColor(.black)
        })
    }
}
.sheet(isPresented: $showSettings) {
    SettingView()
}

```

·`toolbar` 修饰器和`ToolbarItem` 可让你在导航列加入一个按钮，你可在导航列前缘（navigationBarLeading） 或后缘（navigationBarTrailing ）位置建立一个按钮。由于我们想在右上角显示按钮，因此我们使用 `navigationBarTrailing` 参数。`sheet`修饰器用于以强制回应视图的形式显示 `SettingView`。

在画布中，你应该会在导航列中看到一个齿轮图示，如图 13.8 所示。如果你运行 App，并点选齿轮图示，即会弹出设定视图。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-8.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-8.png)

图 13.8. 建立一个导航列按钮

### 作业

现在解除设定视图的唯一方式是使用向下滑动手势。在第 12 章中，你已经学过如何以程序设计方式来解除强制回应视图。我们来练习修改，请在导航列建立“Save”与“Cancel”按钮，如图13.9 所示。你不需要实现这些按钮，当使用者点击任一按钮时，你只需解除设定视图即可。

![%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-9.png](%E4%BB%A5%E9%80%89%E5%8F%96%E5%99%A8(Picker)%E3%80%81%E5%BC%80%E5%85%B3(Toggle)%E4%B8%8E%E6%AD%A5%E8%BF%9B%E5%99%A8(Stepper)%E6%9D%A5%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20Swi%20371f3cd2b6314d60b3a3c02dac159b54/swiftui-form-9.png)

图 13.9. 在导航列建立“Save”与“Cancel”按钮

### 接下来的任务

我希望你了解 Form 组件，并知道如何使用选择器与步进器等组件来建立一个表单 UI。至目前为止，这个 App 无法永久储存使用者偏好。每次启动 App 后，设定都会重置回原始设定。在下一章中，我将教你如何在本地储存器中储存这些设定。更重要的是，我们将会依照使用者偏好来修改清单视图。

在本章所准备的示例档中，有完整的项目可供下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIForm.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIForm.zip))