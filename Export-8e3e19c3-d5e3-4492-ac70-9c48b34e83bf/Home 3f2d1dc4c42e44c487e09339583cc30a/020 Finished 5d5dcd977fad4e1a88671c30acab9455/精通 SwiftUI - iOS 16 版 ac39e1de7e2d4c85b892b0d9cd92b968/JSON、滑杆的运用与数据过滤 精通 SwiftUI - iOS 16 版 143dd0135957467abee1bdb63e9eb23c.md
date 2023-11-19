# JSON、滑杆的运用与数据过滤 | 精通 SwiftUI - iOS 16 版

JSON 是 JavaScript Object Notation 的缩写，是用户端- 服务器应用程序中用于数据交换的通用数据格式。即使我们是行动装置 App 的开发者，也不可避免地要使用 JSON，因为几乎所有的 Web API 或后端网页服务都使用 JSON 作为数据交换的格式。

在本章中，我们将讨论当使用 SwiftUI 框架建立 App 时如何使用 JSON。如果你对于不了解 JSON 的话，我建议看一下在《iOS 程序设计进阶攻略》一书中的 [免费试阅章节](https://www.appcoda.com/intermediate-swift-tips/json.html) ，这里会详细解释在 Swift 中处理 JSON 的两种不同方法。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-1.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-1.png)

图 21.1. 示例App

和往常一样，为了掌握 JSON 及其相关的 API 知识，你将建立一个简单的 JSON App， 该 App 利用 [Kiva.org](http://kiva.org/).提供的 JSON API。若是你没有听过 Kiva， 这是一个非营利组织，其使命是通过借贷将人们联击在一起，以减轻贫困问题；Kiva 让每个人借出至少 25 美元，来帮助世界各地的人创造机会。Kiva 为开发者提供了免费的 Web API 来存取他们的数据。对于我们的示例 App，我们将调用一个免费的 Kiva API 来取得最近的募资借款，并在清单视图中显示，如图 21.1 所示。

除此之外，我们将示范滑杆（Slider ）的用法，滑杆是SwiftUI 提供的众多内建 UI 控制组件之一。你将在 App 中实现一个数据筛选选项，以让使用者可以筛选清单中的贷款数据，如图 21.2 所示。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-2.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-2.png)

图 21.2. 滑杆控制组件

### 了解 JSON 与 Codable

首先，JSON 格式是什么样子呢？如果你不了解 JSON，则开启浏览器，并指向下列由 Kiva 提供的 Web API：

```
https://api.kivaws.org/v1/loans/newest.json

```

你应该会看到下列的内容：

```
{
    "loans": [
        {
            "activity": "Fruits & Vegetables",
            "basket_amount": 25,
            "bonus_credit_eligibility": false,
            "borrower_count": 1,
            "description": {
                "languages": [
                    "en"
                ]
            },
            "funded_amount": 0,
            "id": 1929744,
            "image": {
                "id": 3384817,
                "template_id": 1
            },
            "lender_count": 0,
            "loan_amount": 250,
            "location": {
                "country": "Papua New Guinea",
                "country_code": "PG",
                "geo": {
                    "level": "town",
                    "pairs": "-9.4438 147.180267",
                    "type": "point"
                },
                "town": "Port Moresby"
            },
            "name": "Mofa",
            "partner_id": 582,
            "planned_expiration_date": "2020-04-02T08:30:11Z",
            "posted_date": "2020-03-03T09:30:11Z",
            "sector": "Food",
            "status": "fundraising",
            "tags": [],
            "themes": [
                "Vulnerable Groups",
                "Rural Exclusion",
                "Underfunded Areas"
            ],
            "use": "to purchase additional vegetables to increase her currrent sales."
        },

        ...

        "paging": {
        "page": 1,
        "page_size": 20,
        "pages": 284,
        "total": 5667
    }
}

```

你的显示结果可能不是相同的格式，但这就是 JSON 回应的样式。如果你使用的是 Chrome，则可以下载并安装一个名为“JSON Formatter”（[http://link.appcoda.com/json-formatter](http://link.appcoda.com/json-formatter) ）的外挂程序，来美化 JSON 回应。

或者，你可以在 Mac 上使用下列的指令，来对格式化 JSON 数据：

```
curl https://api.kivaws.org/v1/loans/newest.json | python -m json.tool > kiva-loans-data.txt

```

这将格式化 JSON 回应，并将其储存到文字档中。

现在你对 JSON 有一些了解了，我们如何在 Swift 中解析 JSON 数据呢？从 Swift 4 开始， Apple 导入一个编码及解码 JSON 数据的新方式，即采用了一个名为 `Codable`的协议。

`Codable` 为开发者提供一个不同的方式来解码（或编码）JSON，从而简化了整个过程。只要你的类型遵守`Codable` 协议以及新的 `JSONDecoder`，你就能够将 JSON 数据解码到你指定的实例（instance ）中。

图 21.3 说明了使用 `JSONDecoder`，将示例借贷数据解码为一个 `Loan` 实例。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-3.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-3.png)

图 21.3. JSONDecoder 解码 JSON 数据，并将其转换为一个 Loan 实例

### 使用 JSONDecoder 与 Codable

在建立示例 App 之前，我们在 Playgrounds 上尝试 JSON 解码。启动 Xcode，并开启一个新的 Playgrounds 项目（在 Xcode 菜单并选择 *File > New > Playground...*），当你建立你的 Playground 项目后，声明下列的 `json` 变量：

```
let json = """
{

"name": "John Davis",
"country": "Peru",
"use": "to buy a new collection of clothes to stock her shop before the holidays.",
"amount": 150

}
"""

```

假设你是 JSON 解析的新手，我们简单说明一下。上面是一个简化的 JSON 回应，类似于上一小节所示的回应。

要解析数据，声明 `Loan` 结构如下：

```
struct Loan: Codable {
    var name: String
    var country: String
    var use: String
    var amount: Int
}

```

如你所见，该结构采用了 `Codable`协议。而且，结构中定义的变量与 JSON 回应的键值相符，这是让解码器知道如何解码数据的方法。

现在，让我们来看看它的魔力 ！

继续在Playground 文件中插入下列的代码：

```
let decoder = JSONDecoder()

if let jsonData = json.data(using: .utf8) {

    do {
        let loan = try decoder.decode(Loan.self, from: jsonData)
        print(loan)

    } catch {
        print(error)
    }
}

```

如果你运行这个项目，则应该在主控台中看到一个信息，这是 `Loan` 实例，其中填满了解码后的值。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-4.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-4.png)

图 21.4. 在主控台中显示解码后的借贷数据

让我们再次研究代码片段。我们实例化一个 `JSONDecoder` 实例，然后将 JSON 字串转换为 `Loan`。而魔法发生在下列这行代码中：

```
let loan = try decoder.decode(Loan.self, from: jsonData)

```

你只需要调用解码器的 `decode` 方法来对 JSON 数据解码，并指定要解码的值的类型（即 `Loan.self` ）。解码器会自动解析 JSON 数据，并将其转换为一个 `Loan` 物件。

很酷，是吧？

### 使用自定义属性名称

现在，让我们进入更复杂的内容。如果属性名称与 JSON 的键不同的话，该怎么办？你如何定义映射（mapping ）呢？

举例而言，我们修改 `json` 变量如下：

```
let json = """
{

"name": "John Davis",
"country": "Peru",
"use": "to buy a new collection of clothes to stock her shop before the holidays.",
"loan_amount": 150

}
"""

```

如你所见，`amount` 这个键现在改为 `loan_amount`。为了解码 JSON 数据，你可以将属性名称从 `amount` 改为`loan_amount`。不过，我们真的想要保留名称 `amount`，在这种的情况下，我们如何定义映射呢？

要定义键与属性名称之间的映射，你需要声明一个名为“CodingKeys”的枚举， `CodingKeys` 枚举具有一个 `String` 类型的原始值，并遵守 `CodingKey` 协议。

现在，修改 `Loan` 结构如下：

```
struct Loan: Codable {
    var name: String
    var country: String
    var use: String
    var amount: Int

    enum CodingKeys: String, CodingKey {
        case name
        case country
        case use
        case amount = "loan_amount"
    }
}

```

在枚举中，你定义了模型的所有属性名称，以及其在 JSON 数据中的对应键。例如：`amount` 定义为映射`loan_amount` 这个键，如果属性名称与 JSON 数据的键相同，则可以省略这个指定。

### 使用巢状 JSON 物件

现在，你应该了解基础知识了，让我们深入研究并解码一个更切合实际的 JSON 回应。首先，修改 `json` 变量如下：

```
let json = """
{

"name": "John Davis",
"location": {
"country": "Peru",
},
"use": "to buy a new collection of clothes to stock her shop before the holidays.",
"loan_amount": 150

}
"""

```

我们已加入了 `location` 这个键，它具有巢状 JSON 物件以及 `country` 巢状键。那么，我们如何从巢状物件中解码 `country` 的值呢？

我们修改 `Loan` 结构如下：

```
struct Loan: Codable {
    var name: String
    var country: String
    var use: String
    var amount: Int

    enum CodingKeys: String, CodingKey {
        case name
        case country = "location"
        case use
        case amount = "loan_amount"
    }

    enum LocationKeys: String, CodingKey {
        case country
    }

    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)

        name = try values.decode(String.self, forKey: .name)

        let location = try values.nestedContainer(keyedBy: LocationKeys.self, forKey: .country)
        country = try location.decode(String.self, forKey: .country)

        use = try values.decode(String.self, forKey: .use)
        amount = try values.decode(Int.self, forKey: .amount)

    }
}

```

和我们之前所做的类似，我们必须定义一个枚举 `CodingKeys`。对于 `country` 这个 case， 我们指定要映射`location` 键。而要处理巢状的 JSON 物件，我们需要定义另一个枚举，在上面的代码中，我们将其命名为`LocationKeys`，并声明与巢状物件的 `country` 键相符的 `country` 这个 case。

因为它不是直接映射，我们需要实现 `Decodable` 协议的初始器，来处理所有属性的解码。在 `init`方法中，我们首先使用 `CodingKeys.self` 调用解码器的 `container` 方法，以取得与指定的编码键相关的数据，即`name`、`location`、`use`与 `amount`。

要解码一个特定值，我们使用特定键（例如：.name ）和关联类型（例如：String.self ） 来调用 `decode` 方法。`name`、`use` 与 `amount` 的解码非常简单。对于 `country`属性，解码有点棘手，我们必须使用`LocationKeys.self` 调用 `nestedContainer` 方法，来取得巢状的 JSON 物件。从回传的值，我们进一步解码`country`的值。

以上为使用巢状物件来解码JSON 数据的方式。

### 使用数组

从 Kiva API 所回传的 JSON 数据中，通常不只一笔贷款，多笔贷款以数组的形式建构， 现在我们来看如何使用Codable 解码 JSON 物件的数组。

首先，修改 `json` 变量如下：

```
let json = """
{
"loans":
[{
"name": "John Davis",
"location": {
"country": "Paraguay",
},
"use": "to buy a new collection of clothes to stock her shop before the holidays.",
"loan_amount": 150
},
{
"name": "Las Margaritas Group",
"location": {
"country": "Colombia",
},
"use": "to purchase coal in large quantities for resale.",
"loan_amount": 200
}]
}
"""

```

在上面的示例中，`json` 变量中有两笔贷款数据，你如何将其解码为 `Loan` 的数组呢？

为此，声明另一个名称为 `LoanStore`的结构，该结构也采用 `Codable` 协议：

```
struct LoanStore: Codable {
    var loans: [Loan]
}

```

该 `LoanStore` 只有一个 `loans` 属性，它与 JSON 数据的 `loans` 键相符。并且，其类型定义为 `Loan` 的数组。

要解码贷款数据，将下列这行代码：

```
let loan = try decoder.decode(Loan.self, from: jsonData)

```

修改为：

```
let loanStore = try decoder.decode(LoanStore.self, from: jsonData)

```

解码器将自动解码 `loans` JSON 物件，并将其储存至 `LoanStore` 的 `loans` 数组中。如果你印出 `loans`，应该会看到如图 21.5 所示的类似信息。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-5.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-5.png)

图 21.5. 印出loans 数组

以上就是使用 Swift 解码 JSON 的方式。

*注意：作为参考，Playgrounds 项目包含在最终下载档内。 你可以在本章末端找到下载链接。*

### 🧭建立 Kiva 贷款 App

好的，你现在应该已经了解如何处理 JSON 解码，让我们开始建立一个示例 App，来看如何运用刚刚所学的技术。

倘若你已经开启 Xcode，至菜单并选择“File → New → Projects”来建立一个新项目。如往常一样，使用“App”模板，并将项目名称命名为“SwiftUIKivaLoan”或是你所喜爱的其他名称。

我们将从建立模型类别来开始，该模型类别储存从 Kiva 取得的所有最新贷款数据。对于使用者介面的实现，我们将稍后处理。

### 取得 Kiva 最新的贷款数据

首先，使用“Swift File”模板来建立一个新文件，并将其命名为 `Loan.swift`。这个文件储存了采用 Codable 协议来进行 JSON 解码的 `Loan` 结构。

在文件中插入下列的代码：

```
struct Loan: Identifiable {
    var id = UUID()
    var name: String
    var country: String
    var use: String
    var amount: Int

    init(name: String, country: String, use: String, amount: Int) {
        self.name = name
        self.country = country
        self.use = use
        self.amount = amount
    }

}

extension Loan: Codable {
    enum CodingKeys: String, CodingKey {
        case name
        case country = "location"
        case use
        case amount = "loan_amount"
    }

    enum LocationKeys: String, CodingKey {
        case country
    }

    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)

        name = try values.decode(String.self, forKey: .name)

        let location = try values.nestedContainer(keyedBy: LocationKeys.self, forKey: .country)
        country = try location.decode(String.self, forKey: .country)

        use = try values.decode(String.self, forKey: .use)
        amount = try values.decode(Int.self, forKey: .amount)

    }
}

```

代码与我们在前一节中所讨论的几乎相同。我们只是使用扩展（extension ）来采用 `Codable` 协议，除了`Codable` 协议之外，这个结构也采用了 `Identifiable` 协议，并具有默认为 `UUID()` 的id 属性。稍后，我们将会使用 SwiftUI 的 `List` 控制组件来呈现这些贷款数据， 这就是为什么我们让这个结构采用 `Identifiable` 协议的原因。

接下来， 使用“Swift File”模板来建立另一个文件， 并将其命名为“LoanStore. swift”。这个类别是负责连接到 Kiva 的 Web API、解码 JSON 数据，并将数据储存在本地端。

我们来逐步撰写 `LoanStore` 类别， 如此你就可以更加了解我是如何实现的。在 `LoanStore.swift` 中插入下列的代码：

```
class LoanStore: Decodable {
    var loans: [Loan] = []
}

```

稍后，解码器将解码 `loans` JSON 物件，并将其储存至 `LoanStore` 的 `loans` 数组中，这就是为什么我们建立上述 `LoanStore` 的原因。代码看起来与我们之前建立的 `LoanStore` 结构非常相似，但是它采用 `Decodable` 协议而不是`Codable` 协议。

如果你研究 `Codable` 的文件，它只是一个协议组成的类型别名：

```
typealias Codable = Decodable & Encodable

```

`Decodable` 与 `Encodable` 是你需要实际使用的两个协议。由于`LoanStore` 只有负责处理 JSON 解码，因此我们采用 `Decodable` 协议。

如前所述，我们将使用一个清单视图来显示 `loans`。因此，除了 `Decodable`之外，我们必须采用`ObservableObject` 协议，并使用 `@Published` 属性包裹器标记 `loans` 变量，如下所示：

```
class LoanStore: Decodable, ObservableObject {
    @Published var loans: [Loan] = []
}

```

如此，当 `loans` 变量有任何变动时，SwiftUI 将自动管理 UI 的修改。如果你已经忘记什么是 `ObservableObject`，则请再次阅读第 14 章。

不过，当你加上 `@Published` 属性包裹器后，Xcode 就会显示一个错误信息，如图 21.6 所示。`Decodable`（或Codable ）协议在 `@Published`上不好发挥作用。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-6.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-6.png)

图 21.6. Xcode 指出 LoanStore 并没有遵守 Decodable 协议的错误信息

要修正这个错误，需要做一些额外的工作。当使用 `@Published` 属性包裹器时，我们需要手动实现 `Decodable`所需的方法。如果你研究文件（[https://developer.apple.com/documentation/swift/decodable)），可以采用以下的方法：](https://developer.apple.com/documentation/swift/decodable)%EF%BC%89%EF%BC%8C%E5%8F%AF%E4%BB%A5%E9%87%87%E7%94%A8%E4%BB%A5%E4%B8%8B%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%9A)

```
init(from decoder: Decoder) throws

```

实际上，在解码巢状的 JSON 物件时，我们已经实现这个方法，现在修改类别如下：

```
class LoanStore: Decodable, ObservableObject {
    @Published var loans: [Loan] = []

    enum CodingKeys: CodingKey {
        case loans
    }

    required init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        loans = try values.decode([Loan].self, forKey: .loans)
    }

    init() {

    }
}

```

我们加上 `CodingKeys` 枚举来明确指定要解码的键，然后我们实现自订的初始器来处理解码。

好的，错误现在已经修正了，那下一步呢？

### 调用 Web API

到目前为止，我们只设定了 JSON 解码的所有内容，但我们尚未使用 Web API。现在于 `LoanStore` 类别中声明一个新变量，来储存 Kiva API 的URL：

```
private static var kivaLoanURL = "https://api.kivaws.org/v1/loans/newest.json"

```

Next, insert the following methods in the class:

```
func fetchLatestLoans() {
    guard let loanUrl = URL(string: Self.kivaLoanURL) else {
        return
    }

    let request = URLRequest(url: loanUrl)
    let task = URLSession.shared.dataTask(with: request, completionHandler: { (data, response, error) -> Void in

        if let error = error {
            print(error)
            return
        }

        // 解析 JSON 数据
        if let data = data {
            DispatchQueue.main.async {
                self.loans = self.parseJsonData(data: data)
            }

        }
    })

    task.resume()
}

func parseJsonData(data: Data) -> [Loan] {

    let decoder = JSONDecoder()

    do {

        let loanStore = try decoder.decode(LoanStore.self, from: data)
        self.loans = loanStore.loans

    } catch {
        print(error)
    }

    return loans
}

```

`fetchLatestLoans()` 方法通过使用 `URLSession` 来连接到 Web API。当它接收到 API 回传的数据后，它会传送数据给 `parseJsonData` 方法来解码 JSON，并转换贷款数据为 `Loan` 数组。

你可能想知道为什么我们需要使用 `DispatchQueue.main.async` 来包裹这行代码？

```
DispatchQueue.main.async {
    self.loans = self.parseJsonData(data: data)
}

```

当调用 Web API 时， 该操作是在背景伫列中运行。在此，`loans` 变量标记为 `@Published`，这意味着对于变量的任何修改，SwiftUI 将触发使用者介面的修改。而且， UI 修改需要在主伫列中运行，这就是我们使用`DispatchQueue.main.async` 包裹它的原因。

### 实现使用者介面

现在，我们已经建立了用于取得贷款数据的类别，让我们继续进行使用者介面的实现。我猜想你可能忘了UI 的外观，请参考图 21.7，这是我们将要建立的 UI。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-7.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-7.png)

图 21.7. 示例 App 的使用者介面

而且，我们将UI 分成三个视图，而不是在同一个文件中撰写UI 的代码：

- *ContentView.swift* - 这是呈现贷款清单的主视图。
- *LoanCellView.swift* - 这是单元格视图。
- *LoanFilterView.swift* - 这是显示筛选选项的视图。

我们从单元格视图开始。在项目导航器中，在 `SwiftUIKivaLoan` 按右键，选择“New file...”，然后选取“SwiftUI View”模板，并将文件命名为 `LoanCellView.swift`。

修改 `LoanCellView` 如下：

```
struct LoanCellView: View {

    var loan: Loan

    var body: some View {
        HStack(alignment: .top) {
            VStack(alignment: .leading) {
                Text(loan.name)
                    .font(.system(.headline, design: .rounded))
                    .bold()
                Text(loan.country)
                    .font(.system(.subheadline, design: .rounded))
                Text(loan.use)
                    .font(.system(.body, design: .rounded))
            }

            Spacer()

            VStack {
                Text("$\(loan.amount)")
                    .font(.system(.title, design: .rounded))
                    .bold()
            }
        }
        .frame(minWidth: 0, maxWidth: .infinity)

    }
}

```

这个视图带入 `Loan`，并渲染单元格视图。该代码一目了然，但如果你想要预览单元格视图，你将需要修改`LoanCellView_Previews` 如下：

```
struct LoanCellView_Previews: PreviewProvider {
    static var previews: some View {
        LoanCellView(loan: Loan(name: "Ivan", country: "Uganda", use: "to buy a plot of land", amount: 575)).previewLayout(.sizeThatFits)
    }
}

```

我们实例化一个虚构的贷款数据，并传送至单元格视图做渲染。只要你将预览设定为 *Selectable* 模式，你的预览面板看起来应该与图 21.8 类似。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-8.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-8.png)

图 21.8. 贷款单元格视图

现在回到 `ContentView.swift` 来实现清单视图，首先，声明一个名为 `loanStore` 的变量：

```
@ObservedObject var loanStore = LoanStore()

```

由于我们要观察贷款商店的变化并修改UI，因此将 `loanStore` 标记为 `@ObservedObject` 属性包裹器。

接着，修改 `body`变量如下：

```
var body: some View {
    NavigationStack {

        List(loanStore.loans) { loan in

            LoanCellView(loan: loan)
                .padding(.vertical, 5)
        }

        .navigationTitle("Kiva Loan")

    }
    .task {
        self.loanStore.fetchLatestLoans()
    }
}

```

如果你已经阅读了第 10 章与第 11 章，则应该了解如何显示清单视图，并将其嵌入到导航视图中，这就是上面的代码所做的事情。在视图出现时将调用 `task` 函数，并且我们调用 `fetchLatestLoans()` 方法来从 Kiva 取得最新的贷款数据。

如果这是你第一次使用 .task ，它与 .onAppear 非常相似。 两者都允许你在视图出现时运行异步任务。 主要区别在于`.task`会在视图被销毁时自动取消任务。 在这种情况下更合适。

现在，在预览中或模拟器上运行这个 App，你应该能够看到如图 21.9 所示的贷款纪录。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-9.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-9.png)

图 21.9. 在清单视图中呈现贷款数据

### 使用滑杆建立筛选器视图

在完成本章之前，我想要介绍如何实现一个筛选功能。这个筛选功能可让使用者定义最大贷款金额，并只显示低于该金额的那些纪录。图 21.7 为筛选器视图的示例，使用者可以使用滑杆来设定最大数量。

同样的，为了让程序更有组织性，因此为筛选器视图建立一个新文件，并将其命名为 `LoanFilterView.swift`。

现在修改 `LoanFilterView` 结构如下：

```
struct LoanFilterView: View {

    @Binding var amount: Double

    var minAmount = 0.0
    var maxAmount = 10000.0

    var body: some View {
        VStack(alignment: .leading) {

            Text("Show loan amount below $\(Int(amount))")
                .font(.system(.headline, design: .rounded))

            HStack {

                Slider(value: $amount, in: minAmount...maxAmount, step: 100)
                    .accentColor(.purple)

            }

            HStack {
                Text("\(Int(minAmount))")
                    .font(.system(.footnote, design: .rounded))

                Spacer()

                Text("\(Int(maxAmount))")
                .font(.system(.footnote, design: .rounded))
            }

        }
        .padding(.horizontal)
        .padding(.bottom, 10)
    }
}

```

我假设你完全了解堆叠视图，因此我将不讨论如何使用它们来实现布局，不过让我们再多谈一下滑杆控制组件，其是 SwiftUI 所提供的一个标准组件，你可以通过传送滑杆的绑定（Binding ）、范围与滑杆刻度（step ）来实例化滑杆。绑定保存滑杆的目前值。以下是用于建立滑杆的示例代码：

```
Slider(value: $amount, in: minAmount...maxAmount, step: 100)

```

刻度可控制使用者拖曳滑杆时的修改量。如果你让使用者拥有很好的控制组件，请将刻度设定更小一点。对于上列的代码，我们将其设定为“100”。

为了预览筛选器视图，修改 LoanFilterView_Previews 如下：

```
struct LoanFilterView_Previews: PreviewProvider {
    static var previews: some View {
        LoanFilterView(amount: .constant(10000))
    }
}

```

现在，你的预览应该如图 21.10 所示。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-10.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-10.png)

图 21.10. 用于设定显示条件的筛选器视图

好的，我们已经实现了筛选器视图，但是我们尚未实现筛选纪录的实际逻辑。让我们增强 `LoanStore.swift` 的筛选功能。

首先，声明以下的变量，该变量用来储存贷款纪录的复本，以作为筛选操作之用。

```
private var cachedLoans: [Loan] = []

```

要储存该副本，请插入下列这行代码，并将其放在 `self.loans = self.parseJsonData(data: data)` 的后面：

```
self.cachedLoans = self.loans

```

最后，为筛选建立一个新函数：

```
func filterLoans(maxAmount: Int) {
    self.loans = self.cachedLoans.filter { $0.amount < maxAmount }
}

```

这个函数带入最大金额的值，并筛选低于该限额的那些贷款项目。

酷 ！我们快完成了。

让我们回到 `ContentView.swift` 来显示筛选器视图，我们要做的是在右上角加上一个导航列按钮。当使用者点击按钮时，App 会显示筛选器视图。

我们首先声明两个状态变量：

```
@State private var filterEnabled = false
@State private var maximumLoanAmount = 10000.0

```

`filterEnabled` 变量储存筛选器视图的目前状态。默认是设定为“false”，表示没有显示筛选器视 图。`maximumLoanAmount` 储存用于显示的最大贷款金额，任何大于该限额的贷款纪录都将被隐藏。

接下来，将 `NavigationView` 的代码修改如下：

```
NavigationStack {
    VStack {
        if filterEnabled {
            LoanFilterView(amount: self.$maximumLoanAmount)
                .transition(.opacity)
        }

        List(loanStore.loans) { loan in

            LoanCellView(loan: loan)
                .padding(.vertical, 5)
        }
    }

    .navigationTitle("Kiva Loan")
}

```

我们添加了`LoanFilterView`并将其嵌入到 `VStack` 中。 `LoanFilterView` 的外观由 `filterEnabled` 控制。 当 `filterEnabled` 设置为 `true` 时，App将在列表视图的顶部插入贷款过滤器视图。 剩下的部分是导航列按钮，插入下列的代码，并将其放在 `.navigationBarTitle("Kiva Loan")` 的后面：

```
.toolbar {
    ToolbarItem(placement: .navigationBarTrailing) {
        Button {
            withAnimation(.linear) {
                self.filterEnabled.toggle()
                self.loanStore.filterLoans(maxAmount: Int(self.maximumLoanAmount))
            }
        } label: {
            Text("Filter")
                .font(.subheadline)
                .foregroundColor(.primary)
        }
    }
}

```

这将在右上角加入一个导航列按钮。当点击按钮后，我们切换 `filterEnabled` 的值来显示/ 隐藏筛选器视图。除此之外，我们调用 `filterLoans` 函数来筛选贷款项目。

现在运行 App 来测试它，你应该会在导航列上看到一个筛选器按钮，点击它一次，将弹出筛选器视图，然后你可以设定新的上限（例如：$500）。再次点击按钮，App 只会显示低于 $500的贷款纪录。

![JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-11.png](JSON%E3%80%81%E6%BB%91%E6%9D%86%E7%9A%84%E8%BF%90%E7%94%A8%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%BF%87%E6%BB%A4%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20143dd0135957467abee1bdb63e9eb23c/swiftui-json-11.png)

图 21.11. 显示筛选器视图

### 本章小结

本章介绍了很多的内容，你应该知道如何使用 Web API、解析 JSON 内容，以及在清单视图中显示数据，我们还简要介绍了滑杆控制组件的用法。

如果你之前使用 UIKit 开发过 App，你可能会对 SwiftUI 的简单性感到惊讶。再看一下 ContentView 的代码，只需要 40 行的代码就可以建立清单视图。最重要的是，你不需要手动处理 UI 修改及传送数据，一切都在背后运作。

为了参考，你可以至下列的网址下载完整的项目：

- 示例项目 ([https://www.appcoda.com/resources/swiftui4/SwiftUIKivaLoan.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIKivaLoan.zip))