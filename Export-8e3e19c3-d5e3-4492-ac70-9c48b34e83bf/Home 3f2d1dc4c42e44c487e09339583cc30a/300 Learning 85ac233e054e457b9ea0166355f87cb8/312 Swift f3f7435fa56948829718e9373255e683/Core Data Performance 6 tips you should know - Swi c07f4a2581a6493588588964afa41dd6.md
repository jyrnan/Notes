# Core Data Performance: 6 tips you should know - SwiftLee

[imagegenerator.php](Core%20Data%20Performance%206%20tips%20you%20should%20know%20-%20Swi%20c07f4a2581a6493588588964afa41dd6/imagegenerator.php)

Writing Core Data code with performance in mind helps to prepare your app for the future. Your database might be small in the beginning but can easily grow, resulting in slow queries and decreased experience for the user.

Since I started writing the [Collect by WeTransfer](https://collect.bywetransfer.com/) app in 2017 I’ve been writing a lot of Core Data related code, touching it almost every day. With millions of users adding lots of content, performing Core Data related code has become an important skill in our team.

Runway connects with all your existing tools (think GitHub, CI, App Store Connect, etc.) to level-up your team’s release coordination and automation. Track sign-offs, automate everything from kickoff to submit to release, and avoid the usual cat-herding. [Get started for free](https://runway.team/?utm_source=sponsorship&utm_medium=website&utm_campaign=swiftlee&utm_content=feb_mar_22).

Over the years, we’ve developed lots of insights, which I’m happy to share with you through 5 tips you should know.

## 1: Make use of a background managed object context

One thing we didn’t from the start is making use of a background managed object context. We only used the view context to perform any Core Data related tasks: inserting new content, deleting content, fetching content, etc.

In the beginning, our app was relatively small. Making only use of the view context wasn’t really an issue and didn’t result in any visible performance penalties related to Core Data. Obviously, once our app started to grow, we realized that the view context was associated with the main queue. Slow queries blocked our UI and our app became less responding.

In general, best practice is to perform data processing on a background queue as it can be CPU intensive. Examples like importing JSON into Core Data could otherwise block the view context and result in unresponsiveness in the user interface.

The solution is to make use of a background managed object context. The latest APIs make it easy to create a new context from your persistent container:

```
let backgroundContext = persistentContainer.newBackgroundContext()
```

I recommend this method over the `NSManagedObjectContext(concurrenyType:)` initializer as it will automatically be associated with the `NSPersistentStoreCoordinator` and it will be set to consume `NSManagedObjectContextDidSave` broadcasts too. This keeps your background context in sync with the view context.

You can save this background context on a custom persistent container subclass. This way, you can reuse your background context and you only have to manage two contexts. This keeps your Core Data structure simple to understand and it prevents having multiple out of sync contexts.

If you only have to use the background context in a few places, you can also decide to use the `performBackgroundTask(_:)` method which creates a background context in place:

```
persistentContainer.performBackgroundTask { (backgroundContext) in
    // .. Core Data Code
}
```

However, this method creates a new `NSManagedObjectContext` each time it is invoked. You might want to consider using the shared background context if you’re dispatching more often to a background context.

Writing multi-threaded Core Data code is a lot more complex than using a single view context. The reason for this is that you can’t simply pass an `NSManagedObject` instantiated from a view context to a background context. Doing so would result in a crash and potential data corruption.

When it’s necessary to move a managed object from one queue to another you can make use of the `NSManagedObjectID` which is thread-safe:

```
let managedObject = NSManagedObject(context: persistentContainer.viewContext)

backgroundContext.perform {
    let object = try? backgroundContext.existingObject(with: managedObject.objectID)
}
```

## 2: Only save a managed object context if needed

Saving a managed object context commits all current changes to the context’s parent store. As you can imagine, this is not a cheap operation and it should only be used if needed to ensure performance in Core Data.

First of all, it’s important to check if there’s even anything to save. If there are no changes to commit, there’s also no reason to perform a save. By creating a `saveIfNeeded` method you allow yourself to easily built-in a check for this:

```
extension NSManagedObjectContext {

    /// Only performs a save if there are changes to commit.
    /// - Returns: `true` if a save was needed. Otherwise, `false`.
    @discardableResult public func saveIfNeeded() throws -> Bool {
        guard hasChanges else { return false }
        try save()
        return true
    }
}
```

Apart from using `saveIfNeeded` instead of `save()` you also need to consider whether a save makes sense. Although a context could have changes it’s not always needed to directly commit these changes.

For example, if you import multiple items into your database, you might only want to save after you’ve imported all items on your background context. A save is often followed by UI updates and multiple saves after each other could easily result in unnecessary reloads. Besides that, take into account that saved changes in a background context are merged into the view context, blocking the main queue shortly as well. Therefore, be conscious!

Fetching data is an expensive task and need to be as performant as possible to make your app prepared for large datasets. The following code is an often made mistake:

```
let fetchRequest: NSFetchRequest<Content> = NSFetchRequest(entityName: "Content")
let allContent = try! persistentContainer.viewContext.fetch(fetchRequest)
let allContentWithNames = allContent.filter { $0.name != nil && $0.name?.isEmpty == false }
```

This code will load all inserted objects into memory while it’s being filtered directly after to only remain with content having a name.

It’s much more performant to use predicates to only fetch the objects that are needed. The above filter can be written as followed with a `NSPredicate`:

```
let fetchRequest: NSFetchRequest<Content> = NSFetchRequest(entityName: "Content")
fetchRequest.predicate = NSPredicate(format: "%K != nil AND name != ''", #keyPath(Content.name))
let allContentWithNames = try! persistentContainer.viewContext.fetch(fetchRequest)
```

This has two advantages:

- Only the needed objects are loaded into memory
- You don’t need to iterate over all objects

Predicates are very flexible and should allow you to fetch the desired dataset in most cases while maintaining performance in Core Data.

Following up on the previous example it’s important to set fetch limits when you’re only going to display a part of the dataset.

For example, say that you only need the first 3 names of all content items. In this case, it would be unnecessary to load all content items having a name into memory. We could prevent this by setting a fetch limit:

```
let fetchRequest: NSFetchRequest<Content> = NSFetchRequest(entityName: "Content")
fetchRequest.predicate = NSPredicate(format: "%K != nil AND name != ''", #keyPath(Content.name))

// Set a fetch limit to only fetch the first 3 content items with a name.
fetchRequest.fetchLimit = 3
let contentWithNames = try! persistentContainer.viewContext.fetch(fetchRequest)
```

This code will only return the first 3 content items having a name.

Instead of iterating over a dataset deleting each object one by one it’s often more performant to use a `NSBatchDeleteRequest` which runs faster as it operates at the SQL level in the persistent store itself.

You can learn more about batch delete requests in my blog post [Using NSBatchDeleteRequest to delete batches in Core Data](https://www.avanderlee.com/swift/nsbatchdeleterequest-core-data/).

As with all code you write it’s important to know how to optimise and debug it once it’s not performing as expected. There’s many ways of debugging which are best explained in my dedicated blog post: [Core Data Debugging in Xcode using launch arguments](https://www.avanderlee.com/debugging/core-data-debugging-xcode/).

Runway connects with all your existing tools (think GitHub, CI, App Store Connect, etc.) to level-up your team’s release coordination and automation. Track sign-offs, automate everything from kickoff to submit to release, and avoid the usual cat-herding. [Get started for free](https://runway.team/?utm_source=sponsorship&utm_medium=website&utm_campaign=swiftlee&utm_content=feb_mar_22).

Writing performant Core Data code from the beginning helps you to prepare your app for future large datasets. Although your app might be performing at the beginning it can easily slow down once your database and model grows. By making use of a background context, smart fetch requests, and batch delete requests you’re making your Core Data code already more performant.

If you like to improve your Swift knowledge, even more, check out the [Swift category page](https://www.avanderlee.com/category/swift/). Feel free to [contact me](mailto:contact@avanderlee.com) or tweet to me on [Twitter](https://www.twitter.com/twannl) if you have any additional tips or feedback.

Thanks!