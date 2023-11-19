# @PublishedObject思考

这是肘子的一个代码的解读。

其实这段代码的主要目的是实现类似与ObservableObject里面的@Published的功能，但是更进一步的是针对引用性质的实例来实现类似的让外层ObservableObject能发送通知的功能。

因为如果@Published标注的是值类型，其改变能够通过包装器的willSet方法来实现，但是@PublishedObject标注的是引用类型，所以有所不同。这里的要求是@PublishedObject标注的引用类型必须是ObseavalbeObject，这样可以通过将内外层的ObservalbeObject的`objectWillChange.send()`方法联动起来，从而实现内层更改，也会导致外层发送更改通知。具体方法是**外层订阅内层的通知**。

# ObservableObject

首先要了解ObservableObject这个协议有一个特性：

<aside>
💡 By default an `ObservableObject`synthesizes an `objectWillChange`publisher that emits the changed value before any of its `@Published`properties changes.  

缺省下 `ObservableObject` 一定会绑定一个 `objectWillChange` publisher在任何用`@Published` 标注的属性发生变化的时候发送通知

</aside>

所以，这个协议的一个要求就是：

`var objectWillChange: Self.ObjectWillChangePublisher`

# PropertyWrapper对外层容器实例的引用

包装器必须要能够对外层容器实例进行引用，这样才能触发外层`objectWillChange.send()`

所以，在PropertyWrapper提案中定义了一个static subscript方法，通过这个方法可以实现外层在实现包装器方法代码时候调用这个subscript方法，从而将外层容器实例引用传入到包装器内部。

```swift
public static subscript<OuterSelf: Observed>(
      instanceSelf observed: OuterSelf, //外层
      wrapped wrappedKeyPath: ReferenceWritableKeyPath<OuterSelf, Value>,
      storage storageKeyPath: ReferenceWritableKeyPath<OuterSelf, Self>
    ) -> Value {
    get {
      observed[keyPath: storageKeyPath].stored
    }
    set {
      let oldValue = observed[keyPath: storageKeyPath].stored
      if newValue != oldValue {
        observed.broadcastValueWillChange(newValue: newValue)
      }
      observed[keyPath: storageKeyPath].stored = newValue
    }
  }
}
```

在编译器补充的包装器代码里调用这个方法

```swift
public class MyClass: Superclass {
  @Observable public var myVar: Int = 17
  
  // desugars to... 这里添加了包装器的真正实现代码，调用了基于keypath的subscript方法
  private var _myVar: Observable<Int> = Observable(wrappedValue: 17)
  public var myVar: Int {
    get { Observable<Int>[instanceSelf: self, wrapped: \MyClass.myVar, storage: \MyClass._myVar] }
    set { Observable<Int>[instanceSelf: self, wrapped: \MyClass.myVar, storage: \MyClass._myVar] = newValue }
  }
}
```

# 完整范例

有了以上铺垫，就可以看看完整范例

```swift
//总体思路：利用ObservalbeObject的协议一定会有一个属性 var objectWillChange: ObservableObjectPublisher
//可以将包含 @publishedObjecet 修饰属性的物体称之为实例（父），而其内部通过 @publishedObjecet 修饰属性称之为属性（子）
//通过 @propertyWrapper 修饰属性（子）内部的storage本身也是一个observableObject
@propertyWrapper
public struct PublishedObject<Value: ObservableObject> {
    
    //该方法是PropertyWrapper提案中实现的用于编译器在补充完整包装器代码中，实例（父）来调用，实现对于这个属性（子）的存取
    // 对该属性的存取，实际上是利用keypath来存取，所以重写subscript方法，来实现自定义的get/set方法，加入功能代码
    public static subscript<EnclosingSelf: ObservableObject>(
        _enclosingInstance observed: EnclosingSelf, //是包含这个 @Published修饰属性（子）的 实例（父）
        wrapped wrappedKeyPath: ReferenceWritableKeyPath<EnclosingSelf, Value>,
        storage storageKeyPath: ReferenceWritableKeyPath<EnclosingSelf, PublishedObject>
    ) -> Value where EnclosingSelf.ObjectWillChangePublisher == ObservableObjectPublisher {
        get {
            
            //建立起实例（父）objectWillChange的send方法和属性（子）notifier.send方法的绑定
            observed[keyPath: storageKeyPath].setChanger(observer: observed)
            return observed[keyPath: storageKeyPath].wrappedValue
        }
        set {
            observed[keyPath: storageKeyPath].wrappedValue = newValue
        }
    }

    public var wrappedValue: Value{
        willSet{
            
            // 更改一层的属性，直接发送更改通知
            notifier.send?()
            
            //建立对对属性（子）进行的订阅，属性（子）里面storage所引用的值发生改变的时候，就会调用notifier.send?()
            //而notifier.send?() 其实就是调用 实例（父）的objectwillChang的send方法
            cancelled = newValue.objectWillChange.sink(receiveValue: { [notifier] _ in
                DispatchQueue.main.async {
                    notifier.send?()
                }
            })
        }
    }

    public init(wrappedValue: Value) where Value: ObservableObject {
        self.wrappedValue = wrappedValue
        
        //建立初始订阅
        cancelled = wrappedValue.objectWillChange.sink(receiveValue: { [notifier] _ in
            DispatchQueue.main.async {
                notifier.send?()
            }
        })
    }
    
    //目的是保存对实例（父）对引用
    private let notifier = SendHolder()
    private var cancelled: AnyCancellable?

    private class SendHolder {
        var send: (() -> Void)?
        init() {}
    }
    
    //建立起实例（父）objectWillChange的send方法和属性（子）notifier.send方法的绑定
    mutating func setChanger<Observer: ObservableObject>(observer: Observer) where Observer.ObjectWillChangePublisher == ObservableObjectPublisher {
        guard notifier.send == nil else { return }
        notifier.send = { [weak observer] in
            observer?.objectWillChange.send()
        }
    }
}

```

# 总结

其实是一个nested observableObject。利用PropertyWrapper把内外层的联动起来：外层订阅内层的publisher，内层的数值改变后发布objectWillChange.send()，然后外层收到发布消息后也进行外层的objectWillChange.send()给外层的那些订阅者。