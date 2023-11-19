# 认识 Swift 中的异步与并发 - Cubox

> 
> 
> 
> 作者：nuomi1，目前就职于格隆汇，Swift with iOS，果粉 / 米家粉。
> 
> 审核：
> 
> 四娘，iOS 开发，老司机技术周报成员。目前就职于格隆汇，对 Swift 和编译器相关领域感兴趣
> 
> Damonwong，iOS 开发，老司机技术周报编辑，就职于淘系技术部
> 

本文基于 Session 10132[1] / 10134[2] / 10133[3] 梳理，使用「获取缩略图」的例子对 Swift 5.5 新增的异步与并发编程进行讲解，如需了解更多，欢迎查看其它 Session。

## 异步

异步编程在 iOS 开发里是一个常见的操作，例如我们经常需要在网络请求回来之后更新数据模型和视图。但是当异步操作嵌套时，不仅容易出现逻辑错误，还可能会陷入回调地狱。

### `completion` 回调

我们用「获取缩略图」来举例，先看下下面这段代码，找下有哪些逻辑问题：

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else {
                return
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    return
                }
                completion(thumbnail, nil)
            }
        }
    }
    task.resume()
}

```

是的，少部分人可以发现，以上代码有两处错误，都是在 `guard` 语句之后直接退出，没有调用 `completion`，这样外部调用就无法处理所有可能的情况。而且不调用 `completion` 是一个合法（但不符合预期）的行为，编译器不会产生错误。

把遗留的两个 `completion` 补全后，代码如下：

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else {
                completion(nil, FetchError.badImage)
                return
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    completion(nil, FetchError.badImage)
                    return
                }
                completion(thumbnail, nil)
            }
        }
    }
    task.resume()
}

```

为了处理这些异步函数的结果，需要使用五个 `completion` 来通知调用方，并且调用方需要判断 `UIImage?` 和 `Error?` 的 4 种组合结果，但实际上有效的只有 2 种。

我们先用 Swift 标准库里面的 `Result` 来解决这个问题，代码如下：

```swift
func fetchThumbnail(for id: String, completion: @escaping (Result<UIImage, Error>) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(.failure(error))
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(.failure(FetchError.badID))
        } else {
            guard let image = UIImage(data: data!) else {
                completion(.failure(FetchError.badImage))
                return
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    completion(.failure(FetchError.badImage))
                    return
                }
                completion(.success(thumbnail))
            }
        }
    }
    task.resume()
}

```

只要调用方可以知道结果，那么结果一定只有 2 种，要么成功，要么失败，不需要判断其他情况，更加符合使用习惯，但是这样仍然无法避免 `completion` 被忽略调用的情况。

### 使用 `async` 和 `await` 重构函数

为了解决 `completion` 被忽略调用的情况，我们可以尝试使用 Swift 5.5 新增的 `async` 和 `await` 来解决这个问题，代码如下：

```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)
    let (data, response) = try await URLSession.shared.data(for: request)
    
		guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }
    
		let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
    return thumbnail
}

```

步骤非常简单：

1. 生成 `request` 请求，这是一个同步操作；
2. 请求数据，`try` 标记可能抛出错误，`await` 标记潜在暂停点，这是一个异步操作；
3. 判断返回状态码是否 200，不是则抛出错误；
4. 生成原始图片，这是一个同步操作；
5. 生成缩略图，`await` 标记潜在暂停点，无法生成则抛出错误，这是一个异步操作；
6. 返回成功生成的缩略图。

相较于前面 `completion` 版本的 23 行，这里 `async` 版本仅需要 8 行，没有函数的层层嵌套，都是一步接一步线性的，易读性大大提升，同时编译器也可以检查错误。

### 它们是如何工作的

[a normal function call](%E8%AE%A4%E8%AF%86%20Swift%20%E4%B8%AD%E7%9A%84%E5%BC%82%E6%AD%A5%E4%B8%8E%E5%B9%B6%E5%8F%91%20-%20Cubox%207f686e118ad14efa836652aa2af7c04d/filtersno_upscale())

a normal function call

上图是一个普通函数调用，当 `thumbnailURLRequest` 完成之后，返回 `fetchThumbnail` 继续执行后续代码，如果这个函数是一个耗时任务，那么当前线程就会持续等待，直到完成。

[an asynchronous function call](%E8%AE%A4%E8%AF%86%20Swift%20%E4%B8%AD%E7%9A%84%E5%BC%82%E6%AD%A5%E4%B8%8E%E5%B9%B6%E5%8F%91%20-%20Cubox%207f686e118ad14efa836652aa2af7c04d/filtersno_upscale()%201)

an asynchronous function call

上图是一个异步函数调用，当 `data(for:)` 调用后，函数被挂起进行等待，当任务完成后，系统恢复 `data(for:)` 的调用，返回 `fetchThumbnail` 继续执行后续代码。

### async / await facts

`async` / `await` 实际上的意思：

1. `async` 允许一个函数被挂起；
2. `await` 标记一个异步函数的潜在暂停点；
3. 在挂起期间可能会进行其他工作；
4. 一旦等待的异步调用完成，在 `await` 之后恢复执行。

也就是说，我们用 `async` 标记一个函数是异步函数，这仅仅是一个标记，并不意味着函数一定是异步操作，函数内可以是一个简单的加法（例如 `1 + 1`），也可以是一个异步的网络请求。当我们调用 `async` 函数时，需要使用 `await` 关键字进行调用，`await` 标记一个潜在暂停点。异步函数在执行同步代码时，不能放弃自己的线程，当执行到潜在暂停点时，会放弃自己的线程，挂起并等待被调用的异步函数的结果。当被调用的异步函数完成时，控制返回原来的异步函数的潜在暂停点（需要注意这时的线程不一定和之前的相同），继续执行之后的代码。这些线程切换的工作不需要我们手动处理，由 Swift 自动管理，极大地提高了编程效率。

### 小结

`async` / `await` 的加入让我们得以使用与同步编程类似的控制流来进行异步编程，不仅可以解决 `completion` 容易被忽略的问题，还可以更好地进行错误处理。与此同时，线性的控制流也避免了回调地狱的问题，大大提高代码的可读性。

## 并发

上个章节我们介绍了如何获取单张图片的缩略图，但在我们日常开发中更多的是展示列表，这就需要同时处理多张缩略图。

### `completion` 回调处理多个缩略图

如果我们一个接着一个地处理多个缩略图，可能会写出以下代码：

```swift
func fetchThumbnails(
    for ids: [String],
    completion handler: @escaping ([String: UIImage]?, Error?) -> Void
) {
    guard let id = ids.first else { return handler([:], nil) }
    let request = thumbnailURLRequest(for: id)
    let dataTask = URLSession.shared.dataTask(with: request) { data, response, error in
        guard let response = response,
              let data = data
        else {
            return handler(nil, error)
        }
        UIImage(data: data)?.prepareThumbnail(of: thumbSize) { image in
            guard let image = image else {
                return handler(nil, ThumbnailFailedError())
            }
            fetchThumbnails(for: Array(ids.dropFirst())) { thumbnails, error in
                // 添加图片到 thumbnails
            }
        }
    }
    dataTask.resume()
}

```

代码非常糟糕，递归调用依次获取缩略图，并且难以整合这些缩略图。除此之外，代码可读性也很低。

### 使用 `async` 和 `await` 处理多个缩略图

我们参考上个章节对单张缩略图的处理，试着使用 `async` 和 `await` 进行重构，代码如下：

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    
		for id in ids {
        let request = thumbnailURLRequest(for: id)
        let (data, response) = try await URLSession.shared.data(for: request)
        try validateResponse(response)
        guard let image = await UIImage(data: data)?.byPreparingThumbnail(ofSize: thumbSize) else {
            throw ThumbnailFailedError()
        }
        thumbnails[id] = image
    }
    return thumbnails
}

```

步骤非常简单：

1. 声明 `thumbnails` 保存返回值；
2. 使用 `for-in` 循环创建任务；
3. 创建请求；
4. 下载图片；
5. 检查资源合法性；
6. 生成缩略图；
7. 保存缩略图。
8. 返回 `thumbnails`。

可以发现，仅仅是增加了 `for-in` 循环，就可以非常方便地顺序处理多张缩略图。

### 使用 `async-let` 处理结构化并发

上面例子的 `thumbSize` 是本地提供的，处理起来比较简单。假设 `thumbSize` 也要从网络接口获取，那么在生成缩略图的过程中会有两个网络请求，代码如下：

```swift
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
    let (data, _) = try await URLSession.shared.data(for: imageReq)
    let (metadata, _) = try await URLSession.shared.data(for: metadataReq)
    guard let size = parseSize(from: metadata),
          let image = await UIImage(data: data)?.byPreparingThumbnail(ofSize: size)
    else {
        throw ThumbnailFailedError()
    }
    return image
}

```

可以看到，在 `imageReq` 请求完成之后，才发起 `metadataReq` 请求，只有两个请求都完成之后，才能执行之后的代码。我们能不能让这两个请求同时进行呢？另外如果在两个请求和使用他们的返回结果之间有其他的无关任务，无关任务的执行必须等到两个请求完成之后，这无疑是一种浪费。

我们可以用 `async-let` 来解决这个问题，代码如下：

```swift
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
    
		async let (data, _) = URLSession.shared.data(for: imageReq)
    async let (metadata, _) = URLSession.shared.data(for: metadataReq)
    
		guard let size = parseSize(from: try await metadata),
          let image = try await UIImage(data: data)?.byPreparingThumbnail(ofSize: size)
    else {
        throw ThumbnailFailedError()
    }
    return image
}

```

使用 `async let` 标记 `data` 和 `metadata`，使用 `await` 进行访问，如果函数可以抛出错误，那么还需要使用 `try` 关键字。两个网络请求会同时进行，并且执行后续代码，即上文提到的无关任务。当需要访问结果的时候，系统会进行等待直到完成或者抛出错误。这样便能更高效地完成整个任务。

需要注意的是，这两个网络请求是并发的，假如第一个获取图片的任务失败了抛出错误需要退出函数，Swift 会自动将未等待的任务标记取消，然后等待它完成再退出函数。任务被标记取消并不意味着停止任务，只是通知不需要该任务的返回值。

### 调用异步函数处理多个缩略图

既然 `for-in` 内部的逻辑就是处理单个缩略图的逻辑，自然而然我们就会想到直接调用封装好的异步函数，代码如下：

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        try Task.checkCancellation()
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}

```

你应该发现了，在获取缩略图之前，调用了 `checkCancellation` 来进行任务检查。如上文所述，子任务被标记取消并不会停止任务，我们可以尽可能地检查任务是否被取消，提前退出，不再请求后续的缩略图。

如果希望任务取消后返回之前已经处理的缩略图，可以使用 `isCancelled` 进行判断，代码如下：

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        if Task.isCancelled { break }
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}

```

### 使用任务组进行处理

即使图片数据和元数据已经可以同时请求了，但是对于整个获取缩略图这个任务而言，仍然是一个接一个进行的，**如果我们想要同时进行多个任务，就需要引入任务组进行并发编程**，代码如下：

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
		     ///group是关键。先在group里创建任务   
				for id in ids {
            group.async {
                return (id, try await fetchOneThumbnail(withID: id))
            }
        }
				/// 再等待group获取任务的返回值并处理,
				/// 这里是asyncSequence
        for try await (id, thumbnail) in group {
            thumbnails[id] = thumbnail
        }
    }
    return thumbnails
}

```

步骤如下：

1. 声明 `thumbnails` 保存返回值；
2. 调用 `withThrowingTaskGroup` 创建任务组；
3. 使用 `for-in` 循环依次把任务放到任务组，任务被添加时会立刻执行，到达最大并发数量后进行等待；
4. 使用 `for-await-in` 循环依次取回已处理的缩略图，如果任务还没完成，等待直到完成，或者抛出错误。
5. 返回 `thumbnails`。

使用任务组可以非常方便地执行并发任务，线程切换和派发任务都由 Swift 进行管理，不需要我们编写复杂的控制逻辑。

### `async-let` 与任务组的关系

事实上，`async-let` 和任务组都有任务依赖树的概念，即一个父任务有一个或多个子任务，而这个父任务又是它的父任务的子任务。当一个子任务因为错误而导致父任务需要抛出错误退出时，这个子任务的兄弟任务会被标记取消，父任务需要等待这些任务完成或者抛出错误（但会忽略这些任务的返回值或者错误），才能真正退出。

`async-let` 可以看做简化版的任务组（实际上并不是任务组的语法糖），非常适合处理**不同返回类型**的异步任务，等待这些异步任务的返回结果，然后组合它们再继续后面的任务。

而任务组更适合处理**数量不定的相同返回类型**的异步任务，在遍历任务组的返回结果时可以使用一个或者多个返回结果。例如两个相同返回类型的任务，只需要处理第一个返回的结果，任务组非常方便，但 `async-let` 难以实现。

### 小结

<aside>
💡 并发的套路是：
* 先创建并发任务 `async let` 或者 `withThrowingTaskGroup { group in ...}`
* 再await获取任务的返回值

</aside>

只有 `async` / `await` 的代码是不具备并发能力的，结构化并发的加入让我们得以用少量的代码在 `async` / `await` 的基础上实现并发编程，同时能够继续使用错误处理和线性控制流。

## Actor

上两个章节我们介绍了异步与并发，语法简洁的同时功能强大。并发虽然高效，但是如果没有采取手段加以控制，很容易产生数据竞争的问题。

### 数据竞争

想象一下，下面代码的输出结果是什么：

```swift
class Counter {
    var value = 0    
		func increment() -> Int {
	        value = value + 1
        return value
    }
}
let counter = Counter()
let queue1 = DispatchQueue(label: "queue_1")
let queue2 = DispatchQueue(label: "queue_2")
queue1.async {
    print(counter.increment())
}
queue2.async {
    print(counter.increment())
}
```

`1,1`、`1,2`、`2,1`、`2,2`，都有可能，取决于 `value` 的写入和读取时机。

共享的可变状态会引发数据竞争，通常我们使用锁来解决数据竞争，但是锁的粒度不好控制，处理得不好还会引发死锁。

### Actor 模型

锁的粒度不好控制，也容易造成死锁，我们可以使用 Swift 5.5 新增的 `actor` 来解决数据竞争的问题。

那什么是 `Actor` 呢？（摘自 这里[4]）

`Actor` 是一种并发模型，由状态（State）、行为（Behavior）、邮箱（Mailbox）三者组成。

1. 状态：`actor` 持有的变量，由自身管理，避免并发环境下的锁问题；
2. 行为：`actor` 中的计算逻辑，通过 `actor` 接收到的消息来改变自身的状态；
3. 邮箱：`actor` 之间通讯的桥梁，内部使用 `FIFO` 队列来存储和处理消息，接收方从邮箱中获取消息。

`Actor` 模型描述了一组为避免并发编程问题的公理。

1. 所有 `actor` 状态都是本地的，外部无法访问；
2. `actor` 之间必须通过消息传递进行通讯；
3. 一个 `actor` 可以响应消息、退出新的 `actor`、改变内部状态、把消息发送给一个或多个 `actor`；
4. `actor` 可能会阻塞自己但是不应该阻塞运行的线程。

在 Swift 中，

- `actor` 是一种引用类型，并且不可以被继承。
- `actor` 内定义的变量和方法都是隔离的，在内部使用可以直接访问，而在外部需要使用 `await` 进行异步访问。
- 对 `actor` 的变量访问和方法调用都是消息，在同一时刻只有一条消息可以被处理，多条消息按照顺序依次被处理。当某一条消息需要 `await` 等待其他异步任务时，消息会被挂起进行等待，`actor` 得以处理邮箱中的后续消息，表现为 `actor` 是可重入的。这就需要我们使用额外的机制来处理重复的请求，避免资源的浪费。

### 使用 actor 实现 Counter

我们使用 `actor` 来解决 `Counter` 的数据竞争问题，代码如下：

```swift
actor Counter {
    var value = 0    
func increment() -> Int {
        value = value + 1
        return value
    }
}
let counter = Counter()
asyncDetached {
    print(await counter.increment())
}
asyncDetached {
    print(await counter.increment())
}
```

结果要么是 `1,2`，要么是 `2,1`，不会存在 `1,1` 和 `2,2` 的情况。

### 使用 actor 处理图片下载

我们试着使用 `actor` 实现一个图片下载器，代码如下：

```swift
actor ImageDownloader {
    private var cache: [URL: Image] = [:]    
func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            return cached
        }        
let image = try await downloadImage(from: url)        
cache[url] = image
        return image
    }
}
```

当我们同时获取两种相同的图片时，图片下载器会先发起一个异步任务，当进行到 `await` 等待下载结果时，函数被挂起，从而发起第二个异步任务。假如先取回的图片是正确的，后取回的却是错误的，那么最终缓存下来的会是坏的。我们需要解决这类问题，一种解决方案是把先取回的当做正确的，后取回的忽略，代码如下：

```swift
actor ImageDownloader {
    private var cache: [URL: Image] = [:]    
func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            return cached
        }        
let image = try await downloadImage(from: url)        
cache[url] = cache[url, default: image] // 保存的时候判断是不是已经有了数据
        return cache[url]
    }
}
```

更好的解决方案是把下载任务保存起来，先发起第一个异步任务并保存，如果在返回结果之前触发了第二次请求，那么就把上面保存的任务取出，并等待它完成。由于任务是同一个，所以只会发出一个网络请求，代码如下：

```swift
actor ImageDownloader {    
private enum CacheEntry {
        case inProgress(Task.Handle<Image, Error>)
        case ready(Image)
    }    
private var cache: [URL: CacheEntry] = [:]    
func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            switch cached {
            case .ready(let image):
                return image
            case .inProgress(let handle):
                return try await handle.get()
            }
        }        
let handle = async {
            try await downloadImage(from: url)
        }        
cache[url] = .inProgress(handle)        
do {
            let image = try await handle.get()
            cache[url] = .ready(image)
            return image
        } catch {
            cache[url] = nil
            throw error
        }
    }
}
```

原文上面代码中handle已经deprecated，我在下面更新了代码。但未验证。

```swift
actor ImageDownloader{
    private enum CacheEntry {
        case inProgress(Task<Image, Error>)
        case ready(Image)
    }
    
    private var cache: [URL: CacheEntry] = [:]
    
		/// 额外补充的方法（不确定）
    func downloadImage(from url: URL) async throws -> Image {
        return Image(systemName: "home")
    }
    
    func image(from url: URL) async throws -> Image? {
        if let cached = cache[url] {
            switch cached {
            case .ready(let image):
                return image
            case .inProgress(let task):
                return try await task.value
            }
        }
        
        let task = Task {
            try await downloadImage(from:url)
        }
        
        cache[url] = .inProgress(task)
        
        do {
            let image = try await task.value
            cache[url] = .ready(image)
            return image
        } catch {
            cache[url] = nil
            throw error
        }
    }
```

### 小结

`actor` 的加入，让我们得以用一种新的方式来进行异步与并发编程，同时不需要处理数据竞争的问题。在 `actor` 内访问自有属性和方法都是同步逻辑，对外则表现为异步逻辑，同一时刻只能处理一条消息，多条消息顺序处理，这些规则为 `actor` 的高并发性提供了保证。

## 总结

1. 异步代码的控制流难以编写，难以阅读，Swift 引入 `async` / `await` 解决异步编程的问题；
2. 异步不代表并发，只用 `async` / `await` 编写的代码不具有并发性，Swift 引入结构化并发让 `async` / `await` 编写的代码执行并发任务，同时解决一些控制流上的问题；
    1. async let / async group 的区别
3. 异步与并发虽好，随之而来的是数据竞争，Swift 引入 `actor` 来解决共享可变状态的问题，状态被隔离在 `actor` 内部，修改只能通过向 `actor` 发送信息并等待结果，消息被 `actor` 以同步的方式进行处理，避免同一时间对数据进行修改。

## 关注我们

我们是「老司机技术周报」，一个持续追求精品 iOS 内容的技术公众号。欢迎关注。

**关注有礼，关注【老司机技术周报】，回复「WWDC」，领取 《WWDC20 内参》**

### 支持作者

这篇文章的内容来自于 **《WWDC21 内参》**。在这里给大家推荐一下这个专栏，专栏目前已经创作了 102 篇文章，目前正在五折销售，只需要 29.9 元。点击【阅读原文】，就可以购买继续阅读 ~ 我们会在所有文章更新完毕恢复到原价。

**WWDC 内参** 系列是由老司机牵头组织的精品原创内容系列。已经做了几年了，口碑一直不错。主要是针对每年的 WWDC 的内容，做一次精选，并号召一群一线互联网的 iOS 开发者，结合自己的实际开发经验、苹果文档和视频内容做二次创作。

**今年我们也引入了审核机制，内容和质量上也有了比较大的提升，欢迎订阅。**

### 参考资料

[1]

10132: *https://developer.apple.com/videos/play/wwdc2021/10132/*

[2]

10134: *https://developer.apple.com/videos/play/wwdc2021/10134/*

[3]

10133: *https://developer.apple.com/videos/play/wwdc2021/10133/*

[4]

这里: *https://www.jianshu.com/p/d803e2a7de8e*