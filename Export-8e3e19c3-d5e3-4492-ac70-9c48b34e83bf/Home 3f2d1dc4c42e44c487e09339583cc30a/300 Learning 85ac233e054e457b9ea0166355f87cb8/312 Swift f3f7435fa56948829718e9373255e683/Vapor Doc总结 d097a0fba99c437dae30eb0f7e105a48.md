# Vapor Doc总结

# Getting Started

## Folder Structure

### public

This folder contains any public files that will be served by your app if `FileMiddleware` is enabled.

`**FileMiddleware` ：应该是提供文件服务的中间件？**

### Sources

**APP**： 

- **Controllers**: grouping together application logic
- **Migration**: database migrations if using Fluent
- **Models**: store `Content` structs or Fluent `Models`
- **configure**.**swift**: configure `Application`
- **routes.swfit:** called by `configure(_:)` to register routes to `Application`

Run:

- main.swfit: create **Application**

## SPM

<aside>
💡  SPM：它的target所包含的文件内容是靠文件夹来区别？参考Vapor的SPM文件结构：App和Run. 确实如此

</aside>

## Xcode

### Working directory

需要指定**工作目录是项目文件（package.swift)所在的目录**，这样才能顺利找到Vapor所依赖的众多资源文件。例如leaf

<aside>
💡 指定方法：在Scheme/edit Schem里面制定working dirctory

![Untitled](Vapor%20Doc%E6%80%BB%E7%BB%93%20d097a0fba99c437dae30eb0f7e105a48/Untitled.png)

</aside>

# 路由

**对应请求给予返回值的方法**

## HTTP方法：

## HTTP请求路径

请求的URI，以 / 开头的路径和？之后的查询字串组成

## 路由方法

所有常见的HTTP方法都可以作为Application的方法使用。

```swift
app.get(){req in)
app.on(){…}
```

## 路由参数

可以使得请求方法的路径动态化。 通过 “：”路径组件提示这是动态组件

使用req.parameters来访问字符串的值

```swift
app.get("hello", ":name"){req in 
let name = req.parameters.get("name)
return ...}
```

## 路径

**路由为给定的HTTP方法和URI路径指定处理程序（request handler)**

### 方法

[路由方法](Vapor%20Doc%E6%80%BB%E7%BB%93%20d097a0fba99c437dae30eb0f7e105a48.md) 参考上面👆

### 路径组件

路由处理请求在识别方法之外，还要处理各种个样的路径。所以需要一些通用组件来表示各种个样的路径。

- 常量(foo)
- 参数路径(:foo) : 变量
- 任何路径(**) ：放在路径中间代表一个任意元素*
- 通配路径(**) 代表任意多个

### 参数

- 通过“：”保存
    - 通过`req.parameters.get`方法来获得
- 通过“**”
    - 👍通过`**`匹配的URI值将以[String]形式保存在`req.parameters`中，通过`req.parameters.getCatchAll`方法访问

### body数据流

使用`on方法`注册路由时候，可以指定request body如何被处理。

```swift
app.on(.POST, "listings", body: collect(maxSize: "1mb")) {req in ...}
```

默认 情况stream body的大小是16kb，可以实现同步解码请求

- 还有流处理的方式

```swift
app.on(.POST, "upload", body: .stream){req in ...}
```

## 路由组

基于路径前缀或特定中间件的一组路由

- **路径前缀**

```swift
let users = app.group("users") // 创建了一个路由组
users.get { req in ..} // GET /users
users.post {req in ...}  // POST /users
users.get(":id" {req in  // GET /users/:id 以id为变量名保存路径参数
	let id  = req.parameters.get("id")!
	... 
}

**路由组的继续嵌套**

app.group("users") {users in
users.get{...} //GET /users
...
users.group(":id") {user in 
	user.get{...} // Get /users/:id
```

- **中间件** 来产生路由组 这个有点难理解☹️

```swift
app.group(RateLimitMiddleware(requestsPerMinuter: 5)) {rateLimited in
	reateLimited.get("slow-thing") {req in 
	...
	}
}
```

## 重定向

```swift
req.rediect(to: "/some/new/path", type: .permanent)
```

# 内容Content

**Request或Response里面的Data部分，涉及三部分，编解码方法都可以定制！**

- request的content
- request路径里面的query
- response的content

## 概述

应该是对于HTTP协议里面的content部分的处理

<aside>
💡  HTTP的content属性包括：
- content type：  example application/json
- content length：
- 真正的content，一般是data结构，例如通过JSON编码的数据

</aside>

### 内容结构

针对HTTP消息content创建可编码类型（用来解码对应的content data）

```swift
{“hello": "world"}

// 创建与之匹配的类型， 类似于Codable协议
struct Greeting: Content { 
	var hello: String

//范例
app.post("greeting"){req in
	let greeting = try req.content.decode(Greeting.self)
	print(greeting.name) // "world"
	return HTTPStatus.ok
}
```

<aside>
💡 以上代码：
* req.content其实就是HTTP消息里面的body/content，一般是Data数据。所以，content直接进行decode方法，和Codable协议一致
* 可以直接返回`HTTPStatus`

</aside>

🤔️： 解码方法使用请求的content类型来寻找合适的解码器？这部分不太理解

### 支持的媒体类型

**重点看！！**

![Untitled](Vapor%20Doc%E6%80%BB%E7%BB%93%20d097a0fba99c437dae30eb0f7e105a48/Untitled%201.png)

## 查询

针对URL查询字符串中的URL编码数据？

### 解码

也是需要创建一个对于查询字串对的结构体来进行解码，例如

```swift
GET /hello?name=Vapor HTTP/1.1
content-length: 0

//创建与查询字串对应的结构体
struct Hello: Content {
	var name: String?
}

//进行解码
app.get("hello") {req -> String in
	let hello = req.query.decode(Hello.self) // 解码环节，req.query保存查询数据
	return "Hello, \(hello.name ?? "Anonymous")"
}
```

### 单值

也能通过字符串字典形式来获取单个参数值

```swift
let name: String? = req.query["name"]
```

## 钩子hook

Content类型的两个方法： `beforeDecode` `afterDecode`, 这两个方法会在decode前后自动调用，可以自定义。

不明白使用方式。🤔️

## 覆盖默认值

改变Content API 默认的编码器和解码器

### 全局

```swift
let encoder = JSONEncoder()
ContentConfiguration.global.use(encoder: encoder, for .json) //全局修改
```

### 单次生效

```swift
let hello = try req.content.decode(Hello.self, using: decoder) //增加一个using参数实现单次修改
```

## 定制编码器

- 可以针对Content和Query两个方面进行编解码器的定制，分别实现各自协议，协议具体实现方法和Codable类似
    - `ContentEncoder` / `ContentDecoder`
    - `URLQueryEncoder` / `URLQueryDecoder`
- 还可以针对`ResponseEncodable`进行定制，具体实现内部的方法参考如下代码，**重点**！

```swift
 HTML的包装类型
struct HTML {
	let value: String
}

extension HTML: ResponseEncodable {
	public func encodeResponse(for request: Request) async throws -> Response { // A：
		var headers = HTTPHeaders()
		headers.add(name: .contentType, value: "text/html") // 构建header
		return .init(status: .ok, headers: headers, body: .init(string: value))
	}
}
// 符合协议就是可以返回一个Response
// encodeResponse就是返回一个response
// response的生成是三部分： status, headers, body

// 使用方法
app.get { _ in  
// 这里返回的是需要符合ResponseEncodabe协议的类型，
//因为可以调用协议要求的方法最终返回一个Response。 参考上面 A
	HTML(value: """ 
	<html>
		<body>
			<h1>Hello, world!</h1>
		</body>
	<html>
	""")
}

```

# Client

通过client去访问外部资源？

# Validation验证

添加一个验证方法。

方法是生成一个Validations集合， 常见做法是类型继承Validatable

## 添加验证

### 遵循协议

```swift
extension CreateUser: Validatable {
	static func validations(_ validations: inout Validations) {
		validations.add("email", as:String, is: .email)
	}
}
```

### 验证请求的`Content`

在路由处理程序 req.content.decode(CreateUser.self)之前使用

```swift
try CreateUser.validate(content: req)
```

### 验证请求的`Query`

```swift
try CreateUser.validate(query: req)
```

### 其他类型的验证

```swift
// 13以上整数
validataions.add("age", as: Int.self, is: .range(13...))
// 字符串验证
validations.add("name", as: String.self, is: !.empty)
validations.add("username", as: String.self, is: .count(3...) && .alphanumeric)
```

## 验证器

提供了多个验证器用来进行验证

- .ascii 仅包含ASCII字符
- .alphanumeric 仅包含字母数字字符
- …更多

验证器可以通过运算组合符号组合起来构建复杂的验证

# Async

## Async await

使用旧API时候，可以通过调用.get()方法来返回一个`EventLoopFuture`

```swift
return someMethodCallThatReturnAFuture().flatMap{futureSresult in ...}
//变成
let futureResult = try await someMethodThatReturnsAsFuture().get()
```

## EventLoopFuture

Future是基于回调的异步API的替代方案（老办法）

## 转换

一连串的链式调用

- map
- flatMapThrowing
- flatMap
- transform

# 日志

[Vapor:  入门 →  日志](https://docs.vapor.codes/zh/basics/logging/)

Vapor通过SwiftLog来实现日志记录

## 日志级别

![Untitled](Vapor%20Doc%E6%80%BB%E7%BB%93%20d097a0fba99c437dae30eb0f7e105a48/Untitled%202.png)

可以通过修改Log级别来增加或减少日志的生成数量

- 参数修改
- 设置LOG_LEVEL环境变量

# 环境

Vapor默认在development中运行

# 错误

## 中断Abort