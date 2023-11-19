# 利用代码生成image

```swift
var image: UIImage? {
    get {
        guard let context = UIGraphicsGetCurrentContext() else {
            return nil
        }
        let rect = CGRect(x: 0, y: 0, width: 1, height: 1)
        UIGraphicsBeginImageContext(rect.size)
        context.setFillColor(cgColor)
        context.fill(rect)
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return image
    }
}

let im = UIImage(named:"pic") // uses pic@2x.png
if let path = Bundle.main.path(forResource: "pic", ofType: "png") {
		let im2 = UIImage(contentsOfFile:path) // uses pic@2x.png
        }
```