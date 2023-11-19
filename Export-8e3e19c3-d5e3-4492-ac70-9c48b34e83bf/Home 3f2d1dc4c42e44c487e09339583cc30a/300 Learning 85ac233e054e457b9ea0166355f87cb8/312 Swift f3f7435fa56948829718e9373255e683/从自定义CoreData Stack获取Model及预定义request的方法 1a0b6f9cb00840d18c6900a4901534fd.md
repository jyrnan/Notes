# 从自定义CoreData Stack获取Model及预定义request的方法

viewContext会有一个指向persistantStoreCoordinator的索引，通过它可以获取objectModel，从而进一步获取model里面预先定义好的request

```swift
guard let model = coreDataStack.managedContext.persistentStoreCoordinator?.managedObjectModel,
          let fetchRequest = model.fetchRequestTemplate(forName: "FetchRequest") as? NSFetchRequest<Venue>
          else {
      return
    }
```