# 统一类型标识(Uniform Type Identifiers) - 知乎

[https://zhuanlan.zhihu.com/p/311198989](https://zhuanlan.zhihu.com/p/311198989)

![%E7%BB%9F%E4%B8%80%E7%B1%BB%E5%9E%8B%E6%A0%87%E8%AF%86(Uniform%20Type%20Identifiers)%20-%20%E7%9F%A5%E4%B9%8E%20467e7d13b46b496fbe01f62fe3a4dee8/v2-a9468a97c9b8398b1b6746c5427d4720_1440w.jpg](%E7%BB%9F%E4%B8%80%E7%B1%BB%E5%9E%8B%E6%A0%87%E8%AF%86(Uniform%20Type%20Identifiers)%20-%20%E7%9F%A5%E4%B9%8E%20467e7d13b46b496fbe01f62fe3a4dee8/v2-a9468a97c9b8398b1b6746c5427d4720_1440w.jpg)

## 简介

> 希望这篇文章能让你对统一类型标识有一个基本的概念，并且了解以下3点:
> 
1. 在iOS、MacOS下系统是如何判断一个文件是什么类型的
2. 如果你创建了一种新的文件类型,如何让系统识别和处理
3. iOS 14.0 下的新框架`UniformTypeIdentifiers`
- 思考一下，分享文件的时候如何让你的应用出现在下面的红框中

![%E7%BB%9F%E4%B8%80%E7%B1%BB%E5%9E%8B%E6%A0%87%E8%AF%86(Uniform%20Type%20Identifiers)%20-%20%E7%9F%A5%E4%B9%8E%20467e7d13b46b496fbe01f62fe3a4dee8/v2-e7ad85fe89b107ff8bb33bcf7d7c2ca8_720w.webp](%E7%BB%9F%E4%B8%80%E7%B1%BB%E5%9E%8B%E6%A0%87%E8%AF%86(Uniform%20Type%20Identifiers)%20-%20%E7%9F%A5%E4%B9%8E%20467e7d13b46b496fbe01f62fe3a4dee8/v2-e7ad85fe89b107ff8bb33bcf7d7c2ca8_720w.webp)

## 正文

**1. 文件扩展名(Path Extension) 和 媒体类型(MIME)**

思考这么一个问题，假设你用iPad拍摄了一张照片并存储在硬盘上,为什么打开它的时候系统会把它作为图片而不是音频文件进行处理

> 操系统在大部分情况通过文件扩展名(Path Extension)来判断文件的类型
> 

比如我们常见的 `.jpg` `.jpeg` `.jpe` 都表示了文件是JPEG图像。 在此例中, iPad拍摄的图片就是一张JPEG图像,并且以`.jpeg`扩展名存储,所以系统能正确的识别它为一张图片。

> 但是在浏览器或者服务器上,通常使用
> 
> 
> [MIME](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)
> 

比如下面5种形式,都代表了同一种文件格式(左边是文件扩展名、右便是MIME type)

**2. 统一类型标识(Uniform Type Identifiers)**

在Apple的平台上, 系统使用`统一类型标识`来规范的确定文件的格式。 `统一类型标识`可以简单的理解为唯一的单个字符串

比如上述的JPEG图像, 它的统一类型标识就是 `public.jpeg`， 不论他们存储在本地还是在web上。

> 此外,统一类型标识有着类似于swift或则objective-c中协议的关系
> 

比如 `public.jpeg`就可以理解为遵循了`public.image`协议,`public.image` 遵循 `public.data` 。 可以参考下图

这些类型标识符帮助系统确定文件的类型，以及帮助系统决策如何处理这些文件。

**3. 创建你自己的统一类型标识**

> 几个规则
> 
1. 统一类型标识不区分大小写。
2. 命名规则为reverse-DNS，例如 `com.example.imagetemplate`
3. `public.*` `dyn.*` `com.example.*` `com.apple.*` 这些是系统保留的命名空间,你不应该使用它们作为类型标识符

> 创建分为两步
> 
1. 类型声明（仅仅告知系统此类型存在，并不意味着你的App能够处理这种类型）
2. 类型支持（告知系统App能够处理某种类型，不管是谁声明了类型,App都能够处理）

**4. 实战**

我们新建一个餐厅App，并且创建一个新的文件类型(基于json)`restaurantmenu`, 我们希望App能够处理我们自己定义的文件类型

> 新建一个xcode工程, 选择TARGETS, info
> 

> 这里我们可以选择导入或者导出类型。 如果类型是由
> 
> 
> ```
> 其他App
> ```
> 
> ```
> 导入
> ```
> 
> ```
> 导出
> ```
> 

新类型是由App自己创建的,所以选择`Exported Type Identifiers`

现在我们完成了类型的声明(第一步), 我们要对新类型进行支持(第二步)。选择`Document Types` ，支持类型

> 上述操作已经完成了类型的声明及支持
> 

声明完成后同样在`Document Types`中添加类型支持

> 类型创建完成后，我们需要做的就是编写代码处理文件, 示例中我们简单的读取
> 
> 
> ```
> food.restaurantmenu
> ```
> 

**5. 完结**

希望你能通过此篇文章比较系统的了解到Uniform Type Identifiers,如何创建和处理新的文件类型。

此文内容出自WWDC Sessions [Uniform Type Identifiers — a reintroduction](https://link.zhihu.com/?target=https%3A//developer.apple.com/videos/play/tech-talks/10696)

示例代码链接:

> 最后,把App安装到设备上,你可以看到我们的应用可以对
> 
> 
> ```
> restaurantmenu
> ```
>