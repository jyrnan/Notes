# SwiftUI自动生成persistantController理解

```swift
import CoreData
import UIKit

///PersistantceController是自定义的结构，可以自己修改。
///核心是A处的container
struct PersistenceController {
    
    static let shared = PersistenceController()

        
    ///A：container才是CoreData内部定义的类。
    ///Container是一个封装体，内部封装了CoreData Stack的内容
    ///包括： NSManagedObjectModel、NSPersistentStoreCoordinator、context三件
    ///其中Model 和coordinator是不会变的，每次新生成的一个container都会指向同一个
    ///但是context会变化，可能有多个。所以会存在多线程的问题。context的数据不安全，但是CoreData会处理好
    ///每次利用这个PersistantController会产生一个不同的实例，但是都会指向同一个model和coordinator，但生成个字的context。
    let container: NSPersistentCloudKitContainer

    init(inMemory: Bool = false) {
		
///初始化第一步：先初始一个Container
        container = NSPersistentCloudKitContainer(name: "Notes")
        if inMemory {
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }

		///初始化第二步：load persistantStore
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.

                /*
                Typical reasons for an error here include:
                * The parent directory does not exist, cannot be created, or disallows writing.
                * The persistent store is not accessible, due to permissions or data protection when the device is locked.
                * The device is out of space.
                * The store could not be migrated to the current model version.
                Check the error message to determine what the actual problem was.
                */
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        
        setupNotificationHandling()
        print(#line, "Persistant inited")
    }
```