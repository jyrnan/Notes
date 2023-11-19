# 可以自定义Preview的ContextMenu

# 基本方法

ContextMenu两个重要参数：

```swift
public func contextMenu<M, P>(@ViewBuilder menuItems: () -> M, @ViewBuilder preview: () -> P) -> some View where M : View, P : View
```

# menuItems

传入一串视图，一般是Button，用来作为菜单的各个子项

## Preview

利用Context Menu可以定义长按视图弹出的菜单，默认是以长按的视图作为预览

![Untitled](%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%AE%9A%E4%B9%89Preview%E7%9A%84ContextMenu%20dc64ed7f2ab84a47ac835fb3c189af2a/Untitled.png)

### 自定义Preview

可以通过preview这个参数传入一个作为预览的视图，定制更加复杂的预览视图

```swift
ForEach(restaurants) { restaurant in
                BasicImageRow(restaurant: restaurant)
                .contextMenu {
                  Button {
                    //Action
                  } label: {
                    Text(restaurant.name)
                  }

                } preview: {
                  Image(systemName: "house")
                }

            }
```

![Untitled](%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%AE%9A%E4%B9%89Preview%E7%9A%84ContextMenu%20dc64ed7f2ab84a47ac835fb3c189af2a/Untitled%201.png)