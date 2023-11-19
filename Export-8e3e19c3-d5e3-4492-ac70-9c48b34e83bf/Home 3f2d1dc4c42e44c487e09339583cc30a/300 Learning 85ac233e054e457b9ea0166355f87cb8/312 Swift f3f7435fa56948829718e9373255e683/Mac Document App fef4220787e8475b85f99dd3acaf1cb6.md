# Mac Document App

OC/Swift 中的Mac Document app有如下几个关键点：

- 生成的App模版中有两个主要类：ViewController和Document，后者更加是核心
- ViewController负责控制界面显示，有一个属性`representedObject` 表示数据文件，该数据一般由Document类来持有。
- Document类是核心，有如下几个关键方法：
    - `- (**void**)makeWindowControllers` 这个方法是创建一个ViewController，设置成整个App的VC部分。
    - 同时，把Document内的数据文件传给VC的`representedObject` 属性
    - 可以创建一个`contentViewController` 的属性，用来指向相应的ViewController（但这里应该有点问题，如果是多个窗口就不适用）
    
    ```objectivec
    - (void)makeWindowControllers {
        // Override to return the Storyboard file name of the document.
        [self addWindowController:[[NSStoryboard storyboardWithName:@"Main" bundle:nil] instantiateControllerWithIdentifier:@"Document Window Controller"]];
        self.contentViewController = self.windowControllers.firstObject.contentViewController; //这是自己加的代码
    }
    ```
    
    ```swift
    /// - Tag: makeWindowControllersExample
        override func makeWindowControllers() {
            // Returns the storyboard that contains your document window.
            let storyboard = NSStoryboard(name: NSStoryboard.Name("Main"), bundle: nil)
            if let windowController =
                storyboard.instantiateController(
                    withIdentifier: NSStoryboard.SceneIdentifier("Document Window Controller")) as? NSWindowController {
                addWindowController(windowController)
                
                // Set the view controller's represented object as your document.
                if let contentVC = windowController.contentViewController as? ViewController {
                    contentVC.representedObject = content
                    contentViewController = contentVC
                }
            }
        }
    ```
    
    - NSDocument类来响应App的基本操作，例如存储，打开，分别用如下两个基本方法来对应实现。在NSDocment子类中需要重写这两个方法来实现对数据的存储。
    
    ```objectivec
    - (NSData *)dataOfType:(NSString *)typeName error:(NSError **)outError
    - (BOOL)readFromData:(NSData *)data ofType:(NSString *)typeName error:(NSError **)outError
    ```
    
    ```swift
    /// - Tag: readExample
        override func read(from data: Data, ofType typeName: String) throws {
            content.read(from: data)
        }
        
        /// - Tag: writeExample
        override func data(ofType typeName: String) throws -> Data {
            return content.data()!
        }
    ```