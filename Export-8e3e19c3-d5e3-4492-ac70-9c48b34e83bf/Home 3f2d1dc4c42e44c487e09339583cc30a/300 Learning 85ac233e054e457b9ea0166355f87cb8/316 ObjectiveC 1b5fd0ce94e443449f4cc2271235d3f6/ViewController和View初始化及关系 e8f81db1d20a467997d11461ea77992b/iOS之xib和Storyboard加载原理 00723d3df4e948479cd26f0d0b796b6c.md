# iOS之xib和Storyboard加载原理

为了说明加载的过程，我们通过解析UIViewController实例化到呈现给我们能看到视图这中间的过程

UIViewController也可以没有xib和StoryBoard进行实例化，这里我们分析的是带有xib或StoryBoard的情况

## 生成UIViewController

不管xib还是Storyboard苹果都给给我们提供了上层api用于我们视图和控制器的实例化，分别对应的类是UINib和UIStoryboard。
我们使用的是最上层的api。

- **我们带有xib的UIViewController实例化的api如下**

`[[ViewController alloc] init];
[[ViewController alloc] initWithNibName: bundle:];`

- **带有Storyboard的实例化api如下：**

`[storyborad instantiateViewControllerWithIdentifier:]`

其实接下来苹果会调用UIViewController的下列方法

```objectivec
(instancetype)initWithCoder:(NSCoder *)aDecoder{
	return [super initWithCoder:aDecoder];
}
```

打断点可以看到aDecoder这个不为空，这个对象就是UINibDecoder的实例，这个类继承于NSCoder。通过这个类我们可以应该猜出UINibDecoder给我们做了什么。

**其实说白了就是帮我们反序列化了。**上边的方法其实仅仅帮我们反序列化了UIViewController。这个时候我们拉线的视图都是空。

之后会调用`-(void)awakeFromNib`这个方法告我们UIViewController已经成功从xib或Storyboard加载。如果不是通过xib或Storyboard加载的话，这个方法是不会调用的。

## 加载View

通过调用下面方法

```objectivec
(void)loadView{
	[super loadView];
}
```

当我们调用super方法的时候，这个时候其实开始真正加载我们xib上的所有视图，并且会调用每个视图的`-(instancetype)initWithCoder:(NSCoder *)aDecoder`方法,这个时候我们通过xib拉线的视图才真正可以使用了;

如果没有xib，super调用的时候会自动帮我们创建一个UIView,并且设置给self.view

原文链接：[https://blog.csdn.net/qq_42792413/article/details/86103621](https://blog.csdn.net/qq_42792413/article/details/86103621)