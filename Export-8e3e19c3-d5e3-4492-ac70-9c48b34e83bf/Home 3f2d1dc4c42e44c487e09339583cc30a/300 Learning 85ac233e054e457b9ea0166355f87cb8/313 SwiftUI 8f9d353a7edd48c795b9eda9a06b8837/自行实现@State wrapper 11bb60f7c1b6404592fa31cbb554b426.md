# 自行实现@State wrapper

```swift
import Foundation
import SwiftUI

final class Box<T> : ObservableObject {
    var value: T {
        willSet {
            objectWillChange.send()
        }
    }
    
    init(_ value: T) {
        self.value = value
    }
}

@propertyWrapper
struct JyState<V>: DynamicProperty {
    @StateObject private var box: Box<V>
    
    var wrappedValue: V {
        get {
            box.value
        }
        nonmutating  set {
            box.value = newValue
        }
    }
    
    var projectedValue: Binding<V>  {
        Binding(
            get: { wrappedValue},
            set: { wrappedValue = $0 }
        )
    }
    
    init(_ value: V) {
        _box = StateObject(wrappedValue: Box(value))
    }
}
```