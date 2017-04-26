<properties
    pageTitle="MES (Media Encoder Standard) 的任务预设 | Azure"
    description="本主题概述了 MES (Media Encoder Standard) 的任务预设。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter=""
    translationtype="Human Translation" />
<tags
    ms.assetid="f243ed1c-ac9c-4300-a5f7-f092cf9853b9"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/02/2017"
    wacn.date="04/24/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="94cc33cdcc0ef10b4a4b5eaf58e75667f9f79c14"
    ms.lasthandoff="04/14/2017" />

# <a name="task-presets-for-mes-media-encoder-standard"></a>MES (Media Encoder Standard) 的任务预设

**Media Encoder Standard** 定义了一组可以在创建编码作业时使用的编码预设。 如果想要使用媒体服务对视频进行编码以实现流式处理，建议使用“自适应流式处理”预设。 指定此预设时，Media Encoder Standard 将[自动生成比特率阶梯](/documentation/articles/media-services-autogen-bitrate-ladder-with-mes/)。 

但是，如果需要自定义编码预设，应采用此部分中定义的某个编码预设作为模板，以用于自定义配置。 有关这些预设中的每个元素的含义及其有效值的说明，请参阅 [Media Encoder Standard 架构](/documentation/articles/media-services-mes-schema/)主题。  
  
> [AZURE.NOTE]
>  使用预设进行 4k 编码时，应获取 `S3` 保留单位类型。 有关详细信息，请参阅[如何缩放编码](/documentation/articles/media-services-scale-media-processing-overview/)。  
  
使用 Media Encoder Standard 时，默认启用旋转。 如果视频在智能手机或其他移动设备上采用纵向模式录制，则默认情况下，这些预设会在编码之前将其旋转为横向模式（与 Azure 媒体编码器不同，使用 Azure 媒体编码器时，需手动进行视频旋转，如[此](https://azure.microsoft.com/zh-cn/blog/2014/08/21/advanced-encoding-features-in-azure-media-encoder/)博客的“视频旋转”中所述）。  
  
可用预设：  
  
 [H264 多比特率 1080p Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-1080p-Audio-5.1/) 生成一组 8 GOP 对齐的 MP4 文件（范围从 6000 kbps 到 400 kbps）和 AAC 5.1 音频。
  
 [H264 多比特率 1080p](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-1080p/) 生成一组 8 GOP 对齐的 MP4 文件（范围从 6000 kbps 到 400 kbps）和立体声 AAC 音频。  
  
 [H264 多比特率 16x9 (iOS)](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-16x9-for-iOS/) 生成一组 8 GOP 对齐的 MP4 文件（范围从 8500 kbps 到 200 kbps）和立体声 AAC 音频。  
  
 [H264 多比特率 16x9 SD Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD-Audio-5.1/) 生成一组 5 GOP 对齐的 MP4 文件（范围从 1900 kbps 到 400 kbps）和 AAC 5.1 音频。
  
 [H264 多比特率 16x9 SD](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD/) 生成一组 5 GOP 对齐的 MP4 文件（范围从 1900 kbps 到 400 kbps）和立体声 AAC 音频。  
  
 [H264 多比特率 4K Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4K-Audio-5.1/) 生成一组 12 GOP 对齐的 MP4 文件（范围从 20000 kbps 到 1000 kbps）和 AAC 5.1 音频。
  
 [H264 多比特率 4K](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4K/) 生成一组 12 GOP 对齐的 MP4 文件（范围从 20000 kbps 到 1000 kbps）和立体声 AAC 音频。  
  
 [H264 多比特率 4x3 (iOS)](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4x3-for-iOS/) 生成一组 8 GOP 对齐的 MP4 文件（范围从 8500 kbps 到 200 kbps）和立体声 AAC 音频。  
  
 [H264 多比特率 4x3 SD Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD-Audio-5.1/) 生成一组 5 GOP 对齐的 MP4 文件（范围从 1600 kbps 到 400 kbps）和 AAC 5.1 音频。
  
 [H264 多比特率 4x3 SD](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD/) 生成一组 5 GOP 对齐的 MP4 文件（范围从 1600 kbps 到 400 kbps）和立体声 AAC 音频。  
  
 [H264 多比特率 720p Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p-Audio-5.1/) 生成一组 6 GOP 对齐的 MP4 文件（范围从 3400 kbps 到 400 kbps）和 AAC 5.1 音频。
  
 [H264 多比特率 720p](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p/)生成一组 6 GOP 对齐的 MP4 文件（范围从 3400 kbps 到 400 kbps）和立体声 AAC 音频。  
  
 [H264 单比特率 1080p Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-1080p-Audio-5.1/) 生成比特率为 6750 kbps 的单个 MP4 文件和 AAC 5.1 音频。
  
 [H264 单比特率 1080p](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-1080p/) 生成比特率为 6750 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 [H264 单比特率 4K Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4K-Audio-5.1/) 生成比特率为 18000 kbps 的单个 MP4 文件和 AAC 5.1 音频。
  
 [H264 单比特率 4K](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4K/) 生成比特率为 18000 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 [H264 单比特率 4x3 SD Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4x3-SD-Audio-5.1/) 生成比特率为 1800 kbps 的单个 MP4 文件和 AAC 5.1 音频。
  
 [H264 单比特率 4x3 SD](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-4x3-SD/) 生成比特率为 1800 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 [H264 单比特率 16x9 SD Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-16x9-SD-Audio-5.1/) 生成比特率为 2200 kbps 的单个 MP4 文件和 AAC 5.1 音频。
  
 [H264 单比特率 16x9 SD](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-16x9-SD/) 生成比特率为 2200 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 [H264 单比特率 720p Audio 5.1](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-720p-Audio-5.1/) 生成比特率为 4500 kbps 的单个 MP4 文件和 AAC 5.1 音频。
  
 [H264 单比特率 720p (Android)](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-720p-for-Android/) 预设生成比特率为 2000 kbps 的单个 MP4 文件和立体声 AAC。  
  
 [H264 单比特率 720p](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-720p/) 生成比特率为 4500 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 [H264 单比特率高品质 SD (Android)](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-High-Quality-SD-for-Android/) 生成比特率为 500 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 [H264 单比特率低质量 SD (Android)](/documentation/articles/media-services-mes-preset-H264-Single-Bitrate-Low-Quality-SD-for-Android/) 生成比特率为 56 kbps 的单个 MP4 文件和立体声 AAC 音频。  
  
 有关媒体服务编码器的详细信息，请参阅[使用 Azure 媒体服务按需编码](/documentation/articles/media-services-encode-asset/)。
<!--Update_Description: add reference to "自动生成比特率阶梯"; wording update-->