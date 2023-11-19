# 取消NavigationView的返回按钮并附带手势

1 设置返回环境变量

```swift
@Environment(\.presentationMode) var presentationMode: Binding<PresentationMode>
```

2 设置返回按钮

```swift
var btnBack : some View { Button(action: {
                self.presentationMode.wrappedValue.dismiss()
                }) {
                    HStack {
                    Image("Logo") // set image here
                        .resizable()
                        .aspectRatio(contentMode: .fit)
                        .foregroundColor(.white)
                        .frame(width: 24, height: 24, alignment: .center)
                    }
                }
```

3 设置返回按钮和取消原有返回按钮

```swift
.navigationBarBackButtonHidden(true)
        .navigationBarItems(leading: btnBack）
```

4 设置手势变量

```swift
@GestureState private var dragOffset = CGSize.zero
```

5 设置手势modifier

```swift
.gesture(DragGesture().updating($dragOffset, body: {(value, state, transition) in
            
            if (value.startLocation.x < 20 && value.translation.width > 100) {
                self.presentationMode.wrappedValue.dismiss()
            }
        }))
```