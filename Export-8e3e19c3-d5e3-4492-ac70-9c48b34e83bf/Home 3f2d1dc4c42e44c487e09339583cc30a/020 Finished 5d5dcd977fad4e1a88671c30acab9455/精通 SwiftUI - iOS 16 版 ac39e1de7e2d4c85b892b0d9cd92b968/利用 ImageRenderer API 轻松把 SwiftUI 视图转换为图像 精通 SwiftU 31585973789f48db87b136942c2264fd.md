# 利用 ImageRenderer API 轻松把 SwiftUI 视图转换为图像 | 精通 SwiftUI - iOS 16 版

iOS 16 为 SwiftUI 带来的另一个 API 就是 `ImageRenderer`。有了这个 API，我们可以轻松把 SwiftUI 视图转换为图像。这个实现十分简单，让我们利用想要转换为图像的视图，来实例化 `ImageRenderer` 的实例：

```
let renderer = ImageRenderer(content: theView)

```

然后，我们就可以存取 `cgImage` 或 `uiImage` 属性，来取得转换后的图像。

一如以往，我喜欢利用示例来示范一个 API 的用法。在第38章中，我们用了新的 Charts 框架来构建折线图。这次，让我们来看看如何让使用者把折线图保存为 Photo Album 中的图像，并使用 `ShareLink` 进行分享。

### 重温 Chart 视图

![%E5%88%A9%E7%94%A8%20ImageRenderer%20API%20%E8%BD%BB%E6%9D%BE%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftU%2031585973789f48db87b136942c2264fd/swiftui-line-chart-symbol.png](%E5%88%A9%E7%94%A8%20ImageRenderer%20API%20%E8%BD%BB%E6%9D%BE%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftU%2031585973789f48db87b136942c2264fd/swiftui-line-chart-symbol.png)

图 41.1. 折线图

首先，让我们来重温一下 `ChartView` 示例。我们使用了 `Charts` 框架的新 API 来构建一个有关气温的折线图。代码如下：

```
var body: some View {
    VStack {
        Chart {
            ForEach(chartData, id: \.city) { series in
                ForEach(series.data) { item in
                    LineMark(
                        x: .value("Month", item.date),
                        y: .value("Temp", item.temperature)
                    )
                }
                .foregroundStyle(by: .value("City", series.city))
                .symbol(by: .value("City", series.city))
            }
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .month)) { value in
                AxisGridLine()
                AxisValueLabel(format: .dateTime.month(.defaultDigits))

            }
        }
        .chartPlotStyle { plotArea in
            plotArea
                .background(.blue.opacity(0.1))
        }
        .chartYAxis {
            AxisMarks(position: .leading)
        }
        .frame(width: 350, height: 300)
        .padding(.horizontal)

    }
}

```

要跟着本教程学习，我建议你先从 [https://www.appcoda.com/resources/swiftui4/SwiftUIImageRendererStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIImageRendererStarter.zip) 下载 Xcode 项目。 在使用 ImageRenderer 之前，让我们将上面在 ContentView.swift 中的代码重构为一个单独的视图，如下所示：

```
struct ChartView: View {
    let chartData = [ (city: "Hong Kong", data: hkWeatherData),
                      (city: "London", data: londonWeatherData),
                      (city: "Taipei", data: taipeiWeatherData)
    ]

    var body: some View {
        VStack {
            Chart {
                ForEach(chartData, id: \.city) { series in
                    ForEach(series.data) { item in
                        LineMark(
                            x: .value("Month", item.date),
                            y: .value("Temp", item.temperature)
                        )
                    }
                    .foregroundStyle(by: .value("City", series.city))
                    .symbol(by: .value("City", series.city))
                }
            }
            .chartXAxis {
                AxisMarks(values: .stride(by: .month)) { value in
                    AxisGridLine()
                    AxisValueLabel(format: .dateTime.month(.defaultDigits))

                }

            }
            .chartPlotStyle { plotArea in
                plotArea
                    .background(.blue.opacity(0.1))
            }
            .chartYAxis {
                AxisMarks(position: .leading)
            }
            .frame(width: 350, height: 300)

            .padding(.horizontal)

        }
    }
}

```

然后，让我们声明一个变量来保存视图：

```
var chartView = ChartView()

```

### 利用 ImageRenderer 把视图转换为图像

现在，我们可以把 Chart 视图转换为图像了。我们要添加一个名为 *Save to Photos* 的按钮，来把 Chart 视图图像储存到 Photo Album 中。

让我们如此实现按钮：

```
var body: some View {

    VStack(spacing: 20) {
        chartView

        HStack {
            Button {
                let renderer = ImageRenderer(content: chartView)

                if let image = renderer.uiImage {
                    UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
                }
            } label: {
                Label("Save to Photos", systemImage: "photo")
            }
            .buttonStyle(.borderedProminent)
        }
    }

}

```

在按钮的闭包中，我们利用 `chartView` 来建立一个 `ImageRenderer` 的实例，并使用 `uiImage` 属性来撷取生成的图像。然后，让我们调用 `UIImageWriteToSavedPhotosAlbum` 把图像储存到 Photo Album 中。

**备注：**我们需要在 info.plist 中添加一个 key *Privacy - Photo Library Usage Description*，让 App 可以把图像储存到内置的 Photo Album 中。

### 添加 Share 按钮

![%E5%88%A9%E7%94%A8%20ImageRenderer%20API%20%E8%BD%BB%E6%9D%BE%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftU%2031585973789f48db87b136942c2264fd/swiftui-imagerenderer-sharelink.png](%E5%88%A9%E7%94%A8%20ImageRenderer%20API%20%E8%BD%BB%E6%9D%BE%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftU%2031585973789f48db87b136942c2264fd/swiftui-imagerenderer-sharelink.png)

图 41.2. 加入 Share 按钮

在之前的章节，我们学了如何使用 `ShareLink` 来显示一个 Share Sheet。我们可以搭配 `ImageRanderer` 使用，轻松地建立让使用者分享 Chart 视图的功能。

为方便起见，让我们将渲染图像 (rendered image) 的代码重构为一个独立的方法：

```
@MainActor
private func generateSnapshot() -> UIImage {
    let renderer = ImageRenderer(content: chartView)

    return renderer.uiImage ?? UIImage()
}

```

这个 `generateSnapshot` 方法会把 `chartView` 转换为图像。

**备注：**如果你没有用过 `@MainActor`，可以[参考这篇文章](https://www.avanderlee.com/swift/mainactor-dispatch-main-thread/)。

有了这个 helper 方法，我们就可以如此在 `VStack` 视图中建立一个 `ShareLink`：

```
ShareLink(item: Image(uiImage: generateSnapshot()), preview: SharePreview("Weather Chart", image: Image(uiImage: generateSnapshot())))
.buttonStyle(.borderedProminent)

```

现在，当我们点击 *Share* 按钮，App 就会撷取折线图，并让我们以图像形式分享图表。

![%E5%88%A9%E7%94%A8%20ImageRenderer%20API%20%E8%BD%BB%E6%9D%BE%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftU%2031585973789f48db87b136942c2264fd/swiftui-imagerenderer-sharesheet.png](%E5%88%A9%E7%94%A8%20ImageRenderer%20API%20%E8%BD%BB%E6%9D%BE%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%9B%BE%E5%83%8F%20%E7%B2%BE%E9%80%9A%20SwiftU%2031585973789f48db87b136942c2264fd/swiftui-imagerenderer-sharesheet.png)

图 41.3. 显示 share sheet

### 调整图像比例

你可能会发现渲染图像的解析度有点低。`ImageRenderer` 类别有一个 `scale` 属性，用来调整渲染图像的比例。在默认情况下，这个属性的值是 1.0。如果我们想提高图像的解析度，可以将它设置为 `2.0` 或 `3.0`。或者，我们可以将值设置为屏幕比例：

```
renderer.scale = UIScreen.main.scale

```

### 总结

有了 `ImageRenderer` 类别，我们就可以简单地把任何 SwiftUI 视图转换成图像。如果你的 App 支持 iOS 16 以上的版本，就可以用这个新 API 为使用者建立一些方便的功能。除了渲染图像外，我们还可以使用 `ImageRenderer` 来渲染 PDF 文件。详情可以参阅 [Apple 的官方文件](https://developer.apple.com/documentation/swiftui/imagerenderer)。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：