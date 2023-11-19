# 把View存储成Properties

可以使得代码主体看上去更加简洁明了
If you have several views nested inside another view, you might find it useful to create properties for some or all of them to make your layout code easier. You can then reference those properties inline inside your view code, helping to keep it clear.

```swift
struct ContentView: View {
    let title = Text("Paul Hudson")
                    .font(.largeTitle)
    let subtitle = Text("Author")
                    .foregroundColor(.secondary)

    var body: some View {
        VStack {
            title
            subtitle
        }
    }
}
```