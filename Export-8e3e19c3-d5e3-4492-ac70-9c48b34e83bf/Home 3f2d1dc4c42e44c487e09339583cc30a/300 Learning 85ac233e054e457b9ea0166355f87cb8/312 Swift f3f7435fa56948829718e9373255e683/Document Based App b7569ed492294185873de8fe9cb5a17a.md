# Document Based App

# Document App基本关系

<aside>
💡 基本关系：
DocumentGroup

→`FileDocumentConfiguration`

→`FileDocument`

→`[ReadConfiguration](https://developer.apple.com/documentation/swiftui/filedocument/readconfiguration)` / `[WriteConfiguration](https://developer.apple.com/documentation/swiftui/filedocument/writeconfiguration)` 这个关系是不是和第一项一致待定？

→`FileWrapper`

</aside>

# ****FileDocumentConfiguration****

- `var document: Document` The current document model.
- `var $document: Binding<Document>`

****Getting document properties****

- `var fileURL: URL?` The URL of the open file document.
- `var isEditable: Bool` Whether the document is editable.

# ****FileDocument / ReferenceFileDocument****

对文件的抽象可以理解成需要在内存中有一个数据结构， 实现上面的协议。有两种类型， 分别对应struct或者class

## 三个关键步骤

- 提供文档能够读取或写入的内容的类型
    - `[readableContentTypes](https://developer.apple.com/documentation/swiftui/referencefiledocument/readablecontenttypes)` 读取
    - `[writableContentTypes](https://developer.apple.com/documentation/swiftui/referencefiledocument/writablecontenttypes-6x6w9)` 写入
- 初始化方法`[init(configuration:)](https://developer.apple.com/documentation/swiftui/referencefiledocument/init(configuration:))` 可以理解成读取文件的方式
- 序列化内容的，可以理解成保存到磁盘的方式。
    - FileDecument 实现的方法是`[fileWrapper(configuration:)](https://developer.apple.com/documentation/swiftui/filedocument/filewrapper(configuration:))`
    - ReferenceFileDocument需要实现提供文件内容的快照方法 `[snapshot(contentType:)](https://developer.apple.com/documentation/swiftui/referencefiledocument/snapshot(contenttype:))`而序列化内容的方法是`[fileWrapper(snapshot:configuration:)](https://developer.apple.com/documentation/swiftui/referencefiledocument/filewrapper(snapshot:configuration:))`

<aside>
💡 这里读/写的一个关键参数是`configuration:`

</aside>

## configuration

这两个configuration都是TypeAlias

`[ReadConfiguration](https://developer.apple.com/documentation/swiftui/filedocument/readconfiguration)` = `[FileDocumentReadConfiguration](https://developer.apple.com/documentation/swiftui/filedocumentreadconfiguration)`

- `[contentType: UTType](https://developer.apple.com/documentation/swiftui/filedocumentreadconfiguration/contenttype)` The expected uniform type of the file contents.
- `[file: FileWrapper](https://developer.apple.com/documentation/swiftui/filedocumentreadconfiguration/file)` The file wrapper containing the document content.

`[WriteConfiguration](https://developer.apple.com/documentation/swiftui/filedocument/writeconfiguration)` = `[FileDocumentWriteConfiguration](https://developer.apple.com/documentation/swiftui/filedocumentwriteconfiguration)`

- `[contentType: UTType](https://developer.apple.com/documentation/swiftui/filedocumentwriteconfiguration/contenttype)`
- `[existingFile: FileWrapper?](https://developer.apple.com/documentation/swiftui/filedocumentwriteconfiguration/existingfile)` The file wrapper containing the current document content. `nil` if the document is unsaved.

<aside>
💡 而Configuration里面有一个关键的参数是`FileWrapper`

</aside>

## FileWrapper

[Apple Developer Documentation](https://developer.apple.com/documentation/foundation/filewrapper)

FileWrapper代表了用来访问**文件系统中的节点**。应该是实现了从内存数据到存储文件的转变，可以和`[FileManager](https://developer.apple.com/documentation/foundation/filemanager)`关联起来了

### 类型

- **Regular-file file wrapper:** 文件
- **Directory file wrapper:** 目录
- **Symbolic-link file wrapper:** 链接

### 有如下属性

- **Filename.** Name of the file system node the file wrapper represents.
- **file-system attributes.** See `[FileManager](https://developer.apple.com/documentation/foundation/filemanager)` for information on the contents of the `attributes` dictionary.
- **Regular-file contents.** Applicable only to regular-file file wrappers.
- **File wrappers.** Applicable only to directory file wrappers.
- **Destination node.** Applicable only to symbolic-link file wrappers.

---

# ****Defining file and data types for your app****

- Declaring your app’s custom type in the project’s `Info.plist` file.
- Creating a new identifier for exported types, or using existing identifiers for imported types.
- Defining each type’s conformance to system-declared types
- Listing any associated file extensions or MIME types.

## ****Declare a custom type for your app****

define it as an exported or imported type

If your app uses a type that this framework provides, don’t redeclare it in your app’s bundle.

### exported type

uses its own proprietary document format, declare it as an 

### imported type

uses a type that another app defines，

## ****Choose an identifier for your type****

start by using a reverse DNS format that begins with `com.companyName`

## ****Define the conformance****

The type declaration can include a list of type identifiers that the type conforms to. For example, if your app uses a proprietary file format based on `JSON`, use `public.json` in the `[UTTypeConformsTo](https://developer.apple.com/documentation/bundleresources/information_property_list/utexportedtypedeclarations/uttypeconformsto)` string in the `Info.plist`file.

## ****Define the description, extensions, and MIME types****