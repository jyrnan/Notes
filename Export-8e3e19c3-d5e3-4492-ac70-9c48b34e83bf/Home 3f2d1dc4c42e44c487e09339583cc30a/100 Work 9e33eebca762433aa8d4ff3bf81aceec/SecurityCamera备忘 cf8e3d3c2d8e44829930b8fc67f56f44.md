# SecurityCamera备忘

主画面代码逻辑在viewWillDisplay里面？

MQTT？

视频窗口**`开始视频会议`NRTC**模块？

调用视频模块产生视频视图的核心语句：`[**self** startNRTCWith:sn];` // YOE：这里开始调用视频通话模块

```objectivec
- (void)startNRTCWith:(NSString *)sn{
    NSInteger userID = [[STANDARD_USER_DEFAULT objectForKey:@"userID"] integerValue];
    if (0 == userID) {
        return;
    }
    if (self.hud) {
        [self.hud hideAnimated:NO];
    }
    self.hud = [self showLoading:self.view title:@"正在启动监控"];
    
    NRTCChannel *channel = [[NRTCChannel alloc] init];
    self.channl = channel;
    channel.channelName = sn;
    channel.channelMode = NRTCChannelModeVideo;
    channel.myUid = userID;
    channel.appKey = NRTC_KEY;
    channel.meetingMode = YES;
    channel.autoRotateRemoteVideo = NO;
    channel.acousticEchoCanceler = NRTCAcousticEchoCancelerSDKBuiltin;
    channel.meetingRole = NRTCMeetingRoleActor;

    [[NRTCManager sharedManager] setDelegate:self];
    NSError *error = [[NRTCManager sharedManager] joinChannel:channel];
    if (error) {
        if (isDebug_Test) {
            [self.view makeToast:@"加入房间失败"];
        }
    }else{
        if (isDebug_Test) {
            [self.view makeToast:@"加入房间成功"];
        }
    }
//    [[NRTCManager sharedManager] joinChannel:channel];

    self.chatType = FirstPage_RCTType_NRTC;
}
```

然后由 **`NRTCDelegate`**提供回调，实现视频View

```objectivec
- (void)onJoinedChannel:(NRTCChannel *)channel info:(NRTCChannelInfo *)info{
    NSLog(@"onJoinedChannel");
    if (!self.hud) {
        if (_isLandscape) {
            self.hud = [self showLoading:self.landscapeMonitorView title:@"正在启动监控"];
            self.hud.offset = CGPointMake(0.f, 0.f);
        }else{
            self.hud = [self showLoading:self.view title:@"正在加载"];
            self.hud.offset = CGPointMake(0.f, -30.0f);
        }

    }
    [[NRTCManager sharedManager] setAudioMute:YES];

    if (self.listArray.count) {
        NSString *sn = _tvDic[@"sn"];
        NSString *topic = [NSString stringWithFormat:@"%@%@%@", MQTT_PREFIX_TOPIC, sn, IN_CHANNEL_TOPIC];
        
        [self.session publishData:nil onTopic:topic retain:NO qos:MQTTQosLevelAtMostOnce publishHandler:^(NSError *error) {
            NSLog(@"%@", error);
            
        }];
    }
    [[ZTGCDTimerManager sharedInstance] scheduleGCDTimerWithName:@"netWork" interval:15.0f queue:nil repeats:NO option:CancelPreviousTimerAction action:^{
        dispatch_async(dispatch_get_main_queue(), ^{
            [self.hud hideAnimated:YES];
            if (!self.remoteView) {
                self.exceptionView.hidden = NO;
                [self exceptionWithType:0 tvName:self.tvDic[@"name"] adminUserName:nil];
                [[NRTCManager sharedManager] leaveChannel];
            }
        });
    }];
}
```

提供视频View的回调方法：

```objectivec
- (void)displayRemoteView:(UIView *)view forUser:(UInt64)uid{
    NSLog(@"%@ --- %llu", view, uid);
    NSString *sn = _tvDic[@"sn"];
    SInt64 remoteUid = abs([sn SC_hashCode]);
    if (uid == remoteUid) {
        self.remoteView = view; //提供了view
        if (_isLandscape) {
            CGRect bounds = CGRectMake(0, 0, SCREEN_HEIGHT, SCREEN_WIDTH);
            view.frame = bounds;
            [self.landscapeMonitorView insertSubview:self.remoteView atIndex:0];
        }else{
            view.frame = self.remoteDisplayView.bounds;
            [self.remoteDisplayView insertSubview:self.remoteView atIndex:0];
        }
        NSInteger userID = [[STANDARD_USER_DEFAULT objectForKey:@"userID"] integerValue];
        [MobClick beginEvent:monitor_duration];
        [MobClick event:monitor_success attributes:@{@"userID" : @(userID)}];
    }
    dispatch_async(dispatch_get_main_queue(), ^{
        [self.hud hideAnimated:YES];
        self.exceptionView.hidden = YES;
    });
    self.defaultView.hidden = YES;
    self.monitorView.hidden = NO;
    [STANDARD_USER_DEFAULT setObject:sn forKey:@"currentTVsn"];
}
```