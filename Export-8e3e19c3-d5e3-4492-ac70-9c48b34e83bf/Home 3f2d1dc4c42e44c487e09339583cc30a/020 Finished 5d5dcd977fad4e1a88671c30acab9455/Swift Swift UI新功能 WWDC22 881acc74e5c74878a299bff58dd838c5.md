# Swift / Swift UI新功能 WWDC22

# 软件包

## TOUFU

## Swift软件包插件-仅仅提供开发过程中使用

- 命令插件
    - 为源代码做格式化
    - 文档生成
    - 生成测试报告
- 构建工具插件
    - 扩展构建系统的功能
    - 生成源码文件
    - 资源处理
    
    插件用Swift生成
    

### 模块别名？

# Swift驱动（Swift Driver）

整合的编译器

主动编译

主动链接

运行时提速

- 优化的协议一致性检查

# 并发功能更新

## 向后兼容

Mac OS 10.15

iOS 13

## 模型的扩展

内存安全性：数据竞争的安全性

分布式参与者：distributed actor， 和Actor类似，不仅是不同的线程，而是不同的设备。

分布式参与软件包（Distributed actor package）

## 异步算法软件包：asyncSequence

### 时钟协议

- debuance
- chunk
- 。。。

### 集合初始化

从asyncSequence创建集合类型。

## instrument展示并发工具

# 日渐丰富的表现力

## if let：可选性消除

## 允许指针转换

不会为不同类型的指针做转换

C语言允许指针不匹配

在调用C的函数的时候允许指针转换，让swift和C的api无缝接入

## 通过支持正则表达式来描述？Swift Regex

提供了类似SwiftUI的语言来表示正则表达式 `RegexComponent` 更加容易被理解

# 范型代码规范化：any关键词

---

# SwiftUI

## Swift charts

- chart是和SwiftUI一样的View
- 在chart里可以添加SwiftUI的view
- 自动处理环境因素，例如动态字库，环境色

## 导航和窗口

### navigationStack

- 适应于pop、push等界面切换
- 多个程序视图实现数据驱动的api： navigationDestination
- 导航路径

### navigationSplitView

- 支持两列或多列的视图
- 可以和上面混用

### scenen

- 新增window，这是app的唯一窗口？Mac OS only
- 适应Mac Ap
- 创建菜单栏的视窗功能，可以仅使用它

### Forms

新增的表单样式，具有gruop的特点

## Controls增强

- Textfield：
    - 高度能够自动增加
    - lineLimit
- multiDatePicker
- 多个Toggle可以聚合成一个折叠，链接到一个集合Bindding
- stepper
- accessibilityQuickAction用于apple watch

### table

- 可用于iPad os，iPhone，Mac多个平台
- toolBar可用于iPad OS自定义

### searchable

## 分享

### Photos：photosPicker

全新的照片选择界面

### shareLink API

- watch OS标准的分享界面
- shareLink来分享内容

### Transferable

各类数据如何跨平台传输的协议

- drag&drop
- standard transferable type

## 图形渲染和页面布局

### shape Styles

- color新增一个gradient的属性
- shadow的修饰符 和symbol结合
- 文本动画：粗细，样式和布局

### Layout

- Grid，新的容器视图
    - grid row
- layout协议
- ViewThatFits
- AnyLayout：完成不同布局之间的切换动画

---

# iPad APP Charts和symbol设计特性

## iPad OS

### 工具栏

- 可以自定义布局，和mac OS接近