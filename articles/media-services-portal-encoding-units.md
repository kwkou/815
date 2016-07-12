<properties
	pageTitle="如何使用 Azure 管理门户缩放媒体处理"
	description="了解如何通过指定要为帐户设置的“按需流式处理保留单位”和“编码保留单位”数，缩放媒体服务。"
	services="media-services"
	documentationCenter=""
	authors="juliako,milangada"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="03/29/2016"
	wacn.date="05/16/2016"/>


# 如何使用 Azure 管理门户缩放媒体处理

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encoding-units/)
- [门户](/documentation/articles/media-services-portal-encoding-units/)
- [REST](https://msdn.microsoft.com/zh-cn/library/azure/dn859236.aspx)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

## 概述

媒体服务帐户与保留单位类型关联，后者决定了编码处理任务的处理速度。你可以在以下保留单位类型之间进行选择：**S1**、**S2** 或 **S3**。例如，与 **S1** 保留单位类型相比，使用 **S2** 类型时，相同的编码作业运行速度更快。有关详细信息，请参阅[编码保留单位类型](https://azure.microsoft.com/blog/author/milanga/)。

除了指定保留单元类型，你还可以指定如何通过媒体保留单元来设置帐户。设置的媒体保留单元数决定了给定帐户中可并发处理的媒体任务数。例如，如果你的帐户具有 5 个保留单元，则只要有任务要处理，就可以同时运行 5 个媒体任务。其余任务将排队等待，运行的任务完成后才选择它们以按顺序进行处理。如果帐户未设置任何保留单元，则按顺序选择任务进行处理。在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

## 在不同的保留单位类型之间进行选择

下表可帮助你在不同的编码速度之间进行选择时做出决定。它还提供了几个基准案例，并提供可用于下载视频的 SAS URL，你可以对该视频执行自己的测试：

   |**S1**|**S2**|**S3**|
----------|------------|----------|------------
预期的用例| 单比特率编码。<br/>处于 SD 或较低分辨率的文件，对时间不敏感，成本低。|单比特率和多比特率编码。<br/>针对 SD 和 HD 编码的正常使用。 |单比特率和多比特率编码。<br/>完整的 HD 和 4K 分辨率视频。对时间敏感，更快的编码周转。 
基准|[输入文件：5 分钟、640x360p、29.97 帧/秒](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_360p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>编码为具有相同分辨率的单比特率 MP4 文件大约需要 11 分钟。|[输入文件：5 分钟、1280x720p、29.97 帧/秒](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_720p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)<br/><br/>使用“H264 单比特率 720p”预设值进行编码大约需要 5 分钟。<br/><br/>使用“H264 多比特率 720p”预设值进行编码大约需要 11.5 分钟。|[输入文件：5 分钟、1920x1080p、29.97 帧/秒](https://wamspartners.blob.core.windows.net/for-long-term-share/Whistler_5min_1080p30.mp4?sr=c&si=AzureDotComReadOnly&sig=OY0TZ%2BP2jLK7vmcQsCTAWl33GIVCu67I02pgarkCTNw%3D)。<br/><br/>使用“H264 单比特率 1080p”预设值进行编码大约需要 2.7 分钟。<br/><br/>使用“H264 多比特率 1080p”预设值进行编码大约需要 5.7 分钟。

##注意事项

>[AZURE.IMPORTANT] 请注意以下事项：

- 保留单位可用于并行化所有媒体处理，其中使用 Azure Media Indexer 为作业编制索引。但是，与编码不同，索引作业使用更快的保留单位并不能更快地完成处理。

- 如果使用共享的池，即没有任何保留单位，则你的编码任务将具有与 S1 RU 相同的性能。但是，你的任务在排队状态下花费的时间可能没有上限，并且在任何给定时间内，最多一项任务将在运行。

- 下面的数据中心不提供 **S3** 保留单位类型：巴西南部、印度西部、印度中部和印度南部。

- 为 24 小时期间指定的最大单位数将用于计算成本。

## 更改保留单位类型

若要更改保留单位类型和保留单位数目，请执行以下操作：

1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中单击“媒体服务”。然后，单击媒体服务的名称。

2. 选择“编码”页。

	若要更改“保留单位类型”，请按“S1”、“S2”或“S3”。

	若要更改所选保留单位类型的保留单位数，请使用“编码”滑块。


	![“处理器”页](./media/media-services-portal-encoding-units/media-services-encoding-scale.png)

3. 按“保存”按钮保存更改。

	按“保存”后，会立即分配新的保留单位。
 

##配额和限制

有关配额和限制以及如何开具支持票证的信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。




 
<!---HONumber=Mooncake_0509_2016-->