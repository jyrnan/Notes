# 利用GeometryReader获取滚动偏移量

节选这篇文章：[以SwiftUI 手势与 GeometryReader 建立一个底部展开式页面 <值得反复看>](../../../020%20Finished%205d5dcd977fad4e1a88671c30acab9455/%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20ac39e1de7e2d4c85b892b0d9cd92b968/%E4%BB%A5SwiftUI%20%E6%89%8B%E5%8A%BF%E4%B8%8E%20GeometryReader%20%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%BA%95%E9%83%A8%E5%B1%95%E5%BC%80%E5%BC%8F%E9%A1%B5%E9%9D%A2%20%E5%80%BC%E5%BE%97%E5%8F%8D%E5%A4%8D%E7%9C%8B%20ad7dd63ca6c3484bba56998f71728737.md) 

<aside>
💡 如果需要判断ScrollView滚动的偏移量，这个做法值得参考，主要涉及到三个知识点：

- GeometryReader的读取功能
- 通过proxy.frame(in:)方法获取在不同坐标系下的数据
- 利用PreferenceKey把数值传出去
</aside>

## 在ScrollView顶部增加一个GeometryReader

这里是示范代码，后面会改成真正起作用的代码。

```swift
GeometryReader { scrollViewProxy in
    Text("\(scrollViewProxy.frame(in: .named("scrollview")).minY)")
}
.frame(height: 0)
```

## 定义坐标系

为了定义自己的坐标空间，如下，加上 `.coordinateSpace` 修饰器至 `ScrollView`：

```swift
.coordinateSpace(name: "scrollview")

```

你可能想知道，为何我们必须使用自己的坐标空间，而非使用 `.global` 。

## 利用PreferenceKey传值

这一部份也可以参考这篇：[利用Geometry Reader来获取动态高度](%E5%88%A9%E7%94%A8Geometry%20Reader%E6%9D%A5%E8%8E%B7%E5%8F%96%E5%8A%A8%E6%80%81%E9%AB%98%E5%BA%A6%20e7909ffa76494ee7a817f1e85d92b003.md) 

### 创建PreferenceKey 的介绍

现在我们已经找到滚动偏移量取得方式，接下的问题是如何让它的父视图知道偏移量，以利进一步处理。 SwiftUI 框架提供了一个 `PreferenceKey` 协议，可以让你很容易地从子视图传递数据至它的父视图。

为了使用布局偏好（preference）来传递滚动偏移量，我们必须建立一个结构来遵守 `PreferenceKey`协议，如下所示，在 `RestaurantDetailView.swift` 插入以下这段程序：

```swift
struct ScrollOffsetKey: PreferenceKey {
    typealias Value = CGFloat

    static var defaultValue = CGFloat.zero

    static func reduce(value: inout Value, nextValue: () -> Value) {
        value += nextValue()
    }
}
```

这个协议有两个必要的实现内容，首先，你必须定义默认值，以我们这个实现而言默认值为零。第二，你必须实现 `reduce` 函数来将所有偏移量值合并成一个。

接下来，你可以修改前一节所建立的 `GeometryReader`，如下所示：

```swift
GeometryReader { scrollViewProxy in
    Color.clear
			.preference(key: ScrollOffsetKey.self, value: scrollViewProxy.frame(in: .named("scrollview")).minY)
}
.frame(height: 0)

```

程序中，我们取得滚动的偏移量，并将至储存至偏好键（preference key）。

我们使用 `Color.clear` 视图来隐藏视图。

proxy.frame是基于上面定义好的坐标空间`.named("scrollview")`

### 利用.onPreferenceChang(key:value:)把数值传出去

一个简单的方式是使用 `.onPreferenceChange` 修饰器来观察值的修改。你可以加上修饰器至 `VStack`，并置于 `.gesture` 下方，如下所示：

```swift
.onPreferenceChange(ScrollOffsetKey.self) { value in
    print("\(value)")
}

```

声明一个新状态变量来持续追踪滚动偏移量：

```swift
@State private var scrollOffset: CGFloat = 0.0

```

接着，修改 `.onPreferenceChange` 修饰器如下：具体代码内容因情形而异

```swift
.onPreferenceChange(ScrollOffsetKey.self) { value in
    if self.viewState == .full {
        self.scrollOffset = value > 0 ? value : 0

        if self.scrollOffset > 120 {
            self.positionOffset = 0
            self.scrollOffset = 0
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
               self.viewState = .half
            }
        }
    }
}

```

### 应用方式

现在加上另一个 .offset 修饰器至 VStack：

```swift
.offset(y: self.scrollOffset)

```