# 创建RouteCollection

通过创建RouterCollection可以把一系列router打包注册给app。

可以理解成RouteCollection就是一系列router的集合

<aside>
💡 其实routeCollection也就是名字一般，是多个routes的集合，只是通过一个协议方法一次性打包注册给api

</aside>

# 创建

创建的方法是满足RouteCollection协议

- 设置关系：协议方法 boot(routes:) 里面来设置各种请求分别与之对应的处理方法的联系
- 写出具体处理方法

```swift
struct MoviesController: RouteCollection {
    /// 协议方法
    func boot(routes: Vapor.RoutesBuilder) throws {
        let movies = routes.grouped("movies")
        /// 建立请求和对应处理方法的关系
        movies.get( use: index)
    }
    
    /// 协议方法中调用的相应处理方法
    func index(req: Request) async throws -> String {
        return "index"
    }
}
```

# 注册使用

创建好的RouteCollection需要注册使用。在routes方法中和其他单体route一起注册

```swift
// configures your application
public func configure(_ app: Application) async throws {
    // uncomment to serve files from /Public folder
    // app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))

    // register routes
    try routes(app) // 注册所有的route
}

func routes(_ app: Application) throws {
    
    try app.register(collection: MoviesController()) //注册RouteCollection
		
/// 更多单体的router
...
}
```