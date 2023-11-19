# UIKit添加手势识别

添加手势识别可以通过代码或者Storyboard方式

# 代码方式

```objectivec
-(void)addCancelChangeModeGesture{
    UITapGestureRecognizer *recognizer = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(disMissChangeModeView)]; //1 创建一个手势识别对象。
    [self.mChangeModeBgView addGestureRecognizer:recognizer]; //2 附着到View上
}
/// JY：取消模式选择View
-(void)disMissChangeModeView{
    [self hideChoiceModeView];
    NSDictionary *dict = @{ @"Operate_type" : @"取消" };
    //JY：埋点功能
    [MobClick event:@"Remote_Change_Mode" attributes:dict];
}
```

1. 创建一个手势识别对象。方法参数解释如下
    1. 参数Target：该参数指定了第二个action方法（消息）要发送的对象。所以必须是能对该消息进行相应的对象
    2. 参数action：手势产生后执行的方法（消息）
    3. 补充说明：这一堆参数其实就是OC里面最典型的一个概念，对象和消息。二者是相对应的，对象必须要能够对该消息进行响应。虽然可以进行动态替换，但是这个概念是不变的。

<aside>
💡 因为该手势识别对象是通过alloc创建，是创建在堆上，所以不用担心其随着函数退出被销毁。二后续步骤该手势会被添加（附着）到其它对象，例如范例中的mChangeModeBgView，所以会被该对象持有，因而不会被释放）

</aside>

1. 将该手势识别对象附着到某个View上。

# StoryBoard方式

通过Storyboard方式添加其实就是省略了一些代码创建过程。

## 添加并附着手势对象到视图上

1. 首先从Libary里面选择一个手势对象，拖到相应的View中。注意拖到哪个view，就相当于在哪个view添加了这个手势。（这个View成为这个手势的FirstResponder？）
    
    ![截屏2022-06-17 17.03.22.png](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/%E6%88%AA%E5%B1%8F2022-06-17_17.03.22.png)
    
    手势对象会出现在导航栏中，但并不是在相应的View下，
    
    ![截屏2022-06-17 17.15.20.png](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/%E6%88%AA%E5%B1%8F2022-06-17_17.15.20.png)
    
    这一步其实相当于`[self.mChangeModeBgView addGestureRecognizer:recognizer];` //2 附着到View上
    

## 设置手势对象链接代码

1. 添加手势后可以按住ctrl将其拖到某个对象的代码当中来实现手势对象和代码的链接：对象可以是ViewController，也可以是View，其实就是设置了该手势执行的对象？而对象的不同，也会显示出不同的链接可能性。
2. 这部分的操作相当于代码方式设置了对象和方法：`[[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(disMissChangeModeView)]`

3. 直接在导航栏来链接，会在弹出菜单显示不同的链接方式。（应该是只能在该ViewController范围内链接）

![Untitled](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/Untitled.png)

![截屏2022-06-17 17.21.36.png](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/%E6%88%AA%E5%B1%8F2022-06-17_17.21.36.png)

1. 拖入代码中会依据插入代码位置不同产生不同的链接方式：
- 如果是@implementation部分，则是创建一个Action的链接，对应的是一个执行方法。例如下例。
    
    ![Untitled](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/Untitled%201.png)
    
    值得注意的是该方法有一个sender参数可以用来标记是哪个对象发送来这个消息。
    
    ```objectivec
    - (IBAction)test:(id)sender {
        [self touch];
    }
    ```
    
- 如果是插入到@interface 部分，则是创建一个outlet链接。
    
    ![Untitled](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/Untitled%202.png)
    

这样，通过storyboard创建的手势对象就和代码完全结合了。

## 疑问：Exit是什么？

![Untitled](UIKit%E6%B7%BB%E5%8A%A0%E6%89%8B%E5%8A%BF%E8%AF%86%E5%88%AB%207cf72d9ca1e24586bf3b690f8aa322ad/Untitled%203.png)

我從未使用過它，也可能永遠不會使用它，但是您可以將一個物件指定為第一個從UI接收事件的物件。

我想您可能正在建立一個UIView子類並將其新增到UIViewController中，但實際上您希望其他一些物件接收和處理您要新增到的UIViewController以外的事件。

我發現[this link](https://web.archive.org/web/20130601045552/http://cocoadev.com/wiki/FirstResponder)可以更好地解釋它。