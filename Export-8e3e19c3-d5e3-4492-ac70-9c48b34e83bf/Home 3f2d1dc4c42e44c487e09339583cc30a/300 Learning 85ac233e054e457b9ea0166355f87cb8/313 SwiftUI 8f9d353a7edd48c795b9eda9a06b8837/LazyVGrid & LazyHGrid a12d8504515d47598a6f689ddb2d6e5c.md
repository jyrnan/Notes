# LazyVGrid & LazyHGrid

![1*LGg7-MXzgf_yYB1fPzrFQw.png](LazyVGrid%20&%20LazyHGrid%20a12d8504515d47598a6f689ddb2d6e5c/1LGg7-MXzgf_yYB1fPzrFQw.png)

iOS 14 的 SwiftUI 推出了專門呈現格子狀排版的 LazyVGrid & LazyHGrid，實現類似 IG 的照片牆頁面變得容易許多。接下來彼得潘將以小時候看的亞森羅蘋為例，一步步介紹 LazyVGrid & LazyHGrid。

以 ScrollView 包著 LazyVGrid 可實現元件由上而下排列的上下捲動頁面。

```swift
struct ContentView: View {

    let books = [
        "怪盜亞森羅蘋",
        "虎牙",
        "魔女與羅蘋",
        "怪盜與名偵探",
        "惡魔詛咒的紅圈",
        "玻璃瓶塞的祕密",
        "八大奇案",
        "奇巖城"
    ]

    var body: some View {

 ScrollView(.vertical) {
            let columns = [GridItem()]
            LazyVGrid(columns: columns) {
                ForEach(books.indices) { item in
                    BookView(book: books[item], number: item + 1)
                }
            }

        }
    }
}

struct BookView: View {
    let book: String
    let number: Int

    var body: some View {
        VStack {
            Image(book)
                .resizable()
                .scaledToFill()
                .frame(width: 150, height: 150)
                .clipped()
            Text(book)
        }
        .overlay(NumberImage(number: number)
            , alignment: .topLeading)
    }
}

struct NumberImage: View {
    let number: Int
    var body: some View {
        Image(systemName: "\(number).circle.fill")
            .font(.largeTitle)
            .foregroundColor(.white)
    }
}
```

在生成 LazyVGrid 時，我們必須傳入型別 **`[GridItem]`** 的參數 columns 控制 column 的數量。剛剛的例子我們傳入 **`[GridItem()]`**，因此它只會有一個 column。

LazyVGrid 開頭的 lazy 暗示著裡面的元件將在需要時才生成，因此如下圖所示，一開始畫面只看到五本書時，它只需要產生五本書的元件，其它書的元件等滑到時再產生即可。

# 顯示多個 Column 的 LazyVGrid

顯示多個 Column 完全不是問題，array 裡 GridItem 的數量即為 column 的數量，因此以下例子將產生兩個 column 的頁面。

```
let columns = Array(repeating: GridItem(), count: 2)
```

當有多個 column 時，LazyVGrid 的元件將從第一個 row 開始排列，排完再排到第二個 row，其它以此類推。

# GridItem 的 init

剛剛我們以 GridItem() 產生 GirdItem，它的 init 如下。

```swift
init(_ size: GridItem.Size = .flexible(), spacing: CGFloat? = nil, alignment: Alignment? = nil)
```

參數 size 採用的預設值是 .flexible()，表示 column 的寬度將依據可用空間自動算出合適的大小，待會我們會再介紹其它設定 size 的方法。

# 讓圖片寬度隨 column 寬度調整的 LazyVGrid

我們也可以讓圖片寬度隨 column 寬度調整。

```swift
struct BookView: View {
    let book: String
    let number: Int

    var body: some View {
        VStack {
            Image(book)
                .resizable()
                .scaledToFill()
                .frame(height: 100)
                .clipped()
            Text(book)
        }
        .overlay(NumberImage(number: number)
            , alignment: .topLeading)
    }
}
```

以上程式的 Image 我們只設定它的高度為 100，有 2 個 column 時，圖片將平分螢幕扣掉間距的寬度，因此螢幕愈大時圖片也會愈大，以下兩張圖分別為 iPhone & iPad 呈現的模樣。

# 調整 LazyVGrid 的間距

生成 LazyVGrid 時，我們可以傳入參數 **`spacing`** 控制 row 之間的間距。

```
LazyVGrid(columns: columns,spacing: 100) {
```

生成 GridItem 時，從參數 **`spacing`** 可控制 column 之間的間距。

```
let columns = [GridItem(spacing: 100), GridItem()]
```

如下圖所示，兩個 column 時只會有一個 column 間的間距，因此我們在第一個 GridItem 設定間距 100，第二個 GirdItem 不須設定。

# 讓圖片固定比例的 LazyVGrid

我們也可以讓 LazyVGrid 裡的圖片寬度隨 column 寬度調整，而且圖片維持固定比例。

比方程式將顯示正方形的圖片，超出的地方切除。

```
struct ContentView: View {

    let books = [
        "怪盜亞森羅蘋",
        "虎牙",
        "魔女與羅蘋",
        "怪盜與名偵探",
        "惡魔詛咒的紅圈",
        "玻璃瓶塞的祕密",
        "八大奇案",
        "奇巖城"
    ]

    var body: some View {

        ScrollView(.vertical) {
            let columns = Array(repeating: GridItem(), count: 2)
            LazyVGrid(columns: columns) {
                ForEach(books.indices) { item in
                    BookView(book: books[item], number: item + 1)
                }
            }

        }
    }
}struct BookView: View {
    let book: String
    let number: Int

    var body: some View {
        VStack {
 Rectangle()
                .aspectRatio(1, contentMode: .fit)
                .overlay(
                    Image(book)
                        .resizable()
                        .scaledToFill()
                )
                .clipped()
            Text(book)
        }
        .overlay(NumberImage(number: number)
                 , alignment: .topLeading)
    }
}
```

從 Rectangle() 呼叫 `.aspectRatio(1, contentMode: .fit)` 設定比例 1:1，讓 Rectangle 變成正方形，然後利用 overlay 將 Image 加到 Rectangle 上，再搭配 clipped 切成正方形。

aspectRatio 傳入 4/3 時將顯示 4:3 的圖片。

```
Rectangle()
   .aspectRatio(4/3, contentMode: .fit)
```

# 讓 GridItem 自動計算 column 數量的 adaptive size

我們也可以在生成 GridItem 時傳入型別 GridItem.Size 的資料控制 column 的寬度。

```
struct ContentView: View {

    let books = [
        "怪盜亞森羅蘋",
        "虎牙",
        "魔女與羅蘋",
        "怪盜與名偵探",
        "惡魔詛咒的紅圈",
        "玻璃瓶塞的祕密",
        "八大奇案",
        "奇巖城"
    ]

    var body: some View {

        ScrollView(.vertical) {
            let columns = [GridItem(.adaptive(minimum: 150))]
            LazyVGrid(columns: columns) {
                ForEach(books.indices) { item in
                    BookView(book: books[item], number: item + 1)
                }
            }

        }
    }
}struct BookView: View {
    let book: String
    let number: Int

    var body: some View {
        VStack {
            Image(book)
                .resizable()
                .scaledToFill()
                .frame(height: 200)
                .clipped()
            Text(book)
        }
        .overlay(NumberImage(number: number)
            , alignment: .topLeading)
    }
}
```

**`[GridItem(.adaptive(minimum: 150))]`** 表示我們希望 column 的最小寬度 150，但比 150 大也 ok，然後 LazyVGrid 將以此為條件盡可能塞入愈多的 column。因此在 iPhone 上有 2 個 column，但在 iPad 上變成 5 個 column。

# 左右捲動的 LazyHGrid

LazyHGrid 的使用方式跟 LazyVGrid 大同小異，以 ScrollView 包著 LazyHGrid 可實現元件由左而右排列的水平捲動頁面。

```
struct ContentView: View {

    let books = [
        "怪盜亞森羅蘋",
        "虎牙",
        "魔女與羅蘋",
        "怪盜與名偵探",
        "惡魔詛咒的紅圈",
        "玻璃瓶塞的祕密",
        "八大奇案",
        "奇巖城"
    ]

    var body: some View {

        VStack {
ScrollView(.horizontal) {
                let rows = [GridItem()]
                LazyHGrid(rows: rows) {
                    ForEach(books.indices) { item in
                        BookView(book: books[item], number: item + 1)
                    }
                }
            }
            .fixedSize(horizontal: false, vertical: true)
            Spacer()
        }
    }
}struct BookView: View {
    let book: String
    let number: Int

    var body: some View {
        VStack {
            Image(book)
                .resizable()
                .scaledToFill()
                .frame(width: 150, height: 150)
                .clipped()
            Text(book)
        }
        .overlay(NumberImage(number: number)
            , alignment: .topLeading)
    }
}
```

值得注意的，我們在 ScrollView 上呼叫 fixedSize(horizontal: false, vertical: true)，如此 ScrollView 的高度才會剛好等於內容的高度，而非佔滿螢幕。

# 搭配 NavigationLink 點選換頁

```
struct ContentView: View {

    let books = [
        "怪盜亞森羅蘋",
        "虎牙",
        "魔女與羅蘋",
        "怪盜與名偵探",
        "惡魔詛咒的紅圈",
        "玻璃瓶塞的祕密",
        "八大奇案",
        "奇巖城"
    ]

    var body: some View {

        NavigationView {
            ScrollView(.vertical) {
                let columns = Array(repeating: GridItem(), count: 2)
                LazyVGrid(columns: columns) {
                    ForEach(books.indices) { item in
 NavigationLink {
                            Image(books[item])
                                .resizable()
                                .scaledToFit()
                        } label: {
                            BookView(book: books[item], number: item + 1)
                        }
                    }
                }

            }
            .navigationTitle("亞森羅蘋")
        }
    }
}
```