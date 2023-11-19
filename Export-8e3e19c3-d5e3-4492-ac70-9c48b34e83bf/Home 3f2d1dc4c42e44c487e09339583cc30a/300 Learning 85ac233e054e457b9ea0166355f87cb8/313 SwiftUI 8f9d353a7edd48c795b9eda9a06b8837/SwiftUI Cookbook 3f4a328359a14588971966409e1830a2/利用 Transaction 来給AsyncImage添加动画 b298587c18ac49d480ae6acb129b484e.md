# 利用 Transaction 来給AsyncImage添加动画

### 利用 Transaction 来添加动画

同一个 `init` 可以让我们在阶段修改时指定可选 `transaction`。例如，以下代码片段在 `transaction` 参数中指定使用 `spring` 类型的动画：

```swift
AsyncImage(url: URL(string: imageURL), transaction: Transaction(animation: .spring())) { phase in
    switch phase {
    case .empty:
        Color.purple.opacity(0.1)

    case .success(let image):
        image
            .resizable()
            .scaledToFill()

    case .failure(_):
        Image(systemName: "exclamationmark.icloud")
            .resizable()
            .scaledToFit()

    @unknown default:
        Image(systemName: "exclamationmark.icloud")
    }
}
.frame(width: 300, height: 500)
.cornerRadius(20)

```

如此一来，在下载图像后，我们就会看到淡入 (fade-in) 动画。我们无法在预览版面中测试代码，请在模拟器中测试代码以查看动画。

你也可以把 `transition` 修饰符附加到 `image` 视图：

```swift
case .success(let image):
    image
        .resizable()
        .scaledToFill()
        .transition(.slide)

```

这样在显示结果图像时，就会看到滑入 (slide-in) 动画。