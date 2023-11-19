# Async let 和 TaskGroup重点

```swift
import UIKit
import _Concurrency

func asyncWork(num: Int) async -> Int {
    let delay = UInt64(arc4random_uniform(8))
    try? await Task.sleep(nanoseconds: 1000000000 * delay)
    print(#function, "delay:  \(delay) seconds")
    return Int(delay)
}

Task{
    print("Started")
    
    let result = await withTaskGroup(of: Int.self, returning: [Int].self) { group in
        for i in 1..<5 {
            group.addTask {
                await asyncWork(num: i)
            }
        }
        var result: [Int] = []
        for await i in group {
            print(i)
            result.append(i)
        }
        return result
    }
    print(result)
    print("Ended")
}
```

`TaskGroup`中：

- of: 是单个任务返回的类型
- returning: 是整个任务组返回的类型（可选）

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