# 利用MainActor执行方法的方式

# 有三种方法

```swift
@MainActor
func foo() {}
```

```swift
Task{ @MainActor in
	foo()
}
```

```swift
Task {
	await MainActor.run {
		foo()
	}
}
```

# 从DispatchQueue到Task写法的变化

这是官方的推荐方法

```swift
DispatchQueue.main.async {}
转变成：
Task.detached { @MainActor in }
```