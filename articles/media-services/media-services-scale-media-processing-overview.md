<properties
    pageTitle="调整媒体处理的规模概述 | Azure"
    description="本主题概述了如何使用 Azure 媒体服务调整媒体处理的规模。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="780ef5c2-3bd6-4261-8540-6dee77041387"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/13/2017"
    wacn.date="06/15/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="3dc742a89b55435528793df451bbb1ce122c6d5f"
    ms.lasthandoff="06/15/2017" />

# <a name="scaling-media-processing-overview"></a>调整媒体处理的规模概述
此页概述了如何以及为何调整媒体处理的规模。 

## <a name="overview"></a>概述
媒体服务帐户与保留单位类型关联，后者决定了编码处理任务的处理速度。 可以在以下保留单位类型中进行选择：**S1**、**S2** 或 **S3**。 例如，与 **S1** 保留单位类型相比，使用 **S2** 保留单位类型时，同一编码作业运行速度更快。 有关详细信息，请参阅[保留单位类型](https://azure.microsoft.com/zh-cn/blog/high-speed-encoding-with-azure-media-services/)。

除了指定保留单位类型，还可以指定通过保留单位来预配帐户。 设置的保留单位数决定了给定帐户中可并发处理的媒体任务数。 例如，如果帐户具有 5 个保留单元，则只要有任务要处理，就可以同时运行 5 个媒体任务。 其余任务将排队等待，运行的任务完成后才选择它们以按顺序进行处理。 如果帐户未预配任何保留单位，则按顺序选择任务进行处理。 在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

## <a name="choosing-between-different-reserved-unit-types"></a>在不同的保留单位类型之间进行选择
下表可帮助你在不同的编码速度之间进行选择时做出决定。 它还提供了几个基准案例，并提供可用于下载视频的 SAS URL，你可以对该视频执行自己的测试：

| 方案 | **S1** | **S2** | **S3** |
| --- | --- | --- | --- |
| 预期的用例| <p>单比特率编码。</p><p>具有 SD 或更低分辨率的文件，不具有高时效性，成本低。 |单比特率和多比特率编码。</p><p>SD 和 HD 编码的正常使用情况。 |单比特率和多比特率编码。</p><p>全高清和 4K 分辨率视频。对时间敏感，更快的编码周转。 </p> |
| 基准|<p>输入文件：5 分钟、640x360p、29.97 帧/秒。</p><p>编码为具有相同分辨率的单比特率 MP4 文件大约需要 11 分钟。</p>|<p>输入文件：5 分钟、1280x720p、29.97 帧/秒</p><p>预设为“H264 单比特率 720p”的编码大约需要 5 分钟。</p><p>预设为“H264 多比特率 720p”的编码大约需要 11.5 分钟。</p>|<p>输入文件：5 分钟、1920x1080p、29.97 帧/秒。</p><p>预设为“H264 单比特率 1080p”的编码大约需要 2.7 分钟。</p><p>预设为“H264 多比特率 1080p”的编码大约需要 5.7 分钟。</p>|

## <a name="considerations"></a>注意事项
> [AZURE.IMPORTANT]
> 查看本节中所述的注意事项。  
> 
> 

* 保留单位可用于并行化所有媒体处理，其中使用 Azure Media Indexer 为作业编制索引。  但是，与编码不同，索引作业使用更快的保留单位并不能更快地完成处理。
* 如果使用共享的池（即没有任何保留单位），则编码任务将具有与 S1 RU 相同的性能。 但是，任务在排队状态下花费的时间可能没有上限，并且在任何给定的时间内，最多只会运行一项任务。


## <a name="billing"></a>计费

将根据媒体保留单位的实际使用分钟数对你进行收费。 有关详细说明，请参阅[媒体服务定价](/pricing/details/media-services/)页的“常见问题”部分。   

## <a name="quotas-and-limitations"></a>配额和限制
有关配额和限制以及如何开具支持票证的信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。

## <a name="next-step"></a>后续步骤
通过以下技术之一完成调整媒体处理的规模任务：
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encoding-units/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/media/operations/encodingreservedunittype)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
<!--Update_Description: add "计费" section-->