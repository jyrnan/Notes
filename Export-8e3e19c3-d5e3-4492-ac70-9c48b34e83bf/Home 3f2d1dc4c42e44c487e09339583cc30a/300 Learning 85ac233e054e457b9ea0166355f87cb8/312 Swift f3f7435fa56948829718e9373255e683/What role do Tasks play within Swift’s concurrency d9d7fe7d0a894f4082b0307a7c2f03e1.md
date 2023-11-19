# What role do Tasks play within Swift’s concurrency system? | Swift by Sundell

![the-role-tasks-play-in-swift-concurrency.png](What%20role%20do%20Tasks%20play%20within%20Swift%E2%80%99s%20concurrency%20d9d7fe7d0a894f4082b0307a7c2f03e1/the-role-tasks-play-in-swift-concurrency.png)

When writing asynchronous code using Swift’s new built-in concurrency system, creating a `Task` gives us access to a new asynchronous context, in which we’re free to call `async`-marked APIs, and to perform work in the background.

But besides enabling us to encapsulate a piece of asynchronous code, the `Task` type also lets us control the way that such code is run, managed, and potentially cancelled.

**[Bitrise:** Rock-solid, fast continuous integration that’s super easy to configure. In just a few minutes, you can set up builds, tests, and automatic App Store and beta deployments for your project, all running in the cloud on every pull request and commit. Try it for free today.](https://bitrise.io/swift)

Perhaps the most common way to use a `Task` within UI-based apps is to have it act as a bridge between our synchronous, main thread-bound UI code, and any background operations that are used to fetch or process the data that our UI is rendering.

For example, here we’re using a `Task` within a UIKit-based `ProfileViewController` to be able to use an `async`-marked API to load the `User` model that our view controller should render:

One thing that’s really interesting about the above code is that there are no `self` captures, no `DispatchQueue.main.async` calls, no tokens or cancellables that need to be retained, or any other kind of “bookkeeping” that we normally have to do when performing asynchronous operations using tools like closures or Combine.

```swift
class ProfileViewController: UIViewController {
    private let userID: User.ID
    private let loader: UserLoader
    private var user: User?
    ...

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        Task {
            do {
                let user = try await loader.loadUser(withID: userID)
                userDidLoad(user)
            } catch {
                handleError(error)
            }
        }
    }

    ...

    private func handleError(_ error: Error) {
        // Show an error view
        ...
    }

    private func userDidLoad(_ user: User) {
        // Render the user's profile
        ...
    }
}
```

So how exactly are we able to perform a network call (which is definitely going to be executed on a background thread), and then directly call UI-updating methods like `userDidLoad` and `handleError`, without first manually dispatching those calls using `DispatchQueue.main`?

This is where Swift’s new `[MainActor](https://www.swiftbysundell.com/articles/the-main-actor-attribute)` attribute comes in, which automatically ensures that UI-related APIs (such as those defined within a `UIView` or `UIViewController`) are correctly dispatched on the main thread. So, as long as we’re writing our asynchronous code using Swift’s new concurrency system, and within such a `MainActor`-marked context, then we no longer have to worry about accidentally performing UI updates on a background queue. Neat!

Another thing that’s interesting about our above implementation is that we’re not required to manually retain our loading task in order for it to complete. That’s because asynchronous tasks are not automatically cancelled when their corresponding `Task` handle is deallocated — they just keep executing in the background.

## [Referencing and cancelling a task](https://www.swiftbysundell.com/articles/the-role-tasks-play-in-swift-concurrency/#referencing-and-cancelling-a-task)

However, in this particular case, we probably *do* want to maintain a reference to our loading task, since we might want to cancel it when our view controller disappears, and we probably also want to prevent duplicate tasks from being performed in case the system calls `viewWillAppear` while a task is already in progress:

```swift
class ProfileViewController: UIViewController {
    private let userID: User.ID
    private let loader: UserLoader
    private var user: User?
    private var loadingTask: Task<Void, Never>?
    ...

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        guard loadingTask == nil else {
            return
        }

        loadingTask = Task {
            do {
                let user = try await loader.loadUser(withID: userID)
                userDidLoad(user)
            } catch {
                handleError(error)
            }

            loadingTask = nil
        }
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        loadingTask?.cancel()
loadingTask = nil
    }

    ...
}
```

Note how a `Task` has two generic types — the first indicates what type of output that it returns (which is `Void` in our case, since our task simply forwards its loaded `User` model onto our view controller’s methods), and the second is for its error type (and since we handle all errors within our task itself, that error type is `Never` in this case).

Calling the `cancel` method on a `Task` also marks all of its child-tasks as cancelled as well. So by cancelling our top-level `loadingTask` within our view controller, we’re also implicitly cancelling its underlying network operations at the same time.

However, note that it’s up to each individual task to implement the actual cancellation handling code required to cancel its particular operation. So, even though the system will automatically manage and propagate cancellation in terms of *marking*, it’s up to each task to decide how to actually *handle* that cancellation (for example by calling `Task.checkCancellation` within its closure).

## [Context inheritance](https://www.swiftbysundell.com/articles/the-role-tasks-play-in-swift-concurrency/#context-inheritance)

The relationship between a given `Task` and its parent can be quite important, at least within `@MainActor`-marked classes, such as views and view controllers. That’s because child tasks are not only connected to their parents in terms of cancellation — they also automatically inherit the same execution context as their parent uses.

To illustrate when that behavior can become somewhat problematic, let’s imagine that our `ProfileViewController` was loading its `User` model from a local database, rather than over the network, and that our `Database` API is currently completely synchronous.

At first glance, it could seem like the following implementation will be completely fine, since we might expect that our asynchronous work will still be executed on a background thread (even if we no longer perform any `await`-based calls within our `Task`):

```swift
class ProfileViewController: UIViewController {
    private let userID: User.ID
    private let database: Database
    private var user: User?
    private var loadingTask: Task<Void, Never>?
    ...

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        guard loadingTask == nil else {
            return
        }

        loadingTask = Task {
            do {
                let user = try database.loadModel(withID: userID)
                userDidLoad(user)
            } catch {
                handleError(error)
            }

            loadingTask = nil
        }
    }

    ...
}
```

However, although the above `Task` will indeed be performed asynchronously, it will still be executed on the main thread, since it’s being dispatched using the `MainActor` (it inherits that context from the `viewWillAppear` method that it was created in). So, essentially, our above `Task` is more or less equivalent to performing that same database call within a `DispachQueue.main.async` closure.

Since we probably want to move our database call away from the main thread (to prevent that call from interfering with the responsiveness of our UI), we could instead use a `detached` task — which will be executed within its own, stand-alone context. When doing so, we’ll also have to use `await` when calling back into our view controller’s methods, since those methods are isolated by the `MainActor` (we can also no longer set our `loadingTask` property to `nil` directly within our task):

```swift
class ProfileViewController: UIViewController {
    ...

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        guard loadingTask == nil else {
            return
        }

        loadingTask = Task.detached(priority: .userInitiated) { [weak self] in
            guard let self = self else { return }

            do {
                let user = try self.database.loadModel(withID: self.userID)
                await self.userDidLoad(user)
            } catch {
                await self.handleError(error)
            }

            await self.loadingTaskDidFinish() //跨域需要awaitby jy
        }
    }

    ...

    private func loadingTaskDidFinish() {
        loadingTask = nil
    }
}
```

In general, it’s recommended to only use `detached` tasks when we explicitly want to create a new top-level task that uses its own execution context. In other situations, simply using `Task {}` to encapsulate our asynchronous code is the recommended way to go.

Finally, let’s take a look at how we can `await` the result of a given `Task` instance. For example, let’s say that we wanted to extend our above `Database`-based view controller implementation with support for loading the current user’s image over the network.

To do that, we’ll wrap our `detached` task within yet another `Task` instance, and we’ll then use the `await` keyword to wait for our database loading operation to complete before proceeding with our image download — like this:

```swift
class ProfileViewController: UIViewController {
    private let userID: User.ID
    private let database: Database
    private let imageLoader: ImageLoader
    private var user: User?
    private var loadingTask: Task<Void, Never>?
    ...

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        guard loadingTask == nil else {
            return
        }

        loadingTask = Task {
            let databaseTask = Task.detached(
    priority: .userInitiated,
    operation: { [database, userID] in
        try database.loadModel(withID: userID)
    }
)

            do {
                let user = try await databaseTask.value
                let image = try await imageLoader.loadImage(from: user.imageURL)
                userDidLoad(user, image: image)
            } catch {
                handleError(error)
            }

            loadingTask = nil
        }
    }

    ...

    private func userDidLoad(_ user: User, image: UIImage) {
        // Render the user's profile
        ...
    }
}
```

Note how we can once again call our view controller’s methods directly within our top-level `Task`, since it’s now `MainActor`-bound, just like before. That illustrates just how smoothly we can now mix work that’s performed both on and off the main queue, without having to worry about accidentally performing UI updates on the wrong thread.

Support Swift by Sundell by checking out this sponsor:

**[Bitrise:** Rock-solid, fast continuous integration that’s super easy to configure. In just a few minutes, you can set up builds, tests, and automatic App Store and beta deployments for your project, all running in the cloud on every pull request and commit. Try it for free today.](https://bitrise.io/swift)

## [Conclusion](https://www.swiftbysundell.com/articles/the-role-tasks-play-in-swift-concurrency/#conclusion)

Swift’s new `Task` type enables us to encapsulate, observe, and control a unit of asynchronous work — which in turn lets us call `async`-marked APIs, and perform background work, even within code that’s otherwise completely synchronous. That way, we can gradually introduce `async` functions and the rest of Swift’s new concurrency system, even within applications that weren’t designed with those new features in mind.

I hope that you found this article interesting and useful. If you have any questions, comments, or feedback, then feel free to [reach out via email](https://www.swiftbysundell.com/contact).

Thanks for reading!