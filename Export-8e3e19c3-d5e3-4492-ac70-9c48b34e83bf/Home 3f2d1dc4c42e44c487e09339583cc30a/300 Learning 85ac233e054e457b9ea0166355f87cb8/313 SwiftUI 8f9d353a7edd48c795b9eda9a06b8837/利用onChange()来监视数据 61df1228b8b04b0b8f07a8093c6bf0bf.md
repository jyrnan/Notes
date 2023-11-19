# 利用onChange()来监视数据

- Adds a modifier for this view that fires an action when a specific value changes.
- SwiftUI lets us attach an **onChange()** modifier to any view, which will run code of our choosing when some state changes in our program. This is important, because we can’t use property observers like **didSet** with something like **@State**. 用onChange代替didSet……
    
    ![struct ContentViewView.png](%E5%88%A9%E7%94%A8onChange()%E6%9D%A5%E7%9B%91%E8%A7%86%E6%95%B0%E6%8D%AE%2061df1228b8b04b0b8f07a8093c6bf0bf/struct_ContentViewView.png)