# SwiftUI和UIView的整合三种类型

Key points

**To host a SwiftUI view in a UIKit project:**

1. Add the SwiftUI view file to your project. 创建一个SwiftUI页面
2. Add a hosting controller to your storyboard, and create a segue to it. 创建一个hostingController，并在IB中创建一个segue指向这个hostingController
3. Connect the segue to an @IBSegueAction in your view controller code.关键： 将segue在Viewcontroller中创建一个@IBSegueAction
4. Set the hosting controller’s rootView to an instance of your SwiftUI view.在创建的@IBSegueAction 加入代码，来指定hostViewcontroller的rootView是一个需要展现的SwiftUI view的实例

**To host a view controller in a SwiftUI project:**

1. Add the view controller and storyboard files to the SwiftUI project.
2. In the storyboard’s identity inspector, set the Storyboard ID for the view controller.
3. Create a representation struct for the view controller, and implement the makeUIViewController(context:) method to instantiate the view controller from Main.storyboard.
4. Add a NavigationLink to ContentView, with the view controller representation as the link’s destination.

**To host a UIKit view with data dependencies:**

1. Create a SwiftUI view that conforms to UIViewRepresentable.
2. Implement the make method to instantiate the UIKit view.
3. Implement the update method to update the UIKit view from the SwiftUI view.
4. Create a Coordinator, and implement its valueChanged method to update the SwiftUI view from the UIKit view.

**下附第三种类型的详细范例和解释**

```swift
import SwiftUI
import UIKit
import Combine

//该范例实现了一个在SwiftUI的View里面添加一个UIView（UISlider），并实现了双向的数据同步。
struct ColorSlider: UIViewRepresentable {
    var color: UIColor
    @Binding var value: Double
    //0、实现UIViewRepresentable第一步需要指定UIView的具体类型，这个范例是UISlider
    typealias UIViewType = UISlider //用来明确UIView的具体类型。
    
    //4、为了实现从UIView到SwiftUI的数据同步，在3、实现了Coordinator定义后，该函数用来生成一个Coordinator的实例，实例初始化的就是把实例的变量和SwiftUI里面的BindingObject绑定起来，具体数量可以根据需要在Coordinator里面进行定义。实例生成后，实例的变量其实就是指向了希望联动的各个SwiftUI里面的BindingObject，例如范例中的Coordinator.value就是指向了ColorSlider.value（4.2），指向完成后将来就可以在UIView里面调用3.2定义的valueChanged方法实现对ColorSlider.value的修改，从而实现了从UIView到SwiftUI方向的数据同步。
    func makeCoordinator() -> ColorSlider.Coordinator {
        Coordinator(value: $value) //4.2 实现Coordinator.value指向了ColorSlider.value，见3.1
    }
    
    //1、实现UIVIewRepresentable协议的两个必须方法之一，这是定义一个UIView的生成方法，UIView的类型在typealias UIViewType = UISlider来明确。
    func makeUIView(context: Context) -> UISlider {
        let slider = UISlider(frame: .zero) //1.1 初始化该类型的实例，并在一些设置语句后将该实例返回
        slider.thumbTintColor = color
        slider.value = Float(value)
        slider.addTarget(context.coordinator, action: #selector(Coordinator.valueChanged(_:)), for: .valueChanged)
        return slider //1.2 返回实例。实现该方法的返回值
    }
    
    //2、实现UIVIewRepresentable协议的两个必须方法之二，该方法可以实现从SwiftUI里传递参数到UIView的，从而实现SwiftUI到UIViewe方向的数据同步
    func updateUIView(_ uiView: UISlider, context: Context) {
        uiView.value = Float(value)
    }
    
    //3、定义该Viw的一个Coordinator，其作用是为了生成一个实例，用来该实例来实现该UIView的数据与SwitUI视图里的BindingObject绑定，UIV从而为实现从UIView到SwiftUI方向的数据同步实现可能。
    //该类定义一个Value变量，类型是BindingObject（这里是具体的Double类型），初始化函数的作用就是将来该类型的一个实例实现自身的Value变量指向SwiftUI里面的某个BindingObject。 见3.1
    //事实上应该可以根据需要定义更多的变量，用来绑定更多的SwiftUI里面的BindingObject
    class Coordinator: NSObject {
        var value: Binding<Double>
        init(value: Binding<Double>) {
            self.value = value //3.1 通过这个赋值，实现了Coordinator的value变量指向了SwiftUI里面的某个BindingObject，将来通过修改Coordingnator.value，也就是在修改SwiftUI里的BindingObject
        }
        //3.2这个方法可以被UIView调用，从而实现对Coordinator.value，也就是指向的SwiftUI某个BindingObject，进行修改。
        @objc func valueChanged(_ sender: UISlider) {
            self.value.wrappedValue = Double(sender.value)
        }
    }
    
}

struct ColorSlider_Previews: PreviewProvider {
   
    static var previews: some View {
        VStack {
            ColorSlider(color: .red, value: .constant(0.5))
        }
        
    }
}

struct AttributedText: UIViewRepresentable {
    var attributedText: NSAttributedString

    init(_ attributedText: NSAttributedString) {
        self.attributedText = attributedText
    }

    func makeUIView(context: Context) -> UITextView {
        return UITextView()
    }

    func updateUIView(_ label: UITextView, context: Context) {
        label.attributedText = attributedText
    }
}

//usage: AttributedText(NSAttributedString())
```