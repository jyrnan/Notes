# UITableViewController

几个关键点：

![Untitled](UITableViewController%201b984916121e48c1a6abf650cf5dec0b/Untitled.png)

# tableView属性

- 类型是UITableView。看起来是和view属性重合

# 两个关键协议

- tableView需要有两个关键协议来支持，一般都设置成UITableViewController本身来满足实现
    - UITableViewDataSource协议：提供数据要遵守UITableViewDataSource协议，就必须实现两个必须方法
        - `tableView:numberOfRowsInSection:` 返回某个section里row的个数
        - `tableView:cellForRowAtIndexPath:` 返回对应indexPath的[cell视图](UITableViewController%201b984916121e48c1a6abf650cf5dec0b.md)
    - delegate：针对tableView的一些操作来实现对应的方法。使UITableView可以响应用户操作

# UITableViewCell对象

UITableViewCell对象有一个`UIScrollView`，里面再包含有一个子视图：`contentView` 

ScrollView的作用是可以实现Cell的左右滑动操作。而所有的子视图都必须先放在`contentView` 是因为它会在某些情况下改变大小，例如进入编辑模式时候contentView会缩小，为编辑控件留出位置。

![Screen Shot 2022-03-25 at 14.44.52.png](UITableViewController%201b984916121e48c1a6abf650cf5dec0b/Screen_Shot_2022-03-25_at_14.44.52.png)

## 创建cell的方法

- 默认：直接返回一个默认视图
- 自定义Cell视图：必须在ViewDidLoad方法里面注册Cell类。按照约定，应该将UITableViewCell或者UITableViewCell子类的类名用作`reuseIdentifier`。

```objectivec
[self.tableView registerClass:[UITableViewCell class]forCellReuseIdentifier:@"UITableViewCell"];
```

### 重用UITableViewCell对象

是取回的对象是否是某个特定的类型。每个UITableViewCell对象都有一个类型为NSString的reuseIdentifier属性。当数据源向UITableView对象获取可重用的UITableViewCell对象时，可传入一个字符串并要求UITableView对象返回相应的UITableViewCell对象，这些UITableViewCell对象的reuseIdentifier属性必须和传入的字符串相同。

```objectivec
“UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"UITableViewCell" forIndexPath:indexPath];
```

# TableView的编辑

## 编辑属性

UITableView有一个名为`editing`的属性，如果将editing属性设置为YES，UITableView就会进入编辑模式。

通过 `[self setEditing:YES animated:YES];` 来设置

## HeaderView & FooterView

表头视图是指UITableView对象可以在其表格上方显示的特定视图，适合放置针对某个表格段或整张表格的标题和控件。表头视图可以是任意的UIView对象。
表头视图有两种，分别针对表格段和表格。类似地，还有表尾视图（footer view），也具有表格段和表格两种

## 其他若干Delegate的方法来实现移动、删除等方法

## TableView的reload

很多操作以后要记得reloadData

```objectivec
[self.tableView reloadData];
```