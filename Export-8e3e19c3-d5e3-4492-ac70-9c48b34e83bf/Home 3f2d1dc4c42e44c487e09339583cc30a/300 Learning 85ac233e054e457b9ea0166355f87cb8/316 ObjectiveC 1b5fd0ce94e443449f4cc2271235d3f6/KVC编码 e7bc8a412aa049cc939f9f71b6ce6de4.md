# KVC编码

# KVC原理

KVC(Key-Value Coding)键值编码是通过一系列定义在NSObject中的方法实现的，使用这些方法可以通过属性的名称存取属性的值，以下是其中的两种方法：

```objectivec
- (id)valueForKey:(NSString *)k;
- (void)setValue:(id)v forKey:(NSString *)k;
```

例如NIB文件被载入后，其中的IBOutlet变量就是通过KVC来设置的。

<aside>
💡 注意：KVC是基于实例变量的存储方法。也就是说KVC其实是调用实例变量的存储方法，不要弄反。

</aside>

例如，selectedObj对象有一个fido属性，那么可以使用如下代码获取或设置fido属性：

```objectivec
id currentFido = [selectedObj valueForKey:@"fido"];
[selectedObject setValue:userChoice forKey:@"fido"];
```

- 如果selectedObj中定义了fido属性的存取方法，那么valueForKey:和setValue:forKey:就会调用相应的存取方法；
- 如果没有存取方法，则会查找名为_fido或fido的实例变量并直接设置或返回实例变量的值；
- 如果既没有相应的存取方法，也没有_fido或fido实例变量，就会抛出异常。”

# 补充实例

### 常见错误提示

NIB文件在加载时会使用`setValue:forKey:`设置IBOutlet变量。例如，如果需要为某个对象设置一个名为rex的插座变量，那么该对象必须定义了`setRex:`方法或者具有_rex实例变量（或rex实例变量），否则在加载NIB文件时会抛出异常，类似以下提示：
　　`[<BNRSunsetViewController 0x68c0740> setValue:forUndefinedKey:]:this class is not key value coding-compliant for the key rex.'`
　　如果读者看到了这类异常提示，则很有可能是因为在Interface Builder中关联好插座变量之后又更改了该插座变量的名称。

### 错误命名

由此本节传达出一条最重要的编程法则：请务必遵守存取方法命名规范。其目的不仅仅是方便其他开发者阅读代码，系统也具有一套依赖于命名规范的工作机制，如果不遵守命名规范，很有可能会发生意外错误。
　　下面来看一个例子。某个视图控制器定义了clock插座变量，指向一个表示时钟的视图；同时，它还作为一个按钮的目标，为其定义了动作方法setClock:。该方法用于获取网络最新时间并更新时钟视图，方法声明如下：
　　`- (IBAction)setClock:(id)sender;`
　　这样就会产生一个奇怪的问题：当NIB文件被加载时，该方法会立即执行，同时也无法正确设置clock插座变量。`**原因是系统会使用setClock:动作方法设置clock**`——系统会将setClock:视为clock的存方法。因此需要重新命名该动作方法，例如updateClock:。
　　如果不遵守命名规范，就可能在这类奇怪的问题上浪费大量时间。

<aside>
💡 时刻记住实例属性的默认存取设置方法

</aside>

```objectivec
get： - (id) valueName;

set： - (void)setValueName;
```