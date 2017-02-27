<properties
    pageTitle="Azure 按需媒体编码器概述和比较 | Azure"
    description="本主题概述并比较 Azure 按需媒体编码器。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="e6bfc068-fa46-4d68-b1ce-9092c8f3a3c9"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="02/24/2017"
    ms.author="juliako" />  


#Azure 按需媒体编码器概述和比较

##编码概述

Azure 媒体服务提供了多个用于在云中对媒体进行编码的选项。

开始使用媒体服务时，了解编解码器与文件格式之间的区别很重要。编解码器是实现压缩/解压缩算法的软件，而文件格式是用于保存压缩视频的容器。

媒体服务所提供的动态打包允许以媒体服务支持的流格式（MPEG DASH、HLS、平滑流式处理）传送自适应比特率 MP4 或平滑流式处理编码内容，而无须重新打包成这些流格式。

>[AZURE.NOTE]
创建 AMS 帐户时，系统会将**默认**流式处理终结点以“已停止”状态添加到用户的帐户。若要开始对内容进行流式处理并利用动态打包和动态加密功能，必须确保要从其流式获取内容的流式处理终结点处于“正在运行”状态。若要使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)，必须执行下列操作：
>
>另外，将源文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流式处理文件（本教程稍后将演示编码步骤）。

媒体服务支持将在本文中介绍的以下按需编码器：

- [媒体编码器标准版](/documentation/articles/media-services-encode-asset/#media-encoder-standard)

本文简要概述了按需媒体编码器，并提供了指向介绍更多详细信息的文章的链接。本主题还提供编码器的比较。

请注意，默认情况下每个媒体服务帐户同时只能有一个活动的编码任务。你可以保留编码单元，使用它们可以同时运行多个编码任务，你购买的每个编码保留单元对应一个任务。有关信息，请参阅[缩放编码单位](/documentation/articles/media-services-portal-encoding-units/)。

##<a name="media-encoder-standard"></a>媒体编码器标准版

###如何使用

[如何使用媒体编码器标准版进行编码](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)

###格式

[格式和编解码器](/documentation/articles/media-services-media-encoder-standard-formats/)

###预设

媒体编码器标准版使用[此处](/documentation/articles/media-services-mes-presets-overview/)所述的编码器预设之一进行配置。

### 输入和输出元数据
编码器输入元数据在[此处](/documentation/articles/media-services-input-metadata-schema/)说明。

[此处](/documentation/articles/media-services-output-metadata-schema/)说明了编码器输出元数据。

###生成缩略图

有关信息，请参阅[如何使用媒体编码器标准生成缩略图](/documentation/articles/media-services-advanced-encoding-with-mes/#thumbnails)。

###修剪视频（裁剪）

有关信息，请参阅[如何使用媒体编码器标准修剪视频](/documentation/articles/media-services-advanced-encoding-with-mes/#trim_video)。

###创建覆盖层

有关信息，请参阅[如何使用媒体编码器标准创建覆盖层](/documentation/articles/media-services-advanced-encoding-with-mes/#overlay)。

###另请参阅

[媒体服务博客](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)
 
##媒体编码器高级工作流

###概述

[在 Azure 媒体服务中引入高级编码](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services/)

###如何使用



[如何在 Azure 媒体服务中使用高级编码](https://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services/)

### 已知问题
如果输入视频不包含隐藏式字幕，输出资产仍将包含一个空的 TTML 文件。


## 相关文章
* [通过自定义媒体编码器标准预设执行高级编码任务](/documentation/articles/media-services-custom-mes-presets-with-dotnet/)
* [配额和限制](/documentation/articles/media-services-quotas-and-limitations/)

<!--Reference links in article-->

[1]: /pricing/details/media-services/

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: add one note for creating AMS account; remove "比较编码器" section-->