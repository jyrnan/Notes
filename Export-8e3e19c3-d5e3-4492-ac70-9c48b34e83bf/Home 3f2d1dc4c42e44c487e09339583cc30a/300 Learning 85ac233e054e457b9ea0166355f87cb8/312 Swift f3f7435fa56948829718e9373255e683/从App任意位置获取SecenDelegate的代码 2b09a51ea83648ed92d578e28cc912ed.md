# 从App任意位置获取SecenDelegate的代码

```swift
guard let windowsScenen = UIApplication.shared.connectedScenes.first as? UIWindowScene ,
              let sceneDelegate = windowsScenen.delegate as? SceneDelegate
        else {
            return result
        }
        let viewContext = sceneDelegate.context
```