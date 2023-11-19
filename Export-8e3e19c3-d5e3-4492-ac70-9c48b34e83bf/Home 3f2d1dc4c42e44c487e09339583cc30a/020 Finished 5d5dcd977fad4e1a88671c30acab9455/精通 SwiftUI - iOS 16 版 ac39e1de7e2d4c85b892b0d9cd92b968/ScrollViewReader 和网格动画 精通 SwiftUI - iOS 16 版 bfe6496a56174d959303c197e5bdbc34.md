# ScrollViewReader 和网格动画 | 精通 SwiftUI - iOS 16 版

在前一章，我已介绍了新的 `matchedGeometryEffect` 修饰器 (modifier) ，并向你展示了如何创建一些基本的视图动画。 在本章中，让我们看看如何在网格视图中使用修饰器和加入过场动画。 此外，你还将学习另一个名为`ScrollViewReader`的全新 UI 组件。

### 示例 App

在我们开始实现之前，让我先向你展示最终的成果。 这应该让你对要即将构建的内容有所了解。 当你开发 App时，你可能需要以网格（Grid）形式显示照片并让使用者选择其中的一些项目。

示例 App 在屏幕底部显示一个Dock，当一个项目被选中时，它会从网格中移除并插入到 Dock 中。 当你选择更多项目时，个 Dock 自动扩大以容纳更多项目。 你可以水平滑动以浏览Dock中的项目。 如果你点击 Dock 中的其中一个项目，该项目就会被移除并重新加到网格中。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-1.jpg](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-1.jpg)

图 34.1. 示例 App

我们将实现这个示例App，并会使用 `matchedGeometryEffect` 修饰符加入绚丽的过场动画。 开始之前，请到 [https://www.appcoda.com/resources/swiftui4/SwiftUIGridViewAnimationStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIGridViewAnimationStarter.zip) 下载Starter项目。 该项目已包含样本数据和图像。

### 构建照片网格（Photo Grid）

首先，让我们建立照片网格。 在 `ContentView` 结构体中，声明一个状态变量，如下所示：

```
@State private var photoSet = samplePhotos

```

`在Starter项目中，我已经预先准备好 samplePhotos` 常数用于存放示范照片。而 `photoSet` 被定义为状态变量，主要原因是，随着使用者的选择，我们会修改它存放的照片。

为了用网格形式布局照片，我们会使用`LazyVGrid`组件。 在 `body` 中加入以下代码：

```
VStack {
    ScrollView {

        HStack {
            Text("Photos")
                .font(.system(.title, design: .rounded))
                .fontWeight(.heavy)

            Spacer()
        }

        LazyVGrid(columns: [ GridItem(.adaptive(minimum: 50)) ]) {

            ForEach(photoSet) { photo in

                Image(photo.name)
                    .resizable()
                    .scaledToFill()
                    .frame(minWidth: 0, maxWidth: .infinity)
                    .frame(height: 60)
                    .cornerRadius(3.0)
            }
        }
    }
}
.padding()

```

假设你已经阅读了前面有关网格视图的章节，你一定能够明白当中的代码。 我们只需使用自适应布局将一组照片排列在一个网格中。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-2.png](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-2.png)

图 34.2. 照片网格（Photo Grid）

### 加入 Dock

为了显示和存放使用者所选的照片，我们将创建一个Dock。 在 `VStack` 中插入以下代码：

```
ScrollView(.horizontal, showsIndicators: false) {

}
.frame(height: 100)
.padding()
.background(Color(.systemGray6))
.cornerRadius(5)

```

这就会建立一个可滚动的矩形区域来存放选定的照片。 当然，它现在还没有任何相片，只是一个空白区。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-3.png](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-3.png)

图 34.3. 添加灰色区域

### 处理使用者所选的照片

选择照片后，我们会将其从照片网格中移除并将它加入到 Dock 中。 为了处理使用者所选的照片，我们将创建一个状态变量来存放所选照片。 在 `ContentView` 中加入以下代码来声明变量：

```
@State private var selectedPhotos: [Photo] = []

```

`photoSet` 中的每张照片都有自己的ID，而类型是 `UUID` 。 要储存当前选定的照片，请声明另一个 `UUID` 类型的状态变量：

```
@State private var selectedPhotoId: UUID?

```

要侦测照片选择，请将 `onTapGesture` 修饰器附加到 `LazyVGrid` 的 `Image` 组件，如下所示：

```
Image(photo.name)
    .resizable()
    .scaledToFill()
    .frame(minWidth: 0, maxWidth: .infinity)
    .frame(height: 60)
    .cornerRadius(3.0)
    .onTapGesture {
        selectedPhotos.append(photo)
        selectedPhotoId = photo.id
        if let index = photoSet.firstIndex(where: { $0.id == photo.id }) {
            photoSet.remove(at: index)
        }
    }

```

在 `onTapGesture` 中，我们将所选照片添加到 `selectedPhotos` 数组并修改 `selectedPhotoId`。 此外，我们从 `photoSet` 中删除所选择的照片。 由于 `photoSet` 是一个状态变量，一旦从数组中删除选定的照片，它就会自动从网格中删除。

App 会将所选的照片添加到Dock中。 因此，像这样修改 Dock 的 `ScrollView`：

```
ScrollView(.horizontal, showsIndicators: false) {
    LazyHGrid(rows: [ GridItem() ]) {
        ForEach(selectedPhotos) { photo in
            Image(photo.name)
                .resizable()
                .scaledToFill()
                .frame(minWidth: 0, maxWidth: .infinity)
                .frame(height: 100)
                .cornerRadius(3.0)
                .onTapGesture {
                    photoSet.append(photo)
                    if let index = selectedPhotos.firstIndex(where: { $0.id == photo.id }) {
                        selectedPhotos.remove(at: index)
                    }
                }
        }
    }
}

```

我们创建一个水平网格（Horizontal Grid）来显示选定的照片。 对于每张照片，我们将 `onTapGesture` 修饰器附加到它上面。 当有人点击 Dock 中的照片时，我们会将照片放回网格并且从`selectedPhotos`中删除。 换句话说，照片将从Dock中删除。

如果你在预览画面中运行App，你应该能够选择网格中的任何照片。 当你点击一张照片时，它会自动添加到 Dock 中，并且该照片将从网格中删除。 相反，你可以点击 Dock 中的照片将其移回照片网格。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-4.png](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-4.png)

图 34.4. 将所选照片添加到 Dock

### 使用 MatchedGeometryEffect 设置过场动画

现在 App 的图片选择效果已很不错，但我们可以通过建立动画再进一步提升使用者体验。 目前，所选照片会立即出现在 Dock 中。 我想要做的为这个选择动作加入动画。 选择后，照片应该看起来像是从照片网格飞到 Dock。

要做出这种类型的动画，最简单就是使用 `matchedGeometryEffect` 修饰器。 首先，在 `ContentView` 中，声明一个Namespace变量：

```
@Namespace private var photoTransition

```

接下来，将 `.matchedGeometryEffect` 修饰器附加到两个 `Image` 视图：

```
.matchedGeometryEffect(id: photo.id, in: photoTransition)

```

这个实现的技巧是为每张图片分配一个不同的 ID，这样App只会对所选照片的变化作动画化处理。

要启用动画，请将 `.animation` 修饰器附加到 `VStack` 并在 `.padding()` 下插入以下代码：

```
.animation(.interactiveSpring(), value: selectedPhotoId)

```

在模拟器或预览画面上运行App。 当你点击网格中的照片时，就可以在将它添加到 Dock ，而照片应该看起来像是从照片网格飞到 Dock。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-5.png](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-5.png)

图 34.5. 所选照片将添加到Dock中

### 使用 ScrollViewReader 移动滚动视图

动画效果应该不错吧？ 但是你是否注意到App还有一个小问题？ 当你不断将照片加入至Dock，你会发现Dock是不会自动滚动至最近选择的照片。 如果你选择的照片超过 4 张，则需要自己滚动Dock以显示其他选定的照片。

我们如何修复这个错误？ 在 iOS 14 中，Apple 引入了一个名为 `ScrollViewReader` 的新组件。 顾名思义，此阅读器要与 `ScrollView` 一起使用。 它允许开发者以编程方式将滚动视图（ScrollView）移动到特定位置。 要使用 `ScrollViewReader`，请将它包裹 `ScrollView` 。 每个子视图都应该有自己的ID。 之后，你就可以使用指定 ID 并调用 `ScrollViewProxy` 的 `scrollTo` 函数来将滚动视图移动到该特定位置。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-6.png](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-6.png)

图 34.6. 了解 ScrollViewReader

现在让我们回到示例App。 要以编程方式滚动Dock的`ScrollView`，我们首先要给每张照片指定一个ID。就是通过这个 ID，我们才可以指定滚动位置。 `scrollTo` 函数内的参数就是指这个ID。 由于每张照片本身都已经有一个ID，我们就直接使用它吧。

要在 Dock 中设置 `Image` 视图的ID，请附加 `.id` 修饰器：

```
.id(photo.id)

```

完成后，将`ScrollViewReader`包裹整个`ScrollView`，如下所示：

```
ScrollViewReader { scrollProxy in
    ScrollView(.horizontal, showsIndicators: false) {
        LazyHGrid(rows: [ GridItem() ]) {
            ForEach(selectedPhotos) { photo in
                Image(photo.name)
                    .resizable()
                    .scaledToFill()
                    .frame(minWidth: 0, maxWidth: .infinity)
                    .frame(height: 100)
                    .cornerRadius(3.0)
                    .id(photo.id)
                    .matchedGeometryEffect(id: photo.id, in: photoTransition)
                    .onTapGesture {
                        photoSet.append(photo)
                        if let index = selectedPhotos.firstIndex(where: { $0.id == photo.id }) {
                            selectedPhotos.remove(at: index)
                        }
                    }
            }
        }
    }
    .frame(height: 100)
    .padding()
    .background(Color(.systemGray6))
    .cornerRadius(5)
}

```

最后，将`.onChange`函数附加到dock的`ScrollView`，如下所示：

```
.onChange(of: selectedPhotoId, perform: { id in
    guard id != nil else { return }

    scrollProxy.scrollTo(id)
})

```

我们使用`.onChange`来侦测`selectedPhotoId`的变动。 每当使用者选择照片时，我们就使用该照片 ID 并调用`scrollTo`，这样滚动视图就会滚动到该相片位置。而最重要的是，确保 Dock 永远都显示最近选择的照片。 你可以再次运行App进行试一试。

![ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-7.gif](ScrollViewReader%20%E5%92%8C%E7%BD%91%E6%A0%BC%E5%8A%A8%E7%94%BB%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20bfe6496a56174d959303c197e5bdbc34/swiftui-gridanimation-7.gif)

图 33.7. 自动滚动Dock

### 总结

在本章中，我们继续探讨 `matchedGeometryEffect` 的用法，并使用这个修饰器创建另一种视图转换效果。 只要活用 `matchedGeometryEffect` ，你可以轻易加入过场动画并改善App的使用者体验。 我们还试验了 `ScrollViewReader` 以编程方式移动滚动视图。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

[https://www.appcoda.com/resources/swiftui4/SwiftUIGridViewAnimation.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIGridViewAnimation.zip)