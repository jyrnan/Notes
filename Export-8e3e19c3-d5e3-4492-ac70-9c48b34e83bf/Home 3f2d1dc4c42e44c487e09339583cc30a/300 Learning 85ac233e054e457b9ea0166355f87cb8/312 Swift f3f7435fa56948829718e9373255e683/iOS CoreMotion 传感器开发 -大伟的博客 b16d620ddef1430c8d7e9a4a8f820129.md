# iOS CoreMotion 传感器开发 -大伟的博客

> CoreMotion.framework集中了iOS设备大多数传感器的API接口，这些传感器包括陀螺仪、加速度计、磁力计等。这些传感器的值可以反映手机设备的空间姿态及运动状态。
> 

### 传感器的坐标系统

使用网络上的一张图片：

![Untitled](iOS%20CoreMotion%20%E4%BC%A0%E6%84%9F%E5%99%A8%E5%BC%80%E5%8F%91%20-%E5%A4%A7%E4%BC%9F%E7%9A%84%E5%8D%9A%E5%AE%A2%20b16d620ddef1430c8d7e9a4a8f820129/Untitled.png)

所有传感器的原始数据都是一个以手机作为参考系的三维向量：

- X轴

手机正放时，屏幕右侧方向为X轴正方向，左侧方向为X轴负方向；

- Y轴

手机屏幕顶部向上方向记为Y轴正方向，屏幕底部向下方向为Y轴负方向；

- Z轴

手机正放时，垂直屏幕向上方向为Z轴正方向，向下为Z轴负方向。

### 获取传感器的原始值

通过CMMotionManager类获取，陀螺仪、加速度计、磁力计的获取方式类似，以陀螺仪为例：

```objectivec
self.operationQueue = [[NSOperationQueue alloc]init];
self.motionManager = [[CMMotionManager alloc]init];

if (self.motionManager.isGyroAvailable) {
    //方式1
    [self.motionManager startGyroUpdates];
    //Returns the latest sample of gyro data
self.motionManager.gyroData

    //方式2
self.motionManager.gyroUpdateInterval = 0.1;
    [self.motionManager startGyroUpdatesToQueue:self.operationQueue withHandler:^(CMGyroData * _Nullable gyroData, NSError * _Nullable error) {
if (error)return;

        CMRotationRate rate = gyroData.rotationRate;
        //rate.x, rate.y,rate.z,gyroData.timestamp
    }];
}
```

三个传感器返回数据的具体含义：

- 磁力计：CMMagneticField

表示设备在各轴方向上检测到的磁场强度，以微特斯拉为单位。

- 陀螺仪：CMRotationRate

表示设备围绕各轴的转动速度，单位是弧度每秒。

- 加速计：CMAcceleration

设备在各轴方向上检测到的加速度值（包含重力加速度的影响），以重力加速度g为单位。

你也许已经发现了，磁力计可以判定手机的朝向；陀螺仪可以判定手机的转动；加速度可以判定手机的运动状态。这几个传感器的值结合起来，再配合GPS信息，不就可以计算出手机的空间姿态和运动状态了吗？实际上，当你开始处理这些传感器的数据时就会发现没那么简单。主要有下面几个影响因素需要考虑：

- 磁力计受所处的地磁场以及周围设备的磁场影响；
- 加速计的结果需要去除重力加速度的影响；
- 三个传感器都会在X、Y、Z轴产生分量，且X、Y、Z是相对手机的，手机参考系很容易变化；

将这些影响因素带入进来，计算就很麻烦。好在Apple封装了CMDeviceMotion，将传感器检测到的原始值经过滤波和修正之后返回更加直观的结果供开发者使用。

### CMDeviceMotion

CMDeviceMotion数据的获取也是使用类似的API：

```objectivec
if (self.motionManager.isDeviceMotionAvailable) {
self.motionManager.deviceMotionUpdateInterval = 0.1;
    [self.motionManager startDeviceMotionUpdatesUsingReferenceFrame:CMAttitudeReferenceFrameXArbitraryZVertical toQueue:self.operationQueue withHandler:^(CMDeviceMotion * _Nullable motion, NSError * _Nullable error) {
        printf("姿态角度- %f %f %f\n", motion.attitude.pitch, motion.attitude.roll, motion.attitude.yaw);
    }];
}
```

CMDeviceMotion数据包含attitude、rotationRate、gravity、userAcceleration、magneticField信息。

- attitude

通过陀螺仪、磁力计计算的姿态信息，需要有一个姿态参考系配合。

- heading

返回[0, 360]的角度，设备相对于姿态参考系的夹角，夹角为0表示和当前参考系重合。参考系为CMAttitudeReferenceFrameXArbitraryZVertical或CMAttitudeReferenceFrameXArbitraryCorrectedZVertical时返回负值。

- rotationRate

转速速率：陀螺仪原始数据滤波之后的结果；

- gravity

重力加速度在各方向上的分量；

- userAcceleration

外力加速度在各方向上的分量。gravity和userAcceleration矢量叠加的结果就是通过加速计获取到的原始结果。

- magneticField

在未校准指南针的情况下，数据为0。打开工具中的「指南针」可以进行校准（结合GPS信息对不同地区的数据修正）操作。

### CMAttitude AND CMAttitudeReferenceFrame

CMDeviceMotion.attitude属性是CMAttitude类型，表示设备的空间姿态。CMAttitude包含pitch、roll、yaw信息（以弧度为单位）：

- pitch：以X轴为轴的转动角度。
- roll：以Y轴为轴的转动角度。
- yaw：以Z轴为轴的转动角度。

由于这三个信息都是相对的，必须有一个参考系。CMAttitudeReferenceFrame就表示这个参考系。

```
typedef NS_OPTIONS(NSUInteger, CMAttitudeReferenceFrame) API_UNAVAILABLE(tvos) {
    CMAttitudeReferenceFrameXArbitraryZVertical = 1 << 0,
    CMAttitudeReferenceFrameXArbitraryCorrectedZVertical = 1 << 1,
    CMAttitudeReferenceFrameXMagneticNorthZVertical = 1 << 2,
    CMAttitudeReferenceFrameXTrueNorthZVertical = 1 << 3
};
```

默认是CMAttitudeReferenceFrameXArbitraryZVertical，表示以水平面为参考平面，此时可以计算出pitch和roll的相对值，但yaw信息却无法计算出来，因为Z轴的转动方向正好和水平面平行，找不到一个参考点。系统采取的方式是，默认yaw值为0，也就是以最开始采集传感器数据时设备在X轴的方向为0参考。

你也许也想到了，水平面可以确定pitch和roll的参考系，那么地磁南北极不就可以确定yaw的参考系吗？实际上CMAttitudeReferenceFrameXMagneticNorthZVertical和CMAttitudeReferenceFrameXTrueNorthZVertical正是用地磁方向作为参考。实际验证中发现，CMAttitudeReferenceFrameXTrueNorthZVertical需要开启GPS定位且打开系统服务中的「指南针校准」选项。通常采用默认的参考系即可。

- 四个参考面解释
    
    `CMAttitudeReferenceFrameXArbitraryZVertical`
    
    Describes a reference frame in which the Z axis is vertical and the X axis points in an arbitrary direction in the horizontal plane.
    
    `CMAttitudeReferenceFrameXArbitraryCorrectedZVertical`
    
    Describes the same reference frame as `CMAttitudeReferenceFrameXArbitraryZVertical` except that the magnetometer, when available and calibrated, is used to improve long-term yaw accuracy. Using this constant instead of `CMAttitudeReferenceFrameXArbitraryZVertical` results in increased CPU usage.
    
    `CMAttitudeReferenceFrameXMagneticNorthZVertical`
    
    Describes a reference frame in which the Z axis is vertical and the X axis points toward magnetic north. Note that using this reference frame may require device movement to calibrate the magnetometer.
    
    `CMAttitudeReferenceFrameXTrueNorthZVertical`
    
    Describes a reference frame in which the Z axis is vertical and the X axis points toward true north. Note that using this reference frame may require device movement to calibrate the magnetometer. It also requires the location to be available in order to calculate the difference between magnetic and true north.
    

### multiplyByInverseOfAttitude

CMAttitude的multiplyByInverseOfAttitude接口可以计算相对于指定CMAttitude实例的差值，适用于用任意CMAttitude实例（通常为采集的第一个CMAttitude）作为参考系的情况。浏览自由派大伟博客的其它内容获取更多[iOS开发技术文章](https://easeapi.com/blog/tags/iOS.html)，不定期更新。

### 其他文章

[iOS启动优化之二进制重排](https://easeapi.com/blog/blog/143-ios-startup.html)[iOS CLLocationManager的弹窗问题](https://easeapi.com/blog/blog/141-cllocationmanager.html)[iOS NSURLProtocol详解及使用陷阱](https://easeapi.com/blog/blog/140-nsurlprotocol.html)[iOS URLSession Authentication Challenge及SSL Pinning](https://easeapi.com/blog/blog/137-ssl-pinning.html)[iOS Method Swizzling使用陷阱](https://easeapi.com/blog/blog/131-method-swizzling.html)

• [技术笔记](https://easeapi.com/blog/category/%E6%8A%80%E6%9C%AF%E7%AC%94%E8%AE%B0.html)  
• [iOS](https://easeapi.com/blog/tags/iOS.html)

**关于本博客**
如未特殊说明，本博客内容均为原创，转载需获得授权。如发现内容有错别字、与事实不