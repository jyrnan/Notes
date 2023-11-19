# 如何把 SwiftUI 视图转换为 PDF 文件 | 精通 SwiftUI - iOS 16 版

在上一章，我们学习了如何使用 `ImageRenderer` 撷取 SwiftUI 视图，并储存为图像。这个在 iOS 16 推出的新类别还可以把视图转换为 PDF 文件。

在这篇教学中，我们会以上次的示例为基础进行构建，并添加 *Save to PDF* 功能。

## 重温上一篇文章的示例 App

如果你还没有读过上一章，我建议你先读过再看此章节。在上一篇文章中，我们已经说明过 `ImageRenderer` 的基本用法，并解释了示例 App 的实现过程。

![%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf.png](%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf.png)

图 42.1. 示例 App

要跟着实现的话，可以先下载这个 Starter 项目：[https://www.appcoda.com/resources/swiftui4/SwiftUIImageRendererPDFStarter.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIImageRendererPDFStarter.zip).

我对上次的示例 App 做了一些改动，为折线图添加了 heading 和 caption。示例 App 现在还有一个 *PDF* 按钮，用来把 Chart 视图保存为 PDF 文件。以下是 `ChartView` 结构的代码：

```
struct ChartView: View {
    let chartData = [ (city: "Hong Kong", data: hkWeatherData),
                      (city: "London", data: londonWeatherData),
                      (city: "Taipei", data: taipeiWeatherData)
    ]

    var body: some View {
        VStack {
            Text("Building Line Charts in SwiftUI")
                .font(.system(size: 40, weight: .heavy, design: .rounded))
                .multilineTextAlignment(.center)
                .padding()

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

            Text("Figure 1. Line Chart")
                .padding()

        }
    }
}

```

### 把 Chart 视图储存为 PDF 文件

现在，我们想要做的就是使用 `ImageRenderer`，把 `ChartView` 储存为 PDF 文件。虽然，我们只需要几行代码就可以把 SwiftUI 视图转换为图像，PDF rendering 就需要多一点步骤。

如果我们是想把 Chart 转换为图像，我们可以存取 `uiImage` 属性，来取得渲染图像 ( rendered image)。而如果我们想把 Chart 转换成 PDF，就会使用 `ImageRenderer` 的 `render` 方法。我们将会实现：

- 寻找文档目录，并为 PDF 文件准备渲染路径 (rendered path)（例如 linechart.pdf）。
- 准备一个用于绘图的 `CGContext` 实例。
- 调用渲染器 (renderer) 的 `render` 方法来渲染 PDF 文件。

在实现中，我们会建立一个新方法 `exportPDF`。以下是这个方法的代码：

```
@MainActor
private func exportPDF() {
    guard let documentDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else { return }

    let renderedUrl = documentDirectory.appending(path: "linechart.pdf")

    if let consumer = CGDataConsumer(url: renderedUrl as CFURL),
       let pdfContext = CGContext(consumer: consumer, mediaBox: nil, nil) {

        let renderer = ImageRenderer(content: chartView)
        renderer.render { size, renderer in
            let options: [CFString: Any] = [
                kCGPDFContextMediaBox: CGRect(origin: .zero, size: size)
            ]

            pdfContext.beginPDFPage(options as CFDictionary)

            renderer(pdfContext)
            pdfContext.endPDFPage()
            pdfContext.closePDF()
        }
    }

    print("Saving PDF to \(renderedUrl.path())")
}

```

在代码的前两行，我们检索了使用者的文档目录，并设置 PDF 文件的文件路径（即 `line chart.pdf`）。然后，我们创建了 `CGContext` 的实例，并把 `mediaBox`参数设置为 nil。在这种情况下，Core Graphics 会使用默认页面大小 8.5 x 11 inches（612 x 792 points）。

`renderer` 闭包会接收两个参数：视图当前的大小、和将视图渲染到 `CGContext` 的函数。让我们调用上下文的 `beginPDFPage` 方法，来打开 PDF 页面。`renderer` 方法会绘制 Chart 视图。请记住，我们需要关闭 PDF 文件才能完成整个操作。

让我们要创建一个 `PDF` 按钮，来调用这个 `exportPDF` 方法：

```
Button {
    exportPDF()
} label: {
    Label("PDF", systemImage: "doc.plaintext")
}
.buttonStyle(.borderedProminent)

```

我们可以试着在模拟器上运行 App 来进行测试，点击 *PDF* 按钮后，你应该会在控制台看到以下信息：

```
Saving PDF to /Users/simon/Library/Developer/CoreSimulator/Devices/CA9B849B-36C5-4608-9D72-B04C468DA87E/data/Containers/Data/Application/04415B8A-7485-48F0-8DA2-59B97C2B529D/Documents/linechart.pdf

```

如果你在 Finder 打开文件，应该会看到以下的 PDF 文件：

![%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf-sample.png](%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf-sample.png)

图 42.2. PDF 文件

如果想调整图表的位置，我们可以在调用 `renderer` 前插入这行代码：

```
pdfContext.translateBy(x: 0, y: 200)

```

如此一来，图表就会出现在文件上面的部分。

![%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf-chart.png](%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf-chart.png)

图 42.3. 改变图表的位置

## 将 PDF 档加至 Files app

你可能会发现我们无法在 *Files* App 中找到 PDF 文件。如果我们想 PDF 文件可用于内置的 *Files* App，我们就需要在 `Info.plist` 中修改一些设置。让我们切换到 `Info.plist` 并添加以下的 key：

- `UIFileSharingEnabled` - 让 App 支持 iTunes 文件分享
- `LSSupportsOpeningDocumentsInPlace` - 支持在本地打开文件

然后，把 key 的值设置为 *Yes*。启用了这两个选项后，让我们再在模拟器上运行 App。接着，打开 *Files* App 并导航到 *On My iPhone* 位置，你应该会看到 App 的数据夹，而 PDF 文件就在数据夹中。

![%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf-files-app.png](%E5%A6%82%E4%BD%95%E6%8A%8A%20SwiftUI%20%E8%A7%86%E5%9B%BE%E8%BD%AC%E6%8D%A2%E4%B8%BA%20PDF%20%E6%96%87%E4%BB%B6%20%E7%B2%BE%E9%80%9A%20SwiftUI%20-%20iOS%2016%20%E7%89%88%20b98d2f800ac646fdb097a5216e3dd53c/swiftui-imagerenderer-pdf-files-app.png)

图 42.4. 储存 PDF 档至 Files app

### 总结

你不仅可以从视图创建图像，`ImageRenderer` 还允许开发人员将视图转换为 PDF 文档。 借助 iOS 16 中的这个新 API，你可以轻松地在 iOS 应用程序中添加一些 PDF 相关的功能。

在本章所准备的示例档中，有最后完整的 Xcode 项目，可供你下载参考：

- [https://www.appcoda.com/resources/swiftui4/SwiftUIImageRendererPDF.zip](https://www.appcoda.com/resources/swiftui4/SwiftUIImageRendererPDF.zip)