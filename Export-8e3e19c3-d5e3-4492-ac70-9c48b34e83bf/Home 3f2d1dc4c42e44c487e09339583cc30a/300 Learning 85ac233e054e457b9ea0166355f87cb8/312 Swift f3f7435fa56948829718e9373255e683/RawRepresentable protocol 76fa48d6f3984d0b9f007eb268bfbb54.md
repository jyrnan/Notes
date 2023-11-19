# RawRepresentable protocol

# 协议条件

RawRepresentable protocol 是一种类型可以从自身和RawValue直接自由转换。所以条件如下：

- 从RawValue初始化生成`init?(rawValue: Self.RawValue)`
- 提供一个RawValue值 `var rawValue: Self.RawValue`
- 相关类型 `associatedtype RawValue`

根据协议条件，实现以下代码即可

```swift
class A: RawRepresentable {
    required init?(rawValue: String) {
        <#code#>
    }
    
    var rawValue: String
    
    typealias RawValue = String
    
    
}
```

# 引申思考

- 符合Codable协议的类型，可以转换成Data，Data可以转换成String，所以可以通过转换产生`RawValue = String` 的RawRepresentable类型。例如Array

```swift
extension Array: RawRepresentable where Element: Codable {
    public init?(rawValue: String) {
        self.init()
        guard let data = rawValue.data(using: .utf8),
                let array = try? JSONDecoder().decode([Element].self, from: data) else {return}
        self = array
    }
    
    public var rawValue: String {
        guard let data = try? JSONEncoder().encode(self), let result = String(data: data, encoding: .utf8)  else {
            return "[]"
        }
        return result
    }
}
```