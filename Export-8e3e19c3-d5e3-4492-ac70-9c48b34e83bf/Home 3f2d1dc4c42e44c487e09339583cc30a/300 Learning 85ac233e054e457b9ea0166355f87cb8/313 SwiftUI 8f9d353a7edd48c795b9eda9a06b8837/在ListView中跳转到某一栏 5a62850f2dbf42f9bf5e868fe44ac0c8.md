# 在ListView中跳转到某一栏

• If you want to programmatically make SwiftUI’s **List** move to show a specific row, you should embed it inside a **ScrollViewReader**. This provides a **scrollTo()** method on its proxy that can move to any row inside the list, just by providing its ID and optionally also an anchor.

![struct ContentView View{.png](%E5%9C%A8ListView%E4%B8%AD%E8%B7%B3%E8%BD%AC%E5%88%B0%E6%9F%90%E4%B8%80%E6%A0%8F%205a62850f2dbf42f9bf5e868fe44ac0c8/struct_ContentView_View.png)