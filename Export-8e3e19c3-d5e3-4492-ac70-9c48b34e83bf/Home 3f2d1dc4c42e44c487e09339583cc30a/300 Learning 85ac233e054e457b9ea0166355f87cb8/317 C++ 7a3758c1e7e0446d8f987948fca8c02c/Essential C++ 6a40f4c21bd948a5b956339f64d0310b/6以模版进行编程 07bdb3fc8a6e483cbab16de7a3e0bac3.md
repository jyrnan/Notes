# 6以模版进行编程

# 6.1 被参数化的类型

理解了

# 6.3 template类型参数处理

在编写构造器代码时候，注意以下两点

1. 尽量通过引用传入参数
2. 尽量在构造体方法具体执行内容之前实现内部变量的赋值

```cpp
template<typename valType>
inline BTNode<valType>::
BTNode( const valType &val): _val(val) { //这里实现了在构造方法之前调用valType的默认构造方法。
    _cnt = 1;
    _lchild = _rchild = 0;
}
```

> 在方法内部实现赋值并不会产生错误，但是效率更低
> 

# 6.4 实现一个 Class Template

不能将模版类轻易分开放到头文件和实现文件中。（原因待深究）

# 6.6 常量表达式与默认参数

模版参数可以是具体的类型，并作为参数输入

```cpp
template <int length, int beg_pos>
class num_sequence {
}
```

# 6.7 以Template参数作为一种设计策略

# 6.8 Member Template Function

```cpp
class PrintIt {
public:
    PrintIt(ostream &os):_os(os){}
    
    template<typename elemType>
    void print(const elemType &elem, char delimiter = '\n'){
        _os << elem << delimiter;
    }
private:
    ostream& _os;
    
};
```