#iOS中的视频播放(二) - 视频控制对象
[前文](url)描述了视频播放的元数据类型，本文描述和元数据类型和控制相关的类族。
先上一段代码：

```
//初始化AVAsset
self.playerAsset = [AVAsset assetWithURL:_videoUrl];

// 初始化playerItem
self.playerItem  = [AVPlayerItem playerItemWithAsset:self.playerAsset];

// 初始化Player
self.player = [AVPlayer playerWithPlayerItem:self.playerItem];
    
// 初始化playerLayer
self.playerLayer = [AVPlayerLayer playerLayerWithPlayer:self.player];
```

上述是一段播放视频的代码，通过视频资源 url 初始化视频数据容器 `AVAsset`，使用 AVAsset 初始化视频描述和控制对象 `AVPlayerItem`，然后通过 AVPlayerItem 初始化视频播放器 `AVPlayer`，最后将视频播放器绑定到具体的播放 layer 进行视频图层的渲染。本文将详细讲解这些对象类型。

###  [AVAsset](https://developer.apple.com/reference/avfoundation/avasset)
`AVAsset` 承载了媒体的视频和声音等元数据，是一个抽象的基类。由多种 `AVAssetTrack`（音频轨道、字幕轨道、视频轨道等）集合组成。
来看看 AVAsset 的初始化方法。

```
  + (instancetype)assetWithURL:(NSURL *)URL
```
当通过 url 来初始化的时候，AVAsset会初始化 `AVURLAsset` ，AVURLAsset 是 AVAsset 类族的子类，用来通过本地或者远程的 url 初始化媒体资源。

AVAsset 有如下重要属性。

```
// 媒体的时长。
@property (nonatomic, readonly) CMTime duration;
// 媒体的默认播放速度。这个值绝大部分时间为1.0。
@property (nonatomic, readonly) float preferredRate;
// 媒体的默认播放音量。这个值绝大部分时间为1.0。
@property (nonatomic, readonly) float preferredVolume;
// 媒体的旋转，缩放，平移量。 The identity transform: [ 1 0 0 1 0 0 ]。
@property (nonatomic, readonly) CGAffineTransform preferredTransform;
// 此资源中包含的所有的AVAssetTrack , AVAsset 可以通过标识符,媒体类型或媒体特征等信息找到相应的track。
@property (nonatomic, readonly) NSArray<AVAssetTrack *> *tracks;
// 包含着当前视频常见格式类型的元数据。
@property (nonatomic, readonly) NSArray<AVMetadataItem *> *commonMetadata;
// 包含当前视频所有格式类型的元数据。
@property (nonatomic, readonly) NSArray<AVMetadataItem *> *metadata;
// 包含当前视频所有可用元数据的格式类型。
@property (nonatomic, readonly) NSArray<NSString *> *availableMetadataFormats;
```

前面说过，多种 `AVAssetTrack` 组成了 AVAsset，一般的视频至少由声音轨道和画面轨道组成。AVAssetTrack 映射了元数据中的`track atom` ，重要属性如下。

```
// AVAssetTrack 通过属性trackID唯一标记了track。
@property (nonatomic, readonly) CMPersistentTrackID trackID;
// AVAssetTrack 通过属性 mediaType描述了track的媒体类型。
@property (nonatomic, readonly) NSString *mediaType;
// track的语言描述，符合ISO 639-2/T。
@property (nonatomic, readonly) NSString *languageCode;
// track内容的默认音量。
@property (nonatomic, readonly) float preferredVolume;
// track内容的默认速度。
@property (nonatomic, readonly) float nominalFrameRate;
// track的元数据。
@property (nonatomic, readonly) NSArray<AVMetadataItem *> *metadata;
// track所有可用元数据的格式类型。
@property (nonatomic, readonly) NSArray<NSString *> *availableMetadataFormats;
```

 `trackID` 是 track 的唯一映射。可以通过方法

```
- (nullable AVAssetTrack *)trackWithTrackID:(CMPersistentTrackID)trackID;
```

使用 trackID 获得唯一的 AVAssetTrack。

 `mediaType` 总共描述了8种类型的轨道，

```
AVMediaTypeVideo
AVMediaTypeAudio
AVMediaTypeText
AVMediaTypeClosedCaption
AVMediaTypeSubtitle
AVMediaTypeSubtitle
AVMediaTypeTimecode
AVMediaTypeMetadata
AVMediaTypeMuxed
```

可以通过方法

```
- (NSArray<AVAssetTrack *> *)tracksWithMediaType:(NSString *)mediaType
```

使用 mediaType 获得一组符合需求的 AVAssetTrack。

如果要查看视频的元数据，例如 Title、Creator、CreationDate、LastModifiedDate等，可以通过方法

```
- (NSArray<AVMetadataItem *> *)metadataForFormat:(NSString *)format;
```

找到指定类型的元数据。

AVAsset 和 AVAssetTrack 所有的元数据格式类型定义在 [AVMetadataFormat](https://developer.apple.com/library/mac/documentation/AVFoundation/Reference/AVFoundationMetadataKeyReference/#//apple_ref/doc/constant_group/Common_Metadata_Keys) 中，例如 AVMetadataCommonKeyTitle， AVMetadataCommonKeyCreator 等。

###  [AVPlayerItem](https://developer.apple.com/library/mac/documentation/AVFoundation/Reference/AVPlayerItem_Class/)

AVAsset 描述了视频内容的静态内容，例如时长创建日期等。而视频的动态内容和在播放资源时呈现状态，则由 AVPlayerItem 描述。AVPlayerItem 由多种 `AVPlayerItemTrack`建立模型。

AVPlayerItem 有如下重要属性。

```
//视频的播放状态
@property (nonatomic, readonly) AVPlayerItemStatus status;
//视频的播放时长
@property (nonatomic, readonly) CMTime duration;
//视频缓存区域大小
@property (nonatomic, readonly) NSArray<NSValue *> *loadedTimeRanges;
//callback,缓冲区有足够数据可以播放
@property (nonatomic, readonly, getter=isPlaybackLikelyToKeepUp) BOOL playbackLikelyToKeepUp;
//callback,缓冲区满了
@property (nonatomic, readonly, getter=isPlaybackBufferFull) BOOL playbackBufferFull;
//callback,缓冲区空了，需要等待数据
@property (nonatomic, readonly, getter=isPlaybackBufferEmpty) BOOL playbackBufferEmpty;
```

以上控制属性在视频播放着很常用，我们通常使用 KVO 来监控视频播放状态和缓冲区的状态。

首先为这些控制数据添加监听者。

```
[_playerItem addObserver:self forKeyPath:@"status" options:NSKeyValueObservingOptionNew context:nil];
[_playerItem addObserver:self forKeyPath:@"loadedTimeRanges" options:NSKeyValueObservingOptionNew context:nil];
[_playerItem addObserver:self forKeyPath:@"playbackBufferEmpty" options:NSKeyValueObservingOptionNew context:nil];
[_playerItem addObserver:self forKeyPath:@"playbackLikelyToKeepUp" options:NSKeyValueObservingOptionNew context:nil];

```

而后我们就可以监听播放的状态变化，比如播放准备就绪，缓冲区大小等。

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    if (object == self.player.currentItem) {
        if ([keyPath isEqualToString:@"status"]) {
            if (self.player.currentItem.status == AVPlayerItemStatusReadyToPlay) {
                NSLog(@"可以播放");
            } else if (self.player.currentItem.status == AVPlayerItemStatusFailed) {
                NSLog(@"播放失败");
            }
        } else if ([keyPath isEqualToString:@"loadedTimeRanges"]) {
            NSArray *loadedTimeRanges = [[_player currentItem] loadedTimeRanges];
            CMTimeRange timeRange     = [loadedTimeRanges.firstObject CMTimeRangeValue];// 获取缓冲区域
            float startSeconds        = CMTimeGetSeconds(timeRange.start);
            float durationSeconds     = CMTimeGetSeconds(timeRange.duration);
            NSTimeInterval result     = startSeconds + durationSeconds;// 计算缓冲总进度

            NSLog(@"缓冲区大小 %f",result);
        }
    } else if ([keyPath isEqualToString:@"playbackBufferEmpty"]) {
        NSLog(@"缓冲区为空了");
    } else if ([keyPath isEqualToString:@"playbackLikelyToKeepUp"]) {
        NSLog(@"缓冲区数据足够播放");
    }
}
```

通过对属性的监听，我们可以对视频播放做其他的控制。比如播放、暂停和一些 UI 的处理。

另外，在视频播放完成时，AVPlayerItem 会发送 `AVPlayerItemDidPlayToEndTimeNotification`通知，我们可以为其绑定视频播放完毕以后的策略。

### [AVPlayer](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVPlayer_Class/) 

`AVPlayer` 是一个用来播放基于时间的视听媒体的控制器对象，支持播放从本地、分布下载或通过 `HTTP Live Streaming` 协议得到的流媒体。AVPlayer 只管理一个单独资源的播放，如果想要管理一个资源队列，需要使用到 AVPlayer 的子类 `AVQueuePlayer`。

AVPlayer 提供了视频播放的基本控制方法。例如，`play`，`pause`等。同时 AVPlayer 也提供了一个常用的方法

```
- (void)seekToTime:(CMTime)time toleranceBefore:(CMTime)toleranceBefore toleranceAfter:(CMTime)toleranceAfter completionHandler:(void (^)(BOOL finished))completionHandler
```

使用这个方法，我们可以跳转到想要播放的时间节点，当跳转成功时，会触发成功回调。

AVPlayer 是一个不可见组件,要将视频资源导出到用户界面目标位置,需要使用 AVPlayerLayer 类。

`AVPlayerLayer` 是构建于 Core Animation 框架之上（[注1](#jump)）的图层类型，扩展了 Core Animation 的 CALayer 类，为 iOS 的视频渲染提供支持。AVPlayerLayer 的属性 `videoGravity` 描述了视频承载层的缩放状态，有如下三种类型。

```
//保留长宽比，未填充部分会有黑边。
AVLayerVideoGravityResizeAspect
//保留长宽比，填充所有的区域。
AVLayerVideoGravityResizeAspectFill
//拉伸填满所有的空间。
AVLayerVideoGravityResize
```

videoGravity 是 AVPlayerLayer 中唯一需要我们设置的地方，默认情况下我们会设置为 AVLayerVideoGravityResizeAspect 。



###回顾
本文简单总结了视频相关的一些类族，当我们需要做一些复杂的操作，比如视频裁剪合并滤镜等就会对上述对象的方法有更深入的使用。



<span id="jump">注1：AVPlayerLayer不是Core Animation框架的一部分，AVPlayerLayer由AVFoundation提供。</span>
