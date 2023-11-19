# 自定义NavigationBar的字体和颜色

[设定字型与颜色](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E5%AF%BC%E8%88%AAUI%E4%B8%8E%E5%AF%BC%E8%88%AA%E5%88%97%E5%AE%A2%E5%88%B6%E5%8C%96%E8%BF%90%E7%94%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20560c759146fd46fa989167bf2fa22ff7.md) 

### 设定字型与颜色

接下来，我们来看如何修改标题的字型与颜色。在撰写本章时，SwiftUI 还没有修饰器来让开发人员设定导航列的字型及颜色，而我们需要使用 UIKit 所提供的 `UINavigation BarAppearance` API。

举例而言，我们要将导航列的标题颜色修改为红色、字型修改为 `Arial Rounded MT Bold`，则我们可以在 init() 函数中建立一个 `UINavigationBarAppearance` 物件，并相应地设定属性。在 `ContentView` 中插入下列的函数：

```
init() {
    let navBarAppearance = UINavigationBarAppearance()
    navBarAppearance.largeTitleTextAttributes = [.foregroundColor: UIColor.red, .font: UIFont(name: "ArialRoundedMTBold", size: 35)!]
    navBarAppearance.titleTextAttributes = [.foregroundColor: UIColor.red, .font: UIFont(name: "ArialRoundedMTBold", size: 20)!]

    UINavigationBar.appearance().standardAppearance = navBarAppearance
    UINavigationBar.appearance().scrollEdgeAppearance = navBarAppearance
    UINavigationBar.appearance().compactAppearance = navBarAppearance
}

```

`largeTitleTextAttributes` 属性用于设定大尺寸标题的文字属性，而 `titleTextAttributes` 属性则用于设定标准尺寸标题的文字属性。当我们设定 `navBarAppearance` 后，将其它指定给三个外观属性，包括`standardAppearance`、`scrollEdgeAppearance` 与 `compactAppearance`。如前所述，如果需要的话，你可以为 `scrollEdgeAppearance` 与 `compactAppearance` 建立及指定一个单独的外观物件。

![../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E5%AF%BC%E8%88%AAUI%E4%B8%8E%E5%AF%BC%E8%88%AA%E5%88%97%E5%AE%A2%E5%88%B6%E5%8C%96%E8%BF%90%E7%94%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20560c759146fd46fa989167bf2fa22ff7/swiftui-navigation-7.png](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E5%AF%BC%E8%88%AAUI%E4%B8%8E%E5%AF%BC%E8%88%AA%E5%88%97%E5%AE%A2%E5%88%B6%E5%8C%96%E8%BF%90%E7%94%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20560c759146fd46fa989167bf2fa22ff7/swiftui-navigation-7.png)

图 11.7. 对大尺寸标题与标准尺寸标题修改字型与颜色