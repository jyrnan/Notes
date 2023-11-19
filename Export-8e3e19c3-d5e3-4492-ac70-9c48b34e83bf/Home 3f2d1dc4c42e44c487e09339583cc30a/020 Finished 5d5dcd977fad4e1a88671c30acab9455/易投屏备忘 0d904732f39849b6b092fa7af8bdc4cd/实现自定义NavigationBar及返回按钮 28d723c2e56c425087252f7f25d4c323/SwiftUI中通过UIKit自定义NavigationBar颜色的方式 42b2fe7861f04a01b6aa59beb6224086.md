# SwiftUI中通过UIKit自定义NavigationBar颜色的方式

> 如果需要在SwiftUI中来定义NavigationBar的颜色，目前只能通过UIKit的方式来实现。当然，以下方法直接在UIKit中也是可以使用的
> 

# NavigationBar颜色最基本设置

可以理解NavigationBar的颜色有三种状态：

- 默认：半透明
- 完全透明
- 完全不透明

## 默认

不做任何设置的时候，是半透明的效果

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled.png)

## 完全透明

可以设置成完全透明的效果

```swift
navAppearance.configureWithTransparentBackground()
```

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%201.png)

## 完全不透明

设置成完全不透明的时候，默认会用前景色来绘制

```swift
navAppearance.configureWithOpaqueBackground()
```

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%202.png)

### 指定颜色来绘制

会展示出指定的颜色，并且是完全不透明效果，也就是完整展示指定的颜色

```swift
navAppearance.configureWithOpaqueBackground()
navAppearance.backgroundColor = .green
```

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%203.png)

### 特别注意：

如果指定的颜色是`.clear`这个时候产生的是类似于指定完全透明的效果🤔️

```swift
navAppearance.configureWithOpaqueBackground()
navAppearance.backgroundColor = .clear
```

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%204.png)

## NavigationBar的阴影

NavigationBar下有一条阴影，可以通过如下设置成不可见来配合上面三张不同的效果。

这也是间接证明了NavigationBar并不是如以前所想完全是透明空间。😳

```swift
navAppearance.shadowColor = .clear
```

# 三个不同appreance属性的区别

```swift
UINavigationBar.appearance().standardAppearance = navAppearance
UINavigationBar.appearance().scrollEdgeAppearance = navAppearance
UINavigationBar.appearance().compactAppearance = navAppearance
```

`scrollEdgeAppearance`:当可滚动内容的边缘与导航栏的边缘对齐时，导航栏的外观设置。如果这个属性的值为nil, UIKit使用导航栏的standardAppearance外观属性的值，修改为有一个透明的背景。这是iOS15后才有的新属性

`standardAppearance`:设置导航栏标准高度的样式设置，默认样式。此属性的默认值是一个包含系统默认外观设置的外观对象。`compactAppearance`:紧凑高度导航栏的外观设置。

`compactScrollEdgeAppearance`:当可滚动内容的边缘与导航栏的边缘对齐时，紧凑高度导航栏的外观设置。

### 上面范例的代码

参考来源

[导航UI与导航列客制化运用 | 精通 SwiftUI - iOS 16 版](../../%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E5%AF%BC%E8%88%AAUI%E4%B8%8E%E5%AF%BC%E8%88%AA%E5%88%97%E5%AE%A2%E5%88%B6%E5%8C%96%E8%BF%90%E7%94%A8%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20560c759146fd46fa989167bf2fa22ff7.md)

```swift
struct Navigation: View {
  init() {
    let navAppearance = UINavigationBarAppearance()

    navAppearance.configureWithOpaqueBackground()
//    navAppearance.configureWithTransparentBackground()
    navAppearance.backgroundColor = .clear

    navAppearance.shadowColor = .clear

    navAppearance.largeTitleTextAttributes = [.foregroundColor: UIColor.red, .font: UIFont(name: "ArialRoundedMTBold", size: 35)!]
    navAppearance.titleTextAttributes = [.foregroundColor: UIColor.red, .font: UIFont(name: "ArialRoundedMTBold", size: 20)!]

    UINavigationBar.appearance().backIndicatorImage = UIImage(systemName: "list.bullet")!
    UINavigationBar.appearance().backIndicatorTransitionMaskImage = UIImage(systemName: "list.bullet")!

    UINavigationBar.appearance().standardAppearance = navAppearance
    UINavigationBar.appearance().scrollEdgeAppearance = navAppearance
    UINavigationBar.appearance().compactAppearance = navAppearance

    UINavigationBar.appearance().tintColor = .green
  }

  var body: some View {
    NavigationStack {
      ScrollView {
        Rectangle()
          .fill(.blue)
          .frame(height: 500)
        
        NavigationLink("NextPage", value: "Next Page")

          
      }
      .ignoresSafeArea()
      .border(.red, width: 5)
      .navigationDestination(for: String.self, destination: { string in
        ScrollView {
          Text(string)
        }
        .navigationBarTitleDisplayMode(.inline)
      })
      .navigationTitle("Navigation")
    }
  }
}
```

# PARTII：网络上获得关于在SwiftUI中实现自定义NavigationBar颜色的代码参考：

<aside>
💡 方法是把NavigationBar设置成完全透明，然后再在NavigationBar区域填充一个自定义的View🤔️，但是为啥要这样？！

</aside>

<aside>
💡 ~~分析了NavigationBar产生的实质其实就是当前View避让顶部空间，从而露出底层View，看起来像是一个Bar条；自定义颜色就是在这个空间再塞入一个填满空间的自定义颜色的View。~~
新的分析感觉并不是如上面所述通过留白露出底层View的底色，实际上bar是有一个底色在此的，之所以会出现上面的情况，实际应该是因为如下代码：

```swift
/// 这行代码会让
coloredAppearance.configureWithTransparentBackground()

coloredAppearance.backgroundColor = .clear // The key is here. Change the actual bar to clear.
```

</aside>

例如下图中List无视安全区域，而NavigationBar的背景相关设置14-17行注释掉，可以看到List藏到了Navigationbar的后面，并不会是完全透明。

PS：17行的shadowColor的设置，也说明其实这里应该是有一个视图，而不是真空地带区域。

但是：后面在这个空间塞入一个自定义颜色的View又是什么原因呢？

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%205.png)

## 显式设置 `ignoresSafeArea` 带来的效果

例如下面的代码中，`.ignoresSafeArea` 会导致Image扩展到顶部空间，

```swift
var body: some View {
    ZStack(alignment: .top) {
      Image("Main_backGroundImg")
        .resizable()
        .ignoresSafeArea() // 影响到NavigationBar背景色，因为背景会覆盖最底层View的颜色，从而NavigationBar部分就不显示
```

![显式设置 `ignoresSafeArea` 带来的效果](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%206.png)

显式设置 `ignoresSafeArea` 带来的效果

![不设置的效果，看上去有NavigationBar](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%207.png)

不设置的效果，看上去有NavigationBar

## 不设置`ignoresSafeArea` 的效果

~~不设置的时候，系统会自动将safeAreaInsets.top增到指定宽度，看上去是当前View顶部被往下压缩，从而露出后面的view的底色。看上去就相当于存在了一个NavigationBar区域。而本质上压缩当前View产生的留白区域。~~

*补充说明： 之所以会有以上误解，是因为NavigationBar采用了完全透明的效果🫥*

```swift
struct PersonalCenterQuestionsView: View {
  var body: some View {
    ScrollView {
      VStack(spacing: 0) {
        ForEach(qATexts) { qaText in
          CellView(qaText: qaText)
            .border(.clear)
        }
      }
    }
    .background(Color("BgColor1")
      .ignoresSafeArea(edges: .bottom)) // 影响到NavigationBar背景色，没有覆盖上部安全区域，会露出底部View的本色，从而NavigationBar部分会显示显示出来。
    .navigationTitle("常见问题")
    .navigationBarTitleDisplayMode(.inline)
  }
}
```

## 代码的证明

从这段代码的可以看出NavigationBar的宽度其实就是safeAreaInsets.top宽度

```swift
func body(content: Content) -> some View {
    ZStack {
      content
      VStack { // VStack的尺寸其实就和content一致
        GeometryReader { geometry in // 获取VStack的尺寸
          Color(self.backgroundColor ?? .clear)
            .frame(height: geometry.safeAreaInsets.top) // 获取VStack上方避让高度，也就是NavigationBar高度
            .edgesIgnoringSafeArea(.top) // 色块位置会忽视上方安全区域，导致上移，也就正好占据NavigationBar位置
            
          Spacer() // 这个视图其实并不起作用，创建原因未知
        }
      }
    }
  }
```

## 最终实现

所以，SwiftUI中改变navigationBar颜色，其实就是在这个留白区域塞了一个View，高度恰好就是`geometry.safeAreaInsets.top`

# 在SwiftUI中实现自定义NavigationBar颜色的代码

这段代码也就是对上面等观点的一个很好的例证

```swift
/// 修改NavigationTitle底色的modifier，来自现成代码
struct NavigationBarModifier: ViewModifier {
  var backgroundColor: UIColor?
  var titleColor: UIColor?

  init(backgroundColor: Color, titleColor: UIColor?) {
    self.backgroundColor = UIColor(backgroundColor)

    let coloredAppearance = UINavigationBarAppearance()
    
		coloredAppearance.configureWithTransparentBackground()
    coloredAppearance.backgroundColor = .clear // The key is here. Change the actual bar to clear.
    
		coloredAppearance.titleTextAttributes = [.foregroundColor: titleColor ?? .black]
    coloredAppearance.largeTitleTextAttributes = [.foregroundColor: titleColor ?? .black]
    coloredAppearance.shadowColor = .clear

    UINavigationBar.appearance().standardAppearance = coloredAppearance
    UINavigationBar.appearance().compactAppearance = coloredAppearance
    UINavigationBar.appearance().scrollEdgeAppearance = coloredAppearance
    UINavigationBar.appearance().tintColor = titleColor

    print(#line, #function, "NavigationBar changed")
  }

  func body(content: Content) -> some View {
    ZStack {
      content
      VStack { // VStack的尺寸其实就和content一致
        GeometryReader { geometry in // 获取VStack的尺寸
          Color(self.backgroundColor ?? .clear)
            .frame(height: geometry.safeAreaInsets.top) // 获取VStack上方避让高度，也就是NavigationBar高度
            .edgesIgnoringSafeArea(.top) // 色块位置会忽视上方安全区域，导致上移，也就正好占据NavigationBar位置
            
          Spacer() // 这个视图其实并不起作用，创建原因未知
        }
      }
    }
  }
}

extension View {
  func navigationBarColor(backgroundColor: Color, titleColor: UIColor?) -> some View {
    self.modifier(NavigationBarModifier(backgroundColor: backgroundColor, titleColor: titleColor))
  }
}
```

## 使用方式

```swift
struct PersonalCenterQuestionsView: View {
  var body: some View {
    ScrollView {
      VStack(spacing: 0) {
        ForEach(qATexts) { qaText in
          CellView(qaText: qaText)
            .border(.clear)
        }
      }
    }
    .background(Color("BgColor1")
      .ignoresSafeArea(edges: .bottom)) // 影响到NavigationBar背景色，没有覆盖上部安全区域，会露出底部View的本色，从而NavigationBar部分会显示显示出来。
    .navigationTitle("常见问题")
    .navigationBarTitleDisplayMode(.inline)
//    .navigationBarBackButtonHidden()
    /// 从目前效果来看，其实并不需要这个modifier
    /// 该modifier的作用是在NavigationBar区域创建一个新的视图来填充NavigationBar区域
    /// 从而挡住底层的View背景
    /// 实现充当NavigationBar的背景
//    .navigationBarColor(backgroundColor: .red, titleColor: .blue) // 实现说明见上方定义
  }
}
```

# 💡关于`ignoreSafeArea`的补充

`ignoreSafeArea` 并不会对其修饰的view本体的真正边框frame产生改变。

从这个范例中可以看出，无论是最外层ZStack，还是内层使用了`ignoreSafeArea` 的Color，他们的实际边框始终都是被最外层的SafeArea限制，并不会占满整个屏幕。而只是效果看上去占据了整个屏幕。

这也是要引入VStack或background里面添加一个GeometryReader来获取当前View实际大小，并获得`geometry.safeAreaInsets.top` 的值的原因

![Untitled](SwiftUI%E4%B8%AD%E9%80%9A%E8%BF%87UIKit%E8%87%AA%E5%AE%9A%E4%B9%89NavigationBar%E9%A2%9C%E8%89%B2%E7%9A%84%E6%96%B9%E5%BC%8F%2042b2fe7861f04a01b6aa59beb6224086/Untitled%208.png)