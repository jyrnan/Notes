# SwiftUI tips

**SwiftUI tips**

1. apply a **clipShape()** modifier and ask it to be clipped to a circle shape
2. add an **overlay()** modifier so we place a shape on top of our image
3. Spacer() will automatically take up all available free space
4. If you find your menu item labels being clipped, try adding **.layoutPriority(1)** to the **VStack** in **ItemRow** – that should give SwiftUI the hint that the items in the **VStack** should be shown in full. Hopefully this won’t be needed soon!
5. 为开关切换添加动态效果

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled.png)

- 列表中激活编辑模式

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled%201.png)

- 停用某个组件 .disable()

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled%202.png)

- If you want SwiftUI to use your image’s original color, you should attach a **renderingMode()** modifier to it, like this:

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled%203.png)

- Although both **Group** and **AnyView** achieve the same result for our layout, it’s generally preferable to use **Group** because it’s more efficient for SwiftUI.

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled%204.png)

- 利用计算属性来生成一些需要更多设置的属性，例如：

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled%205.png)

—————————————————————————————

- SwiftUI has an **exportFiles** environment key that lets us export files from our app to anywhere the user wants – any folder in iCloud, or on their local device实现输出文件到某个位置urls，但是输出什么么？还是不太明白，需要进一步了解。

To use it, first create a property to store the action as provided by the system:

@Environment(\.exportFiles) **var** exportAction

Finally, call your **exportAction** property with that URL, and provide a closure that will be called when the operation completes. This must accept a **Result<URL, Error>?**: a **URL** to the newly moved file on success, an error if something went wrong, and nil if the user cancelled.

Try this, for example:

![Untitled](SwiftUI%20tips%209d1b3ad5b11f4d3f8b1395d35c183eda/Untitled%206.png)

- **How to make print() work**

If you press play in the SwiftUI preview to try out your designs, you’ll find that any calls to **print()** are ignored. If you’re using **print()** for testing purposes – e.g. as simple button tap actions – then this can be a real headache.

Fortunately, there’s a simple fix: right-click on the play button in the preview canvas and choose “Debug Preview”. With that small change made you’ll find your **print()** calls work as normal.