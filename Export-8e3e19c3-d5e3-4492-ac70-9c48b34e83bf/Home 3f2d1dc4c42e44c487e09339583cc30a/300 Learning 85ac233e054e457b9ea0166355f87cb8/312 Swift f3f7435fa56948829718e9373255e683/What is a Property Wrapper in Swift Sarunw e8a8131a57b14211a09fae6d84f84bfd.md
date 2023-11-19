# What is a Property Wrapper in Swift | Sarunw

In this article

As the name implies, a property wrapper is a new **type that wraps a property** to add additional logic. To understand the benefit of property wrapper and what it is, let's see how we do this property manipulation before the coming of property wrapper.

You can easily support sarunw.com by checking out this sponsor.

![What%20is%20a%20Property%20Wrapper%20in%20Swift%20Sarunw%20e8a8131a57b14211a09fae6d84f84bfd/codeshot.png](What%20is%20a%20Property%20Wrapper%20in%20Swift%20Sarunw%20e8a8131a57b14211a09fae6d84f84bfd/codeshot.png)

## Custom logic over Swift properties

Swift property provides **two ways** for us to implement a custom logic over it.

### Property Observers

You can observe and respond to changes in a property's value with two **[Property Observers](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID262)**, `willSet` and `didSet`.

The following example observe a `counts` properties and print logging message to the console.

```swift
class Counter {
    var counts: Int = 0 {
        willSet {
            print("About to set counts to \(newValue)")
        }
        didSet {
            if counts > oldValue  {
                print("Added \(counts - oldValue)")
            } else {
		print("Removed \(oldValue - counts)")
	    }
        }
    }
}
```

### Computed properties

You can also manage underlying value yourself with a **[Computed Properties](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID259)**.

Here is an example of using computed property to implement lazy properties.

```swift
struct Foo {
  private var _foo: Int?
  var foo: Int {
    get {
      if let value = _foo { return value }
      let initialValue = 1738
      _foo = initialValue
      return initialValue
    }
    set {
      _foo = newValue
    }
  }
}
```

The above code tries to replicate lazy stored properties, `lazy var foo = 1738`.

## Problem

As you can see, some property implementation is not likely to be shared, e.g., the log in our `Counter` contains specific logic to observe the `counts`. But the lazy property has many use cases and likely to be used in many places.

Some of the property implementation patterns that come up repeatedly are hardcoded into the compiler, e.g., `lazy`. Building these patterns into the language has some **disadvantages**. It makes the language and compiler **more complex and inflexible**. You can imagine that the language can't have all the patterns build into the language.

So, How do we do before the time of Property Wrapper?

### Example

Let's start by creating a new property pattern. I want to make a property that stores a truncated string. It will be a very basic implementation where a string property that longer than ten characters will get truncated, and ellipsis (`...`) will be added. If the string is shorter than that, we will keep it as is.

I can build this using a computed property. I use this property in a **BlogTeaser** struct to present a teaser for a blog post.

```swift
struct BlogTeaser {
    var title: String

    private var _body: String = "" // 1
    var body: String { // 2
        set {
            _body = truncate(string: newValue)
        }

        get {
            return _body
        }
    }

    init(title: String, body: String) {
        self.title = title
        self.body = body
    }

    private func truncate(string: String) -> String { // 3
        if string.count > 10 {
            return string.prefix(10) + "..."
        } else {
            return string
        }
    }
}
```

<1> Private data store.
 <2> A computed property that contain truncate logic.
 <3> Our truncate logic.

Run this code, and everything works as expected.

```swift
let teaser = BlogTeaser(
    title: "Hello, SwiftUI",
    body: "Lorem Ipsum is simply dummy text of the printing and typesetting industry.")
print(teaser.body)
// Lorem Ipsu...
```

What if we want to use this pattern elsewhere? If I want to use this again in **CommentTeaser**? I can copy over the code above into my **CommentTeaser** property.

```swift
struct CommentTeaser {
    private var _comment: String = ""
    var comment: String {
        set {
            _comment = truncate(string: newValue)
        }

        get {
            return _comment
        }
    }

    init(comment: String) {
        self.comment = comment
    }

    private func truncate(string: String) -> String {
        if string.count > 10 {
            return string.prefix(10) + "..."
        } else {
            return string
        }
    }
}
```

This approach work, but the truncation logic will scatter everywhere, which is hard to maintain. The better way is to wrap the truncation logic and data storage into a separate struct.

### Create a wrapper for a property

To make our truncated property more reusable, we try to extract it to a dedicated structure.

```swift
struct TruncateWrapper {
    private var _value: String = ""

    init(wrappedValue: String) {
        _value = truncate(string: wrappedValue)
    }

    var wrappedValue: String {
        set {
            _value = truncate(string: newValue)
        }

        get {
            return _value
        }
    }

    private func truncate(string: String) -> String {
        if string.count > 10 {
            return string.prefix(10) + "..."
        } else {
            return string
        }
    }
}
```

As you can see, I copy most truncation-related code out into a separte structure, **TruncateWrapper**. We can remove most of the boilerplate code from BlogTeaser and CommentTeaser this way.

```swift
struct BlogTeaser {
    var title: String
    var body: TruncateWrapper
}

struct CommentTeaser {
    var comment: TruncateWrapper
}
```

The truncation logic is more organized now, but this would make initializing and accessing the value a little bit complicated. We need to wrap it in the `TruncateWrapper` and access the string value from the computed property `wrappedValue`.

```swift
var teaser = BlogTeaserUsingWrapper(
    title: "Hello, SwiftUI",
    body: TruncateWrapper( // 1
        wrappedValue: "Lorem Ipsum is simply dummy text of the printing and typesetting industry."))
print(teaser.body.wrappedValue) // 2
// Lorem Ipsu...

teaser.body.wrappedValue = "What is property Wrapper in Swift" // 3
print(teaser.body.wrappedValue)
// What is pr...
```

<1> Need to wrap our property with a wrapper to get the truncate function.
 <2> Reading it needs to access the nested computed property, `wrappedValue`.
 <3> Changing it also needs to access `wrappedValue`.

You can see that even though we can wrap our property manipulating logic into a wrapper, it feels unnatural and contains a lot of boilerplate code.

To create this kind of property implementation that is easy to reuse, Swift 5.1 has introduced **[Property Wrapper](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)** as a mechanism to allow these patterns to be defined as libraries.

## Property wrapper

A property wrapper **adds a layer of separation between code that manages how a property is stored and the code that defines a property**. This is precisely the [problem](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#problem) we are trying to solve in previous sections.

We already implement a code that manages how a property is stored. That is our [TruncateWrapper](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#create-a-wrapper-for-a-property). You will see that the concept of property wrapper is not rocket science. We can reuse most of the code in our TruncateWrapper and [convert it to property wrapper type](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#property-wrapper-types).

The best thing about property wrappers is the part that helps to define a property and to access them. Because even we can [create a custom wrapper](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#create-a-wrapper-for-a-property), using it still feel alienated and not fit how we usually work with property. Having to wrap and access nested property is troublesome. You will see this change when we try to [convert our TruncateWrapper to property wrapper](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#property-wrapper-types).

In the end, we will convert our TruncateWrapper to a property wrapper that can easily use like a Swift lazy property.

```swift
lazy var df = DateFormatter()

@Truncate var body = "Lorem Ipsum is simply dummy text of the printing and typesetting industry."
body = "What is property Wrapper in Swift"
print(body)
// What is pr...
```

There are **three** areas that you should know about property wrappers.
 I will visit them one by one.

1. [Property wrapper types](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#property-wrapper-types)

## Property wrapper types

To define a property wrapper, we mark a structure, enumeration, or class as property wrapper type (`@propertyWrapper`). A property wrapper type is a type that can be used as a property wrapper. There are **two** requirements for a property wrapper type.

<aside>
💡 **必要条件**

1. It must be defined with the attribute `@propertyWrapper`.
2. It must have a property named `wrappedValue`. This is the property that represents the underlying value of the wrapper instance. In our case, this is a string.
</aside>

Once we know these requirements, we can convert our `TruncateWrapper` into a property wrapper type.

```swift
@propertyWrapper // 1
struct Truncate {
    private var _value: String = ""

    var wrappedValue: String { // 2
        set {
            _value = truncate(string: newValue)
        }

        get {
            return _value
        }
    }

    private func truncate(string: String) -> String {
        if string.count > 10 {
            return string.prefix(10) + "..."
        } else {
            return string
        }
    }
}
```

<1> Add the `@propertyWrapper` attribute to let the compiler know that this is a property wrapper type, so it can verify and synthesize appropriated [helpers](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#synthesized-storage-properties).
 <2> We must have a property named `wrappedValue` with a type that we want to use. In this case, a string.

By adding `@propertyWrapper`, we now have a functional property wrapper type. To use it, we annotate any property which has the same type as our wrapped value (string) with a property wrapper type prefixed by the `@` sign. This tells the compiler that we want this property to be managed by that property wrapper.

```swift
struct BlogTeaser {
    var title: String
    @Truncate var body: String // 1
}

var teaser = BlogTeaser(
    title: "Hello, SwiftUI")
print(teaser.body) // 2
// empty string

teaser.body = "What is property Wrapper in Swift" // 3
print(teaser.body)
// What is pr...
```

<1> You apply a wrapper to a property by writing the wrapper’s name before the property as an attribute.
 <2> You can treat it like a normal string.
 <3> Try to set it with a string longer than ten characters, and you will see our string got truncated as expected.

This small change is all you need to do to create a property wrapper type. One noticeable change of property wrapper type over our TruncateWrapper is that we no longer need to reference the `wrappedValue`. This makes a huge difference from our custom wrapper.

Right now, you can't initialize BlogTeaser with a body yet. Doing so at this stage and you will get the following error.

```
var teaser = BlogTeaser(
    title: "Hello, SwiftUI",
    body: "What is property Wrapper in Swift")
    // Cannot convert value of type 'String' to expected argument type 'Truncate'
```

To be able to do that, you need to understand what the `@Truncate` attribute is doing behind the scene.

### Synthesized storage properties

The compiler does **three** things when we add a property wrapper to a property.

1. It **makes that property computed** (generate a getter/setter).
2. It **introduces a stored property** whose **type is the wrapper type**.
3. Initialize the stored property in one of **[three ways](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#how-to-initialize-property-wrapper)**.

This is what the compiler did to our `@Truncate var body: String` behind the scene.

```
struct BlogTeaser {
    var title: String

    // synthesis code for
    // @Truncate var body: String
    private var _body = Truncate() // 1
    var body: String { // 2
        get {
            return _body.wrappedValue

        }
        set {
            _body.wrappedValue = newValue
        }
    }
}
```

<1> Introduce a stored property for wrapper type and initialize it with `init()`.
 <2> Generate a computed property that accesses `wrappedValue` with the same name as our declaration, `body`.

The `_body` property store an instance of the property wrapper, `Truncate`. The `body` accesses the `wrappedValue` property throught the `get` and `set`. This is the magic behind the property wrapper attribute.

You can verify this by trying to access `_body` in BlogTeaser. 

`struct BlogTeaser {    var title: String    @Truncate var body: String    func foo() {        print(_body) // You can access _body of type Truncate    }}`

Knowing this means you can create your own helpers without relying on a compiler. You are not likely to do so, though.

This might remind you of an old friend, `@synthesize`, in Objective-C.

Understand the synthesized storage properties process is the first step to understand the initialization of the property wrapper. So far, you have seen only one way to initialize the property wrapper (implicitly via `init()`). As I mentioned earlier, there are three ways of doing it. Knowing all of them will open you to all possibilities of a property wrapper.

## How to initialize property wrapper

The [stored property](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#synthesized-storage-properties) can be initialized in one of **three** ways. These options available based on [initializers](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#type-of-initializer) you have in a property wrapper type.

1. [Implicitly](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#implicitly).
2. [Initial value provided on the **property declaration**](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#at-property-declaration).
3. [Placing the initializer arguments **after the property wrapper type**](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#after-the-property-wrapper-type).

### Implicitly

When **nothing is assigned during a property declaration** and the property wrapper type **provides a no-parameter initializer (`init()`)**. In such cases, the wrapper type's `init()` will be invoked to initialize the stored property.

This is a case for our `@Truncate`.

```swift
@Truncate var body: String

// Implemented as
private var _body: Truncate = Truncate()
var body: String { /* access via _body.wrappedValue */ }
```

### At property declaration

If a property wrapper type provides an initializer with the original property's type (`String` in our case), it will be used when we provide an initial value at declaration time.

The initializer must have a single parameter of the same type as the `wrappedValue`.
 You must name the parameter `wrappedValue` (`init(wrappedValue: String)`).
 Have the same access level as the property wrapper type itself.

If we want to make our Truncate property wrapper initializable at declaration time, we have to add `init(wrappedValue: String)` initializer.

```swift
@propertyWrapper
struct TruncateWithInitWrappedValue {
    private var _value: String = ""

    var wrappedValue: String {
        set {
            _value = truncate(string: newValue)
        }

        get {
            return _value
        }
    }

    init(wrappedValue: String) { // 1
        _value = truncate(string: wrappedValue)
    }

    private func truncate(string: String) -> String {
        if string.count > 10 {
            return string.prefix(10) + "..."
        } else {
            return string
        }
    }
}
```

<1> Add a new initializer. That initializer must have a single parameter of the same type as the `wrappedValue` property and named `wrappedValue`.

With this change, we can now initialize our property wrapper at declaration time.

```swift
struct BlogTeaser {
    var title: String
    @Truncate var body: String = "Hello, SwiftUI!"

    // Implemented as
    private var _body: Truncate = Truncate(wrappedValue: "Hello, SwiftUI!")
    var body: String { /* access via _body.wrappedValue */ }
}
```

This change also affects memberwise initializers. The compiler will create an initializer that accepts wrapped value.

You can now initialize BlogTeaser like this without any warning.

```swift
var blog = BlogTeaser(
    title: "Hello, SwiftUI",
    body: "What is property Wrapper in Swift")
```

Under the hood, you get an initializer that took advantage of the newly created `init(wrappedValue: String)`.

```swift
struct BlogTeaser {
    var title: String
    @Truncate var body: String

    // implicit memberwise initializer:
    init(title: String,
        body: String) {
        self.title = title
        self._body = Truncate(wrappedValue: body)
    }
}
```

Without `init(wrappedValue: String)`, we will get the error that we see in the previous section.

You can use this knowledge to create a custom initializer that initializes a property wrapper.

### After the property wrapper type

This is **the most flexible way to initialize** a property wrapper of all three forms. You can use any initializer signature you want. You specify the initializer by placing the initializer arguments after the property wrapper type.

I add a new initializer where a string is generated from an integer.

```swift
@propertyWrapper
struct Truncate {
    private var _value: String = ""

    var wrappedValue: String {
        set {
            _value = truncate(string: newValue)
        }

        get {
            return _value
        }
    }

    init(customInt: Int) { // 1
        let stringFromInt = String(repeating: "Ho ", count: customInt)
        _value = truncate(string: stringFromInt)
    }

    private func truncate(string: String) -> String {
        if string.count > 10 {
            return string.prefix(10) + "..."
        } else {
            return string
        }
    }
}
```

<1> New initializer that generates string based on the passing argument.

You can use it like this:

```swift
struct BlogTeaser {
    var title: String
    @Truncate(customInt: 5) var body: String // 1

    // Implemented as
    private var _body: Truncate = Truncate(customInt: 5)
    var body: String { /* access via _body.wrappedValue */ }
}

var blog = BlogTeaser(title: "Custom Init")
print(blog.body)
// Ho Ho Ho H...
```

<1> Placing the initializer arguments after the property wrapper type.

### After the property wrapper type (Continue)

You can also combine [the third form](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#after-the-property-wrapper-type) with [the second form](https://sarunw.com/posts/what-is-property-wrappers-in-swift/#at-property-declaration). So you can initialize your property wrapper at declaration with one or more arguments.

This work with two conditions:

1. If your property wrapper type provided an initializer with the first parameter as the original property's type (`String` in our case) and named it `wrappedValue`.
2. That initialize has one or more extra parameters.

Let's say I want my wrapper to have an adjustable character limit instead of a hard-coded value of ten. Here is my updated version.

```swift
@propertyWrapper
struct Truncate {
    private var _value: String = ""
    private let maximumLength: Int// 1

    var wrappedValue: String {
        set {
            _value = truncate(string: newValue)
        }

        get {
            return _value
        }
    }

    init(wrappedValue: String, maximumLength: Int = 10) { // 2
        self.maximumLength = maximumLength
        _value = truncate(string: wrappedValue)
    }

    private func truncate(string: String) -> String {
        if string.count > maximumLength { // 3
            return string.prefix(maximumLength) + "..."
        } else {
            return string
        }
    }
}
```

<1> I declare a new private variable to keep a maximum length allow for my `body`.
 <2> I create a new initializer that follows the rules. I also set default value for the `maximumLength`, so we can use it in two form as you will see.
 <3> I use the value from `maximumLength` instead of a hard-coded value.

Then, create a new struct to use our new Truncate.

```swift
struct BlogWithTeaserOfTen {
    var title: String
    @Truncate var body: String = "12345678910"

    // Implemented as
    private var _body: Truncate = Truncate(wrappedValue: "12345678910") // with maximumLength of 10 (default value)
    var body: String { /* access via _body.wrappedValue */ }
}

struct BlogWithTeaserOfThree {
    var title: String
    @Truncate(maximumLength: 3) var body: String = "12345678910"

    // Implemented as
    private var _body: Truncate = Truncate(wrappedValue: "12345678910", maximumLength: 3)
    var body: String { /* access via _body.wrappedValue */ }
}
```

Then, you can use it like this:

```swift
var blog1 = BlogWithTeaserOfTen(title: "Ten")
print(blog1.body)
// 1234567891...

var blog2 = BlogWithTeaserOfThree(title: "Three")
print(blog2.body)
// 123...
```

That all you need to know to use a property wrapper. The final piece is optional. Not every property wrapper needs this, but you should know it nonetheless. It is **Projection**.

## Projections

In addition to the wrapped value, a property wrapper can **expose additional functionality** by defining a projected value (`projectedValue`). Projected value has the following requirements.

1. The `projectedValue` property must have the **same access level as its property wrapper type**.
2. It must be named `projectedValue`.
3. It can be any type.
4. You can refer to a projected value by the same name as your wrapped value, except it **begins with a dollar sign (`$`)**.

The code below adds a `projectedValue` property to the Truncate structure to keep track of whether the property wrapper truncated the new value or not.

```swift
@propertyWrapper
struct Truncate {
    private var _value: String = ""
    private let maximumLength: Int

    var projectedValue: Bool = false // 1
    var wrappedValue: String {
        set {
            _value = truncate(string: newValue)
        }

        get {
            return _value
        }
    }

    init(wrappedValue: String, maximumLength: Int = 10) {
        self.maximumLength = maximumLength
        _value = truncate(string: wrappedValue)
    }

    private mutating func truncate(string: String) -> String {
        if string.count > maximumLength {
            projectedValue = true // 2
            return string.prefix(maximumLength) + "..."
        } else {
            projectedValue = false // 3
            return string
        }
    }
}
```

<1> Add `projectedValue` property.
 <2> <3> Update `projectedValue` once we set a new `wrappedValue`.

After you add `projectedValue`, you can access it using a dollar sign (`$`).

```swift
struct BlogTeaser {
    var title: String
    @Truncate var body: String
}

var blog = Blog(
    title: "Projected Value",
    body: "Not long")
print(blog.body)
// Not long
print(blog.$body) // 1
// false

blog.body = "Long enough"
print(blog.body)
// Long enoug...
print(blog.$body) // 2
// true
```

<1>, <2> Access projected value by prefix variable name with a dollar sign.

The projected value interface got generated the same way as the wrapped value.

```swift
@Truncate var body: String

// Implemented as
public var $body: Bool {
  get { return _body.projectedValue }
  set { _body.projectedValue = newValue }
}
```

A property wrapper can return a value of any type as its projected value. In this example, the property wrapper exposes only one piece of information—whether the string was truncated—so it exposes that Boolean value as its projected value. A wrapper that needs to expose more information can return an instance of some other data type or return itself to expose the wrapper's instance as its projected value.

One example of a property wrapper that you might already see is `[@State](https://developer.apple.com/documentation/swiftui/state)`, a property wrapper for SwiftUI. State's projected value returns another property wrapper, `[Binding](https://developer.apple.com/documentation/swiftui/binding)`.

You can easily support sarunw.com by checking out this sponsor.

[Turn your code into a snapshot: Codeshot creates a beautiful image of your code snippets. Perfect size for Twitter.](https://codeshotapp.com/)

## Conclusion

Knowing a property wrapper doesn't mean you have to convert everything to it. I think this is a tool that you should use with caution.

Even though you have no plan to use it in the project, it is worth knowing because it is heavily used in SwiftUI. SwiftUI contains a lot of magic, and knowing the property wrapper makes it more understandable for me.

There is a lot more implementation detail about property wrappers. If you want to find out more, you can visit [property wrapper proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md) and [Swift Documentation](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID617).

You may also like

[Data in SwiftUI, Part 1: Data](https://sarunw.com/posts/data-in-swiftui-1/)

Part 1 in a series on understanding data in SwiftUI. In the first part, we will try to understand the importance of data and how they play an essential role in SwiftUI.

[How to parse ISO 8601 date in Swift](https://sarunw.com/posts/how-to-parse-iso8601-date-in-swift/)  [Swift](https://sarunw.com/tags/Swift/)

[Sort array of objects by multiple properties with Swift Tuple](https://sarunw.com/posts/sort-array-of-objects-by-multiple-properties-with-swift-tuple/)  [Swift](https://sarunw.com/tags/Swift/)

Read more article about [Swift](https://sarunw.com/tags/Swift/) or see [all available topic](https://sarunw.com/tags)

## Enjoy the read?

If you enjoy this article, you can subscribe to the weekly newsletter.**Every Friday**, you'll get a quick **recap of all articles and tips posted on this site**. No strings attached. Unsubscribe anytime.

Feel free to follow me on [Twitter](https://twitter.com/sarunw) and ask your questions related to this post. Thanks for reading and see you next time.

If you enjoy my writing, please check out my Patreon [https://www.patreon.com/sarunw](https://www.patreon.com/sarunw) and become my supporter. Sharing the article is also greatly appreciated.

[4 Xcode shortcuts to boost your productivity for SwiftUI](https://sarunw.com/posts/xcode-shortcuts-for-swiftui/)

Leaning tips and tricks about the tool will help you down the road. Today, I will show you 4 Xcode shortcuts that I find helpful when dealing with SwiftUI.

[How to make a simple bevel effect using inner shadows in SwiftUI](https://sarunw.com/posts/how-to-make-bevel-effect-in-swiftui/)

We can make a simple bevel effect using two inner shadows. SwiftUI has a built-in way to add a drop shadow with the shadow modifier. But if you want to add an inner shadow effect, you need to be a bit creative.