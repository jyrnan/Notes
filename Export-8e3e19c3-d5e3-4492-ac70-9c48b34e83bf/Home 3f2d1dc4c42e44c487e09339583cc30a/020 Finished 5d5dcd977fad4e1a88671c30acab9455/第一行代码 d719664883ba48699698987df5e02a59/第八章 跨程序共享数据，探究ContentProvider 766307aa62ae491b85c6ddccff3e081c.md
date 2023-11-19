# 第八章 跨程序共享数据，探究ContentProvider

目前，使 用ContentProvider是Android实现跨程序共享数据的标准方式。

# 8.2 运行时权限

和iOS类似，需要在清单文件中提出权限申请。

# 8.3 访问其他程序中的数据

ContentProvider的用法一般有两种:

- 一种是使用现有的ContentProvider读取和操作相应程 序中的数据;
- 另一种是创建自己的ContentProvider，给程序的数据提供外部访问接口。

## 8.3.1 Contentresolver的基本用法

对于每一个应用程序来说，如果想要访问ContentProvider中共享的数据，就一定要借助 ContentResolver类，可以通过Context中的getContentResolver()方法获取该类的实 例。ContentResolver中提供了一系列的方法用于对数据进行增删改查操作，

### URI

ontentResolver中的增删改查方法都是不接收表名参数的，而是使用一个Uri参数代替，这个参数被称为内容URI。

内容URI给ContentProvider中的数据建立 了唯一标识符，它主要由两部分组成:authority和path。

- authority是用于对不同的应用程序 做区分的，一般为了避免冲突，会采用应用包名的方式进行命名。比如某个应用的包名是 com.example.app，那么该应用对应的authority就可以命名为 com.example.app.provider。
- path则是用于对同一应用程序中不同的表做区分的，通常会添 加到authority的后面。
- 我们还需要在字符串的头部加上协议声明。因此， 内容URI最标准的格式如下:

```swift
content://com.example.app.provider/table1
```

### 解析

```swift
val uri = Uri.parse("content://com.example.app.provider/table1")
```

# 8.4 创建自己的ContentProvider

## 8.4.1 步骤

如果想要实现跨程序共享数据的功能，可以通过新建一个类去继承 ContentProvider的方式来实现。ContentProvider类中有6个抽象方法，我们在使用子类继承 它的时候，需要将这6个方法全部重写

```kotlin
class MyProvider : ContentProvider() {
    override fun onCreate(): Boolean {
        return false
}
    override fun query(uri: Uri, projection: Array<String>?, selection: String?,
            selectionArgs: Array<String>?, sortOrder: String?): Cursor? {
return null }
    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        return null
}
    override fun update(uri: Uri, values: ContentValues?, selection: String?,
            selectionArgs: Array<String>?): Int {
return 0 }
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int {
        return 0
}
    override fun getType(uri: Uri): String? {
        return null
} }
```

### 通过URI参数来实现修改

我们需要对传入的uri参数进行解析，从中分析出调用方期望访问的 表和数据。

### 通配符

- *表示匹配任意长度的任意字符。
- #表示匹配任意长度的数字

### URIMatcher类

我们再借助UriMatcher这个类就可以轻松地实现匹配内容URI的功能。UriMatcher中 提供了一个addURI()方法，这个方法接收3个参数，可以分别把authority、path和一个自 定义代码传进去。

```kotlin
class MyProvider : ContentProvider() {
    private val table1Dir = 0
    private val table1Item = 1
    private val table2Dir = 2
    private val table2Item = 3
    private val uriMatcher = UriMatcher(UriMatcher.NO_MATCH)
    init {
        uriMatcher.addURI("com.example.app.provider", "table1", table1Dir)
        uriMatcher.addURI("com.example.app.provider ", "table1/#", table1Item)
        uriMatcher.addURI("com.example.app.provider ", "table2", table2Dir)
        uriMatcher.addURI("com.example.app.provider ", "table2/#", table2Item)
    }
    ...
    override fun query(uri: Uri, projection: Array<String>?, selection: String?,
            selectionArgs: Array<String>?, sortOrder: String?): Cursor? {
        when (uriMatcher.match(uri)) {
table1Dir -> {
// 查询table1表中的所有数据
            }
            table1Item -> {
// 查询table1表中的单条数据 }
table2Dir -> {
// 查询table2表中的所有数据
            }
            table2Item -> {
// 查询table2表中的单条数据 }
}
... }
... }
```

### getType()方法

所有的 ContentProvider都必须提供的一个方法，用于获取Uri对象所对应的`MIME`类型

URI所对应的MIME字符串主要由3部分组成，Android对这3个部分做了如下格式规定。

- 必须以vnd开头。
- 如果内容URI以路径结尾，则后接android.cursor.dir/;如果内容URI以id结尾，则后接android.cursor.item/。
- 最后接上vnd.<authority>.<path>。

```kotlin
vnd.android.cursor.dir/vnd.com.example.app.provider.table1
//或者
vnd.android.cursor.item/vnd.com.example.app.provider.table1
```

方法

```kotlin
class MyProvider : ContentProvider() {
    ...
    override fun getType(uri: Uri) = when (uriMatcher.match(uri)) {
        table1Dir -> "vnd.android.cursor.dir/vnd.com.example.app.provider.table1"
        table1Item -> "vnd.android.cursor.item/vnd.com.example.app.provider.table1"
        table2Dir -> "vnd.android.cursor.dir/vnd.com.example.app.provider.table2"
        table2Item -> "vnd.android.cursor.item/vnd.com.example.app.provider.table2"
        else -> null
} }
```

### 范例（略）