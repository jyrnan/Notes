# @AppStorage & @scenenStroage

# **@AppStorage**

SwiftUI has a dedicated property wrapper for reading values from **UserDefaults**, which will automatically reinvoke your view’s body property when the value changes. That is, this wrapper effectively watches a key in **UserDefaults**, and will refresh your UI if that key changes.

```swift
struct ContentView: View {
    @AppStorage("username") var username: String = "Anonymous"

    var body: some View {
        VStack {
            Text("Welcome, \(username)!")
            
            TextField("input you name", text: self.$username)

            Button("Log in") {
                self.username = "@twostraws"
            }
        }
    }
}
```

# **@scenenStroage**

If you want to save unique data for each of your screens, you should use SwiftUI’s **@SceneStorage** property wrapper. This works a bit like **@AppStorage** in that you provide it with a name to save things plus a default value, but rather than working with **UserDefaults** it instead gets used for state restoration – and it even works great with the kinds of complex multi-scene set ups we see so often in iPadOS.

```swift
struct ContentView: View {
    @SceneStorage("text") var text = "" //貌似也可以保存好数据，即便app被干掉

    var body: some View {
        NavigationView {
            TextEditor(text: $text)
        }

    }
}
```

<aside>
💡 二者似乎都够保留数据，但是前者是基于**UserDefaults，适用于整个app，而后者则是适用于某个scenen，scenenStorage应该只能保存String，Int，URL，Data，Double等数据**

</aside>