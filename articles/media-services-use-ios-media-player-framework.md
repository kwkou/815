<properties linkid="develop-media-services-how-to-guides-ios-media-player-framework" urlDisplayName="iOS Media Player Framework" pageTitle="Use the iOS Media Player Framework with Azure Media Services" metaKeywords="" description="Learn how to use the Media Services iOS Media Player Framework library to create rich, dynamic apps.," metaCanonical="" services="media-services" documentationCenter="" title="How to use the Azure Media Services iOS Media Player Framework" authors="" solutions="" manager="" editor="" />

如何使用 Azure Media Services iOS Media Player Framework
========================================================

借助 Azure Media Services iOS Media Player Framework 库，iPod、iPhone 和 iPad 开发人员可以轻松创建功能丰富的动态客户端应用程序，以快速创建视频和音频流并将其混合在一起。例如，显示体育内容的应用程序可以轻松插入广告，并可以在任何情况下选择并控制这些广告的播放频率，即使回放主要内容，也可以做到这一点。教育应用程序也可以使用相同的功能，例如，创建一个内容区域，以便在其中插入主要讲座的旁白或边栏，然后在播放（显示）这些内容后返回到主要内容。

通常，生成能够基于应用程序及其用户之间的交互创建内容流的应用程序相对较为复杂 - 一般情况下，你必须提前在服务器上从头开始创建整个流并将其保存。使用 iOS Media Player Framework 生成的客户端应用程序可以实现所有这一切，不需要你亲自控制或修改主要内容流。你可以：

-   提前在客户端设备上排定内容流。
-   排定正片开始前的广告或插播内容。
-   排定正片结束后的广告或插播内容。
-   排定正片播放中途显示的广告或插播内容，并创建广告组合。
-   控制是每次将内容时间线倒回后都播放正片中途广告或插播内容，还是只播放广告或插播内容一次，播放后即将其从时间线中删除。
-   发生任何事件后直接在时间线中动态插入内容，不管是用户按下了某个按钮，还是应用程序收到了来自服务的通知 - 例如，新闻内容节目可以发送突发新闻通知，应用程序可以“暂停”主要内容，以动态加载突发新闻流。

将这些功能与 iOS 设备的媒体播放工具相结合，可以在很短的时间内使用较少的资源提供极其丰富的媒体体验。

SDK 包含一个 SamplePlayer 应用程序，它演示了如何生成一个 iOS 应用程序，以便使用上述大多数功能快速创建内容流，并允许用户通过按钮动态触发插播内容。本教程将演示 SamplePlayer 应用程序的主要组件，以及如何将它用作应用程序的起点。

示例播放器应用程序入门
----------------------

以下步骤描述如何获取该应用程序，并全方位展示该应用程序如何利用框架。

1.  克隆 git 存储库。

    `git clone https://github.com/WindowsAzure/azure-media-player-framework`

2.  打开位于 `azure-media-player-framework/src/iOS/HLSClient/` 中的项目：**SamplePlayer.xcodeproj**。

3.  以下是该示例播放器的结构：

![HLS 示例代码结构](http://mingfeiy.com/wp-content/uploads/2013/01/HLS-Structure.png)

1.  在 iPad 文件夹下，有两个 .xib 文件：**SeekbarViewController** 和 **SamplePlayerViewController**。它们用于创建 iPad 应用程序的 UI 布局。同样，在 iPhone 文件夹下有两个用于定义拖动条和控制器的 .xib 文件。

2.  主应用程序逻辑驻留在 `Shared` 文件夹下的 **SamplePlayerViewController.m** 中。下面所述的大多数代码段都位于该文件中。

了解 UI 布局
------------

有两个用于定义播放器界面的 .xib 文件。（以下内容以 iPad 布局为例，不过，iPhone 布局与此十分相似，并且原理是相同的。）##\# SamplePlayerViewController\_iPad.xib ![示例播放器地址栏](http://mingfeiy.com/wp-content/uploads/2013/01/addressbar.png)

-   **“媒体 URL”**是用于加载媒体流的 URL。该应用程序提供了预先填充的媒体 URL 列表，你可以通过 URL 选择按钮来使用这些 URL。或者，你可以输入自己的 Http 实时流 (HLS) 内容 URL。此媒体内容将用作第一个主要内容。**注意：请不要将此 URL 留空。**

-   **“URL 选择”**按钮用于从媒体 URL 列表中选择备用 URL。

### SeekbarViewController\_iPad.xib

![拖动条控制器](http://mingfeiy.com/wp-content/uploads/2013/01/controller.png) \* 使用**“播放”**按钮来播放媒体和暂停播放。

-   **“拖动条”**用于在整个播放时间线上定位。如果要定位，请按住鼠标，将指针拖到所需的位置，然后在拖动条上松开定位按钮。

**注意**：当观看者定位到广告时，将出现一个新的拖动条，其中显示了广告的持续时间。主拖动条仅显示主要内容的持续时间（也就是说，广告的持续时间在主拖动条中为 0）。

-   **“播放器时间”**控件显示两个时间 (`Label:playerTime`)，例如 00:23/02:10。在本例中，00:23 是当前已播放的时间，02:10 是媒体的总持续时间。

-   “快进”和“快退”**按钮**目前不能按预期工作；我们即将发布更新版本。

-   播放主要内容时按“立即排定”按钮会播入广告**（可以在代码隐藏文件中定义广告源 URL）**。注意：在当前版本中，播放一则广告时无法排定另一则广告。

### 如何排定主要内容

下面排定了从 0 秒播放到 80 秒的内容剪辑：

    //Schedule the main content
    MediaTime *mediaTime = [[[MediaTime alloc] init] autorelease];
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 0;
    mediaTime.clipEndMediaTime = 80;
        
    int clipId = 0;
    if (![framework appendContentClip:[NSURL URLWithString:url] withMediaTime:mediaTime andGetClipId:&clipId])
    {
    [self logFrameworkError];
    }

在上面的示例代码中：

-   **MediaTime** 对象控制你要排定为主要内容的视频剪辑。在上面的示例中，已将视频剪辑排定为持续播放 80 秒（从 0 秒播放到 80 秒）；
-   **clipBeginMediaTime** 表示视频的播放开始时间。例如，如果 **clipBeginMediaTime** = 5，则从视频剪辑的第 5 秒开始播放。
-   **clipEndMediaTime** 表示视频的播放结束时间。如果 **clipEndMediaTime**=100，则在视频剪辑的第 100 秒结束视频播放。\*然后，我们通过要求框架执行 **appendContentClip** 来排定 **MediaTime**。在上面的示例中，主要内容 URL 是在 `[NSURL URLWithString:url]` 中指定的，该媒体的排程是使用以下 **withMedia** 属性设置的：`[framework appendContentClip:[NSURL URLWithString:url] withMediaTime:mediaTime andGetClipId:&clipId])`。

**注意：**请始终在排定任何广告（包括正片开始前的广告）之前排定主要内容。

### 不同的情况：如果你在播放两个主要内容剪辑，则也可以使用以下代码将第二个剪辑排在第一个剪辑的后面：

    //Schedule second content
    NSString *secondContent=@"http://wamsblureg001orig-hs.cloudapp.net/6651424c-a9d1-419b-895c-6993f0f48a26/The%20making%20of%20Microsoft%20Surface-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 30;
    mediaTime.clipEndMediaTime = 80;
    if (![framework appendContentClip:[NSURL URLWithString:secondContent] withMediaTime:mediaTime andGetClipId:&clipId])
    {
    [self logFrameworkError];
    }

根据上面的代码进行这种排程会在主要内容时间线上排定两个内容流。第一个内容流是基于 `URLWithString:url` 排定的，第二个内容流是基于 `URLWithString:secondContent` 排定的。第二段内容从媒体流的第 30 秒开始播放，到第 80 秒结束。

排定广告
--------

在当前版本中，仅支持 **pauseTimeline=false** 广告，也就是说，在广告结束后，播放器将从主要内容暂停的位置接着播放。

请了解以下要点：

-   排定广告时，所有的 \*\*LinearTime.duration\*\* 都应为 0。
-   如果 \*\*clipEndMediaTime\*\* 长于广告持续时间，则广告在完成播放后就会结束，而不会引发任何异常。建议你确认广告固有持续时间是否在呈现时间 (\*\*clipEndMediaTime\*\*) 的范围内，以免遗漏广告的播放。
-   支持正片开始前、正片中途和正片结束后的广告。正片开始前的广告只能排定在所有内容的最前面。例如，在粗剪编辑 (RCE) 方案中，你无法为第二段内容排定正片开始前的广告。
-   支持粘性广告和一次性播放广告，可将此类广告与正片开始前、正片中途和正片结束后的广告结合使用。
-   广告格式可以是 .Mp4 或 HLS。

##\# 如何排定正片开始前、正片中途、正片结束后的广告和广告组合

#### 排定正片开始前的广告

    LinearTime *adLinearTime = [[[LinearTime alloc] init] autorelease];
    NSString *adURLString = @"http://smoothstreamingdemo.blob.core.windows.net/videoasset/WA-BumpShort_120530-1.mp4";
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.clipBeginMediaTime = 0;
    adInfo.mediaTime.clipEndMediaTime = 5;
    adInfo.policy = [[[PlaybackPolicy alloc] init] autorelease];
    adInfo.appendTo = -1;
    adInfo.type = AdType_Preroll;
    adLinearTime.duration = 0;
    if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
    [self logFrameworkError];
    }

**AdInfo** 对象表示有关广告剪辑的所有信息：\* **ClipURL** 是剪辑源的 URL。\* **mediaTime** 属性指示广告的播放持续时间。（**clipBeginMediaTime** 是广告的开始时间，**clipEndMediaTime** 定义广告的结束时间。）在上面的示例代码中，我们将一则广告排定为播放 5 秒：从广告时长的第 0 秒开始，到第 5 秒结束。\* 框架当前不使用 **Policy** 对象。\* 如果不是广告组合，则必须将 **appendTo** 值设置为 -1。\* **type** 值可以指定正片开始前、正片中途、正片结束后的广告或广告组合。由于正片开始前或正片结束后的广告没有关联的时间设置，因此我们在这里指定了该类型。

#### 排定正片中途广告

如果将 `adLinearTime.startTime = 23;` 添加到上面的代码示例，广告将从主要内容时间线中的第 23 秒开始播放。

#### 排定正片结束后的广告

    //Schedule Post Roll Ad
    NSString *postAdURLString=@"http://wamsblureg001orig-hs.cloudapp.net/aa152d7f-3c54-487b-ba07-a58e0e33280b/wp-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    AdInfo *postAdInfo = [[[AdInfo alloc] init] autorelease];
    postAdInfo.clipURL = [NSURL URLWithString:postAdURLString];
    postAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    postAdInfo.mediaTime.clipBeginMediaTime = 0;
    postAdInfo.mediaTime.clipEndMediaTime = 5;
    postAdInfo.policy = [[[PlaybackPolicy alloc] init] autorelease];
    postAdInfo.type = AdType_Postroll;
    adLinearTime.duration = 0;
    if (![framework scheduleClip:postAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
    [self logFrameworkError];
    }

与正片开始前的广告排程的唯一区别在于 `postAdInfo.type = AdType_Postroll;`。上面的代码将一段为时 5 秒的广告排定为正片结束后的广告。

#### 排定广告组合

广告组合是连续插播的多则广告。以下代码将两则广告排定为一个广告组合。

    NSString *adpodSt1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4
        st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    AdInfo *adpodInfo1 = [[[AdInfo alloc] init] autorelease];
    adpodInfo1.clipURL = [NSURL URLWithString:adpodSt1];
    adpodInfo1.mediaTime = [[[ManifestTime alloc] init] autorelease];
    adpodInfo1.mediaTime.clipBeginMediaTime = 0;
    adpodInfo1.mediaTime.clipEndMediaTime = 17;
    adpodInfo1.mediaTime = [[[PlaybackPolicy alloc] init] autorelease];
    adpodInfo1.type = AdType_Midroll;
    adpodInfo1.appendTo=-1;
    int32_t adIndex = 0;
    if (![framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
    [self logFrameworkError];
    }
        
    NSString *adpodSt2=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-532531b8-fca4-4c15-86f6-45f9f45ec980/Windows%208_%20Sign%20in%20with%20a%20Smile.mp4
        st=2012-11-28T16%3A35%3A26Z&se=2014-11-28T16%3A35%3A26Z&sr=c&si=c6ede35c-f212-4ccd-84da-805c4ebf64be&sig=zcWsj1JOHJB6TsiQL5ZbRmCSsEIsOJOcPDRvFVI0zwA%3D";
    AdInfo *adpodInfo2 = [[[AdInfo alloc] init] autorelease];
    adpodInfo2.clipURL = [NSURL URLWithString:adpodSt2];
    adpodInfo2.mediaTime = [[[ManifestTime alloc] init] autorelease];
    adpodInfo2.mediaTime.clipBeginMediaTime = 0;
    adpodInfo2.mediaTime.clipEndMediaTime = 17;
    adpodInfo2.policy = [[[PlaybackPolicy alloc] init] autorelease];
    adpodInfo2.type = AdType_Pod;
    adpodInfo2.appendTo = adIndex;
    if (![framework scheduleClip:adpodInfo2 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
    [self logFrameworkError];
    }

这里要注意几个问题：\* 对于第一个剪辑，**appendTo** 为 -1。当我们调用 `[framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex]` 时，`adIndex` 将收到一个唯一值，该值指示广告组合中这第一个剪辑的结束时间。对于广告组合中的第二个剪辑，我们通过将 **appendTo** 设置为 `adpodInfo2.appendTo = adIndex;`（将第一个剪辑的结束位置指定为第二个剪辑的开始位置），使第二则广告的开始时间与第一则广告的结束时间重合。\* 然后，必须将类型设置为 `AdType_Pod` 以指明这是一个广告组合。

### 如何排定一次性播放广告或“粘性”广告

    AdInfo *oneTimeInfo = [[[AdInfo alloc] init] autorelease];
    oneTimeInfo.deleteAfterPlay = YES;

如上面的代码示例中所示，如果将 **deleteAfterPlay** 设置为 **YES**，则此广告只播放一次。如果将 **deleteAfterPlay** 设置为 **NO**，则此广告会持续播放，我们称之为“粘性广告”。##\# 有关详细信息，请参阅 [Azure Media Player Framework Wiki](https://github.com/WindowsAzure/azure-media-player-framework/wiki)。

