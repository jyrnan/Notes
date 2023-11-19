# KryoCocoa基本使用方法（OC版本）

### 簡介

> Kryo 是一個快速高效的Java對象圖形序列化框架，主要特點是性能、高效和易用。該項目用來序列化對象到文件、數據庫或者網絡。 http://code.google.com/p/kryo
> 

Kryo同其他序列化框架的對比（Java比較流行的序列化框架）

|  | 優點 | 缺點 |
| --- | --- | --- |
| Kryo | 速度快，序列化後體積小 | 跨語言支持較複雜 |
| Hessian | 默認支持跨語言 | 較慢 |
| Protostuff | 速度快，基於protobuf | 需靜態編譯 |

> 其實KryoCocoa就是Java中Kryo序列化框架的Objective-C版，是為了兼容Kryo和基本的Java數據類型的。 這裏是KryoCocoa的地址：https://github.com/Feuerwerk/kryococoa
> 

### 基本使用

一、將對象序列化後寫入某個文件中
 假如有以下實體類:

```objectivec
@interface ClassA : NSObject
@property (assign, nonatomic) SInt32 uid;
@property (copy, nonatomic) NSString *name;
@end

```

將該實體類序列化後寫入到某個路徑下的某個文件

```objectivec
ClassA *a = [[ClassA alloc] init];
a.uid = 1;
a.name = @"lisi";

NSOutputStream *outputStream = [NSOutputStream outputStreamToFileAtPath:filePath append:NO];
KryoOutput *output = [[KryoOutput alloc] initWithStream:outputStream];
[k writeObject:a to:output];

[output close];

```

二、讀取某個文件反序列化成一個對象

```objectivec
NSInputStream *inputStream = [NSInputStream inputStreamWithFileAtPath:filePath];
KryoInput *input = [[KryoInput alloc] initWithInput:inputStream];
ClassA *a = [k readObject:input ofClass:[ClassA class]];

[input close];
NSLog(@"%d %@", a.uid,a.name);

```

序列化對象後通過Socket與Java後台交互

假設有一個這樣的java實體類

```java
package com.test.web.socket.modules.user.svo;

public class TestSVO {

    private Long uid;

    private Integer status;

    public Long getUid() {
        return uid;
    }

    public void setUid(Long uid) {
        this.uid = uid;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

}

```

我們在Xcode中應該這樣定義

```objectivec
#import
#import "SerializationAnnotation.h"
#import "JInteger.h"
#import "JLong.h"

@interface TestSVO : NSObject
// 更多OC對應java類型可在git上面查到，不一一列舉了
@property (strong, nonatomic) JLong *uid;
@property (strong, nonatomic) JInteger *status;

@end

```

```objectivec
#import "TestSVO.h"

@implementation TestSVO

+ (NSString *)serializingAlias
{
    //這裏返回的java中的完整報名+類名，否則會出問題
    return @"com.test.web.socket.modules.user.svo.TestSVO";
}

@end

```

假設我們要將上面的對象通過socket發送給服務器

```objectivec
    Kryo *k = [Kryo new];
    // 創建一個內存輸出流 用於輸出Kryo序列化後的東西
    NSOutputStream *outputStream = [[NSOutputStream alloc] initToMemory];
    // KryoOutput這個類實際上就是NSOutputStream的子類
    KryoOutput *output = [[KryoOutput alloc] initWithStream:outputStream];

    [k writeObject:message to:output];
    // toData獲取到NSData
    NSData *contents = output.toData;

    int len = (int)contents.length;
    // 發送數據
    NSData *lengthData = [NSData dataWithBytes:&len length:sizeof(len)];
    [self.socket writeData:[self descBytesWithNSData:lengthData] withTimeout:WRITE_TIME_OUT tag:0];
    [self.socket writeData:contents withTimeout:WRITE_TIME_OUT tag:0];

    [output close];

```

這樣子，Kryo的基本使用就告一段落了。