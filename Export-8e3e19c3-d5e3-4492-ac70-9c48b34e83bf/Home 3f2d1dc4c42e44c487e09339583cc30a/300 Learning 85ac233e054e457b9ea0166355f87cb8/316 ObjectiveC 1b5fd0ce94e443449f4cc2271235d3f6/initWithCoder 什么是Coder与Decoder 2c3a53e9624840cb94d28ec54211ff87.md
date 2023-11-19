# initWithCoder: 什么是Coder与Decoder

Excerpt From
iOS编程（第4版）([elib.cc](http://elib.cc/))
[美] Christian Keur[美] Aaron Hillegass[美] Joe Conway [elib.cc](http://elib.cc/)[https://itunes.apple.com/WebObjects/MZStore.woa/wa/viewBook?id=0](https://itunes.apple.com/WebObjects/MZStore.woa/wa/viewBook?id=0)
This material may be protected by copyright.

经常看到的一个初始化方法`initWithCoder:`究竟是什么？这涉及到OC中将数据固化的过程；简言之就是利用一个编码器NSCoder，将对象的各个属性转换成可以保存到文件的形式，用属性名作为标记保存好；调用时候从文件读取，利用NSCoder来解释，然后按标记恢复给对象的同名属性。是一个互逆的过程。

# 固化

固化是由iOS SDK提供的一种保存和读取对象的机制，使用非常广泛。**当应用固化某个对象时，会将该对象的所有属性存入指定的文件。当应用解固（unarchive）某个对象时，会从指定的文件读取相应的数据，然后根据数据还原对象。**
　　为了能够固化或解固某个对象，相应对象的类必须遵守**`NSCoding协议`**，并且实现两个必需方法：`encodeWithCoder:`和`initWithCoder:`，代码如下：

```objectivec
@protocol NSCoding
　　- （void）encodeWithCoder:（NSCoder *）aCoder;
　　- （instancetype）initWithCoder:（NSCoder *）aDecoder;
@end
```

## encodeWithCoder:

先实现encodeWithCoder:，它有一个类型为NSCoder的参数，`encodeWithCoder:`方法要将所有的属性都编码至该参数。在固化过程中，**NSCoder会将对象转换为键-值对形式的数据并写入指定的文件**。

```objectivec
- （void）encodeWithCoder:（NSCoder *）aCoder {
　　[aCoder encodeObject:self.itemName forKey:@“itemName”];
　　[aCoder encodeObject:self.serialNumber forKey:@“serialNumber”];
　　[aCoder encodeObject:self.dateCreated forKey:@“dateCreated”]
　　[aCoder encodeObject:self.itemKey forKey:@“itemKey”];
　　[aCoder encodeInt:self.valueInDollars forKey:@“valueInDollars”];
　　}
```

- 在这段代码中，凡是指向对象的指针都会用encodeObject:forKey:编码，而valueInDollars是简单属性int，所以用encodeInt:forKey:编码的。请查阅NSCoder的文档了解编码的数据类型。
- 无论编码哪种类型的数据，必须有相应的键才能存入NSCoder对象。这个键是字符串，负责标识相应的属性。按照约定，**编码某个属性时要使用的键就是该属性的名称**。

### 编码的递归性

当应用需要编码某个对象时（encodeObject:forKey:中的第一个参数），会向该对象发送`encodeWithCoder:`消息。收到该消息的对象需要编码自己的属性，所以也会向这些属性发送`encodeWithCoder:`消息（见图18-1）。**因此，对象的编码过程是一个递归过程：编码中的对象会再编码其他对象。**

![编码BNRItem对象](initWithCoder%20%E4%BB%80%E4%B9%88%E6%98%AFCoder%E4%B8%8EDecoder%202c3a53e9624840cb94d28ec54211ff87/Untitled.png)

编码BNRItem对象

**为了能够编码某个对象**，**该对象的所有属性也必需遵守NSCoding协议**（除了类似于int这样基本属性外）。可以查看类参考手册，检查NSString和NSDate是否遵守NSCoding协议。”

## initWithCoder:

编码某个对象时，需要针对每个属性指定相应的键。

当应用需要根据编码后的数据重新初始化某个对象时，会向该对象发送`initWithCoder:`消息。`initWithCoder:`应该还原之前通过`encodeWithCoder:`编码的所有对象，然后将这些对象赋给相应的属性。在BNRItem.m中实现`initWithCoder:`，代码如下：

```objectivec
（instancetype）initWithCoder:（NSCoder *）aDecode{
	self = [super init];
		if （self） {
				_itemName = [aDecoder decodeObjectForKey:@“itemName”];
				_serialNumber = [aDecoder decodeObjectForKey:@“serialNumber”]
				_dateCreated = [aDecoder decodeObjectForKey:@“dateCreated”];
				_itemKey = [aDecoder decodeObjectForKey:@“itemKey”]
				_valueInDollars = [aDecoder decodeIntForKey:@“valueInDollars”];
		}
		return self;
}
```

`initWithCoder:`也有一个类型为NSCoder的参数，和`encodeWithCoder:`不同，该参数的作用是为初始化BNRItem对象提供数据。**这段代码通过向NSCoder对象发送`decodeObjectForKey:`消息重新设置相应的属性。**

<aside>
💡 NSCoder利用`decodeObjectForKey:`读取某个键值的数据，转换后赋值给ivars（instance vars）， 这个ivars的名字和键值是一致的。

</aside>