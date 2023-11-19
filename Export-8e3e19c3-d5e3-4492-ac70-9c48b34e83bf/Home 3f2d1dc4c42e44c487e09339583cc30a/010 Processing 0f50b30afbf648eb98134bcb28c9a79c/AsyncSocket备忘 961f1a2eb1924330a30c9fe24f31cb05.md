# AsyncSocket备忘

[Reference_GCDAsyncSocket · robbiehanson/CocoaAsyncSocket Wiki](https://github.com/robbiehanson/CocoaAsyncSocket/wiki/Reference_GCDAsyncSocket#reference)

### Tag：

不产生实际作用，主要是方便在回调时候进行请求数据批次的判断

```objectivec
- (void)socket:(GCDAsyncSocket *)sock didWriteDataWithTag:(long)tag
{
    if (tag == 1)
        NSLog(@"First request sent");
    else if (tag == 2)
        NSLog(@"Second request sent");
}
```

又例如：

```objectivec
#define TAG_WELCOME 10
#define TAG_CAPABILITIES 11
#define TAG_MSG 12

- (void)socket:(GCDAsyncSocket *)sender didReadData:(NSData *)data withTag:(long)tag
{
    if (tag == TAG_WELCOME)
    {
        // Ignore welcome message
    }
    else if (tag == TAG_CAPABILITIES)
    {
        [self processCapabilities:data];
    }
    else if (tag == TAG_MSG)
    {
        [self processMessage:data];
    }
}
```

### Tag在读取中的应用

可以利用tag来标注当前数据的类型，这样可以安排下一段读取的长度或位置

In addition to this you've probably noticed the tag parameter. The tag you pass during the read/write operation is passed back to you via the delegate method once the read/write operation completes.

```objectivec
- (void)socket:(GCDAsyncSocket *)sender didReadData:(NSData *)data withTag:(long)tag
{
    if (tag == TAG_FIXED_LENGTH_HEADER)
    {
        int bodyLength = [self parseHeader:data];
        [socket readDataToLength:bodyLength withTimeout:-1 tag:TAG_RESPONSE_BODY];
    }
    else if (tag == TAG_RESPONSE_BODY)
    {
        // Process the response
        [self handleResponseBody:data];

        // Start reading the next response
        [socket readDataToLength:headerLength withTimeout:-1 tag:TAG_FIXED_LENGTH_HEADER];
    }
}
```

### TLS