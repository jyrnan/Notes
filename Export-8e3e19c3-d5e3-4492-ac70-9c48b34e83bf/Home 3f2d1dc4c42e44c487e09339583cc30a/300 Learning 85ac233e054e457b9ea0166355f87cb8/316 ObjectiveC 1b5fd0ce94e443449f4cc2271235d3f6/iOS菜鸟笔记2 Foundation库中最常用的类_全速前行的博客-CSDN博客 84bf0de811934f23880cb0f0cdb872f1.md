# iOS菜鸟笔记2: Foundation库中最常用的类_全速前行的博客-CSDN博客

据说，Foundation是支撑整个Objective-C开发的基础库，重要性不言而喻。 
 偷来一副图，表述一下Foundation的位置。

[iOS%E8%8F%9C%E9%B8%9F%E7%AC%94%E8%AE%B02%20Foundation%E5%BA%93%E4%B8%AD%E6%9C%80%E5%B8%B8%E7%94%A8%E7%9A%84%E7%B1%BB_%E5%85%A8%E9%80%9F%E5%89%8D%E8%A1%8C%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%2084bf0de811934f23880cb0f0cdb872f1/20170317214556347](iOS%E8%8F%9C%E9%B8%9F%E7%AC%94%E8%AE%B02%20Foundation%E5%BA%93%E4%B8%AD%E6%9C%80%E5%B8%B8%E7%94%A8%E7%9A%84%E7%B1%BB_%E5%85%A8%E9%80%9F%E5%89%8D%E8%A1%8C%E7%9A%84%E5%8D%9A%E5%AE%A2-CSDN%E5%8D%9A%E5%AE%A2%2084bf0de811934f23880cb0f0cdb872f1/20170317214556347)

Cocoa和UIKit主要关注于应用程序、UI及用户输入，而Foundation主要关注更底层的组织数据的任务。

本文为菜鸟所做，简单的记录自己使用NSString、NSArray、NSDictionary

# NSString

官方文档：[https://developer.apple.com/reference/foundation/nsstring](https://developer.apple.com/reference/foundation/nsstring) 
 分OC和Swift两个版本讲解。 
 1）创建字符串

```objectivec
  NSString* constantString = @"Text of the string";
```

2)大小写转换

```objectivec
    //变成大写
    NSString* uppercaseString = [constantString uppercaseString];
    //变成小写
    NSString* lowercaseString = [constantString lowercaseString];
    //首字母大写
    NSString* capitalizedString = [constantString capitalizedString];
```

3）获取子字符串

```
    //获取前5个字符串
    NSString* startSubstring = [constantString substringToIndex:5];
    //获取第五个字符之后的字符串
    NSString* endSubstring = [constantString substringFromIndex:5];
    //获取index为2，长度是5的子字符串
    NSRange range = NSMakeRange(2, 5);
    NSString* substring = [constantString substringWithRange:range];
```

4）比较字符串

```
    if([uppercaseString isEqualToString:lowercaseString]) {
        NSLog(@"两个字符串相同");
    } else {
        NSLog(@"两个字符串不相同");
    }
```

5）查找字符串

```
    NSRange result = [constantString rangeOfString:@"the" options:NSCaseInsensitiveSearch];
    if(result.location == NSNotFound) {
        NSLog(@"string not found");
    } else {
        NSLog(@"result found is from : %lu",(unsigned long)result.location);
    }
```

# NSArray

官方文档：[https://developer.apple.com/reference/foundation/nsarray](https://developer.apple.com/reference/foundation/nsarray) 
 1）定义一个不可变数组

```
    NSArray* myArray = @[@"one",@"two",@"three",[NSNull null]];
    NSString* oneString = myArray[0];
    NSString* nullString = myArray[3];
    NSLog(@"null string: %@",nullString);

    //数组元素的索引
    int index = [myArray indexOfObject:@"two"];
    if(index == NSNotFound) {
        //do something
    }
```

2）创建一个子字符串

```
    NSRange subArrayRange = NSMakeRange(1, 2);
    NSArray* subArray = [myArray subarrayWithRange:subArrayRange];
```

3)快速枚举

```
    for(NSString* string in myArray) {
        //do something
    }
```

4)可变数组

```
    NSMutableArray* mutableArray = [NSMutableArray arrayWithArray:@[@"linc",@"lily"]];
    [mutableArray addObject:@"lucy"];
    [mutableArray insertObject:@"lee" atIndex:2];

    [mutableArray removeObject:@"lily"];
    [mutableArray removeObjectAtIndex:1];
    [mutableArray replaceObjectAtIndex:1 withObject:@"linda"];
```

# NSDictionary

官方文档：[https://developer.apple.com/reference/foundation/nsdictionary?language=objc](https://developer.apple.com/reference/foundation/nsdictionary?language=objc) 
 字典就是存储键值对的容器，与Java中的Map类似。 
 1）创建一个字典

```
    NSDictionary* dictionary = @{@"name":@"linc",
                                 @"age":@"18"};
    NSString* name = dictionary[@"name"];
    NSLog(@"name is %@",name);
```

2)NSValue和NSNumber 
 NSArray和NSDictionary中只能包含OC对象，那么其他非OC对象的值该怎么活呢？ 
 这就引入了NSValue，它允许你存储大量非对象的类型。 
 而NSNumber就是其子类，专门存储数字。用法如下。

```
    NSNumber* number = @124;
    int value = [number intValue];
    NSNumber* number2 = @(value+20);
```

参考： 
 0、《Coaco入门–使用Objective-C》 
 1、[http://www.cnblogs.com/kenshincui/p/3885689.html](http://www.cnblogs.com/kenshincui/p/3885689.html) 
 2、[https://developer.apple.com/reference/foundation](https://developer.apple.com/reference/foundation)