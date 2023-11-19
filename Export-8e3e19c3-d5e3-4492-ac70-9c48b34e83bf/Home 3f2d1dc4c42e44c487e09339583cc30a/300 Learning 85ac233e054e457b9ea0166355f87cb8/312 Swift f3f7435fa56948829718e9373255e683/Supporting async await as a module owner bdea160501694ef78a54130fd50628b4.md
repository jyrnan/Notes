# Supporting async/await as a module owner

In this article, you can read what you can do to support async/await in your module, keep it a minor release, and support a variety of versions and platforms. All with a few simple steps.

We’ll also briefly cover when to migrate to async/await and how to bridge existing methods.

![Supporting%20async%20await%20as%20a%20module%20owner%20bdea160501694ef78a54130fd50628b4/cover.png](Supporting%20async%20await%20as%20a%20module%20owner%20bdea160501694ef78a54130fd50628b4/cover.png)

Since Xcode 13.2, you can support Swift’s async/await on older platforms. To quote the

[Xcode 13.2 release notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-13_2-release-notes):

You can now use Swift Concurrency in applications that deploy to macOS Catalina 10.15, iOS 13, tvOS 13, and watchOS 6 or newer. This support includes async/await, actors, global actors, structured concurrency, and the task APIs.

But how does this work if you’re the owner of a module? Because you may have to support a wider range of platforms and versions. And as a module owner, you can’t always assume everyone is already upgraded. Maybe someone who uses your module with Xcode 13.2 on their local machine, might have a CI that uses Xcode 13.1, for instance. In which case your fancy async/await code will *not* compile.

And how do you decide what to update first? Do you need to support older methods? Do you need to update internals to async/await?

Let’s cover some approaches.

## Prioritize updating the public API over internals

In the end, the public API of your module is most important. The people implementing your module don’t “care” if your modules internally uses async/await, completion blocks, or pure handcrafted assembly.

But implementers do care about the public API of your module since this affects them, and they will want async/await there. They also expect that your module compiles, which we risk breaking if we’re not careful.

A good rule of thumb is to deem the public API as most important, and you will want to prioritize supporting async/await in the public API *before* you’re going to rewrite your internals to support async/await.

### Updating the public API

To support a wider range of versions and to keep your public API stable, it’s a good practice to support both async/await *and* the completion block variants of your public API.

Let’s see how.

Below we have an `ArticlesAPI` class which contains a `fetchArticles` method that fetches articles, returning a `Result`. This method uses a completion block.

```
public final class ArticlesAPI {

    public func fetchArticles(completion: @escaping (Result<[Article], ArticleError>) -> Void) {
       // .. implementation omitted
    }
}
```

Side-by-side, we can offer an async function with the same name. Depending on the context, Swift will pick the async/await version over your completion handler.

In other words: It is *safe* to offer an async version of the same name as a non-async method.

For more information about how Swift chooses async variants over completion blocks, please refer to [Overloading and overload resolution](https://github.com/apple/swift-evolution/blob/main/proposals/0296-async-await.md#overloading-and-overload-resolution) in the async/await proposal.

To offer the new method, we can wrap it in a compiler flag and check that Swift’s concurrency can be imported. This is to ensure that machines building with older Xcode versions — before 13.2 — can still compile your module.

On top of that, you can use the `@available` flag for runtime checks to ensure that the methods can be used for the specific platforms. For instance, if an implementer is building for iOS 12, this method will be unavailable as intended.

```swift
public final class ArticlesAPI {

// New compiler flag
#if compiler(>=5.5) && canImport(_Concurrency)

    // New method with an @available flag
    @available(macOS 10.15, iOS 13, *)
    public func fetchArticles() async throws -> [Article] {
      // .. implementation omitted
    }

#endif

    // "Older" method
    public func fetchArticles(completion: @escaping (Result<[Article], ArticleError>) -> Void) {
       // .. implementation omitted
    }

}
```

For the implementation of the new async method, it can use the existing “old” `fetchArticles` method. This way, you don’t have to write a double implementation. For example, let’s say you fix a bug in the original method, you will get the fix in the async version, too.

To convert a `completion` block into an async/await version, you can use the `withCheckedThrowingContinuation` function. In its closure, you call `continuation.resume(returning:)` to return a value, or you call `continuation.resume(throwing:)` to pass an error.

Below you can see how we convert a completion block with `Result` to an async version.

```
public final class ArticlesAPI {

#if compiler(>=5.5) && canImport(_Concurrency)

    @available(macOS 10.15, iOS 13, *)
    public func fetchArticles() async throws -> [Article] {
        return try await withCheckedThrowingContinuation { continuation in
          // We call the completion-block version here.
          fetchArticles { result in
                switch result {
                case .success(let articles):
                    // We pass the value to the continuation.
                    continuation.resume(returning: articles)
                case .failure(let error):
                    // We pass an error to the continuation.
                    continuation.resume(throwing: error)
                }
           }
        }
    }

#endif

   // .. snip
}
```

This works, but since we’re using `Result` we can shorten this, by using a convenience `resume(with:`) method on the continuation.

```
// This...
fetchArticles { result in
      switch result {
      case .success(let articles):
          continuation.resume(returning: articles)
      case .failure(let error):
          continuation.resume(throwing: error)
      }
}

// ... becomes
fetchArticles { result in
      continuation.resume(with: result)
}
```

We can even make it shorter by using a so-called *point-free* style. Which is a fancy way of saying that we pass the name of the function to another function.

```
fetchArticles(completion: continuation.resume(with:))
```

In the end, the entire new method becomes:

```
func fetchArticles() async throws -> [Article] {
    return try await withCheckedThrowingContinuation { continuation in
        fetchArticles(completion: continuation.resume(with:))
    }
}
```

There you go, with a few lines of code we now have an async implementation right next to the original implementation.

## Deprecations

If you’re really dead-set on not supporting completion blocks anymore, then give others some time and a notice to update. You can use the deprecation flag for this.

```
public final class ArticlesAPI {

#if compiler(>=5.5) && canImport(_Concurrency)

    @available(macOS 10.15, iOS 13, *)
    public func fetchArticles() async throws -> [Article] {
    func fetchArticles() async throws -> [Article] {
        return try await withCheckedThrowingContinuation { continuation in
            fetchArticles(completion: continuation.resume(with:))
        }
    }
#endif

    // New deprecation flag
    @available(*, deprecated, message: "Please use the async/await version of `fetchArticles`")
    public func fetchArticles(completion: @escaping (Result<[Article], ArticleError>) -> Void) {
       // .. implementation omitted
    }
}
```

Then after a period of time, you can remove the completion-block version and create a major release. Major releases can be a pain for others though, so I recommend to support the “old” method for a long time.

## Conclusion

As a module owner, you can see it’s painless to support both async and “regular” versions of the same asynchronous code. It goes a long way to keep your module stable, so that you give others the time to update your module to the latest version.

Now that your public API is in place, you can take your time to rewrite your internals to use async/await.

Keep in mind though, that once you replace completion blocks with async/await (even internally), you can’t support older versions anymore (before iOS 13 and such). So to keep your module from becoming a major release, I recommend you support both completion blocks and async/await in your module for a while.