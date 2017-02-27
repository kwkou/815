<properties
    pageTitle="MES (Media Encoder Standard) 任务预设 | Azure"
    description="本主题概述了 MES (Media Encoder Standard) 任务预设。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />
<tags
    ms.assetid="f243ed1c-ac9c-4300-a5f7-f092cf9853b9"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="02/24/2017"
    ms.author="juliako" />  


# MES \(Media Encoder Standard\) 任务预设

`Media Encoder Standard` 定义一组可在创建编码作业时使用的编码预设
  
 存在 XML 或 JSON 的字符串应基于以下文件中所示的预设。可将包含自定义值的预设传递到编码器（其值必须有效）。 有关这些预设中每个元素含义的说明以及每个元素的有效值，请参阅 [Media Encoder Standard 架构](/documentation/articles/media-services-mes-schema/)主题。
  
> [AZURE.NOTE]
使用预设进行 4K 编码时，应获取 `S3` 保留单位类型。有关详细信息，请参阅[如何缩放编码](/documentation/articles/media-services-portal-encoding-units/)。
  
 使用 Media Encoder Standard 时，将默认启用旋转。如果在智能手机或其他移动设备上以纵向模式拍摄了视频，则这些预设默认在编码前将其旋转为横向模式。（而如果使用 Azure 媒体编码器，就必须手动操作视频旋转，如[此博客](http://azure.microsoft.com/blog/2014/08/21/advanced-encoding-features-in-azure-media-encoder/)中“Video Rotation”（视频旋转）所述。
  
 预设名称将映射到以下主题中所示的预设。
  
 [H264 多比特率 1080p 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-1080p-Audio-5.1/) 将生成一组 8 个 GOP 对齐的 MP4 文件，范围为 6000 kbps - 400 kbps，以及 AAC 5.1 音频。
  
 [H264 多比特率 1080p](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-1080p/) 将生成一组 8 个 GOP 对齐的 MP4 文件，范围为 6000 kbps - 400 kbps，以及立体声 AAC 音频。
  
 [适用于 iOS 的 H264 多比特率 16x9](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-16x9-for-iOS/) 将生成一组 8 个 GOP 对齐的 MP4 文件，范围为 8500 kbps - 200 kbps，以及立体声 AAC 音频。
  
 [H264 多比特率 16x9 SD 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD-Audio-5.1/) 将生成一组 5 个 GOP 对齐的 MP4 文件，范围为 1900 kbps - 400 kbps，以及 AAC 5.1 音频。
  
 [H264 多比特率 16x9 SD](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD/) 将生成一组 5 个 GOP 对齐的 MP4 文件，范围为 1900 kbps - 400 kbps，以及立体声 AAC 音频。
  
 [H264 多比特率 4K 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4K-Audio-5.1/) 将生成一组 12 个 GOP 对齐的 MP4 文件，范围为 20000 kbps - 1000 kbps，以及 AAC 5.1 音频。
  
 [H264 多比特率 4K](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4K/) 将生成一组 12 个 GOP 对齐的 MP4 文件，范围为 20000 kbps - 1000 kbps，以及立体声 AAC 音频。
  
 [适用于 iOS 的 H264 多比特率 4x3](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4x3-for-iOS/) 将生成一组 8 个 GOP 对齐的 MP4 文件，范围为 8500 kbps - 200 kbps，以及立体声 AAC 音频。
  
 [H264 多比特率 4x3 SD 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD-Audio-5.1/) 将生成一组 5 个 GOP 对齐的 MP4 文件，范围为 1600 kbps - 400 kbps，以及 AAC 5.1 音频。
  
 [H264 多比特率 4x3 SD](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD/) 将生成一组 5 个 GOP 对齐的 MP4 文件，范围为 1600 kbps - 400 kbps，以及立体声 AAC 音频。
  
 [H264 多比特率 720p 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p-Audio-5.1/) 将生成一组 6 个 GOP 对齐的 MP4 文件，范围为 3400 kbps - 400 kbps，以及 AAC 5.1 音频。
  
 [H264 多比特率 720p](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p/) 将生成一组 6 个 GOP 对齐的 MP4 文件，范围为 3400 kbps - 400 kbps，以及立体声 AAC 音频。
  
 [H264 单比特率 1080p 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-1080p-Audio-5.1/) 将生成单个 MP4 文件，其比特率为 6750 kbps，并且带有 AAC 5.1 音频。
  
 [H264 单比特率 1080p](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-1080p/) 将生成单个 MP4 文件，其比特率为 6750 kbps，并且带有立体声 AAC 音频。
  
 [H264 单比特率 4K 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4K-Audio-5.1/) 将生成单个 MP4 文件，其比特率为 18000 kbps，并且带有 AAC 5.1 音频。
  
 [H264 单比特率 4K](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4K/) 将生成单个 MP4 文件，其比特率为 18000 kbps，并且带有立体声 AAC 音频。
  
 [H264 单比特率 4x3 SD 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4x3-SD-Audio-5.1/) 将生成单个 MP4 文件，其比特率为 18000 kbps，并且带有 AAC 5.1 音频。
  
 [H264 单比特率 4x3 SD](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4x3-SD/) 将生成单个 MP4 文件，其比特率为 1800 kbps，并且带有立体声 AAC 音频。
  
 [H264 单比特率 16x9 SD 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-16x9-SD-Audio-5.1/) 将生成单个 MP4 文件，其比特率为 2200 kbps，并且带有 AAC 5.1 音频。
  
 [H264 单比特率 16x9 SD](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-16x9-SD/) 将生成单个 MP4 文件，其比特率为 2200 kbps，并且带有立体声 AAC 音频。
  
 [H264 单比特率 720p 音频 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-720p-Audio-5.1/) 将生成单个 MP4 文件，其比特率为 4500 kbps，并且带有 AAC 5.1 音频。
  
 [适用于 Android 的 H264 单比特率 720p](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-720p-for-Android/) 预设将生成单个 MP4 文件，其比特率为 2000 kbps，并且带有立体声 AAC。
  
 [H264 单比特率 720p](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-720p/) 将生成单个 MP4 文件，其比特率为 4500 kbps，并且带有立体声 AAC 音频。
  
 [适用于 Android 的 H264 单比特率高品质 SD](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-High-Quality-SD-for-Android/) 将生成单个 MP4 文件，其比特率为 500 kbps，并且带有立体声 AAC 音频。
  
 [适用于 Android 的 H264 单比特率低品质 SD](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-Low-Quality-SD-for-Android/) 将生成单个 MP4 文件，其比特率为 56 kbps，并且带有立体声 AAC 音频。
  
 若要了解媒体服务编码器的更多相关信息，请参阅[使用 Azure 媒体服务按需编码](/documentation/articles/media-services-encode-asset/)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: fix the typo "18000 kbps" to "1800 kbps" for H264 单比特率 4x3 SD-->