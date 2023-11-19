# Combine重要组件详解

目录

本文打算通过combine的一个使用范例来解释Combine内部运作关系，分析Combine里的三个重要核心概念：Publisher，Subscriber，Subscription之间的关系。

阅读本文需要对Combine的内容有一定的了解。

### combine的一般应用形式

先看这个范例：

```swift
var cancellables = Set<AnyCancellable>()

UISlider()
    .publisher(event: .valueChanged) 
    .customSink(receiveValue: {control in 
        guard let slider = control as? UISlider else {
            return
        }
        print(slider.value) 
    })
		.store(in: &cancellables)
```

这是一个典型的Combine应用范例：UISlider提供了一个Publisher，每次UISlider的数值改变时候，Publisher就会输出相应的值（例子中是UISlider本身），然后Subscriber(订阅者)会通过订阅这个Publisher接收到输出值，每次Subscriber接受到Publisher的输出值会执行对应的操作，也就是打印这个UISlider的当前值。

我们可以通过这个范例来一层层揭示Combine里的Publisher和Subscriber的关系。

### 生成一个Publisher

```swift
extension UIControl {
    func publisher(event: UIControl.Event) -> UIControl.EventPublisher {
        return EventPubulisher(control: self, controlEvent: event)
    }
}
```

首先看UISlider是如何提供一个Publisher供订阅。

从上面的代码我们可以看到其实是通过扩展了UIControl这个类的一个.publisher方法。从该函数签名我们可以看出该方法返回了一个类型为EventPublisher的Publisher，这个Publisher被定义在UIControl类下。

事实上Combine框架中并不存在这个EventPublisher，或者说UIControl是无法直接提供这样一个Publisher的，这是我们按照Publisher协议[自己定义的Publisher](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)。该Publisher会由UIControl的某个event（例如Slider的value变化）驱动，来产生新的Output给下游的Subscriber，这个输出值是UIControl本身。

我们可以通过下面的代码来了解如何自定义这样一个Publisher，首先我们看看Publisher这个协议的具体内容：

```swift
protocol Publisher {
    associatedtype Output
    associatedtype Failure
    
    func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input
    }
```

我们实现的这个自定义的EventPublisher很好的遵循了Publisher协议。

```swift
extension UIControl {
   
    struct EventPublisher: Publisher {
        
        typealias Output = UIControl // Publisher.Output类型，这里定义的是输出UIControl本身
        typealias Failure = Never // 定义Publisher.Failure类型，这里定义成Never
        
        let control: UIControl // 定义这个Publisher数据的真正来源，必然是一个UIControl
        let controlEvent: UIControl.Event // 定义驱动UIControl产生数据的事件，例如Slider的value的变化 .valueChanged
        
        /// A1.1
        /// 满足Publisher的协议，必须实现该方法
        /// 该方法的目的是创建一个Subscription，请参考 A1.2
        /// 并将该SubsAcription注入到Subscriber， 请参考A1.3
        /// 有意思的是：该方法一般不直接被调用，而是通过调用Publisher.subscribe(:)来调用这个方法 请参考 A2.2
        func receive<S>(subscriber: S) where S : Subscriber, Never == S.Failure, UIControl == S.Input {
            
            /// A1.2
            /// 创建了一个Subscription。
            /// Subscription的自定义 请参考 C
            let subscription = EventSubscription(control: control, event: controlEvent, subscriber: subscriber)
            
            ///A1.3
            ///将创建的Subscription注入到Subscriber， 方法定义请 参考 [B1.1](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)
            subscriber.receive(subscription: subscription)
        }
    }
}
```

Publisher协议里要求的receive<S>(subscriber:S)是最核心的方法，这个方法者有几点特别需要关注：

- 这个方法创建了一个EventSubscription的实例subscription
- 这个方法接受一个Subscriber作为参数，并且在方法中将创建的EventSubscription的实例通过调用Subscriber的receive(subscription:)注入到Subscriber中

所以通过这个方法，会生成一个Subscription，并将Subscriber， Subscription二者者建立了关系。但有意思的是这个方法一般不会直接调用，而是通过调用Publisher.subscribe(:)来调用。

可以看出这个方法的调用的前提是生成了Subscriber，所以我们只能在生成了Subscriber之后才会调用这个方法。

### 生成Subscriber

一般来说Subscriber都是通过调用Publisher的扩展方法来生成。例如下面这个方法：

```swift

/// A2
/// 这里定义了Publisher一个custumSink的方法，其实等于系统自带的sink方法
/// 作用是产生一个用来产生一个Subscriber，请看 A2.1
/// 并将其附着(attach)到publisher上  请参考 A2.2
extension Publisher {
    func customSink(receiveCompletion: @escaping (Subscribers.Completion<Self.Failure>) -> Void,
                    receiveValue: @escaping (Self.Output) -> Void) -> AnyCancellable {
        
        /// A2.1
        ///产生一个自定义的CustomSink的Subscriber，请参考 B1
        let sink = Subscribers.CustomSink(receiveCompletion: receiveCompletion,
                                    receiveValue: receiveValue)
        
        ///  A2.2
        ///  将产生的Subscriber附着到Publisher上
        ///  该方法内部其实是调用Publisher.receive(:)方法， 请参考 A1.1
        self.subscribe(sink)
        
        /// D
        /// 返回一个持有Subscriber的Cancellable物件来避免Subscriber被释放
				/// 关于Cancellale物件的作用不在本文详述
        return AnyCancellable(sink)
    }
}
```

在这个Publisher的方法里我们可以看到生成了一个自定义的Subscribers.CustomSink的实例sink，并且调用了Publisher自身的方法self.subscribe(sink)，这个方法其实是进一步调用前面提到的协议方法receive(subscriber:)方法，并将新生成的sink作为参数传入；在receive(subscriber:)方法里再生成Subscription的实例，并注入Subscriber里面。至此，Publisher，Subscriber，Subscription三者就都生成了并建立了依赖关系。

我们可以看看Subscriber是如何自定义的。

Subscriber有三个重要的方法：

- receive(subscription:) 必须实现的方法，供Publisher调用,设置传入的Subscription，作为Subscriber的持有，该方法被Publisher在生成 Subscriber的后调用。该方法内部会执行Subscription.request()方法，来设置Subscriber的订阅需要
- receive(_:)，该方法由Subscription来调用，一旦调用方法内部就会执行存储在Subscriber的receiveValue(_:)方法，该方法是由Publisher[生成Subscriber](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)时候传入，作用是Publisher的Output是Value时候执行的命令
- receive(completion:) 和上面类型，该方法由Subscription来调用，一旦调用方法内部就会执行存储在Subscriber的receive(completion:)方法，该方法是由Publisher[生成Subscriber](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)时候传入，作用是Publisher的Output是Completion时候执行的命令

```swift
/// B1
///  自定义一个Subscriber
extension Subscribers {
    
    /// 所有的Subscriber都属于Subscribers范围内
    class CustomSink<Input, Failure: Error>: Subscriber {
        
        let receiveValue: (Input) -> Void // 针对接收到的Publisher的输出值作出的处理闭包
        let receiveCompletion: (Subscribers.Completion<Failure>) -> Void // 针对接收到的Publisher的输出的Completion作出的处理闭包
        var subscription: Subscription? // 持有由Publisher创建的Subscription。 创建位置，请参考 A1.2
        
        init(receiveCompletion: @escaping (Subscribers.Completion<Failure>) -> Void,
             receiveValue: @escaping (Input) -> Void) {
            
            self.receiveCompletion = receiveCompletion
            self.receiveValue = receiveValue
        }
        
        /// B1.1
        /// 必须实现的方法，供Publisher调用
        /// 设置传入的Subscription，作为Subscriber的持有
        /// 该方法被Publisher在附着(attach) Subscriber的时候调用，请参考：A1.3
        func receive(subscription: Subscription) {
            
            /// B1.1.1 自身注入Subscription
            self.subscription = subscription
            
            /// B1.1.2
            /// backpress的实际所在。这里会执行来更新backpress的政策
            /// 方法请参考 C1.1
            subscription.request(.unlimited)
        }
        
        /// B1.2
        /// 必须实现的方法，供Subscription调用
        /// 该方法用来在接收到数值的时候执行
        /// 真正调用该方法的位置 请参考： C1.3
        /// - Parameter input: 输入的数值
        /// - Returns: 后续获取获取数据的策略/数量
        func receive(_ input: Input) -> Subscribers.Demand {
            receiveValue(input)
            return .none
        }
        
        /// B1.3
        /// 必须实现的方法，供Subscription调用
        func receive(completion: Subscribers.Completion<Failure>) {
            receiveCompletion(completion)
        }
    }
}
```

如果要让Subscriber满足Cancellable协议，必须实现cancel()方法，这其实是通过调用Subscription的cancel()来实现

```swift
/// B2
/// 实现Subscriber的cancel功能，其实是通过调用Subscription的Cancel()来实现 请参考： B2.1
extension Subscribers.CustomSink: Cancellable {
    func cancel() {
        
        /// B2.1
        /// 通过调用Subscription的Cancel()来实现，请参考C1.2
        subscription?.cancel()
        
	      /// 去除对Subscription的引用.在C1.2中会去除Subscriber对Subscription的引用
        subscription = nil
    }
}
```

### 生成Subscription

下面的范例是一个Subscription的实现。

```swift
/// C1
/// 定义一个Sucscription，创建位置请参考 A.1.2
extension UIControl {
    class EventSubscription<S: Subscriber>: Subscription where S.Input == UIControl, S.Failure == Never {
        
        let control: UIControl
        let event: UIControl.Event
        
        var subscriber: S? // 持有一个Subscriber
        var currentDemand = Subscribers.Demand.none // 初始的Demand
        
        init(control: UIControl, event: UIControl.Event, subscriber: S) {
            self.control = control
            self.event = event
            self.subscriber = subscriber
            
            /// 绑定到Control的事件上实现Publisher的输出
            control.addTarget(self, action: #selector(eventOccured), for: event)
        }
        
        /// C1.1
        /// 来定义Subscriber向Publisher获取数值的数量
        func request(_ demand: Subscribers.Demand) {
            currentDemand += demand
        }
        
        /// C1.2
	      /// 实现Cancel功能，满足Cancellable协议，请参考B2.1
        func cancel() {

						/// 去除对Subscriber的引用
            self.subscriber = nil
            
            /// 移除和UIControll事件绑定的Publisher输出功能
            control.removeTarget(self, action: #selector(eventOccured), for: event)
        }
        
        @objc
        func eventOccured() {
            if currentDemand > 0 {
                
                /// C1.3
                /// 真正执行数据操作的位置，
								/// 调用Subscriber.receive()方法，请参考B1.2
								/// 通过返回值同步更新currentDemand
                currentDemand += subscriber?.receive(control) ?? .none
                currentDemand -= 1
            }
        }
    }
}
```

### 小结

到这里我们可以小结一下三者的产生顺序关系：

- UIControl调用自身方法publisher(event:) [生成   Publishers.EventPublisher](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)
- EventPublisher调用自身扩展方法[customSink生成](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md) SubScribers.[CustomSink](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md) 的实例sink，并将Subscriber所需要的receive(_:)和receive(completion:)传入
- customSink方法在生成sink后，紧接者调用Publisher自身的方法subscriber(subscriber:)，该方法实际调用了[Publisher自身的方法receive(subscriber:)](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)
- receive(subscriber:)方法生成[Subscription](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)，并立刻调用[Subscriber的receive(subscription:)](Combine%E9%87%8D%E8%A6%81%E7%BB%84%E4%BB%B6%E8%AF%A6%E8%A7%A3%20a2d6383ec4d44a7b9f3175eeb5e597f0.md)方法将Subscription传入Subscriber，建立依赖关系。至此三者生成并建立关系

三者的作用我们可以小结如下：

- Publisher其实主要是用来传递参数来生成Subscriber和Subscription，Publisher生成Subscriber时候，将Subscriber所需要的receive(_:)和receive(completion:)传入
- Subscriber和Subscription互相持有，所以将Subscriber符合Cancellable协议后，保存在Set<AnyCancellable>中就可以一直持有二者。Subscriber调用Subscription里的cancel方法来解除二者互相持有
- **Subscription其实是核心，存储Subscriber的调用Demand。并实现Publisher.Output产生机制，在依据产生机制需要产生Output时再按照Demand状态调用Subscriber里的receive(_:)或receive(completion:)**