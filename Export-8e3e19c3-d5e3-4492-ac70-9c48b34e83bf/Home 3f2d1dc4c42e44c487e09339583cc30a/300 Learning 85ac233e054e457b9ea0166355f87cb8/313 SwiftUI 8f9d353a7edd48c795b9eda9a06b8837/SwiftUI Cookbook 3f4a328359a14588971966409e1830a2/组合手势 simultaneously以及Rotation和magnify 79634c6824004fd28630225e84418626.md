# 组合手势.simultaneously以及Rotation和magnify

<aside>
💡 这三个放在一起是因为相对简单。
手势实施方法基本原理一致
.simultaneously也相对简单，就是讲两个手势结合在一起，共同起作用

</aside>

```swift
struct RotationGestureExample: View {
    @State private var currentRotation = Angle.zero
    @GestureState private var twistAngle = Angle.zero
    @State private var currentMagnification: CGFloat = 1
    @GestureState private var pinchMagnification: CGFloat = 1
    var body: some View {
        Rectangle()
            .fill(Color.red)
            .shadow(radius: 20)
            .frame(width: 200, height: 200)
            .scaleEffect(currentMagnification * pinchMagnification)
            .rotationEffect(currentRotation + twistAngle)
            .gesture(RotationGesture()
                .updating($twistAngle, body: { (value, state, _) in
                    state = value
                })
                .onEnded{ self.currentRotation += $0 }
                .simultaneously(with: MagnificationGesture()
                    .updating($pinchMagnification, body: { (value, state, _) in
                        state = value
                    })
                    .onEnded{ self.currentMagnification *= $0 }
                )
        )
    }
}
```