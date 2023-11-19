# 【潮汐】iOS给多工程的workspace添加CocoaPods - 爱恨的潮汐 - 简书

单个的工程添加CocoaPods时，在执行 pod install 安装需要的第三方库之后，会生成一个与工程同名的workspace，里面添加了一个Pods工程来管理第三方库，但是如果当我的项目中需要集成多个工程或framework，而每个工程又依赖其他的第三方类库时，那么此时需要将所有工程添加到同一个 workspace 中，然后重新配置 Podfile 文件。

1.打开xcode，File->New->Workspace，创建一个 workspace , 选择好存储路径确认即可。

2.打开 workspace 的工作区，在空白处右击选择 Add File to ...，向workspace中添加需要引入的工程。

3.在 workspace 的根目录下，pod init ，创建一个Podfile 文件，然后根据 workspace 中的工程重新配置 Podfile 文件。

```objectivec
workspace 'MyWorkspace.xcworkspace' //workspace文件名
# workspace的主工程路径，是相对于workspace的路径
project 'MyApp1/MyApp1.xcodeproj'

target 'MyApp1' do
  platform :ios, '8.0'
# 第一个工程的相对路径，是相对于workspace的路径
  project 'MyApp1/MyApp1.xcodeproj'
  pod 'Masonry', '~> 1.0.2'
  pod 'AFNetworking', '~> 3.1.0'
  use_frameworks!
end

target 'MyApp2' do
  platform :ios, '8.0'
# 第二个工程的相对路径，是相对于workspace的路径
  project 'MyApp2/MyApp2.xcodeproj'
  pod 'Masonry', '~> 1.0.2'
  use_frameworks!
end

```

注意：Podfile 文件中的workspace文件名，工程名及工程路径一定要与实际的目录保持一致
 4.最后在 workspace 的根目录下，执行 pod install 即可。

==============================================

### 实操案列（SDK项目BRCBTwoAccountSDK为例子）：

image.png

`Podfile文件配置如下，然后在 workspace 的根目录下，执行 pod install 即可。`

```objectivec
workspace 'BRCBTwoAccountSDK.xcworkspace'
source 'https://github.com/CocoaPods/Specs.git'

project 'BRCBTwoAccountDemo/BRCBTwoAccountDemo.xcodeproj'

# Demo工程需要
target 'BRCBTwoAccountDemo' do
   platform :ios, '8.0'
    project 'BRCBTwoAccountDemo/BRCBTwoAccountDemo.xcodeproj'
    pod 'AFNetworking', '~> 3.0'

end
# SDK工程需要
target 'BRCBTwoAccountSDK' do
   platform :ios, '8.0'
    project 'BRCBTwoAccountSDK/BRCBTwoAccountSDK.xcodeproj'
    pod 'AFNetworking', '~> 3.0'

end

```