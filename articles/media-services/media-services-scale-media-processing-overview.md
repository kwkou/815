<properties
	pageTitle="调整媒体处理的规模概述 | Azure"
	description="本主题概述了如何使用 Azure 媒体服务调整媒体处理的规模。"
	services="media-services"
	documentationCenter=""
	authors="juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/29/2016"
	wacn.date="12/27/2016"
	ms.author="juliako"/>


# 调整媒体处理的规模概述

此页概述了如何以及为何调整媒体处理的规模。

## 概述

媒体服务帐户与保留单元类型关联，后者决定了编码处理任务的处理速度。你可以在以下保留单元类型之间进行选择：**S1**、**S2** 或 **S3**。例如，与 S1 保留单元类型相比，使用 S2 类型时，相同的编码作业运行速度更快。有关详细信息，请参阅[保留单元类型](https://azure.microsoft.com/blog/high-speed-encoding-with-azure-media-services/)。

除了指定保留单元类型，你还可以指定通过保留单元来设置帐户。设置的保留单元数决定了给定帐户中可并发处理的媒体任务数。例如，如果帐户具有 5 个保留单元，则只要有任务要处理，就可以同时运行 5 个媒体任务。其余任务将排队等待，运行的任务完成后才选择它们以按顺序进行处理。如果帐户未设置任何保留单元，则按顺序选择任务进行处理。在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

## 在不同的保留单元类型之间进行选择

下表可帮助你在不同的编码速度之间进行选择时做出决定。它还提供了几个基准案例，并提供可用于下载视频的 SAS URL，你可以对该视频执行自己的测试：

方案|**S1**|**S2**|**S3**|
----------|------------|----------|------------
预期的用例| 单比特率编码。<br/>处于 SD 或较低分辨率的文件，对时间不敏感，成本低。|单比特率和多比特率编码。<br/>针对 SD 和 HD 编码的正常使用。 |单比特率和多比特率编码。<br/>完整的 HD 和 4K 分辨率视频。对时间敏感，更快的编码周转。 
基准|[输入文件：5 分钟、640x360p、29.97 帧/秒](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_360p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>编码为具有相同分辨率的单比特率 MP4 文件大约需要 11 分钟。|[输入文件：5 分钟、1280x720p、29.97 帧/秒](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_720p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)<br/><br/>使用“H264 单比特率 720p”预设值进行编码大约需要 5 分钟。<br/><br/>使用“H264 多比特率 720p”预设值进行编码大约需要 11.5 分钟。|[输入文件：5 分钟、1920x1080p、29.97 帧/秒](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_1080p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>使用“H264 单比特率 1080p”预设值进行编码大约需要 2.7 分钟。<br/><br/>使用“H264 多比特率 1080p”预设值进行编码大约需要 5.7 分钟。

##注意事项

>[AZURE.IMPORTANT] 查看本节中所述的注意事项。

- 保留单元可用于并行化所有媒体处理，其中使用 Azure Media Indexer 为作业编制索引。但是，与编码不同，索引作业使用更快的保留单元并不能更快地完成处理。

- 如果使用共享的池（即没有任何保留单元），则编码任务将具有与 S1 RU 相同的性能。但是，任务在排队状态下花费的时间可能没有上限，并且在任何给定时间内，最多一项任务将在运行。

- 下面的数据中心不提供 **S2** 保留单元类型：巴西南部、印度西部、印度中部和印度南部。

- 下面的数据中心不提供 **S3** 保留单元类型：巴西南部、印度西部、印度中部。

- 为 24 小时期间指定的最大单位数将用于计算成本。


##配额和限制

有关配额和限制以及如何开具支持票证的信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。

##后续步骤

通过以下技术之一完成调整媒体处理的规模任务：

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encoding-units/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/media/operations/encodingreservedunittype)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

<!---HONumber=Mooncake_0926_2016-->