# 第五章 Fragment

# **5.1 Fragment**是什么

Fragment是一种可以嵌入在Activity当中的UI片段，它能让程序更加合理和充分地利用大屏幕 的空间，因而在平板上应用得非常广泛。虽然Fragment对你来说是个全新的概念，但我相信你 学习起来应该毫不费力，因为它和Activity实在是太像了，同样都能包含布局，同样都有自己的 生命周期。你甚至可以将Fragment理解成一个**迷你型的Activity**，虽然这个迷你型的Activity 有可能和普通的Activity是一样大的。

<aside>
💡 类似于NavigationSplitView，一个屏幕里面多个View组合

</aside>

# **5.2 Fragment**的使用方式

同样也是Fragment类 + layout的xml文件

### 类

```swift
class LeftFragment : Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
            savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.left_fragment, container, false)
    }
}
```

- 新版继承androidx，依赖添加 `implementation "androidx.fragment:fragment-ktx:1.6.0”`
- 重写相应方法来创建view

### xml

```swift
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:background="#00ff00"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:textSize="24sp"
        android:text="This is right fragment"
        />
</LinearLayout>
```

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled.png)

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%201.png)

### 在Activity中添加Fragment

```swift
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />
    <fragment
        android:id="@+id/rightFrag"
        android:name="com.example.fragmenttest.RightFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />
</LinearLayout>
```

## **5.2.2** 动态添加**Fragment**

```swift
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button.setOnClickListener {
            replaceFragment(AnotherRightFragment())
        }
        replaceFragment(RightFragment())
    }
    private fun replaceFragment(fragment: Fragment) {
        val fragmentManager = supportFragmentManager
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.rightLayout, fragment)
        transaction.commit()
} }
```

通过动态添加Fragment主要分为5步。

1. 创建待添加Fragment的实例。
2. 获取`FragmentManager`，在Activity中可以直接调`getSupportFragmentManager()` 方法获取。
3. 开启一个`transaction`，通过调用beginTransaction()方法开启。
4. 向容器内添加或替换Fragment，一般使用`replace()`方法实现，需要传入容器的id和待添
加的Fragment实例。
5. 提交`transaction`，调用`commit()`方法来完成。

## **5.2.3** 在**Fragment**中实现返回栈

```swift
private fun replaceFragment(fragment: Fragment) {
        val fragmentManager = supportFragmentManager
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.rightLayout, fragment)
        transaction.addToBackStack(null) // 这里把transaction加入栈中
        transaction.commit()
}
```

事务提交之前调用了FragmentTransaction的`addToBackStack()`方法，它可以 接收一个名字用于描述返回栈的状态，一般传入null即可

## **5.2.4 Fragment**和**Activity**之间的交互

如果想要在Activity中调用Fragment里的方法，或者在Fragment中调用 Activity里的方法，应该如何实现呢?

### 从布局中获取Fragment

```swift
val fragment = supportFragmentManager.findFragmentById(R.id.leftFrag) as LeftFragment
```

### Fragment中又该怎样调用Activity

在每个Fragment中都可以通过调用`getActivity()`方法来得到 和当前Fragment相关联的Activity实例，代码如下所示

```swift
if (activity != null) {
    val mainActivity = activity as MainActivity
}
```

# **5.3 Fragment**的生命周期

和Activity一样，Fragment也有自己的生命周期，并且它和Activity的生命周期实在是太像

### 1. 运行状态

当一个Fragment所关联的Activity正处于运行状态时，该Fragment也处于运行状态。

### 2. 暂停状态

当一个Activity进入暂停状态时(由于另一个未占满屏幕的Activity被添加到了栈顶)，与它相关联的Fragment就会进入暂停状态。

### 3. 停止状态

当一个Activity进入停止状态时，与它相关联的Fragment就会进入停止状态，或者通过调用FragmentTransaction的remove()、replace()方法将Fragment从Activity中移除，但在事务提交之前调用了addToBackStack()方法，这时的Fragment也会进入停止状态。总的来说，进入停止状态的Fragment对用户来说是完全不可见的，有可能会被系统回收。

### 4. 销毁状态

Fragment总是依附于Activity而存在，因此当Activity被销毁时，与它相关联的Fragment就会进入销毁状态。或者通过调用FragmentTransaction的remove()、replace()方法将Fragment从Activity中移除，但在事务提交之前并没有调用addToBackStack()方法，这时的Fragment也会进入销毁状态。

# **5.4** 动态加载布局的技巧

## **5.4.1** 使用限定符

- 创建内容不同，名字相同的activity_main.xml，例如包含了两个Fragment
- 放在layout_large文件夹中

这个large就成了限定符，在大屏幕产品上，例如Pad会使用layout_large中的activity_main.xml

<aside>
💡 😧居然是这样来判断和动态加载的

</aside>

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%202.png)

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%203.png)

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%204.png)

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%205.png)

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%206.png)

![Untitled](%E7%AC%AC%E4%BA%94%E7%AB%A0%20Fragment%20632ab4b3ca8a4ade81083b250bcd2bc6/Untitled%207.png)