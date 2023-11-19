# Swift 中 key paths 的能力

# **前言**

自从 swift 刚开始就被设计为是编译时安全和静态类型后，它就缺少了那种我么经常在运行时语言中的动态特性，比如 Object-C, Ruby 和 JavaScript。举个例子，在 Object-C 中，我们可以很轻易的动态去获取一个对象的任意属性和方法 - 甚至可以在运行时交换他们的实现。

虽然缺乏动态性正是 Swift 如此强大的一个重要原因 - 它帮助我们编写更加可以预测的代码以及更大的保证了代码编写的准确性, 但是有的时候，能够编写具有动态特性的代码是非常有用的。

值得庆幸的是，Swift 不断获取越来越多的更具动态性的功能，同时还一直把它的关注点放在代码的类型安全上。其中的一个特性就是 KeyPath。这周，就让我们来看看 KeyPath 是如何在 Swift 中工作的，并且有哪些非常酷非常有用的事情可以让我们去做。

# **基础**

key paths 基本上让我们将任何实例属性引用为单独的值。因此，它们可以通过表达式传递，并使一段代码能够获取或设置一个属性而无需实际了解该属性。

**Key paths 有三种主要变种：**

- `KeyPath`：提供对属性的只读访问权限。
- `WritableKeyPath`: 提供对具有值语义的可变属性的读写访问权限（因此所讨论的实例也需要是可变的，以便允许的写入）。
- `ReferenceWritableKeyPath`: 只能与引用类型（例如类的实例）一起使用，并为任何可变属性提供读写访问权限。

还有一些额外的 key paths 类型，即可以减少内部代码复制并帮助类型擦除，但我们将专注于本文中的主要类型。

让我们深入查看如何使用 key paths，是什么让他们有趣和潜在的强大。

# **功能表达**

假设我们正在构建一个应用程序，让用户读取来自 Web 的文章，并且我们有一个用来代表一个这样的文章的 Article 模型，看起来像这样：

```swift
struct Article {
    let id: UUID
    let source: URL
    let title: String
    let body: String
}
```

每当我们使用这些模型的数组时，希望从每个型号中提取一个数据来形成一个新数组 —— 例如在以下两个示例中，我们正在收集所有 ID 和所有文章的来源：

```bash
let articleIDs = articles.map { $0.id }
let articleSources = articles.map { $0.source }
```

虽然上面完全有效，因为我们仅仅对从每个实例提取单个值有兴趣，但我们真的不需要闭包的全部能力，因此使用 key paths 可能非常适合。
我们将首先扩展 `Sequence` 来添加 `map` 的重载，该 `map` 采用 key paths 而不是闭包。由于我们只对此用例的只读属性访问感兴趣，因此我们将使用标准的 `KeyPath`，并且实际执行数据提取，我们将使用与给定键路径的子项作为参数使用，如下所示：

```swift
extension Sequence {
    func map<T>(_ keyPath: KeyPath<Element, T>) -> [T] {
        return map { $0[keyPath: keyPath] }
    }
}
```

注意：如果您使用的 Swift 5.2 或更高版本，则不再需要上述扩展，因为现在可以将 key paths 自动转换为函数。
通过以上扩展，我们现在能够使用一个非常好的和简单的语法来从任何序列中的每个元素中提取单个值，使得可以从之前转换我们的示例：

```swift
let articleIDs = articles.map(\.id)
let articleSources = articles.map(\.source)
```

这是非常酷的，但是，当 key paths 真正开始发光时，它们用于形成稍微复杂的表达式，例如在排序一系列值时。
标准库能够自动对包含 `Sortable` 元素的任何序列进行排序，但对于所有其他类型，我们必须提供自己的排序闭包。但是，使用 key paths，我们可以通过基于 `Comparable` 的 key patsh 轻松添加用于对任何序列进行排序的支持。就像之前一样，我们将在序列 `Sequence` 协议中添加一个扩展，将给定 key paths 转换为排序表达式闭包：

```swift
extension Sequence {
    func sorted<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        return sorted { a, b in
            return a[keyPath: keyPath] < b[keyPath: keyPath]
        }
    }
}
```

使用上面的扩展，我们现在能够快速且轻松地对任何序列进行排序，只需给出我们想要排序的 key paths。如果我们正在构建任何形式的可排序列表的应用程序 —— 例如包含播放列表的音乐应用程序 —— 这非常方便，因为我们现在自由地对我们的列表进行排序，甚至是嵌套的）：

```swift
playlist.songs.sorted(by: \.name)
playlist.songs.sorted(by: \.dateAdded)
playlist.songs.sorted(by: \.ratings.worldWide)
```

这样做的似乎只是简单地添加了一个语法糖，但可以制作一些更复杂的代码处理的序列同时更容易阅读，并且还可以帮助减少代码复制，因为我们现在能够为任何属性重用相同的排序代码。

# **不需要实例**

虽然适量的语法糖很好，但是关键路径的真正的威力来自于，它可以让我们引用属性而不必与任意的实例相关联。延续使用之前的音乐主题，假设我们正在开发一个展示歌曲列表的 App - 并且在 UI 中为这个列表配置 `UITableViewCell`，我们使用如下的配置类型:

```swift
struct SongCellConfigurator {
    func configure(_ cell: UITableViewCell, for song: Song) {
        cell.textLabel?.text = song.name
        cell.detailTextLabel?.text = song.artistName
        cell.imageView?.image = song.albumArtwork
    }
}
```

再次声明，上面的代码没有一点问题，但是我们期望以这样的方式渲染其他的模型的概率非常的高（非常多的 tableView 的 cells 尝试着去渲染标题，副标题以及图片而不用去管他们代表的是什么模型）- 因此让我们看看，我们能否用关键路径的威力去创建一个共享的配置实现，让他可以被任意的模型使用。
让我们创建一个名叫 CellConfigurator 的泛型，然后因为我们想要用不同的模型去渲染不同的数据，所以我们将会给它提供一组基于关键路径的属性 - 我们先渲染其中的一个数据:

```swift
struct CellConfigurator<Model> {
    let titleKeyPath: KeyPath<Model, String>
    let subtitleKeyPath: KeyPath<Model, String>
    let imageKeyPath: KeyPath<Model, UIImage?>

    func configure(_ cell: UITableViewCell, for model: Model) {
        cell.textLabel?.text = model[keyPath: titleKeyPath]
        cell.detailTextLabel?.text = model[keyPath: subtitleKeyPath]
        cell.imageView?.image = model[keyPath: imageKeyPath]
    }
}
```

上面的实现优雅的地方在于，我们现在可以为每个模型定制我们的 CellConfigurator，使用相同的轻量的关键路径语法，如下所示:

```swift
let songCellConfigurator = CellConfigurator<Song>(
    titleKeyPath: \.name,
    subtitleKeyPath: \.artistName,
    imageKeyPath: \.albumArtwork
)

let playlistCellConfigurator = CellConfigurator<Playlist>(
    titleKeyPath: \.title,
    subtitleKeyPath: \.authorName,
    imageKeyPath: \.artwork
)
```

就像标准库中的 `map` 和 `sorted` 等函数的操作一样，我们曾经可能会使用闭包去实现 `CellConfigurator`。然而，通过关键路径，我们能够使用一个非常好的语法去实现它 - 并且我们也不需要任何的订制化的操作去不得不通过模型实例去处理 - 使它们变得更加的简单，更加的具有说服力。

# **转化为函数**

目前为止，我们仅仅使用关键路径来读取值 - 现在让我们看看我们如何使用它们来动态的写值。在很多不同的代码中，我们常常可以见到一些像下面的代码一样的列子 - 我们通过这段代码来加载一系列的事项，然后在 `ListViewController` 中去渲染它们，然后当加载操作完成后，我们会简单的将加载的事项赋值给视图控制器中的属性。

```swift
class ListViewController {
    private var items = [Item]() { didSet { render() } }

    func loadItems() {
        loader.load { [weak self] items in
            self?.items = items
        }
    }
}
```

让我们看看，通过关键路径赋值能否让上面的语法简单一点，并且能够移除我们经常使用的 `weak self` 的语法（如果我们忘记对 self 的引用前加上 `weak` 关键字的话，那么就会产生循环引用）。

既然所有上面我们做的事情都是获取传递给我们闭包的值，并将它赋值给视图控制器中的属性 - 那么如果我们真的能够将属性的 `setter` 作为函数传递，会不会很酷呢？这样我们就可以直接将函数作为完成闭包传递给我们的加载方法，然后所有的事情都会正常执行。

为了实现这一目标，首先我们先定义一个函数，让任意的可写的转化为一个闭包，然后为关键路径设置属性值。为此，我们将会使用 `ReferenceWritableKeyPath` 类型，因为我们只想把它限制为引用类型（否则的话，我们只会改变本地属性的值）。给定一个对象，以及给这个对象设置关键路径，我们将会自动将捕获的对象作为弱引用类型，一旦我们的函数被调用，我们就会给匹配关键路径的属性赋值。就像这样:

```swift
func setter<Object: AnyObject, Value>(
    for object: Object,
    keyPath: ReferenceWritableKeyPath<Object, Value>
) -> (Value) -> Void {
    return { [weak object] value in
        object?[keyPath: keyPath] = value
    }
}
```

使用上面的代码，我们可以简化之前的代码，将弱引用的 self 去除，然后用看起来非常简洁的语法结尾：

```swift
class ListViewController {
    private var items = [Item]() { didSet { render() } }

    func loadItems() {
        loader.load(then: setter(for: self, keyPath: \.items))
    }
}
```

非常酷有没有！或许它还能变得更加的酷，当上面的代码跟更加先进的函数式编程思想结合在一起的时候，如组合函数 - 因此我们现在可以将多个 setter 函数和其他的函数链接在一起使用。在接下来的文章中，我们将介绍函数式编程和组合函数。

# **总结**

首先，看起来如何以及何时去使用 swift 关键路径这样的功能有点困难，并且很容易将它们看做是简单的语法糖。能够使用更加动态的方法去引用属性是一件非常强大的事情，即使闭包通常可以做很多类似的事情，但是轻量的语法以及关键路径的声明，都使他们能够成为处理非常多种类的数据的好的匹配。