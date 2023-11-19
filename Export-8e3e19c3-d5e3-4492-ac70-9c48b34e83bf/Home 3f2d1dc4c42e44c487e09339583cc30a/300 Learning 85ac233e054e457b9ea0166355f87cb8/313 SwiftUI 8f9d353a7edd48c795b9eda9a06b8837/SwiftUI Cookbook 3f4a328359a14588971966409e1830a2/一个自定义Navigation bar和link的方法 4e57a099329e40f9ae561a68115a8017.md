# 一个自定义Navigation bar和link的方法

[Create a custom navigation bar and link in SwiftUI | Advanced Learning #12](https://youtu.be/aIDT4uuMLHc?si=y9wNXNFP7Y7na4oV)

这哥们的方法很厉害，并没有太多高深的内容。主要方法

- 依托原有的NavigationView的底层导航，因为官方的效率还是不错的
- 创建一个`CustomNavBarContainerview`
- 在NavigationView里面内置`CustomNavBarContainerview` 并且隐藏掉原有的navigationBar， 而在NavigationView外面再包一层自定义的CustomeNavigationView来实现

![Untitled](%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89Navigation%20bar%E5%92%8Clink%E7%9A%84%E6%96%B9%E6%B3%95%204e57a099329e40f9ae561a68115a8017/Untitled.png)

- 自定义NavigationLink也是通过类似的方法
    - CustomNavigtionLink里面包官方的NavigationLink，传入Destination和Label用来构建NavigationLink
    - 传入的Destination 提取出来先包进`CustomNavBarContainerview` 再做为NavigationLink的destination

![Untitled](%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89Navigation%20bar%E5%92%8Clink%E7%9A%84%E6%96%B9%E6%B3%95%204e57a099329e40f9ae561a68115a8017/Untitled%201.png)

### 设置title等内容

- 创建PreferenceKey
- extension View创建方法来设置PreferenceKey

![Untitled](%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89Navigation%20bar%E5%92%8Clink%E7%9A%84%E6%96%B9%E6%B3%95%204e57a099329e40f9ae561a68115a8017/Untitled%202.png)

- 在下层View调用方法设置PreferenceKey

![Untitled](%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89Navigation%20bar%E5%92%8Clink%E7%9A%84%E6%96%B9%E6%B3%95%204e57a099329e40f9ae561a68115a8017/Untitled%203.png)

- 上层View观察PreferenceKey来更新View

![Untitled](%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89Navigation%20bar%E5%92%8Clink%E7%9A%84%E6%96%B9%E6%B3%95%204e57a099329e40f9ae561a68115a8017/Untitled%204.png)

### 恢复默认的返回手势

这里不太理解。应该是利用UIKit的方法

![Untitled](%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89Navigation%20bar%E5%92%8Clink%E7%9A%84%E6%96%B9%E6%B3%95%204e57a099329e40f9ae561a68115a8017/Untitled%205.png)