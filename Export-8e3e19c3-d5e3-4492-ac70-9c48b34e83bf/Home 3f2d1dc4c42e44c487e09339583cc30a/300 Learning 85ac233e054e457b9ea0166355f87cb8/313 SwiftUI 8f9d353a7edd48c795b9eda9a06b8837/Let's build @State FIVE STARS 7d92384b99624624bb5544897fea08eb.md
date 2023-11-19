# Let's build @State | FIVE STARS

# [Let's build @State](https://www.fivestars.blog/articles/lets-build-state)

> Shout out to Mike Ash for the original, historic Let's build NSObject article.
> 

`[@State](https://developer.apple.com/documentation/swiftui/state)` is one of the many SwiftUI's pillars that, once understood, we take for granted and use pretty much everywhere without a second thought. But what is `@State`? What's happening behind the scenes?

In this article, let's try to answer those questions by re-building `@State`, and more.

> As usual, I have no access to the actual SwiftUI code/implementation:what we will find here is a best guess on mocking the original @State behavior, there's probably much more to it in the real implementation.
> 

## Property wrapper

First of all, `@State` is a [property wrapper](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID617), which in short is a fancy getter and setter with extra logic and storage. Let's start by defining our state as following:

```swift
@propertyWrapper
struct FSState {

}
```

A property wrapper requires a `wrappedValue`, letting us read/write the associated value.
Since we want to mock `@State`, we will make our property wrapper generic over a type `V`, and store the original value in a internal `value` property:

```swift
@propertyWrapper
struct FSState<V> {
  // This is where our value is actually stored.
  var value: V

  // And here are our getter/setters.
  var wrappedValue: V {
    get {
      value
    }
    set {
      value = newValue
    }
  }
}

```

Lastly, if we want to provide the same syntax as `@State` and all other property wrappers (e.g. `@State var x = "hello"`), we will need to declare a special initializer:

```swift
@propertyWrapper
struct FSState<V> {
  var value: V

  var wrappedValue: V {
    ...
  }

  init(wrappedValue value: V) {
    self.value = value
  }
}

```

With this definition we can now go ahead and start using `@FSState` in a view, for example:

```swift
struct ContentView: View {
  @FSState var text = "Hello Five Stars"

  var body: some View {
    Text(text)
  }
}

```

## nonmutating

So far our definition is not much different than having a property defined directly in the view itself.
If we remove `@FSState` from our `ContentView` declaration, everything still works great:

```swift
struct ContentView: View {
  var text = "Hello Five Stars"

  var body: some View {
    Text(text)
  }
}

```

![Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image1.png](Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image1.png)

Let's now try to change the text with a button for example:

```swift
struct ContentView: View {
  @FSState var text = "Hello Five Stars"

  var body: some View {
    VStack {
      Text(text)

      Button("Change text") {
        text = ["hello", "five", "stars"].randomElement()!
      }
    }
  }
}

```

Unfortunately, this won't build:
we get a `Cannot assign to property: 'self' is immutable` error on the button action.

The issue is that assigning to `text` would mutate/change `ContentView`.

With structs we can declare `mutating` methods, but we cannot declare `mutating` computed properties (like `body`), nor we can call `mutating` methods in them.

To overcome this we must not change `ContentView`, which means we cannot change `FSState` neither, as our property wrapper is just another value type nested in our view.

First, let's declare our property wrapper setter as `nonmutating`, which tells Swift that setting this value won't change our `FSState` instance:

```swift
@propertyWrapper
struct FSState<V> {
  var value: V

  var wrappedValue: V {
    get { ... }
    nonmutating set { // our setter is now nonmutating
      value = newValue
    }
  }

  ...
}

```

We've now moved the `Cannot assign to property: 'self' is immutable` build error from our `text` assignment to `FSState`'s `wrappedValue` setter.
This makes sense, as we're promising to not mutate the struct instance, but then we're setting `value = newValue`, which is mutating.

This is where Swift's reference types come in: if we replace `FSState`'s `value` property with a class type, and then update that class instance in our setter, we're effectively not changing `FSState` (as `FSState` would only contain the reference to that class, which always stays the same).

Let's define this "container" class type:

```swift
final class Box<V> {
  var value: V

  init(_ value: V) {
    self.value = value
  }
}

```

`Box` is a generic class that only has one function: hold and update our value.

Let's make `@FSState`'s declaration take advantage of this class:

```
@propertyWrapper
struct FSState<V> {
  var box: Box<V>

  var wrappedValue: V {
    get {
      box.value
    }
    nonmutating set {
      box.value = newValue
    }
  }

  init(wrappedValue value: V) {
    self.box = Box(value)
  }
}

```

With this update we can build and run our app!

![Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image2.gif](Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image2.gif)

We tap the button but see no change, if we set breakpoints we will see that everything works: tapping the button correctly sets and updates our state, however the new challenge is letting SwiftUI know.

It's true that we're updating our data, but SwiftUI doesn't know that it should listen to such change and trigger a redraw of its body, let's tackle that next.

## DynamicProperty

Similarly to how SwiftUI has a set of known [view primitives](https://www.fivestars.blog/articles/impossible-swiftui-views/), SwiftUI has also a set of known publishers that each view can listen to, based on the properties defined within that view.

The SwiftUI team has done an astonishing job at hiding SwiftUI's heavy use of Combine:
when we associate a view property with `@State`, `@ObservedObject`, etc SwiftUI will listen to all publishers connected to each property wrapper, which in turn tell SwiftUI when it's time to redraw.

In our case let's use `[@StateObject](https://developer.apple.com/documentation/swiftui/stateobject)` by conforming `Box` to `[ObservableObject](https://developer.apple.com/documentation/Combine/ObservableObject)`. Combine associates an `[objectWillChange](https://developer.apple.com/documentation/combine/observableobject/objectwillchange-2oa5v)` publisher to all `ObservableObject` instances, which we can then use to send events to SwiftUI by calling `[send()](https://developer.apple.com/documentation/combine/observableobjectpublisher/send())`:

```swift
final class Box<V>: ObservableObject {
  var value: V {
    willSet {
      // This is where we send out our "hey, something has changed!" event
      objectWillChange.send()
    }
  }

  init(_ value: V) {
    self.value = value
  }
}

```

> There are easier ways to declare this, but in this article we're trying to see how things work by removing as much "magic" as possible.
> 

With `Box`'s definition updated, we can now go back to `@FSState` and associate `@StateObject` to the `box` property:

```swift
@propertyWrapper
struct FSState<V> {
  @StateObject var box: Box<V>

  var wrappedValue: V {
    ...
  }

  init(wrappedValue value: V) {
    self._box = StateObject(wrappedValue: Box(value))
  }
}

```

Thanks to this update every time `box`'s value changes:

- an `[objectWillChange](https://developer.apple.com/documentation/combine/observableobject/objectwillchange-2oa5v)` event is fired
- and an observer (SwiftUI?) of `box`'s publisher would know about it

Let's run our app once again:

Unfortunately, we're not there yet. While it's true that the new publisher is sending out events when our value changes, we still need to tell SwiftUI about it:
from SwiftUI's point of view, `ContentView` has a `text` property of type `FSState<String>`, which is not something SwiftUI needs to pay attention to.

To change this, we need to make `FSState` conform to `[DynamicProperty](https://developer.apple.com/documentation/swiftui/dynamicproperty)`, described in the documentation as `An interface for a stored variable that updates an external property of a view.`.

Now, this is something SwiftUI is interested in! By making `FSState` conform to `DynamicProperty`, SwiftUI will listen to its events (if any) and trigger a redraw when needed.

`DynamicProperty` requires only the implementation of an `update()` function, however SwiftUI already provides its default implementation, all we need to do is add the `DynamicProperty` conformance and we're good to go:

```swift
@propertyWrapper
struct FSState<V>: DynamicProperty {
  ...
}

```

With this last change let's try to run our app once again:

![Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image3.gif](Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image3.gif)

It works!
Despite adding this `DynamicProperty` conformance, we still didn't declare exactly which properties SwiftUI should listen to:[similarly to how view equatability works](https://www.fivestars.blog/articles/swift-protocols/), I suspect SwiftUI uses Swift's reflection to iterate over all stored properties and look for known property wrapper types to subscribe to.

> For an open source example on how to use reflection this way, refer to my deep dive into Apple's ArgumentParser's implementation, where the same approach is used to find the various command line arguments.
> 

## Binding

An optional feature of property wrappers is to expose a projected value:
a projected value is an alternative look at the value stored within the property wrapper, exposed in a different manner.

Many SwiftUI views use bindings to refer to and potentially mutate values owned and stored somewhere else. An example of this is `[TextField](https://developer.apple.com/documentation/swiftui/textfield)` which uses a `Binding<String>`:

```swift
struct ContentView: View {
  @FSState var text = ""

  var body: some View {
    VStack {
      TextField("Write something", text: $text) // TextField's text is a binding
    }
  }
}

```

As seen above, we can get a binding from a `@State` by calling the associate property with a `$` in front of the property name, what this symbol really does is reaching for the projected value instead of the wrapped one.

Therefore `@State`'s projected value is a generic `[@Binding](https://developer.apple.com/documentation/swiftui/binding)` over its type `V`, let's add the same projected value in `@FSState`:

```swift
@propertyWrapper
struct FSState<V>: DynamicProperty {
  @ObservedObject private var box: Box<V>

  var wrappedValue: V {
    ...
  }

  var projectedValue: Binding<V> {
    Binding(
      get: {
        wrappedValue
      },
      set: {
        wrappedValue = $0
      }
    )
  }

  ...
}

```

And voila', we can now use `@FSState` with bindings!

![Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image4.gif](Let's%20build%20@State%20FIVE%20STARS%207d92384b99624624bb5544897fea08eb/image4.gif)

Here's the final @FSState definition:

```swift
@propertyWrapper
struct FSState<V>: DynamicProperty {
  @StateObject private var box: Box<V>

  var wrappedValue: V {
    get {
      box.value
    }
    nonmutating set {
      box.value = newValue
    }
  }

  var projectedValue: Binding<V> {
    Binding(
      get: {
        wrappedValue
      },
      set: {
        wrappedValue = $0
      }
    )
  }

  init(wrappedValue value: V) {
    self._box = StateObject(wrappedValue: Box(value))
  }
}

final class Box<T>: ObservableObject {
  var value: T {
    willSet {
      objectWillChange.send()
    }
  }

  init(_ value: T) {
    self.value = value
  }
}

```

## Conclusions

Once again the more we explore SwiftUI, the more it shows how much complexity can be hidden in a simple, elegant API.

Most developers won't ever need to worry about how things really work behind the scenes, however I cannot help but appreciate all the effort that has been done in order to get to this beautiful *state*.

I'm sure `@FSState` is not as complete as the real `@State`: if there's something that I missed or if you have more insights on something I glossed over, [I'd love to know](https://twitter.com/zntfdr)!

Thank you for reading and [stay tuned](https://www.fivestars.blog/feed.rss) for more articles!