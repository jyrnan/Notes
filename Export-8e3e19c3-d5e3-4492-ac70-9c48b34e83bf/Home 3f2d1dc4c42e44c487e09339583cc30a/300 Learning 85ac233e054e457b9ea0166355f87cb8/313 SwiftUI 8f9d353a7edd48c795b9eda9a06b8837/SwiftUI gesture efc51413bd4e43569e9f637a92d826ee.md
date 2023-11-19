# SwiftUI gesture

# 基本手势

SwiftUI gives us lots of gestures **for** working with views, and does a great job of taking away most of the hard work so we can focus on the parts that matter. Easily the most common **is** our friend `onTapGesture()`, but there are several others, and there are also interesting ways of combining gestures together that are worth trying out.

## 多次点击手势

I’m going to skip past the simple onTapGesture() because we’ve covered it before so many times, but before we **try** bigger things I **do** want to add that you can pass a count parameter to these to make them handle double taps, triple taps, and more, like this:

```swift
Text("Hello, World!")
    .onTapGesture(count: 2) {
        print("Double tapped!")
    }
```

## 长按手势

OK, **let**’s look at something more interesting than simple taps. For handling long presses you can use `onLongPressGesture()`, like this:

```swift
Text("Hello, World!")
    .onLongPressGesture {
        print("Long pressed!")
    }
```

Like tap gestures, long press gestures are also customizable. For example, you can specify a minimum duration **for** the press, so your action closure only triggers after a specific number of seconds have passed. For example, this will trigger only after two seconds:

```swift
Text("Hello, World!")
    .onLongPressGesture(minimumDuration: 2) {
        print("Long pressed!")
    }
```

### 在长按手势过程中添加一个执行闭包

You can even add a second closure that triggers whenever the state of the gesture has changed. This will be given a single Boolean parameter **as** input, and it will work like this:

1. As soon **as** you press down the change closure will be called with its parameter set to **true**.

2. If you release before the gesture has been recognized (so, **if** you release after 1 second when using a 2-second recognizer), the change closure will be called with its parameter set to **false**.

3. If you hold down **for** the full length of the recognizer, then the change closure will be called with its parameter set to **false** (because the gesture **is** no longer **in** flight), and your completion closure will be called too.

Use code like this to **try** it out **for** yourself:

```swift
var body: some View {
        Text("Hello, world!")
            .padding()
            .onLongPressGesture(minimumDuration: 1,
                                perform: {print("is pressed")},
                                onPressingChanged: {inProgress in
                print("In progress: \(inProgress)!")
            } // 按压的过程中调用此闭包
        )
    }
```

# 复杂手势

For more advanced gestures you should use the `gesture()` modifier with one of the gesture structs: `DragGesture`, `LongPressGesture`, `MagnificationGesture`, `RotationGesture`, and `TapGesture`. These all have special modifiers, usually `onEnded()` and often `onChanged()` too, and you can use them to take action when the gestures are **in**-flight (**for** onChanged()) or completed (**for** onEnded()).

## 缩放手势

As an example, we could attach a magnification gesture to a view so that pinching **in** and out scales the view up and down. This can be done by creating two @State properties to store the scale amount, using that inside a scaleEffect() modifier, then setting those values **in** the gesture, like this:

```swift
struct ContentView: View {
    @State private var currentAmount: CGFloat = 0
    @State private var finalAmount: CGFloat = 1

    var body: some View {
        Text("Hello, World!")
            .scaleEffect(finalAmount + currentAmount)
            .gesture(
                MagnificationGesture()
                    .onChanged { amount in
                        self.currentAmount = amount - 1
                    }
                    .onEnded { amount in
                        self.finalAmount += self.currentAmount
                        self.currentAmount = 0
                    }
            )
    }
}
```

## 旋转手势

Exactly the same approach can be taken **for** rotating views using RotationGesture, except now we’re using the `rotationEffect()` modifier:

```swift
struct ContentView: View {
    @State private var currentAmount: Angle = .degrees(0)
    @State private var finalAmount: Angle = .degrees(0)
    
    var body: some View {
       
        Text("Hello, world!")
            .padding()
            .rotationEffect(currentAmount + finalAmount)
            .gesture(RotationGesture()
                        .onChanged{angle in
                currentAmount = angle
            }
                        .onEnded{angle in
                finalAmount += currentAmount
                currentAmount = .degrees(0)
            })
    }
}
```

# 复合手势

## 多层级手势判定原则

Where things start to get more interesting **is** when gestures clash – when you have two or more gestures that might be recognized at the same time, such **as** **if** you have one gesture attached to a view and the same gesture attached to its parent.

For example, this attaches an on onTapGesture() to a text view and its parent:

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello, World!")
                .onTapGesture { // 子View手势
                    print("Text tapped")
                }
        }
        .onTapGesture { // 父View手势
            print("VStack tapped")
        }
    }
}
```

> **In this situation SwiftUI will always give the child’s gesture priority,** which means when you tap the text view above you’ll see “Text tapped”.
> 

## 优先级手势设置

However, **if** you want to change that you can use the `highPriorityGesture()` modifier to force the parent’s gesture to trigger instead, like this:

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello, World!")
                .onTapGesture {
                    print("Text tapped")
                }
        }
        .highPriorityGesture(
            TapGesture()
                .onEnded { _ in
                    print("VStack tapped")
                }
        )
    }
}
```

## 同时响应手势设置

Alternatively, you can use the `simultaneousGesture()` modifier to tell SwiftUI you want both the parent and child gestures to trigger at the same time, like this:

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello, World!")
                .onTapGesture {
                    print("Text tapped")
                }
        }
        .simultaneousGesture(
            TapGesture()
                .onEnded { _ in
                    print("VStack tapped")
                }
        )
    }
}
//That will print both “Text tapped” and “VStack tapped”.
```

## 顺次手势

Finally, SwiftUI lets us create gesture sequences, **where** one gesture will only become active **if** another gesture has first succeeded. This takes a little more thinking because the gestures need to be able to reference each other, so you can’t just attach them directly to a view.

Here’s an example that shows gesture sequencing, **where** you can drag a circle around but only **if** you long press on it first:

```swift
struct ContentView: View {
    // how far the circle has been dragged
    @State private var offset = CGSize.zero

    // whether it is currently being dragged or not
    @State private var isDragging = false

    var body: some View {
        // a drag gesture that updates offset and isDragging as it moves around
        let dragGesture = DragGesture()
            .onChanged { value in self.offset = value.translation }
            .onEnded { _ in
                withAnimation {
                    self.offset = .zero
                    self.isDragging = false
                }
            }

        // a long press gesture that enables isDragging
        let pressGesture = LongPressGesture()
            .onEnded { value in
                withAnimation {
                    self.isDragging = true
                }
            }

        // a combined gesture that forces the user to long press then drag
        let combined = pressGesture.sequenced(before: dragGesture)

        // a 64x64 circle that scales up when it's dragged, sets its offset to whatever we had back from the drag gesture, and uses our combined gesture
        return Circle()
            .fill(Color.red)
            .frame(width: 64, height: 64)
            .scaleEffect(isDragging ? 1.5 : 1)
            .offset(offset)
            .gesture(combined)
    }
}
```

Gestures are a really great way to make fluid, interesting user interfaces, but make sure you show users how they work otherwise they can just be confusing!

### 官方文档长按范例

**Overview**

To recognize a long-press gesture on a view, create and configure the gesture, then add it to the view using the gesture(_:including:) modifier.

Add a long-press gesture to a Circle to animate its color from blue to red, and then change it to green when the gesture ends:

```swift
struct LongPressGestureView: View {
    @GestureState var isDetectingLongPress = false
    @State var completedLongPress = false

    var longPress: some Gesture {
        LongPressGesture(minimumDuration: 3)
            .updating($isDetectingLongPress) { currentstate, gestureState,
                    transaction in
                gestureState = currentstate
                transaction.animation = Animation.easeIn(duration: 2.0)
            }
            .onEnded { finished in
                self.completedLongPress = finished
            }
    }

    var body: some View {
        Circle()
            .fill(self.isDetectingLongPress ?
                Color.red :
                (self.completedLongPress ? Color.green : Color.blue))
            .frame(width: 100, height: 100, alignment: .center)
            .gesture(longPress)
    }
}
```