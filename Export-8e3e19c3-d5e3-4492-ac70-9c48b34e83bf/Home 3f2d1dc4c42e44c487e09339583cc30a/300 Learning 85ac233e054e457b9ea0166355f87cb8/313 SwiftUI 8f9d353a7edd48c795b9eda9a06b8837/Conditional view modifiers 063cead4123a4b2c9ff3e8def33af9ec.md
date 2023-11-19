# Conditional view modifiers

When working with SwiftUI views, sometimes we would like to apply different modifiers based on conditions/states:

```swift
var body: some view {
  myView
    // if X
    // .padding(8)
    // if Y
    // .background(Color.blue)
}
```

For many cases, we can pass a different modifier *argument* based on the condition, and that will take care of it:

```swift
var body: some view {
  myView
    .padding(X ? 8 : 0)
    .background(Y ? Color.blue : Color.clear)
}
```

While this works here, there are other modifiers, `.hidden()` to name one, where this solution doesn’t work:
in this article, let’s explore how we can take care of such cases.

# The if view extension

The most common solution is to define a new if View extension:

```swift
extension View {
  @ViewBuilder
  func `if`<Transform: View>(
    _ condition: Bool, 
    transform: (Self) -> Transform
  ) -> some View {
    if condition {
      transform(self)
    } else {
      self
    }
  }
}
```

This function will apply transform to our view when condition is true, otherwise it will leave the original view untouched.
Going back to our example, this is one way to use it:

```swift
var body: some view {
  myView
    .if(X) { $0.padding(8) }
    .if(Y) { $0.background(Color.blue) }
}
```

# If else view extension

Depending on how compact we want our declarations to be, applying different modifiers based on the condition true/false value would cost us at least two modifiers:

```swift
var body: some view {
  myView
    .if(X) { $0.padding(8) }
    .if(!X) { $0.background(Color.blue) }
}
```

This is clear and already succinct, however, if we really want to go all-in with View extensions, we can define a new if overload that lets us modify the else branch as well:

```swift
extension View {
  @ViewBuilder
  func `if`<TrueContent: View, FalseContent: View>(
    _ condition: Bool, 
    if ifTransform: (Self) -> TrueContent, 
    else elseTransform: (Self) -> FalseContent
  ) -> some View {
    if condition {
      ifTransform(self)
    } else {
      elseTransform(self)
    }
  }
}
```

Which will make our example use a single modifier:

```swift
var body: some view {
  myView
    .if(X) { $0.padding(8) } else: { $0.background(Color.blue) }
}
```

# IfLet view extension

Similarly to conditions, sometimes we want to apply a modifier only when another value is not nil, similarly to how Swift if let works, and use that value in the modifier itself.
In this case we can define a new View extension that lets us do just that:

```swift
extension View {
  @ViewBuilder
  func ifLet<V, Transform: View>(
    _ value: V?, 
    transform: (Self, V) -> Transform
  ) -> some View {
    if let value = value {
      transform(self, value)
    } else {
      self
    }
  }
}
```

The difference from before is that this new function:

- takes in an optional generic value V instead of a Bool condition
- passes this generic value V as a parameter of the transform function

Here’s an example where a View applies a foreground color only when optionalColor is set:

```swift
var body: some view {
  myView
    .ifLet(optionalColor) { $0.foregroundColor($1) }
}
```

# iOS Availability modifiers

Months ago [we covered an approach](https://fivestars.blog/code/use-new-features-mantain-backward-compatibility.html) on how to use new iOS features while maintaining backward compatibility. We now find ourselves in a similar situation with SwiftUI, where new modifiers have been introduced, and where we would like to ship an app compatible with earlier versions of iOS 13.

Unfortunately, in these situations we cannot use the generic extensions that we just introduced:

- Swift’s #available and @available cannot be passed as arguments in our if modifier
- we can’t guarantee the compiler that our transform function would be applied only on iOS 14/13.4 and later

*If you find a way, [I would love to know](https://twitter.com/zntfdr)!*

One way to overcome this is to define a different view modifier for each of our use cases.

For example, here’s a View extension to ignore the keyboard in our layout:

```swift
extension View {
  @ViewBuilder
  func ignoreKeyboard() -> some View {
    if #available(iOS 14.0, *) {
      ignoresSafeArea(.keyboard)
    } else {
      self // iOS 13 always ignores the keyboard
    }
  }
}
```

And here’s how to have the InsetGroupedList style in both iOS 13 and iOS 14:

```swift
extension List {
  @ViewBuilder
  func insetGroupedListStyle() -> some View {
    if #available(iOS 14.0, *) {
      self
        .listStyle(InsetGroupedListStyle())
    } else {
      self
        .listStyle(GroupedListStyle())
        .environment(\.horizontalSizeClass, .regular)
    }
  }
}
```

To be used as follows:

```swift
var body: some view {
  myView
    .ignoreKeyboard()
}

// ...

var body: some view {
  myView
    .insetGroupedListStyle()
}
```

# Availability Attributes

*Credits to [Ole Begemann](https://twitter.com/olebegemann) for [this tip](https://twitter.com/olebegemann/status/1294583027583123458).*

At some point in the future, our codebases will drop support for iOS 13, making our extensions unnecessary:

wouldn’t it be great if Xcode could let us know when this happens?

Similarly to how libraries/frameworks define API availability, we can use the `@availability` attribute in our new extensions as well:

```swift
@available(
  iOS, introduced: 13, deprecated: 14,
  message: "Use .ignoresSafeArea(.keyboard) directly"
) 
extension View {
  @ViewBuilder
  func ignoreKeyboard() -> some View {
    ...
  }
}

/// ...

@available(
  iOS, introduced: 13, deprecated: 14, 
  message: "Use .listStyle(InsetGroupedListStyle()) directly"
)
extension List {
  @ViewBuilder
  func insetGroupedListStyle() -> some View {
    ...
  }
}
```

Adding this `@available` attribute will trigger a deprecation warning wherever these functions are used, only after iOS 13 support is removed:

we can also add an optional message reminding us what to do once the warning is triggered.

# Conclusions

SwiftUI declarative APIs make Views definition a breeze: when our views need to apply different modifiers based on certain conditions, we can define our own conditional view modifiers, letting us keep the same declarativeness that we’re accustomed to.

Thank you for reading and [stay tuned](https://fivestars.blog/feed.xml) for more SwiftUI articles!