# 对一个View增加效果（操作）的三种方式extension, a custom view, and a modifier

如图，需要对Text() 增加一个相应颜色的圆形背景。

![Hello.png](%E5%AF%B9%E4%B8%80%E4%B8%AAView%E5%A2%9E%E5%8A%A0%E6%95%88%E6%9E%9C%EF%BC%88%E6%93%8D%E4%BD%9C%EF%BC%89%E7%9A%84%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8Fextension,%20a%20custom%20view,%20and%20%208cb2a6f5886246b7980054807618f98a/Hello.png)

```swift
import SwiftUI

struct ContentView: View {
    
    var body: some View {
        
        VStack {
            Text("Hello").circle() //View扩展形式
            CircleWrapper{Text("Morning")} //新建View形式
            Text("Good").circleModifier() //View的Modifier形式
        }
    }
}

//MARK:-利用View的扩展，所以输入的内容是self
extension View {
    func circle(foreground: Color = .white, background: Color = .blue) -> some View {
        Circle()
            .fill(background)
            .overlay(Circle().strokeBorder(foreground).padding(3))
            .overlay(self.foregroundColor(foreground))
            .frame(width: 75, height: 75, alignment: .center)
    }
}

//MARK:-利用@ViewBuilder 来创建一个新的View，
        ///原来的内容作为一个Content的闭包传入
        ///这里需要显性的指定类型：Content：View
struct CircleWrapper<Content: View>: View {
    var foreground: Color = .white
    var background: Color = .accentColor
    
    var content: Content
//    var content: () -> Content
    
    ///传入的content是作为一个闭包，所以是加以@ViewBuilder修饰器使其传入的内容生成一个整体的View
    ///并且是在init过程中执行这一操作
    ///其实也可以在body本体中执行，则相应的内容如备注有两处变化
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
    
    var body: some View {
        Circle()
            .fill(background)
            .overlay(Circle().strokeBorder(foreground).padding(3))
            .overlay(content.foregroundColor(foreground))
//            .overlay(content().foregroundColor(foreground))
            .frame(width: 75, height: 75, alignment: .center)
    }
}

//MARK:-利用Modifier来实现
struct CircleModifier: ViewModifier {
    
    var foreground: Color = .white
    var background: Color = .accentColor
    
    ///因为已经显性制定类型ViewModifier，此处body接受的传入内容类型就是Content
    func body(content: Content) -> some View {
        Circle()
            .fill(background)
            .overlay(Circle().strokeBorder(foreground).padding(3))
            .overlay(content.foregroundColor(foreground))
            .frame(width: 75, height: 75, alignment: .center)
    }
}

///将生成的Modifier做成View的扩展，调用更方便
extension View {
    func circleModifier() -> some View {
        return self.modifier(CircleModifier())
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```