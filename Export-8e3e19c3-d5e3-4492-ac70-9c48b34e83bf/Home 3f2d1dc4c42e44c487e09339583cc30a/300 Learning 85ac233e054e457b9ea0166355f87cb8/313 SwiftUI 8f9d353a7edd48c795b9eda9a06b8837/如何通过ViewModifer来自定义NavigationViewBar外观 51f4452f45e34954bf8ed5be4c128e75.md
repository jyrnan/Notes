# 如何通过ViewModifer来自定义NavigationViewBar外观

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        NavigationView{
        Text("Hello, world!")
            .padding()
            .navigationTitle("Title")
        }
//        .modifier(NavigationBarColor(backgroundColor: .red, tintColor: .white)) //直接用modifier调用
        .navigationBarColor(backgroundColor: .red, tintColor: .white)
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

//基本思路是
//第一步：定义一个ViewModifer
//第二步： 扩展View视图定义一个函数，这个函数的作用是应用modifier
//第三步：在视图调用定义好的函数

///补充说明：其实也可以不用定义这个modifer，直接在视图里调用.modifier

///  定义一个ViewModiier，其实这个modify将是用来配置NavigationBar的相关参数
struct NavigationBarColor: ViewModifier {
    
    init(backgroundColor: UIColor, tintColor: UIColor) {
        let coloredAppearance = UINavigationBarAppearance()
        coloredAppearance.configureWithOpaqueBackground()
        coloredAppearance.backgroundColor = backgroundColor
        coloredAppearance.titleTextAttributes = [.foregroundColor: tintColor]
        coloredAppearance.largeTitleTextAttributes = [.foregroundColor: tintColor]
        
        UINavigationBar.appearance().standardAppearance = coloredAppearance
        UINavigationBar.appearance().scrollEdgeAppearance = coloredAppearance
        UINavigationBar.appearance().scrollEdgeAppearance = coloredAppearance
        UINavigationBar.appearance().tintColor = tintColor
    }
    
    func body(content: Content) -> some View {
        content
    }
    
    
}

//定义一个扩展来调用上面定义的modifier
extension View {
    func navigationBarColor(backgroundColor: UIColor, tintColor: UIColor) -> some View {
        self.modifier(NavigationBarColor(backgroundColor: backgroundColor, tintColor: tintColor))
    }
	}
```