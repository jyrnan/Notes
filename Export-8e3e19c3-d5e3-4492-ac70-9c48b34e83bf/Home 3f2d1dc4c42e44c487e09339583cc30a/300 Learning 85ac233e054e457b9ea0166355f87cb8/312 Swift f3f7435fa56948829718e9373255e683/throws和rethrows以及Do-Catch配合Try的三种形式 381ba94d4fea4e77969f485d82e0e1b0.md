# throws和rethrows以及Do-Catch配合Try的三种形式

• 含有 throws 的方法会抛出一个（但不限定是一种）异常，

```swift
enum E: Error {
    case Negative
}
///这个方法返回是throws和Int，以为着如果正常就会Int值，如果不正常则会Throws错误出来
///当然这个错误也可会有很多类型
///如果只是throws而没有具体的返回值，那就表明会依据情况返回错误类型，但是没有具体的返回值，相当于返回一个Void。
func methodThrowsWhenPassingNegative(number: Int) throws -> Int {
    if number < 0 {
        throw E.Negative
    }
    return number
}
```

- 在 Swift 2.0 中还有一个类似的关键字：rethrows。
- 其实 rethrows 和 throws 做的事情并没有太多不同，它们都是标记了一个方法应该抛出错误。**但是 rethrows 一般用在参数中含有可以 throws 的方法的高阶函数中，来表示它既可以接受普通函数，也可以接受一个能 throw 的函数作为参数。**

```swift
func methodThrows(num: Int) throws {
    if num < 0 {
        throw E.Negative
    }
    print("Excuted!")
}

///rethrows的函数，接受一个可以throws的函数作为参数。
///执行这个作为参数的函数的时候，需要用try，这样可以让该函数抛出错误
///其实在这种情况下，我们简单地把 rethrows 改为 throws，这段代码依然能正确运行。
///但是为了保证代码的可读性，所以把后续的throws改成rethrows
func methodRethrows(num: Int, f: (Int) throws -> Void) rethrows {
    try f(num)
}
///在最外层调用的时候用do—catch语句，用来捕捉抛出的错误并进行处理
do {
    try methodRethrows(num: 1, f: methodThrows(num:))
} catch _ {
    
}
```

- 首先，try 可以接 ! 表示强制执行，这代表你确定知道这次调用不会抛出异常。如果在调用中出现了异常的话，你的程序将会崩溃，这和我们在对 Optional 值用 ! 进行强制解包时的行为是一致的。
- 另外，我们也可以在 try 后面加上 ? 来进行尝试性的运行。try? 会返回一个 Optional 值：如果运行成功，没有抛出错误的话，它会包含这条语句的返回值，否则将为 nil。和其他返回 Optional 的方法类似，一个典型的 try? 的应用场景是和 if let 这样的语句搭配使用。不过如果你用了 try? 的话，就意味着你无视了错误的具体类型。或者说**其实try？ 是把本来可以甩出具体错误的情况转化成为一个optional： 成功就是真正的值，不成功就是nil，至于不成功的原因就不提供了**。
- 所以，**try本身和do catch来配合使用**，因为try是可以throws错误出来的，如果需要知道不成功时是什么原因，必须要用try本身和do catch来配合使用。catch可以用来补获各种具体的错误类型并执行相应的操作。其实这个和switch有点类似

```swift
///定义Error的类型
enum LoginError: Error {
    case UserNotFound, UserPasswordNotMatch
}
///login是可能会throws错误的。所以在可能抛出异常的语句前都添加一个try
func login(user: String, password: String) throws {
    //users 是 [String: String]，存储[用户名:密码]

    if !users.keys.contains(user) {
        throw LoginError.UserNotFound
    }

    if users[user] != password {
        throw LoginError.UserPasswordNotMatch
    }

    print("Login successfully.")
}
///try语句往往和do-catch配合
do {
    try login(user: "onevcat", password: "123")
} catch LoginError.UserNotFound {
    print("UserNotFound")
} catch LoginError.UserPasswordNotMatch {
    print("UserPasswordNotMatch")
}
```

**补充一个范例**

```swift
func nonThrowing() {
    print("non throwing")
}

extension String: Error {} //直接把String扩展成Error以便在下面的func里面直接抛出错误
func throwFunction() throws {
        throw "An Error"
    }

func doSomething(myClosure: () throws -> Void ) rethrows -> Void {
    try myClosure()  //需要使用try
}

//常规的不带throw的闭包可以直接使用
doSomething(myClosure: nonThrowing)

do {
    try doSomething(myClosure: throwFunction)
} catch {
    print("Catch: \(error)")
}
```