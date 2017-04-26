<properties
    pageTitle="概述并比较 Azure 点播媒体编码器 | Azure"
    description="本主题简要介绍并比较了 Azure 点播媒体编码器。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="e6bfc068-fa46-4d68-b1ce-9092c8f3a3c9"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/10/2017"
    wacn.date="04/24/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="9d5d9607ea4256162b55d7f913e960126e5cf0c1"
    ms.lasthandoff="04/14/2017" />

# <a name="overview-and-comparison-of-azure-on-demand-media-encoders"></a>Azure 点播媒体编码器概述和比较
## <a name="encoding-overview"></a>编码概述
Azure 媒体服务提供了多个用于在云中对媒体进行编码的选项。

开始使用媒体服务时，了解编解码器与文件格式之间的区别很重要。
编解码器是实现压缩/解压缩算法的软件，而文件格式是用于保存压缩视频的容器。

媒体服务所提供的动态打包，允许以媒体服务支持的流格式（MPEG DASH、HLS、平滑流式处理）传送自适应比特率 MP4 或平滑流式处理编码内容，而无须重新打包成这些流格式。

>[AZURE.NOTE]
>创建 AMS 帐户后，会将一个处于“已停止”状态的**默认**流式处理终结点添加到帐户。 若要开始流式传输内容并利用动态打包和动态加密，要从中流式传输内容的流式处理终结点必须处于“正在运行”状态。 若要使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)，必须执行下列操作：
>
>此外，将源文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流式处理文件（本教程稍后将演示编码步骤）。

媒体服务支持将在本文中介绍的以下按需编码器：

* [Media Encoder Standard](/documentation/articles/media-services-encode-asset/#media-encoder-standard)

本文简要概述了按需媒体编码器，并提供了指向介绍更多详细信息的文章的链接。 本主题还提供对编码器的比较。

>[AZURE.NOTE]
>默认情况下每个媒体服务帐户同时只能有一个活动的编码任务。 你可以保留编码单元，使用它们可以同时运行多个编码任务，你购买的每个编码保留单位对应一个任务。 有关信息，请参阅[缩放编码单位](/documentation/articles/media-services-scale-media-processing-overview/)。

## <a name="media-encoder-standard"></a>媒体编码器标准版
### <a name="how-to-use"></a>如何使用
[如何使用 Media Encoder Standard 进行编码](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)

### <a name="formats"></a>格式
[格式和编解码器](/documentation/articles/media-services-media-encoder-standard-formats/)

### <a name="presets"></a>预设
Media Encoder Standard 使用[此处](/documentation/articles/media-services-mes-presets-overview/)所述的编码器预设之一进行配置。

### <a name="input-and-output-metadata"></a>输入和输出元数据
[此处](/documentation/articles/media-services-input-metadata-schema/)说明了编码器输入元数据。

[此处](/documentation/articles/media-services-output-metadata-schema/)说明了编码器输出元数据。

### <a name="generate-thumbnails"></a>生成缩略图
有关信息，请参阅[如何使用 Media Encoder Standard 生成缩略图](/documentation/articles/media-services-advanced-encoding-with-mes/#thumbnails)。

### <a name="trim-videos-clipping"></a>修剪视频（裁剪）
有关信息，请参阅[如何使用 Media Encoder Standard 剪裁视频](/documentation/articles/media-services-advanced-encoding-with-mes/#trim_video)。

### <a name="create-overlays"></a>创建覆盖层
有关信息，请参阅[如何使用 Media Encoder Standard 创建覆盖层](/documentation/articles/media-services-advanced-encoding-with-mes/#overlay)。

### <a name="see-also"></a>另请参阅
[媒体服务博客](https://azure.microsoft.com/zh-cn/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)



### <a name="known-issues"></a>已知问题
如果输入视频不包含隐藏式字幕，输出资产仍将包含一个空的 TTML 文件。

## <a name="related-articles"></a>相关文章
* [通过自定义 Media Encoder Standard 预设执行高级编码任务](/documentation/articles/media-services-custom-mes-presets-with-dotnet/)
* [配额和限制](/documentation/articles/media-services-quotas-and-limitations/)

<!--Reference links in article-->
[1]: /pricing/details/media-services/
<!--Update_Description:remove redundant content; update reference link of Media Service scale;add anchors to H2 titles-->