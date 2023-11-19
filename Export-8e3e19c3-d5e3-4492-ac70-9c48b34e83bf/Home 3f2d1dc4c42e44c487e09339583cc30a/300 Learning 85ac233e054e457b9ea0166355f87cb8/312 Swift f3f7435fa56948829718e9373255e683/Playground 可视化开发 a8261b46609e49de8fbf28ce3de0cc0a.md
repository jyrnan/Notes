# Playground 可视化开发

# **用于Swift UI的类型**

```swift
import SwiftUI
import PlaygroundSupport

struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}

PlaygroundPage.current.setLiveView(ContentView())
```

# UIKit

```swift
import UIKit
import PlaygroundSupport

let label = UILabel(frame: CGRect(x: 0, y: 0, width: 400, height: 200))
label.backgroundColor = .white
label.font = UIFont.systemFont(ofSize: 32)
label.textAlignment = .center
label.text = "Hello World"
PlaygroundPage.current.liveView = label
```