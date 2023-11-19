# matchedGeometryEffect构建不同的视图的变化动画

<aside>
💡 不同的视图的变化动画
步骤如下：

- 创建@NameSpace
- `matchedGeometryEffect(id: in: )`

特别注意：变形主体需要类似形状，例如形状的节点尽量匹配

</aside>

在 iOS 14 中，Apple 為 SwiftUI 框架引入了很多新功能，像是 [LazyVGrid](https://www.appcoda.com.tw/swiftui-lazyvgrid-lazyhgrid/) 以及 [LazyHGrid](https://www.appcoda.com.tw/swiftui-lazyvgrid-lazyhgrid/)。其中 `matchedGeometryEffect` 非常引人注目，這個功能讓開發者只需要幾行程式碼，就能夠創造絢麗的視圖動畫。SwiftUI 框架已經讓開發者可以簡單地使用動畫來呈現視圖的變化，而 `matchedGeometryEffect` 修飾器 (modifier) 更將[視圖動畫 (view animations)](https://www.appcoda.com.tw/swiftui-%E5%8B%95%E7%95%AB/) 的實作提升到另一個境界。

對所有手機 App 來說，我們經常需要在多個視圖之間轉換，因此一個令人喜歡的視圖轉換絕對可以提昇整體的使用者體驗。有了 `[matchedGeometryEffect](https://developer.apple.com/documentation/swiftui/view/matchedgeometryeffect(id:in:properties:anchor:issource:))`，你只需要描述兩個視圖的外觀，修飾器就會自動計算兩個視圖的差異，並且自動為大小和位置的變化加上動畫。

可能你會覺得十分困惑，但別擔心，介紹完整個範例 App 之後，你就會明白我在說什麼了。

**編者備註：**本文摘自 Mastering SwiftUI 一書。如果你想深入學習 SwiftUI 動畫及 SwiftUI 框架，請到 [AppCoda 網站](https://www.appcoda.com.tw/swiftui) 購買完整書本。

## 重溫 SwiftUI 動畫

在我們開始介紹 `matchedGeometryEffect` 之前，讓我們先來看一下如何使用 [SwiftUI](https://www.appcoda.com.tw/swiftui-introduction/) 來實作動畫。下面的圖片顯示了一個視圖開始和結束的狀態。當你點擊左邊的圓形視圖，它應該會變大且往上移動；相反地，當你點擊右邊的視圖時，它就會回到原本的大小和位置。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-1-1024x568.png](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-1-1024x568.png)

要實作這個可點擊的圓形視圖非常簡單。開啟一個新的 SwiftUI 專案後，如此更新 `ContentView` 結構：

```
struct ContentView: View {

    @State private var expand = false

    var body: some View {
        Circle()
            .fill(Color.green)
            .frame(width: expand ? 300 : 150, height: expand ? 300 : 150)
            .offset(y: expand ? -200 : 0)
            .animation(.default)
            .onTapGesture {
                self.expand.toggle()
            }
    }
}
```

我們用一個狀態變數 `expand`，來記錄 `Circle` 視圖目前的狀態。當狀態改變時，我們會透過 `.frame` 和 `.offset` 這兩個修飾器，來修改視圖框 (frame) 的大小和位移 (offset)。如果在預覽畫面中執行這個 App ，你應該可以在點擊圓形視圖時看到動畫效果。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-2.gif](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-2.gif)

## 了解 matchedGeometryEffect 修飾器

那麼到底什麼是 `matchedGeometryEffect` 呢？這個功能如何簡化實作視圖動畫的步驟？讓我們再來看一下第一張圖片、以及圓形視圖動畫的程式碼。我們需要找出開始及結束狀態時的確切數值差異。在這個例子當中，就是視圖框的大小及位移。

有了 `matchedGeometryEffect` 修飾器，你不再需要找出兩個狀態之間的差異了。你只需要描述兩個視圖：*一個是開始的狀態，而另一個是結束的狀態*，`matchedGeometryEffect` 會自動添加在兩個視圖之間大小和位置的差異。

要使用 `matchedGeometryEffect` 建立跟之前一樣的動畫效果，你需要先宣告一個命名空間 (namespace) 變數：

```swift
@Namespace private var shapeTransition
```

然後，如此重寫 `body` 的部分：

```swift
var body: some View {
    if expand {

        // Final State
        Circle()
            .fill(Color.green)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(width: 300, height: 300)
            .offset(y: -200)
            .animation(.default)
            .onTapGesture {
                self.expand.toggle()
            }

    } else {

        // Start State
        Circle()
            .fill(Color.green)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(width: 150, height: 150)
            .offset(y: 0)
            .animation(.default)
            .onTapGesture {
                self.expand.toggle()
            }
    }
}
```

在這段程式碼中，我們建立了兩個圓形視圖，一個表示開始狀態，而另一個表示結束狀態。當它一開始被初始化之後，我們會得到一個 `Circle` 視圖，位置置中以及寬度為 150 點。當 `expand` 狀態變數從 `false` 變成 `true` 時， App 會顯示另一個 `Circle` 視圖，其位置是從中間往上 200 點以及寬度為 300 點。

對於兩個 `Circle` 視圖，我們都添加了 `matchedGeometryEffect` 修飾器，並且指定了相同的 ID 和命名空間。透過這樣的設定，SwiftUI 可以計算兩個視圖之間大小和位置的差異，並且添加視圖轉換。隨著後續加上的 `animation` 修飾器，SwiftUI 框架會自動動畫化視圖轉換。

ID 以及命名空間的用途，是用來標記屬於同一個轉換的視圖，所以兩個 `Circle` 視圖會使用相同的 ID 和命名空間。

以上我們介紹了如何使用 `matchedGeometryEffect`，來實作兩個視圖之間的動畫轉換效果。如果你有使用過Keynote 的 Magic Move，這個新的修飾器跟 Magic Move非常類似。我建議你使用 iPhone 模擬器來執行這個 App，以便測試這個動畫效果。在我寫這篇文章時， [Xcode 12](https://www.appcoda.com.tw/xcode-12-swift-53/) 中有一個 bug，導致我們無法在預覽畫面測試這個動畫效果。

## 從圓形變為圓角長方形

現在，讓我們來試試實作另一個視圖轉換動畫。這一次，我們會將一個圓形變化成為一個圓角長方形 (rounded rectangle)。圓形位置於螢幕的上端，而圓角長方形則位於螢幕的底端。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-3.png](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-3.png)

我們可以使用剛剛學到的技巧，準備兩個視圖：一個圓形視圖和一個圓角長方形視圖，`matchedGeometryEffect` 修飾器就會處理視圖轉換的部分。現在，讓我們將 `body` 變數的 `ContentView` 結構改成這樣：

```swift
VStack {
    if expand {

        // Rounded Rectangle
        Spacer()

        RoundedRectangle(cornerRadius: 50.0)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(minWidth: 0, maxWidth: .infinity, maxHeight: 300)
            .padding()
            .foregroundColor(Color(.systemGreen))
            .animation(.easeIn)
            .onTapGesture {
                expand.toggle()
            }

    } else {

        // Circle
        RoundedRectangle(cornerRadius: 50.0)
            .matchedGeometryEffect(id: "circle", in: shapeTransition)
            .frame(width: 100, height: 100)
            .foregroundColor(Color(.systemOrange))
            .animation(.easeIn)
            .onTapGesture {
                expand.toggle()
            }

        Spacer()
    }
}
```

我們同樣使用 `expand` 狀態變數，來切換圓形視圖和圓角長方形視圖。這段程式碼跟之前的範例非常相似，只是我們在這邊加上了 `VStack` 和 `Spacer` 來定位視圖。或許你會問，為什麼要使用 `RoundedRectangle` 來建立圓形物件呢？主要原因是這樣可以讓視圖轉換就會更順暢。

在這兩個視圖中，我們都加上了 `matchedGeometryEffect` 修飾器，並且指定了一樣的 ID 以及命名空間，我們要做的事就完成了。修飾器會自動比較兩個視圖的差異，並且使用動畫呈現這個改變。如果你在預覽畫面或是 iPhone 模擬器上執行這個 App，會看到視圖在圓形和圓角長方形之間完美的轉換。這就是 `matchedGeometryEffect` 的威力。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-4.gif](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-4.gif)

不過，你可能注意到這個修飾器無法執行改變顏色的動畫。沒錯，`matchedGeometryEffect` 只能用來處理位置和大小的變化。

## 練習一

讓我們來做一個小小的練習，來測試你對 `matchedGeometryEffect` 的了解。你的任務是要建立如下圖的動畫視圖轉換。開始時，它是一個橘色的圓形視圖，圓形視圖被點擊後，就會轉換成全螢幕的背景圖。你可以在專案檔中找到完整程式碼。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-5.gif](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-5.gif)

### 使用動畫轉換來交換兩個視圖

現在你應該對 `matchedGeometryEffect` 有了基礎的認識，讓我們繼續來看看它如何幫助我們建立一些絢麗的動畫。在這個範例中，我們會交換兩個圓形視圖的位置，並且套用修飾器來建立順暢的視圖轉換。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-6.gif](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-6.gif)

我們會使用一個狀態變數，來儲存交換的狀態，並建立一個命名空間變數給 `matchedGeometryEffect` 來使用。讓我們在 `ContentView` 宣告這些參數：

```
@State private var swap = false

@Namespace private var dotTransition
```

橘色圓形預設位在螢幕的左邊，而綠色圓形則在螢幕的右邊。當使用者點擊任意一個圓形圖案，就會觸發互換的動畫。使用 `matchedGeometryEffect` 時，你不用了解交換動畫是如何達成的。要建立視圖轉換，你只需要做到以下事情：

1. 建立橘色和綠色圓形交換之前的版面配置
2. 建立兩個圓形在交換之後的版面配置

如果你要將版面配置轉換成程式碼，你可以如此編寫 `body` 變數：

```
if swap {

    // After swap
    // Green dot on the left, Orange dot on the right

    HStack {
        Circle()
            .fill(Color.green)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "greenCircle", in: dotTransition)

        Spacer()

        Circle()
            .fill(Color.orange)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "orangeCircle", in: dotTransition)
    }
    .frame(width: 100)
    .animation(.linear)
    .onTapGesture {
        swap.toggle()
    }

} else {

    // Start state
    // Orange dot on the left, Green dot on the right

    HStack {
        Circle()
            .fill(Color.orange)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "orangeCircle", in: dotTransition)

        Spacer()

        Circle()
            .fill(Color.green)
            .frame(width: 30, height: 30)
            .matchedGeometryEffect(id: "greenCircle", in: dotTransition)
    }
    .frame(width: 100)
    .animation(.linear)
    .onTapGesture {
        swap.toggle()
    }
}
```

我們使用 `HStack` 來將兩個圓形配置成水平排列，並且利用 `Spacer` 在兩個圓形中間建立一些空間。當 `swap` 變數設定成 `true` 之後，綠色圓形會被放在橘色圓形的左邊。相反地，綠色圓形則會被放在橘色圓形的右邊。

如你所見，我們只需要描述不同狀態下圓形視圖的配置，`matchedGeometryEffect` 就會處理餘下的事情。我們在每個 `Circle` 視圖加上修飾器，不過這次有一點不同，因為我們有兩個不同的 `Circle` 視圖需要配置，我們使用了兩個不同的 ID 來建立 `matchedGeometryEffect` 修飾器。我們將橘色圓形的識別名稱設定為 `orangeCircle`，而綠色圓形則是設定為 `greenCircle`。

現在，如果你在模擬器執行這個 App，應該可以在點擊任何一個圓形時看到互換的動畫。

## 練習二

在剛剛的練習中，我們在兩個圓形上使用了 `matchedGeometryEffect`，並交換它們的位置。這個練習會使用到一樣的技巧，不過這次是要使用兩張圖片。下面的圖展示了這個範例，點擊交換按鈕時，這個 App 就會用漂亮的動畫來交換這兩張圖片。

![matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-7.gif](matchedGeometryEffect%E6%9E%84%E5%BB%BA%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A7%86%E5%9B%BE%E7%9A%84%E5%8F%98%E5%8C%96%E5%8A%A8%E7%94%BB%2045520ad0da474b51b72cdcf95caac5c6/swiftui-matchedgeometryeffect-7.gif)

你可以隨意使用自己的圖片。我在範例中使用了這些 Unsplash.com 的免費圖片：

- [https://unsplash.com/photos/pMW4jzELQCw](https://unsplash.com/photos/pMW4jzELQCw)
- [https://unsplash.com/photos/PM4Vu1B0gxk](https://unsplash.com/photos/PM4Vu1B0gxk)

## 總結

引進 `matchedGeometryEffect` 修飾器之後，視圖動畫的實作提昇到了另一個層次。你可以用更少的程式碼來創造漂亮的視圖動畫。即便你是 SwiftUI 的新手，你也可以從這個新修飾器中得益，並且讓你的 App 更棒。

**編者備註：**本文摘自 Mastering SwiftUI 一書。如果你想學習更多不同的視圖動畫及參考其原始碼，請到 [AppCoda 網站](https://www.appcoda.com.tw/swiftui) 購買完整書本。這本書已經為Xcode 12 和 iOS 14 所更新。

**譯者簡介：**周竑翊 – 只待過新創公司的 iOS 開發者，本職是兩個兒子的爸。有空的時間喜歡看看新的技術跟科技時事。用 Playground 寫寫 Swift，但是 side project 仍然難產。其他興趣喜歡攝影、運動及看電影。歡迎寄信與我聯絡：[paul.chou.dev@gmail.com](mailto:paul.chou.dev@gmail.com)

**原文**：[Using matchedGeometryEffect to Create View Animations in iOS 14](https://www.appcoda.com/matchedGeometryEffect/)