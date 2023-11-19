# typedef用于简化block签名

为了隐藏复杂的块类型，可以用到C语言装名为“类型定义”（type definition）的特性

> `typedef`关键字用于给某个类型起个易读的名字， 
这是C语言的特性，不是OC的
> 

例如我们想把某个签名形式相对复杂的block定义成某个名字的类别，以后直接使用这个名字来定义这类block

```objectivec
typedef int(^EOSSomeBlock)(BOOL flag, int value);
```

这样我们就获得来一个名字为`EOSSomeBlock`的类型，代表的是这样一种block签名：接受一个BOOL参数，一个int参数，并且会返回一个int值。我们可以用这个类型来创建新的block了：

```objectivec
EOSSomeBlock newBlock = ^(BOOL flag, int value) {
	//implementation
};
```

这样代码读起来顺畅多了。

另外在一些方法中，需要传入一个block作为参数，如果不用类型定义，那么方法的签名会变得比较复杂，例如：

```objectivec
- (void)startWithCompletionHandler:(void(^)(NSData *Data, NSError *error)completion;
```

我们可以把传入的这个方法参数类型写成一个词，那就顺口多了。所以给这个block参数类型起个别名，后续使用此别名来定义：

```objectivec
typedef void(^EOCCompletionHandler)(NSData *data, NSError *error);
```

上面的方法代码就可以写得更简洁了：

```objectivec
- (void)startWithCompletionHandler:(EOCCompletionHandler)completion;
```