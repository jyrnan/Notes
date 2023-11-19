# 如何在SwiftUI自定义触摸不同阶段

在易投屏中使用，但最终用了其他方式，不过如何利用UIKit桥接到SwiftUI来实现触摸各个阶段的自定义行为可以参考如下代码

```swift
struct TapView: UIViewRepresentable {
    var tappedBeginCallback: (() -> Void)
    var tappedEndCallback: (() -> Void)
  

    func makeUIView(context: UIViewRepresentableContext<TapView>) -> TapView.UIViewType {
        let v = UIView(frame: .zero)
        let gesture = SingleTouchDownGestureRecognizer(target: context.coordinator,
                                                       action: #selector(Coordinator.tapped))
        v.addGestureRecognizer(gesture)
        return v
    }

    class Coordinator: NSObject {
        var tappedBeginCallback: (() -> Void)
        var tappedEndCallback: (() -> Void)

        init(tappedBeginCallback: @escaping (() -> Void), tappedEndCallback: @escaping (() -> Void)) {
            self.tappedBeginCallback = tappedBeginCallback
          self.tappedEndCallback = tappedEndCallback
        }

        @objc func tapped(gesture:UITapGestureRecognizer) {
          switch gesture.state {
          case .began:
            tappedBeginCallback()
          case .ended:
            tappedEndCallback()
          default:
            return
          }
        }
    }

    func makeCoordinator() -> TapView.Coordinator {
      return Coordinator(tappedBeginCallback: self.tappedBeginCallback, tappedEndCallback: self.tappedEndCallback)
    }

    func updateUIView(_ uiView: UIView,
                      context: UIViewRepresentableContext<TapView>) {
    }
}

class SingleTouchDownGestureRecognizer: UIGestureRecognizer {
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent) {
        if self.state == .possible {
            self.state = .began
        }
    }
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent) {
        self.state = .cancelled
    }
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent) {
        self.state = .ended
    }
}
```