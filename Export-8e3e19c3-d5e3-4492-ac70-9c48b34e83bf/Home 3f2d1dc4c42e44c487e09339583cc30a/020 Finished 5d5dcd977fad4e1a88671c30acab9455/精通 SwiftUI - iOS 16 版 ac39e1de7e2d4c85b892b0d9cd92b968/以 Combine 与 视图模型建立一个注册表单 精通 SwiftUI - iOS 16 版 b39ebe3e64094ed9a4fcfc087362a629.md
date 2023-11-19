# 以 Combine 与 视图模型建立一个注册表单 | 精通 SwiftUI - iOS 16 版

现在你应该对于 Combine 有了基本的了解，我们将继续探索 Combine 如何使 SwiftUI 出类拔萃。当开发一个真实的 App 时，让使用者注册帐户是很常见的。在本章中，我将建立一个带有三个文字栏位的简单注册画面。我们的重点是表单验证，因此将不进行实际注册，你将学习如何利用Combine 的强大功能，来验证每个输入栏位，并在视图模型中编写代码。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-1.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-1.png)

图 15.1. 使用者注册示例

在深入研究代码之前，请先看一下图 15.1，这是我们将建立的使用者注册画面。在每个输入栏位下会列出要求，一旦使用者填入资讯后，App 就会即时验证所输入的资讯， 如果验证结果正确，则栏位要求文字就会划掉。而在满足所有的要求之前，将先禁用“注册”（Sign up ）按钮。

如果你对 Swift 与 UIKit 有一些经验，你就会知道有各种类型的实现方式可处理表单验证。不过，在本章中，我们将探索如何利用 Combine 框架来运行表单验证。

### 使用 SwiftUI 布局表单

我们从一个练习来开始本章，使用至目前为止所学的内容，来布局如图 15.1 所示的表单 UI。要在 SwiftUI 中建立一个文字栏位，你可以使用 TextField 组件，而对于密码栏位， SwiftUI 提供了一个名为 `SecureField`安全文字栏位。

要建立一个文字栏位，需要使用栏位名称与绑定（binding ）来初始化 TextField，这将渲染一个可编辑的文字栏位，而使用者输入会储存在给定的绑定中。和其他表单栏位类似，你可以通过使用相关的修饰器来修改它的外观。下面是示例代码片段：

```
TextField("Username", text: $username)
    .font(.system(size: 20, weight: .semibold, design: .rounded))
    .padding(.horizontal)

```

这两个组件的用法非常相似，除了安全栏位会自动遮蔽使用者的输入：

```
SecureField("Password", text: $password)
    .font(.system(size: 20, weight: .semibold, design: .rounded))
    .padding(.horizontal)

```

我知道这两个组件对你而言比较陌生，不过在观看解答之前，试着尽力建立表单。

那么，你能建立表单吗？即使你无法完成这个练习，也没有关系，现在至 [https://www.appcoda.com/resources/swiftui4/SwiftUIFormRegistrationUI.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIFormRegistrationUI.zip) 。来下载这个项目。我将会介绍我的做法让你参考。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-2.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-2.png)

图 15.2 初始项目

开启 `ContentView.swift` 档，并预览画布中的布局，你渲染的视图应该如图 15.2 所示。现在，我们简要检查一下代码，并从 `RequirementText` 视图开始。

```
struct RequirementText: View {

    var iconName = "xmark.square"
    var iconColor = Color(red: 251/255, green: 128/255, blue: 128/255)

    var text = ""
    var isStrikeThrough = false

    var body: some View {
        HStack {
            Image(systemName: iconName)
                .foregroundColor(iconColor)
            Text(text)
                .font(.system(.body, design: .rounded))
                .foregroundColor(.secondary)
                .strikethrough(isStrikeThrough)
            Spacer()
        }
    }
}

```

首先，为什么我对栏位要求文字建立一个单独视图（如图 15.3 所示）？如果你检查任何一个栏位要求文字，它都有一个图示及一个叙述。我们无须从头开始建立每一个栏位要求文字，我们可以将代码一般化，并为其建立一个通用的代码。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-3.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-3.png)

图 15.3 示例文字栏位及其要求文字

`RequirementText` 视图有四个属性，包括 `iconName`、`iconColor`、`text`与 `isStrikeThrough`。它足以弹性支持栏位要求文字的不同样式，如果你接受默认的图示与颜色，则可建立如下的栏位要求文字：

```
RequirementText(text: "A minimum of 4 characters")

```

这将渲染如图 15.3 所示的栏位要求文字。在某些情况下，应要划掉栏位要求文字，并显示不同的图示或颜色。代码可撰写如下：

```
RequirementText(iconName: "lock.open", iconColor: Color.secondary, text: "A minimum of 8 characters", isStrikeThrough: true)

```

你可指定一个不同的系统图示名称、颜色，并将 `isStrikeThrough` 选项设定为 `true`，这可让你建立如图 15.4 所示的栏位要求文字。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-4.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-4.png)

图 15.4. 栏位要求文字被划掉

现在，你应该了解 `RequirementText` 视图的工作原理，以及为何建立它。我们来看一下FormField 视图。同样的，如果你检查所有的文字栏位，它们都有一个共同的样式，即圆体字型样式的文字栏位，这就是为何我取出一些通用代码，并建立 `FormField` 视图的原因。

```swift
struct FormField: View {
    var fieldName = ""
    @Binding var fieldValue: String

    var isSecure = false

    var body: some View {

        VStack {
            if isSecure {
                SecureField(fieldName, text: $fieldValue)
                    .font(.system(size: 20, weight: .semibold, design: .rounded))
                    .padding(.horizontal)

            } else {
                TextField(fieldName, text: $fieldValue)
                    .font(.system(size: 20, weight: .semibold, design: .rounded))
                    .padding(.horizontal)
            }

            Divider()
                .frame(height: 1)
                .background(Color(red: 240/255, green: 240/255, blue: 240/255))
                .padding(.horizontal)

        }
    }
}

```

由于这个通用的 `FormField` 需要同时负责文字栏位与安全栏位，因此它有一个名为 `isSecure`的属性。如果设定为 `true`，表单栏位将建立为安全栏位。在 SwiftUI 中，你可以使用 `Divider` 组件来建立一条线。在代码中，我们使用 `frame` 修饰器来修改高度为“1 点”。

要建立使用者名称栏位，你可以将代码撰写如下：

```
FormField(fieldName: "Username", fieldValue: $username)

```

对于密码栏位，除了 `isSecure` 参数设定为 true 之外，代码也非常相似：

```
FormField(fieldName: "Password", fieldValue: $password, isSecure: true)

```

那么，我们回到 `ContentView` 结构，来看表单是如何布局。

```swift
struct ContentView: View {

    @State private var username = ""
    @State private var password = ""
    @State private var passwordConfirm = ""

    var body: some View {
        VStack {
            Text("Create an account")
                .font(.system(.largeTitle, design: .rounded))
                .bold()
                .padding(.bottom, 30)

            FormField(fieldName: "Username", fieldValue: $username)
            RequirementText(text: "A minimum of 4 characters")
                .padding()

            FormField(fieldName: "Password", fieldValue: $password, isSecure: true)
            VStack {
                RequirementText(iconName: "lock.open", iconColor: Color.secondary, text: "A minimum of 8 characters", isStrikeThrough: true)
                RequirementText(iconName: "lock.open", text: "One uppercase letter", isStrikeThrough: false)
            }
            .padding()

            FormField(fieldName: "Confirm Password", fieldValue: $passwordConfirm, isSecure: true)
            RequirementText(text: "Your confirm password should be the same as password", isStrikeThrough: false)
                .padding()
                .padding(.bottom, 50)

            Button(action: {
                // 进入下一个画面
            }) {
                Text("Sign Up")
                    .font(.system(.body, design: .rounded))
                    .foregroundColor(.white)
                    .bold()
                    .padding()
                    .frame(minWidth: 0, maxWidth: .infinity)
                    .background(LinearGradient(gradient: Gradient(colors: [Color(red: 251/255, green: 128/255, blue: 128/255), Color(red: 253/255, green: 193/255, blue: 104/255)]), startPoint: .leading, endPoint: .trailing))
                    .cornerRadius(10)
                    .padding(.horizontal)

            }

            HStack {
                Text("Already have an account?")
                    .font(.system(.body, design: .rounded))
                    .bold()

                Button(action: {
                    // 进入登入画面
                }) {
                    Text("Sign in")
                        .font(.system(.body, design: .rounded))
                        .bold()
                        .foregroundColor(Color(red: 251/255, green: 128/255, blue: 128/255))
                }
            }.padding(.top, 50)

            Spacer()
        }
        .padding()
    }

}

```

首先，我们有一个 `VStack` 存放所有的表单元素，它由标题开始，接着是所有的表单栏位与栏位要求文字。我已经解释过这些表单栏位与栏位要求文字是如何建立的，因此我将不再详细说明。我加入栏位的是 `padding` 修饰器，这可以让文字栏位之间加入间距。

“注册”按钮是使用 `Button` 组件所建立，目前没有动作。我打算将这个动作闭包留空， 因为我们的重点是表单验证。同样的，我相信你应该知道如何自订按钮，因此我将不再对其详细介绍，你可以随时参考按钮一章的内容。

最后，是“已经有帐号”的叙述文字，这个文字与“登入”按钮并不一定需要，我只是想模仿常见的注册表单。

以上是我布局使用者注册画面的方式。如果你已试着做这个练习，或许可提出其他的解决方案，这完全没问题。我只是告诉你建立表单的其中一种方法，你可以使用它作为参考，并提出更好的实现方式。

### 了解 Combine

在深入研究表单验证代码之前，我们先来了解更多关于 Combine 框架的背景资讯。如前一章所述，这个新框架提供了一个声明式API，用于随着时间处理值。

“随着时间处理值”是什么意思？这些值又是什么？

我们以注册表单为例。App 与使用者互动时，持续产生 UI 事件，假设使用者在文字栏位中输入的每个按键，都会触发一个事件，而这可以成为值的串流，如图 15.5 所示。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-5.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-5.png)

图 15.5. 输入数据串流

这些 UI 事件是框架所参照的一种类型的值。这些值的另一个例子是网路事件（例如： 从远端服务器下载一个文件）。

> Combine 框架提供一个声明式方法，供App 处理事件。你可以为给定的事件源建立单一处理链（chain），而不是实现多个委派回呼（delegate callback ）或完成处理闭包（completion handler closure ）。链的每一个部分都是一个Combine 运算子，对上一个步骤收到的元素运行不同的动作。
> 
> - Apple 的官方文件 ([https://developer.apple.com/documentation/combine/receiving_and_handling_events_with_combine](https://developer.apple.com/documentation/combine/receiving_and_handling_events_with_combine))

发布者与订阅者是框架的两个核心元素。使用 Combine，发布者传送事件，订阅者订阅，以从发布者接收值。同样的，我们以文字栏位为例。通过使用 Combine，使用者在文字栏位中输入的每个按键，都会触发一个值修改的事件。而对监控这些值感兴趣的订阅者，可以订阅接收这些事件，并进一步运行操作（例如：验证）。

举例而言，你正在写一个表单验证器，它有指示表单是否准备好送出的属性。在这个例子中，你可以使用`@Published` 标注来标记该属性，如下所示：

```
class FormValidator: ObservableObject {
    @Published var isReadySubmit: Bool = false
}

```

每次修改 `isReadySubmit` 的值时，它都会向订阅者发布一个事件。订阅者接收修改后的值，并继续处理，例如：订阅者可以使用该值来判断是否应启用“送出”按钮。

你可能会觉得在 SwiftUI 中 `@Published` 和 `@State` 的运作方式非常相似。虽然在这个例子，它的工作原理几乎相同，不过 `@State` 只能应用在属于特定SwiftUI 视图的属性。如果你想要建立不属于特定视图或可以用于多个视图的自订类型的话，则你需要建立一个遵守 `ObservableObject` 的类别，并以 `@Published` 标注来标记这些属性。

### Combine 与 MVVM

现在你应该具备 Combine 的一些基本概念，我们来开始使用框架实现表单验证，下列是我们将要做的事情：

1. 建立一个视图模型来表示使用者注册表单。
2. 在视图模型中实现表单验证。

我知道你心里可能会有几个问题。首先，为什么我们需要建立视图模型呢？我们可以在 ContentView 加入这些表单属性并运行表单验证吗？

你绝对可以这样做，但是随着项目规模持续成长，或者视图变得更加复杂，将复杂的组件拆成多层是一个良好做法。

“关注点分离”（Separation of concerns ）是编写优秀软体的基本原则。我们可以把视图分成视图及其视图模型等两个组件，而不是将所有的东西放在一个视图中。视图本身是负责 UI 布局，而视图模型存放要在视图中显示的状态与数据，并且视图模型还处理数据验证与转换。对于有经验的开发者而言，你知道我们正应用众所周知的“MVVM”（Model- View-ViewModel 的缩写）设计模式。

所以，这个视图模型将保存哪些数据呢？

再看一次注册表单，我们有三个文字栏位，包括：

- 使用者名称。
- 密码。
- 密码确认。

除此之外，这个视图模型将存放这些栏位要求文字的状态，以指示是否应该删掉它们：

- 最少4个字元（使用者名称）。
- 最少8个字元（密码）。
- 一个大写字母（密码）。
- 你的确认密码应与密码相同（密码确认）。

因此，视图模型将具有七个属性，并且每一个属性将其值的修改发布给那些有兴趣接收值的人。视图模型的基本架构可以定义如下：

```
class UserRegistrationViewModel: ObservableObject {
    // Input
    @Published var username = ""
    @Published var password = ""
    @Published var passwordConfirm = ""

    // Output
    @Published var isUsernameLengthValid = false
    @Published var isPasswordLengthValid = false
    @Published var isPasswordCapitalLetter = false
    @Published var isPasswordConfirmValid = false
}

```

这就是表单视图的数据模型。`username`、`password` 与 `passwordConfirm` 属性分别存放使用者名称、密码与密码确认的文字栏位的值。这个类别应该遵守 `ObservableObject`。所有这些属性都使用 `@Published` 来做标注，因为我们想要在值发生修改时通知订阅者，并相应运行验证。

### 使用Combine 验证使用者名称

好的，以上为数据模型，不过我们还没有处理表单验证。我们要如何依照要求来验证使用者名称、密码与密码确认呢？

使用 Combine，你就必须培养发布者/ 订阅者思维模式来解决问题。考虑到使用者名称， 我们这里其实有两个发布者：`username` 与`isUsernameLengthValid`。每当使用者在使用者名称栏位上敲击键盘输入时，`username` 发布者就会发布值的修改，而 `isUsernameLengthValid` 发布者将使用者输入的验证状态通知订阅者。几乎所有SwiftUI 中的控制组件都是订阅者，因此栏位要求文字视图将监听验证结果的变化，并相应修改其样式（即是否有删除线）。图15.6 说明了我们如何使用 Combine 来验证使用者名称的输入。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-6.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-6.png)

图 15.6. username 与 isUsernameValid 发布者

这里缺少的是连接两个发布者之间的东西，并且这个“东西”要处理下列的任务：

- 监听 `username` 的变化。
- 验证使用者名称与回传验证结果（true/false）。
- 指定结果至 `isUsernameLengthValid`。

如果你将以上的需求转换成代码，下面是代码片段：

```
$username
    .receive(on: RunLoop.main)
    .map { username in
        return username.count >= 4
    }
    .assign(to: \.isUsernameLengthValid, on: self)

```

Combine 框架提供两个内建的订阅者：“sink”与“assign”。 `sink` 建立一个通用订阅者来接收值，`assign` 则让你建立另一种类型的订阅者，用于修改物件的特定属性。举例而言，它将验证结果（true/false ）指定给 `isUsernameLengthValid`。

我将深入说明并逐行检查上列的代码。`$username` 是我们想要监听的修改值的来源。由于我们订阅 UI 事件的变化，因此调用 `receive(on:)` 函数来确保订阅者在主运行绪（即 `RunLoop.main` ）上接收到值。

发布者传送的值是使用者所输入的使用者名称，不过订阅者感兴趣的是使用者名称的长度是否能够满足最低要求。这里，`map` 函数被认为是Combine 中的运算子，它接受输入、处理输入，并将输入转换为订阅者所期望的内容，因此我们在上列的代码中做了下列事情：

1. 我们将使用者名称作为输入。
2. 然后，我们验证使用者名称并确认至少有4个字元，并验证其是否至少包含4个字元。
3. 最后，我们将验证结果以布林值回传给订阅者。

对于验证结果， 订阅者只需将结果设定给 `isUsernameLengthValid` 属性。请记得 `isUsernameLengthValid` 也是一个发布者，我们可以修改 `RequirementText` 控制组件来订阅修改，并相应修改 UI，如下所示：

```
RequirementText(iconColor: userRegistrationViewModel.isUsernameLengthValid ? Color.secondary : Color(red: 251/255, green: 128/255, blue: 128/255), text: "A minimum of 4 characters", isStrikeThrough: userRegistrationViewModel.isUsernameLengthValid)

```

图示的颜色与删除线的状态都取决于验证结果而定（也就是 `isUsernameLengthValid` ）。

这就是我们如何使用 Combine 来验证表单栏位的方式。我们还没有将代码修改放入我们的项目中，不过我希望让你了解发布者/ 订阅者的概念，以及我们如何使用这个方法运行验证。在后面的小节中，我们将应用所学的知识，并对代码进行修改。

### 使用 Combine 验证密码

如果你了解使用者名称栏位的验证是如何完成的，我们将对密码与密码确认验证应用类似的实现。

对于密码栏位，有两个要求：

1. 密码长度至少有 8 个字元。
2. 应至少包含一个大写字母。

为此，我们可以设定两个订阅者，如下所示：

```
$password
    .receive(on: RunLoop.main)
    .map { password in
        return password.count >= 8
    }
    .assign(to: \.isPasswordLengthValid, on: self)

$password
    .receive(on: RunLoop.main)
    .map { password in
        let pattern = "[A-Z]"
        if let _ = password.range(of: pattern, options: .regularExpression) {
            return true
        } else {
            return false
        }
    }
    .assign(to: \.isPasswordCapitalLetter, on: self)

```

第一个订阅者订阅了密码长度的验证结果，并指定给 `isPasswordLengthValid` 属性。第二个是处理大写字母的验证。我们使用 `range` 方法来测试密码是否至少包含一个大写字母。同样的，订阅者将验证结果直接指定给属性。

好的，剩下的是密码确认栏位的验证。对于这个栏位，输入要求是密码确认应与密码栏位的密码相同。`password` 与 `passwordConfirm` 都是发布者，为了验证两个发布者是否具有相同的值，我们使用 `Publisher.combineLatest`来接收与结合来自发布者的最新值，然后我们可以验证两个值是否相同。下列为代码片段：

```
Publishers.CombineLatest($password, $passwordConfirm)
    .receive(on: RunLoop.main)
    .map { (password, passwordConfirm) in
        return !passwordConfirm.isEmpty && (passwordConfirm == password)
    }
    .assign(to: \.isPasswordConfirmValid, on: self)

```

同样的，我们指定验证结果给 `isPasswordConfirmValid` 属性。

### 实现 UserRegistrationViewModel

现在我已经解释了实现的方法，我们将所有内容放至项目中。首先，使用 “Swift File” 模板建立一个新 Swift 档，命名为 `UserRegistrationViewModel.swift`，然后使用下列代码替换整个文件内容：

```swift
import Foundation
import Combine

class UserRegistrationViewModel: ObservableObject {
    // 输入
    @Published var username = ""
    @Published var password = ""
    @Published var passwordConfirm = ""

    // 输出
    @Published var isUsernameLengthValid = false
    @Published var isPasswordLengthValid = false
    @Published var isPasswordCapitalLetter = false
    @Published var isPasswordConfirmValid = false

    private var cancellableSet: Set<AnyCancellable> = []

    init() {
        $username
            .receive(on: RunLoop.main)
            .map { username in
                return username.count >= 4
            }
            .assign(to: \.isUsernameLengthValid, on: self)
            .store(in: &cancellableSet)

        $password
            .receive(on: RunLoop.main)
            .map { password in
                return password.count >= 8
            }
            .assign(to: \.isPasswordLengthValid, on: self)
            .store(in: &cancellableSet)

        $password
            .receive(on: RunLoop.main)
            .map { password in
                let pattern = "[A-Z]"
                if let _ = password.range(of: pattern, options: .regularExpression) {
                    return true
                } else {
                    return false
                }
            }
            .assign(to: \.isPasswordCapitalLetter, on: self)
            .store(in: &cancellableSet)

        Publishers.CombineLatest($password, $passwordConfirm)
            .receive(on: RunLoop.main)
            .map { (password, passwordConfirm) in
                return !passwordConfirm.isEmpty && (passwordConfirm == password)
            }
            .assign(to: \.isPasswordConfirmValid, on: self)
            .store(in: &cancellableSet)
    }
}

```

以上的代码几乎与前面章节的代码相同。要使用 Combine，你需要先汇入 Combine 框架。在 `init()` 方法中，我们初始化所有的订阅者来监听文字栏位的值的修改，并运行相应的验证。

代码与我们之前讨论的代码几乎相同，但你可能注意到一件事，即 `cancellableSet` 变量。对于每一个订阅者，我们在最后调用 `store` 函数。

这些是什么呢？

`assign` 函数建立订阅者，并回传一个可取消的实例。你可以使用该实例在适当的时间取消订阅。`store` 函数可让我们将可取消的的参照储存到一个集合中，以便稍后的清理。如果你不储存这个参照，则 App 可能出现记忆体泄漏的问题。

那么，这个示例何时会进行清理呢？由于 `cancellableSet` 被定义为该类别的属性，因此当类别被取消初始化时，将会对订阅进行清理及取消。

现在切换回 `ContentView.swift`，并修改 UI 控制组件。首先，将下列的状态变量：

```
@State private var username = ""
@State private var password = ""
@State private var passwordConfirm = ""

```

以一个视图模型取代，并命名为 `userRegistrationViewModel`：

```
@ObservedObject private var userRegistrationViewModel = UserRegistrationViewModel()

```

接下来，修改文字栏位与使用者名称的栏位要求文字如下：

```
FormField(fieldName: "Username", fieldValue: $userRegistrationViewModel.username)
RequirementText(iconColor: userRegistrationViewModel.isUsernameLengthValid ? Color.secondary : Color(red: 251/255, green: 128/255, blue: 128/255), text: "A minimum of 4 characters", isStrikeThrough: userRegistrationViewModel.isUsernameLengthValid)
    .padding()

```

现在，`fieldValue` 参数已经修改为 `$userRegistrationViewModel.username`。对于栏位要求文字，SwiftUI 监控 `userRegistrationViewModel.isUsernameLengthValid` 属性，并相应修改栏位要求文字。

同样的，修改密码与密码确认栏位的UI 代码如下所示：

```swift
FormField(fieldName: "Password", fieldValue: $userRegistrationViewModel.password, isSecure: true)
VStack {
    RequirementText(iconName: "lock.open", iconColor: userRegistrationViewModel.isPasswordLengthValid ? Color.secondary : Color(red: 251/255, green: 128/255, blue: 128/255), text: "A minimum of 8 characters", isStrikeThrough: userRegistrationViewModel.isPasswordLengthValid)
    RequirementText(iconName: "lock.open", iconColor: userRegistrationViewModel.isPasswordCapitalLetter ? Color.secondary : Color(red: 251/255, green: 128/255, blue: 128/255), text: "One uppercase letter", isStrikeThrough: userRegistrationViewModel.isPasswordCapitalLetter)
}
.padding()

FormField(fieldName: "Confirm Password", fieldValue: $userRegistrationViewModel.passwordConfirm, isSecure: true)
RequirementText(iconColor: userRegistrationViewModel.isPasswordConfirmValid ? Color.secondary : Color(red: 251/255, green: 128/255, blue: 128/255), text: "Your confirm password should be the same as password", isStrikeThrough: userRegistrationViewModel.isPasswordConfirmValid)
    .padding()
    .padding(.bottom, 50)

```

如此，现在可以测试 App 了。如果你已正确进行所有的修改，则 App 现在应该能验证使用者输入，如图 15.7 所示。

![%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-7.png](%E4%BB%A5%20Combine%20%E4%B8%8E%20%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%9E%8B%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E6%B3%A8%E5%86%8C%E8%A1%A8%E5%8D%95%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b39ebe3e64094ed9a4fcfc087362a629/swiftui-registration-7.png)

图 15.7. 注册表单现在可验证使用者输入

### 本章小结

我希望你现在已经掌握了一些 Combine 框架的基本认识。SwiftUI 与 Combine 的导入彻底改变了建立 App 的方式。函式响应式程序设计（Functional Reactive Programming，简称 FRP），近年来越来越流行，不过这是Apple 首次释出自己的函式响应式框架。在我看来，这是一个重要的典范转移（paradigm shift ）。Apple 公司最终对 FRP 做出定位，并推荐 Apple 开发者拥抱这个新的程序设计方法。

就像导入任何一个新技术的导入一样，会有一个学习曲线，即使你之前有 iOS 开发经验，也要花一些时间从委派的程序设计方法变成到发布者/ 订阅者的设计方法。

不过，一旦你管理了 Combine 框架，你将会非常高兴，因为它可以帮你实现更可维护与模组化的代码，与SwiftUI 一起使用，如你所见，视图与视图模型之间的沟通变得轻而易举了。

在本章所准备的示例档中，有完整的项目可供下载：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIFormRegistration.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIFormRegistration.zip))