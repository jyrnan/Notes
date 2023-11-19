# iOS利用VideoToolbox实现视频硬解码 - 简书

[https://www.jianshu.com/p/18ace93bb2ff](https://www.jianshu.com/p/18ace93bb2ff)

### 需求

本文主要将含有编码的H.264,H.265视频流文件解码为原始视频数据,解码后即可渲染到屏幕或用作其他用途.

### 实现原理

正如我们所知,编码数据仅用于传输,无法直接渲染到屏幕上,所以这里利用苹果原生框架VideoToolbox解析文件中的编码的视频流,并将压缩视频数据(h264/h265)解码为指定格式(yuv,RGB)的视频原始数据,以渲染到屏幕上.

注意: 本例主要为解码,需要借助FFmpeg搭建模块,视频解析模块,渲染模块,这些模块在下面阅读前提皆有链接可直接访问.

### 阅读前提

- 音视频基础
- [iOS FFmpeg环境搭建](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5ceff73df265da1bb13f16f4)
- [FFmpeg解析视频数据](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5cffac756fb9a07f08708d20)
- [OpenGL渲染视频数据](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5d08f861518825699040ecac)
- [H.264,H.265码流结构](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5ce9f36bf265da1bbd4b5084)

### 代码地址 : [Video Decoder](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FXiaoDongXie1024%2FXDXVideoDecoder)

### 掘金地址 : [Video Decoder](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5d1243b75188257b791527d5)

### 简书地址 : [Video Decoder](https://www.jianshu.com/p/18ace93bb2ff)

### 博客地址 : [Video Decoder](https://links.jianshu.com/go?to=https%3A%2F%2Fxiaodongxie1024.github.io%2F2019%2F06%2F25%2F20190625_video_decoder_videotoobox%2F)

### 总体架构

总体思想即将FFmpeg parse到的数据装到`CMBlockBuffer`中,将extra data分离出的vps,sps,pps装到`CMVideoFormatDesc`中,将计算好的时间戳装到`CMTime`中,最后即可拼成完成的`CMSampleBuffer`以用来提供给解码器.

CMSampleBufferCreate

### 简易流程

FFmpeg parse流程

- 创建format context: `avformat_alloc_context`
- 打开文件流: `avformat_open_input`
- 寻找流信息: `avformat_find_stream_info`
- 获取音视频流的索引值: `formatContext->streams[i]->codecpar->codec_type == (isVideoStream ? AVMEDIA_TYPE_VIDEO : AVMEDIA_TYPE_AUDIO)`
- 获取音视频流: `m_formatContext->streams[m_audioStreamIndex]`
- 解析音视频数据帧: `av_read_frame`
- 获取extra data: `av_bitstream_filter_filter`

VideoToolbox decode流程

- 比较上一次的extra data,如果数据更新需要重新创建解码器
- 分离并保存FFmpeg parse到的extra data中分离vps, sps, pps等关键信息 (比较NALU头)
- 通过`CMVideoFormatDescriptionCreateFromH264ParameterSets`,`CMVideoFormatDescriptionCreateFromHEVCParameterSets`装载vps,sps,pps等NALU header信息.
- 指定解码器回调函数与解码后视频数据类型(yuv,RGB...)
- 创建解码器`VTDecompressionSessionCreate`
- 生成`CMBlockBufferRef`装载解码前数据,再将其转为`CMSampleBufferRef`以提供给解码器.
- 开始解码`VTDecompressionSessionDecodeFrame`
- 在回调函数中`CVImageBufferRef`即为解码后的数据,可转为`CMSampleBufferRef`传出.

### 文件结构

image

### 快速使用

- 
    
    初始化preview
    
    解码后的视频数据将渲染到该预览层
    

```
- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupUI];
}

- (void)setupUI {
    self.previewView = [[XDXPreviewView alloc] initWithFrame:self.view.frame];
    [self.view addSubview:self.previewView];
    [self.view bringSubviewToFront:self.startBtn];
}

```

- 解析并解码文件中视频数据

```
- (void)startDecodeByVTSessionWithIsH265Data:(BOOL)isH265 {
    NSString *path = [[NSBundle mainBundle] pathForResource:isH265 ? @"testh265" : @"testh264"  ofType:@"MOV"];
    XDXAVParseHandler *parseHandler = [[XDXAVParseHandler alloc] initWithPath:path];
    XDXVideoDecoder *decoder = [[XDXVideoDecoder alloc] init];
    decoder.delegate = self;
    [parseHandler startParseWithCompletionHandler:^(BOOL isVideoFrame, BOOL isFinish, struct XDXParseVideoDataInfo *videoInfo, struct XDXParseAudioDataInfo *audioInfo) {
        if (isFinish) {
            [decoder stopDecoder];
            return;
        }

        if (isVideoFrame) {
            [decoder startDecodeVideoData:videoInfo];
        }
    }];
}

```

- 将解码后数据渲染到屏幕上

> 
> 
> 
> 注意: 如果数据中含有B帧则需要做一个重排序才能渲染,本例提供两个文件,一个不含B帧的h264类型文件,一个含B帧的h265类型文件.
> 

```
- (void)getVideoDecodeDataCallback:(CMSampleBufferRef)sampleBuffer {
    if (self.isH265File) {
        // Note : the first frame not need to sort.
        if (self.isDecodeFirstFrame) {
            self.isDecodeFirstFrame = NO;
            CVPixelBufferRef pix = CMSampleBufferGetImageBuffer(sampleBuffer);
            [self.previewView displayPixelBuffer:pix];
        }

        XDXSortFrameHandler *sortHandler = [[XDXSortFrameHandler alloc] init];
        sortHandler.delegate = self;
        [sortHandler addDataToLinkList:sampleBuffer];
    }else {
        CVPixelBufferRef pix = CMSampleBufferGetImageBuffer(sampleBuffer);
        [self.previewView displayPixelBuffer:pix];
    }
}

- (void)getSortedVideoNode:(CMSampleBufferRef)sampleBuffer {
    int64_t pts = (int64_t)(CMTimeGetSeconds(CMSampleBufferGetPresentationTimeStamp(sampleBuffer)) * 1000);
    static int64_t lastpts = 0;
    NSLog(@"Test marigin - %lld",pts - lastpts);
    lastpts = pts;

    [self.previewView displayPixelBuffer:CMSampleBufferGetImageBuffer(sampleBuffer)];
}

```

### 具体实现

### 1. 从Parse到的数据中检测是否需要更新extra data.

使用FFmpeg parse的数据装在`XDXParseVideoDataInfo`结构体中,结构体定义如下,parse模块可在上文链接中学习,本节只将解码模块.

```
struct XDXParseVideoDataInfo {
    uint8_t                 *data;
    int                     dataSize;
    uint8_t                 *extraData;
    int                     extraDataSize;
    Float64                 pts;
    Float64                 time_base;
    int                     videoRotate;
    int                     fps;
    CMSampleTimingInfo      timingInfo;
    XDXVideoEncodeFormat    videoFormat;
};

```

通过缓存当前extra data可以将当前获取的extra data与上一次的进行对比,如果改变需要重新创建解码器,如果没有改变则解码器可复用.(此代码尤其适用于网络流中的视频流,因为视频流可能会改变)

```
        uint8_t *extraData = videoInfo->extraData;
        int     size       = videoInfo->extraDataSize;

        BOOL isNeedUpdate = [self isNeedUpdateExtraDataWithNewExtraData:extraData
                                                                newSize:size
                                                               lastData:&_lastExtraData
                                                               lastSize:&_lastExtraDataSize];

......

- (BOOL)isNeedUpdateExtraDataWithNewExtraData:(uint8_t *)newData newSize:(int)newSize lastData:(uint8_t **)lastData lastSize:(int *)lastSize {
    BOOL isNeedUpdate = NO;
    if (*lastSize == 0) {
        isNeedUpdate = YES;
    }else {
        if (*lastSize != newSize) {
            isNeedUpdate = YES;
        }else {
            if (memcmp(newData, *lastData, newSize) != 0) {
                isNeedUpdate = YES;
            }
        }
    }

    if (isNeedUpdate) {
        [self destoryDecoder];

        *lastData = (uint8_t *)malloc(newSize);
        memcpy(*lastData, newData, newSize);
        *lastSize = newSize;
    }

    return isNeedUpdate;
}

```

### 2. 从extra data中分离关键信息(h265:vps),sps,pps.

创建解码器必须要有NALU Header中的一些关键信息,如vps,sps,pps,以用来组成一个`CMVideoFormatDesc`描述视频信息的数据结构,如上图

> 
> 
> 
> 注意: h264码流需要sps,pps, h265码流则需要vps,sps,pps
> 
- 分离NALU Header

首先确定start code的位置,通过比较前四个字节是否为`00 00 00 01`即可. 对于h264的数据,start code之后紧接着的是sps,pps, 对于h265的数据则是vps,sps,pps

- 确定NALU Header长度

通过sps索引与pps索引值可以确定sps长度,其他类似,注意,码流结构中均以4个字节的start code作为分界符,所以需要减去对应长度.

- 分离NALU Header数据

对于h264类型数据将数据`&`上`0x1F`可以确定NALU header的类型,对于h265类型数据,将数据`&`上`0x4F`可以确定NALU header的类型,这源于h264,h265的码流结构,如果不懂请参考文章最上方阅读前提中码流结构相关文章.

得到对应类型的数据与大小后,将其赋给全局变量,即可供后面使用.

```
        if (isNeedUpdate) {
            log4cplus_error(kModuleName, "%s: update extra data",__func__);

            [self getNALUInfoWithVideoFormat:videoInfo->videoFormat
                                   extraData:extraData
                               extraDataSize:size
                                 decoderInfo:&_decoderInfo];
        }

......

- (void)getNALUInfoWithVideoFormat:(XDXVideoEncodeFormat)videoFormat extraData:(uint8_t *)extraData extraDataSize:(int)extraDataSize decoderInfo:(XDXDecoderInfo *)decoderInfo {

    uint8_t *data = extraData;
    int      size = extraDataSize;

    int startCodeVPSIndex  = 0;
    int startCodeSPSIndex  = 0;
    int startCodeFPPSIndex = 0;
    int startCodeRPPSIndex = 0;
    int nalu_type = 0;

    for (int i = 0; i < size; i ++) {
        if (i >= 3) {
            if (data[i] == 0x01 && data[i - 1] == 0x00 && data[i - 2] == 0x00 && data[i - 3] == 0x00) {
                if (videoFormat == XDXH264EncodeFormat) {
                    if (startCodeSPSIndex == 0) {
                        startCodeSPSIndex = i;
                    }
                    if (i > startCodeSPSIndex) {
                        startCodeFPPSIndex = i;
                    }
                }else if (videoFormat == XDXH265EncodeFormat) {
                    if (startCodeVPSIndex == 0) {
                        startCodeVPSIndex = i;
                        continue;
                    }
                    if (i > startCodeVPSIndex && startCodeSPSIndex == 0) {
                        startCodeSPSIndex = i;
                        continue;
                    }
                    if (i > startCodeSPSIndex && startCodeFPPSIndex == 0) {
                        startCodeFPPSIndex = i;
                        continue;
                    }
                    if (i > startCodeFPPSIndex && startCodeRPPSIndex == 0) {
                        startCodeRPPSIndex = i;
                    }
                }
            }
        }
    }

    int spsSize = startCodeFPPSIndex - startCodeSPSIndex - 4;
    decoderInfo->sps_size = spsSize;

    if (videoFormat == XDXH264EncodeFormat) {
        int f_ppsSize = size - (startCodeFPPSIndex + 1);
        decoderInfo->f_pps_size = f_ppsSize;

        nalu_type = ((uint8_t)data[startCodeSPSIndex + 1] & 0x1F);
        if (nalu_type == 0x07) {
            uint8_t *sps = &data[startCodeSPSIndex + 1];
            [self copyDataWithOriginDataRef:&decoderInfo->sps newData:sps size:spsSize];
        }

        nalu_type = ((uint8_t)data[startCodeFPPSIndex + 1] & 0x1F);
        if (nalu_type == 0x08) {
            uint8_t *pps = &data[startCodeFPPSIndex + 1];
            [self copyDataWithOriginDataRef:&decoderInfo->f_pps newData:pps size:f_ppsSize];
        }
    } else {
        int vpsSize = startCodeSPSIndex - startCodeVPSIndex - 4;
        decoderInfo->vps_size = vpsSize;

        int f_ppsSize = startCodeRPPSIndex - startCodeFPPSIndex - 4;
        decoderInfo->f_pps_size = f_ppsSize;

        nalu_type = ((uint8_t) data[startCodeVPSIndex + 1] & 0x4F);
        if (nalu_type == 0x40) {
            uint8_t *vps = &data[startCodeVPSIndex + 1];
            [self copyDataWithOriginDataRef:&decoderInfo->vps newData:vps size:vpsSize];
        }

        nalu_type = ((uint8_t) data[startCodeSPSIndex + 1] & 0x4F);
        if (nalu_type == 0x42) {
            uint8_t *sps = &data[startCodeSPSIndex + 1];
            [self copyDataWithOriginDataRef:&decoderInfo->sps newData:sps size:spsSize];
        }

        nalu_type = ((uint8_t) data[startCodeFPPSIndex + 1] & 0x4F);
        if (nalu_type == 0x44) {
            uint8_t *pps = &data[startCodeFPPSIndex + 1];
            [self copyDataWithOriginDataRef:&decoderInfo->f_pps newData:pps size:f_ppsSize];
        }

        if (startCodeRPPSIndex == 0) {
            return;
        }

        int r_ppsSize = size - (startCodeRPPSIndex + 1);
        decoderInfo->r_pps_size = r_ppsSize;

        nalu_type = ((uint8_t) data[startCodeRPPSIndex + 1] & 0x4F);
        if (nalu_type == 0x44) {
            uint8_t *pps = &data[startCodeRPPSIndex + 1];
            [self copyDataWithOriginDataRef:&decoderInfo->r_pps newData:pps size:r_ppsSize];
        }
    }
}

- (void)copyDataWithOriginDataRef:(uint8_t **)originDataRef newData:(uint8_t *)newData size:(int)size {
    if (*originDataRef) {
        free(*originDataRef);
        *originDataRef = NULL;
    }
    *originDataRef = (uint8_t *)malloc(size);
    memcpy(*originDataRef, newData, size);
}

```

### 3. 创建解码器

根据编码数据类型确定使用h264解码器还是h265解码器,如上图我们可得知,我们需要将数据拼成一个CMSampleBuffer类型以传给解码器解码.

- 生成 `CMVideoFormatDescriptionRef`

通过(vps)sps,pps信息组成`CMVideoFormatDescriptionRef`. 这里需要注意的是, h265编码数据有的码流数据中含有两个pps, 所以在拼装时需要判断以确定参数数量.

- 确定视频数据类型

通过指定`kCVPixelFormatType_420YpCbCr8BiPlanarFullRange`将视频数据类型设置为yuv 420sp, 如需其他格式可自行更改适配.

- 指定回调函数
- 创建编码器

通过上面提供的所有信息,即可调用`VTDecompressionSessionCreate`生成解码器上下文对象.

```
    // create decoder
    if (!_decoderSession) {
        _decoderSession = [self createDecoderWithVideoInfo:videoInfo
                                              videoDescRef:&_decoderFormatDescription
                                               videoFormat:kCVPixelFormatType_420YpCbCr8BiPlanarFullRange
                                                      lock:_decoder_lock
                                                  callback:VideoDecoderCallback
                                               decoderInfo:_decoderInfo];
    }

- (VTDecompressionSessionRef)createDecoderWithVideoInfo:(XDXParseVideoDataInfo *)videoInfo videoDescRef:(CMVideoFormatDescriptionRef *)videoDescRef videoFormat:(OSType)videoFormat lock:(pthread_mutex_t)lock callback:(VTDecompressionOutputCallback)callback decoderInfo:(XDXDecoderInfo)decoderInfo {
    pthread_mutex_lock(&lock);

    OSStatus status;
    if (videoInfo->videoFormat == XDXH264EncodeFormat) {
        const uint8_t *const parameterSetPointers[2] = {decoderInfo.sps, decoderInfo.f_pps};
        const size_t parameterSetSizes[2] = {static_cast<size_t>(decoderInfo.sps_size), static_cast<size_t>(decoderInfo.f_pps_size)};
        status = CMVideoFormatDescriptionCreateFromH264ParameterSets(kCFAllocatorDefault,
                                                                     2,
                                                                     parameterSetPointers,
                                                                     parameterSetSizes,
                                                                     4,
                                                                     videoDescRef);
    }else if (videoInfo->videoFormat == XDXH265EncodeFormat) {
        if (decoderInfo.r_pps_size == 0) {
            const uint8_t *const parameterSetPointers[3] = {decoderInfo.vps, decoderInfo.sps, decoderInfo.f_pps};
            const size_t parameterSetSizes[3] = {static_cast<size_t>(decoderInfo.vps_size), static_cast<size_t>(decoderInfo.sps_size), static_cast<size_t>(decoderInfo.f_pps_size)};
            if (@available(iOS 11.0, *)) {
                status = CMVideoFormatDescriptionCreateFromHEVCParameterSets(kCFAllocatorDefault,
                                                                             3,
                                                                             parameterSetPointers,
                                                                             parameterSetSizes,
                                                                             4,
                                                                             NULL,
                                                                             videoDescRef);
            } else {
                status = -1;
                log4cplus_error(kModuleName, "%s: System version is too low!",__func__);
            }
        } else {
            const uint8_t *const parameterSetPointers[4] = {decoderInfo.vps, decoderInfo.sps, decoderInfo.f_pps, decoderInfo.r_pps};
            const size_t parameterSetSizes[4] = {static_cast<size_t>(decoderInfo.vps_size), static_cast<size_t>(decoderInfo.sps_size), static_cast<size_t>(decoderInfo.f_pps_size), static_cast<size_t>(decoderInfo.r_pps_size)};
            if (@available(iOS 11.0, *)) {
                status = CMVideoFormatDescriptionCreateFromHEVCParameterSets(kCFAllocatorDefault,
                                                                             4,
                                                                             parameterSetPointers,
                                                                             parameterSetSizes,
                                                                             4,
                                                                             NULL,
                                                                             videoDescRef);
            } else {
                status = -1;
                log4cplus_error(kModuleName, "%s: System version is too low!",__func__);
            }
        }
    }else {
        status = -1;
    }

    if (status != noErr) {
        log4cplus_error(kModuleName, "%s: NALU header error !",__func__);
        pthread_mutex_unlock(&lock);
        [self destoryDecoder];
        return NULL;
    }

    uint32_t pixelFormatType = videoFormat;
    const void *keys[]       = {kCVPixelBufferPixelFormatTypeKey};
    const void *values[]     = {CFNumberCreate(NULL, kCFNumberSInt32Type, &pixelFormatType)};
    CFDictionaryRef attrs    = CFDictionaryCreate(NULL, keys, values, 1, NULL, NULL);

    VTDecompressionOutputCallbackRecord callBackRecord;
    callBackRecord.decompressionOutputCallback = callback;
    callBackRecord.decompressionOutputRefCon   = (__bridge void *)self;

    VTDecompressionSessionRef session;
    status = VTDecompressionSessionCreate(kCFAllocatorDefault,
                                          *videoDescRef,
                                          NULL,
                                          attrs,
                                          &callBackRecord,
                                          &session);

    CFRelease(attrs);
    pthread_mutex_unlock(&lock);
    if (status != noErr) {
        log4cplus_error(kModuleName, "%s: Create decoder failed",__func__);
        [self destoryDecoder];
        return NULL;
    }

    return session;
}

```

### 4. 开始解码

- 将parse出来的原始数据装在`XDXDecodeVideoInfo`结构体中,以便后续扩展使用.

```
typedef struct {
    CVPixelBufferRef outputPixelbuffer;
    int              rotate;
    Float64          pts;
    int              fps;
    int              source_index;
} XDXDecodeVideoInfo;

```

- 将编码数据装在`CMBlockBufferRef`中.
- 通过`CMBlockBufferRef`生成`CMSampleBufferRef`
- 解码数据

通过`VTDecompressionSessionDecodeFrame`函数即可完成解码一帧视频数据.第三个参数可以指定解码采用同步或异步方式.

```

    // start decode
    [self startDecode:videoInfo
              session:_decoderSession
                 lock:_decoder_lock];

......

- (void)startDecode:(XDXParseVideoDataInfo *)videoInfo session:(VTDecompressionSessionRef)session lock:(pthread_mutex_t)lock {
    pthread_mutex_lock(&lock);
    uint8_t *data  = videoInfo->data;
    int     size   = videoInfo->dataSize;
    int     rotate = videoInfo->videoRotate;
    CMSampleTimingInfo timingInfo = videoInfo->timingInfo;

    uint8_t *tempData = (uint8_t *)malloc(size);
    memcpy(tempData, data, size);

    XDXDecodeVideoInfo *sourceRef = (XDXDecodeVideoInfo *)malloc(sizeof(XDXParseVideoDataInfo));
    sourceRef->outputPixelbuffer  = NULL;
    sourceRef->rotate             = rotate;
    sourceRef->pts                = videoInfo->pts;
    sourceRef->fps                = videoInfo->fps;

    CMBlockBufferRef blockBuffer;
    OSStatus status = CMBlockBufferCreateWithMemoryBlock(kCFAllocatorDefault,
                                                         (void *)tempData,
                                                         size,
                                                         kCFAllocatorNull,
                                                         NULL,
                                                         0,
                                                         size,
                                                         0,
                                                         &blockBuffer);

    if (status == kCMBlockBufferNoErr) {
        CMSampleBufferRef sampleBuffer = NULL;
        const size_t sampleSizeArray[] = { static_cast<size_t>(size) };

        status = CMSampleBufferCreateReady(kCFAllocatorDefault,
                                           blockBuffer,
                                           _decoderFormatDescription,
                                           1,
                                           1,
                                           &timingInfo,
                                           1,
                                           sampleSizeArray,
                                           &sampleBuffer);

        if (status == kCMBlockBufferNoErr && sampleBuffer) {
            VTDecodeFrameFlags flags   = kVTDecodeFrame_EnableAsynchronousDecompression;
            VTDecodeInfoFlags  flagOut = 0;
            OSStatus decodeStatus      = VTDecompressionSessionDecodeFrame(session,
                                                                           sampleBuffer,
                                                                           flags,
                                                                           sourceRef,
                                                                           &flagOut);
            if(decodeStatus == kVTInvalidSessionErr) {
                pthread_mutex_unlock(&lock);
                [self destoryDecoder];
                if (blockBuffer)
                    CFRelease(blockBuffer);
                free(tempData);
                tempData = NULL;
                CFRelease(sampleBuffer);
                return;
            }
            CFRelease(sampleBuffer);
        }
    }

    if (blockBuffer) {
        CFRelease(blockBuffer);
    }

    free(tempData);
    tempData = NULL;
    pthread_mutex_unlock(&lock);
}

```

### 5. 解码后的数据

解码后的数据可在回调函数中获取.这里需要将解码后的数据`CVImageBufferRef`转为`CMSampleBufferRef`.然后通过代理传出.

```
#pragma mark - Callback
static void VideoDecoderCallback(void *decompressionOutputRefCon, void *sourceFrameRefCon, OSStatus status, VTDecodeInfoFlags infoFlags, CVImageBufferRef pixelBuffer, CMTime presentationTimeStamp, CMTime presentationDuration) {
    XDXDecodeVideoInfo *sourceRef = (XDXDecodeVideoInfo *)sourceFrameRefCon;

    if (pixelBuffer == NULL) {
        log4cplus_error(kModuleName, "%s: pixelbuffer is NULL status = %d",__func__,status);
        if (sourceRef) {
            free(sourceRef);
        }
        return;
    }

    XDXVideoDecoder *decoder = (__bridge XDXVideoDecoder *)decompressionOutputRefCon;

    CMSampleTimingInfo sampleTime = {
        .presentationTimeStamp  = presentationTimeStamp,
        .decodeTimeStamp        = presentationTimeStamp
    };

    CMSampleBufferRef samplebuffer = [decoder createSampleBufferFromPixelbuffer:pixelBuffer
                                                                    videoRotate:sourceRef->rotate
                                                                     timingInfo:sampleTime];

    if (samplebuffer) {
        if ([decoder.delegate respondsToSelector:@selector(getVideoDecodeDataCallback:)]) {
            [decoder.delegate getVideoDecodeDataCallback:samplebuffer];
        }
        CFRelease(samplebuffer);
    }

    if (sourceRef) {
        free(sourceRef);
    }
}

- (CMSampleBufferRef)createSampleBufferFromPixelbuffer:(CVImageBufferRef)pixelBuffer videoRotate:(int)videoRotate timingInfo:(CMSampleTimingInfo)timingInfo {
    if (!pixelBuffer) {
        return NULL;
    }

    CVPixelBufferRef final_pixelbuffer = pixelBuffer;
    CMSampleBufferRef samplebuffer = NULL;
    CMVideoFormatDescriptionRef videoInfo = NULL;
    OSStatus status = CMVideoFormatDescriptionCreateForImageBuffer(kCFAllocatorDefault, final_pixelbuffer, &videoInfo);
    status = CMSampleBufferCreateForImageBuffer(kCFAllocatorDefault, final_pixelbuffer, true, NULL, NULL, videoInfo, &timingInfo, &samplebuffer);

    if (videoInfo != NULL) {
        CFRelease(videoInfo);
    }

    if (samplebuffer == NULL || status != noErr) {
        return NULL;
    }

    return samplebuffer;
}

```

### 6.销毁解码器

用完后记得销毁,以便下次使用.

```
    if (_decoderSession) {
        VTDecompressionSessionWaitForAsynchronousFrames(_decoderSession);
        VTDecompressionSessionInvalidate(_decoderSession);
        CFRelease(_decoderSession);
        _decoderSession = NULL;
    }

    if (_decoderFormatDescription) {
        CFRelease(_decoderFormatDescription);
        _decoderFormatDescription = NULL;
    }

```

### 7. 补充:关于带B帧数据重排序问题

注意,如果视频文件或视频流中含有B帧,则渲染时需要对视频帧做一个重排序,本文重点讲解码,排序将在后面文章中更新,代码中以实现,如需了解请下载Demo.

最后编辑于 ：2019.06.25 08:57:33

©著作权归作者所有,转载或内容合作请联系作者

25人点赞

[Video](https://www.jianshu.com/nb/38610343)

[小东邪啊](https://www.jianshu.com/u/23f3ec991fed)一生负气成今日，四海无人对夕阳

总资产35共写了11.8W字获得507个赞共610个粉丝

- [人面猴](https://www.jianshu.com/p/1003a129be45)序言：七十年代末，一起剥皮案震惊了整个滨河市，随后出现的几起案子，更是在滨河造成了极大的恐慌，老刑警刘岩，带你破解...[沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 102,839评论 1
- [死咒](https://www.jianshu.com/p/1c4506f51019)序言：滨河连续发生了三起死亡事件，死亡现场离奇诡异，居然都是意外死亡，警方通过查阅死者的电脑和手机，发现死者居然都...[沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 41,616评论 1
- [救了他两次的神仙让他今天三更去死](https://www.jianshu.com/p/1ded57e57939)文/潘晓璐 我一进店门，熙熙楼的掌柜王于贵愁眉苦脸地迎上来，“玉大人，你说我怎么就摊上这事。” “怎么了？”我有些...[开封第一讲书人](https://www.jianshu.com/u/5891e866c93e)阅读 55,397评论 0
- [道士缉凶录：失踪的卖姜人](https://www.jianshu.com/p/25685c1b1f2b) 文/不坏的土叔 我叫张陵，是天一观的道长。 经常有香客问我，道长，这世上最难降的妖魔是什么？ 我笑而不...[开封第一讲书人](https://www.jianshu.com/u/5891e866c93e)阅读 28,175评论 0
- [港岛之恋（遗憾婚礼）](https://www.jianshu.com/p/553802eff5d6)正文 为了忘掉前任，我火速办了婚礼，结果婚礼上，老公的妹妹穿的比我还像新娘。我一直安慰自己，他们只是感情好，可当我...[茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 34,454评论 0
- [恶毒庶女顶嫁案：这布局不是一般人想出来的](https://www.jianshu.com/p/59985a89b4ef)文/花漫 我一把揭开白布。 她就那样静静地躺着，像睡着了一般。 火红的嫁衣衬着肌肤如雪。 梳的纹丝不乱的头发上，一...[开封第一讲书人](https://www.jianshu.com/u/5891e866c93e)阅读 28,402评论 1
- [城市分裂传说](https://www.jianshu.com/p/62a01de427e0)那天，我揣着相机与录音，去河边找鬼。 笑死，一个胖子当着我的面吹牛，可吹牛的内容都是我干的。 我是一名探鬼主播，决...[沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 20,640评论 2
- [双鸳鸯连环套：你想象不到人心有多黑](https://www.jianshu.com/p/6ccdc163474a)文/苍兰香墨 我猛地睁开眼，长吁一口气：“原来是场噩梦啊……” “哼！你这毒妇竟也来了？” 一声冷哼从身侧响起，我...[开封第一讲书人](https://www.jianshu.com/u/5891e866c93e)阅读 19,883评论 0
- [父亲被人害死了，幕后凶手却是我最亲的人！](https://www.jianshu.com/p/6fe2053108f7)想象着我的养父在大火中拼命挣扎，窒息，最后皮肤化为焦炭。我心中就已经是抑制不住地欢快，这就叫做以其人之道，还治其人...[爱写小说的胖达](https://www.jianshu.com/u/c70b17da4d0e)阅读 18,665评论 5
- [万荣杀人案实录](https://www.jianshu.com/p/8796e3463067)序言：老挝万荣一对情侣失踪，失踪者是张志新（化名）和其女友刘颖，没想到半个月后，有当地人在树林里发现了一具尸体，经...[沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 22,503评论 0
- [护林员之死](https://www.jianshu.com/p/8a691dd8fa34)正文 独居荒郊野岭守林人离奇死亡，尸身上长有42处带血的脓包…… 初始之章·张勋 以下内容为张勋视角 年9月15日...[茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 20,225评论 1
- [白月光启示录](https://www.jianshu.com/p/a5293fa3b5e0)正文 我和宋清朗相恋三年，在试婚纱的时候发现自己被绿了。 大学时的朋友给我发了我未婚夫和他白月光在一起吃饭的照片。...[茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 21,315评论 0
- [惨遭霸总抛弃后，我靠赚来的钱成了富豪榜第一名](https://www.jianshu.com/p/a68a12246696)白月光回国，霸总把我这个替身辞退。还一脸阴沉的警告我。[不要出现在思思面前， 不然我有一百种方法让你生不如死。]我...[爱写小说的胖达](https://www.jianshu.com/u/c70b17da4d0e)阅读 15,875评论 0赞 20
- [活死人](https://www.jianshu.com/p/a83aa7e71001)序言：一个原本活蹦乱跳的男人离奇死亡，死状恐怖，灵堂内的尸体忽然破棺而出，到底是诈尸还是另有隐情，我是刑警宁泽，带...[沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 18,597评论 2
- [日本核电站爆炸内幕](https://www.jianshu.com/p/bee7d9c3fcf9)正文 年R本政府宣布，位于F岛的核电站，受9级特大地震影响，放射性物质发生泄漏。R本人自食恶果不足惜，却给世界环境...[茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 22,117评论 3
- [男人毒药：我在死后第九天来索命](https://www.jianshu.com/p/c2cfc4cb0aa7)文/蒙蒙 一、第九天 我趴在偏房一处隐蔽的房顶上张望。 院中可真热闹，春花似锦、人声如沸。这庄子的主人今日做“春日...[开封第一讲书人](https://www.jianshu.com/u/5891e866c93e)阅读 17,148评论 0赞 4
- [一桩弑父案，背后竟有这般阴谋](https://www.jianshu.com/p/c329b54bd638)文/苍兰香墨 我抬头看了看天上的太阳。三九已至，却和暖如春，着一层夹袄步出监牢的瞬间，已是汗流浃背。 一阵脚步声响...[开封第一讲书人](https://www.jianshu.com/u/5891e866c93e)阅读 17,550评论 0赞 99
- [情欲美人皮](https://www.jianshu.com/p/d79d2f48417f)我被黑心中介骗来泰国打工， 没想到刚下飞机就差点儿被人妖公主榨干…… 1. 我叫王不留，地道东北人。 一个月前我还...[沈念sama](https://www.jianshu.com/u/dcd395522934)阅读 23,116评论 2赞 168
- [代替公主和亲](https://www.jianshu.com/p/fc890ed5083c)正文 我出身青楼，却偏偏与公主长得像，于是被迫代替她去往敌国和亲。 传闻我的和亲对象是个残疾皇子，可洞房花烛夜当晚...[茶点故事](https://www.jianshu.com/u/0f438ff0a55f)阅读 23,503评论 2赞 162