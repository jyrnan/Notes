# 用 Xcode Configuration 和 Scheme来实现代码条件编译满足测试需求

# 需求现状

在我们进行项目开发并测试的时候，可能会需要在测试的时候对项目代码进行适当的修改以满足测试需求，为了实现这一需求，我们可以将其拆解成两部份来实现：

- 创建和设置测试的配置环境；
- 根据配置环境来进行代码条件编译。

# **创建项目的Test配置环境**

Xcode创建项目支持多个Configuations，默认已经存在Debug和Release两种。可以创建一个新的，例如Test，最好复制原来的“Debug” Configuaration。这样我们项目就可以有一个Test的配置环境，

![Untitled](%E7%94%A8%20Xcode%20Configuration%20%E5%92%8C%20Scheme%E6%9D%A5%E5%AE%9E%E7%8E%B0%E4%BB%A3%E7%A0%81%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91%E6%BB%A1%E8%B6%B3%E6%B5%8B%E8%AF%95%E9%9C%80%E6%B1%82%20af91d2aadefc4884bd73aae6045249e6/Untitled.png)

# 利用Scheme来指定Target的环境配置

## **区分Project、Target、Scheme**

- `Project`：是一个项目的整体，相当于一个仓库，包括了所有的代码和资源文件；
- `Target`：相当于一个具体的产品，包含了对于代码，资源文件的具体使用规则和配置；
- `Scheme`: 对指定`Target`的环境进行配置；

## 指定测试的配置环境

通过编辑Scheme来指定生产Test product时采用的Build Configuration，选择成上一步创建的Test

![Untitled](%E7%94%A8%20Xcode%20Configuration%20%E5%92%8C%20Scheme%E6%9D%A5%E5%AE%9E%E7%8E%B0%E4%BB%A3%E7%A0%81%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91%E6%BB%A1%E8%B6%B3%E6%B5%8B%E8%AF%95%E9%9C%80%E6%B1%82%20af91d2aadefc4884bd73aae6045249e6/Untitled%201.png)

# 设置针对Test的Build Configuration下的预处理宏

这里需要注意的是虽然测试的Target是DemoSchemeTests，但是进行测试时依赖的是编译DemoScheme生成的Product，实际上需要进行条件编译的代码也是属于DemoScheme这个Target里面。所以我们是针对DemoScheme这个Target的“Test” Build Configuration来进行设置。

### 设置Swift compiler - Custom Flags

如果是纯Swift语言，则如图设置Active Compilation Conditions，增加一个“TEST”字段。这个字段的目的是进行条件编译的分支条件。

![Untitled](%E7%94%A8%20Xcode%20Configuration%20%E5%92%8C%20Scheme%E6%9D%A5%E5%AE%9E%E7%8E%B0%E4%BB%A3%E7%A0%81%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91%E6%BB%A1%E8%B6%B3%E6%B5%8B%E8%AF%95%E9%9C%80%E6%B1%82%20af91d2aadefc4884bd73aae6045249e6/Untitled%202.png)

### 如果是OC语言环境

则设置Preprocessor Macros

# 实现条件编译

在C 系语言(例如Objective C)中，我们可以通过预处理宏定义一些参数，使用#if或者#ifdef编译条件分支来控制哪些代码需要编译，而哪些代码不需要。

但是在swift中没有宏定义的概念，虽然不能使用 #ifdef 的方法来检查某个符号是否经过宏定义，但是可以支持“#if/#else/#endif”语句。

这里介绍Swift的方法：

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundColor(.accentColor)
            
						#if TEST
            Text("Hello, Test!")
            #else
            Text("Hello, world!")
            #endif
        }
        .padding()
    }
}
```

上面的代码如果在编译环境中检测到TEST值（上面已经设置），则会选择编译`Text("Hello, Test!")`，否则就会编译`Text("Hello, world!")`

# 总结

可以总结成如下几个步骤

1. 创建用于测试的Build Configuration
2. 编辑Scheme，指定进行测试时将选择的的Build Configuation
3. 在需要进行条件编译的target的Build setting的Swift complie - Custom Flags选项中给Test环境下增加检测字段
4. 代码中利用“#if/#else/#endif”语法检测第3步设置的检测字段来实现不同条件下的代码编译