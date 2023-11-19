# 使用续体改写函数

# 基本方法

对于基于回调的异步操作，一般性的转换原则就是

- 将回调去掉
- 添加async修饰
- 如果回调接受Error？标识错误，则添加throws
- 回调参数作为异步函数返回值

# 简要

## 不含返回值的方法

```swift
func calculate(input: Int, completion: @escaping (Int) -> Void)

转换成：

func calculate(input: Int) async -> In

```

### 含有返回值的方式

如果原来的闭包回调函数的返回值只是一个简单的基本类型的话，也许直接通过 inout 参数，在暂停点之前获取这个值，也是一种可选方案。

```swift

func syncFunc(completion: @escaping (Int) -> Void) -> Bool {
	someAsyncMethod {
		completion(1)
	}
	return true
}
转换成：
func asyncFunc(started: inout Bool) async -> Int {
	started = true
	await someAsyncMethod()
	return 1
	}
```

# 使用续体改写函数

<aside>
💡 可以理解成利用一个续体来包住原函数，原来函数需要执行闭包的地方执行续体.resume。也可以理解成**原来方法是在一定时间后（异步）会通过参数（可能没有）来执行闭包**，现在换成利用续体Hold住当前状态，等原方法跑**到要通过参数（可能没有）来执行闭包的时刻**，直接把那个参数（可能是Void）作为**返回值**传回来，
这时候拿到这个返回值（可能Void）就能继续从容的往下执行原来通过闭包形式传递的操作，也就是说这个时候才需要写那些拿到返回值后执行的代码。

</aside>

比如对于下面的闭包回调函数：

```swift
func load(completion: @escaping ([String]?, Error?) -> Void)
```

典型情况下，利用 withUnsafeThrowingContinuation 进行包装：

```swift
func load() async throws -> [String] {
	try await withUnsafeThrowingContinuation { continuation in
		load { values, error in
			if let error = error {
				continuation.resume(throwing: error)
			} else if let values = values {
				continuation.resume(returning: values)
			} else {
				assertFailure("Both parameters are nil")
			}
		}
	}
}
```

## 实际范例

把人脸识别的一个闭包方法转换成async模式

```swift
extension UIImage {
    func detectFaces(completion: @escaping ([VNFaceObservation]?) -> ()) {
        guard let image = self.cgImage else {
            return completion(nil)
        }
        let request = VNDetectFaceRectanglesRequest()
        
        DispatchQueue.global().async {
            let handler = VNImageRequestHandler(cgImage: image,orientation: self.cgImageOrientation
                                                )
            
            try? handler.perform([request])
            
            guard let observations = request.results else {return completion(nil)}
            
            completion(observations)
            
        }
        
    }
  
  /// 修改成Async模式
  /// 这个方法不但把上面的方法改成了async形式
  /// 还直接继续调用了原来需要作为闭包传入的操作
  /// 所以相当于是把上面方法和调用它的方法，两个方法合并起来了。
  /// - Returns: <#description#>
  func detectFaces() async -> UIImage {
    ///本来是回调提供参数([VNFaceObservation]?的方法
    ///改成了用续体异步获得([VNFaceObservation]?
    let result = await withUnsafeContinuation {contiuation in
      self.detectFaces(completion: {
        contiuation.resume(with: .success($0))
      })
    }
    
    ///在上面拿到了([VNFaceObservation]）这个结果后，就可以进行后续的操作
    ///并返回操作后的值
    let croppedImage = result?.cropByFace(self)
    return croppedImage ?? self
  }
}
```