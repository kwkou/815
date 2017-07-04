<properties
	pageTitle="如何使用 Azure 经典管理门户缩放媒体处理"
	description="了解如何通过指定要为帐户预配的“按需流式处理保留单元”和“编码保留单元”数，缩放媒体服务。"
	services="media-services"
	documentationCenter=""
	authors="milangada"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/15/2016"
	wacn.date="06/15/2017"
	ms.author="juliako;milanga"/>


# 如何使用 Azure 经典管理门户缩放媒体处理

## 通过其他技术完成此任务

此页概述了如何缩放媒体处理，并说明了如何使用 Azure 经典管理门户来完成它。还可以通过下面的其他技术来完成此任务：

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encoding-units/)
- [门户](/documentation/articles/media-services-portal-encoding-units/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/media/operations/encodingreservedunittype)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

## 概述

媒体服务帐户与保留单元类型关联，后者决定了编码处理任务的处理速度。可以在以下保留单元类型之间进行选择：**S1**、**S2** 或 **S3**。例如，与 **S1** 类型相比，使用 **S2** 保留单元类型时，相同的编码作业运行速度更快。有关详细信息，请参阅[保留单元类型](https://azure.microsoft.com/blog/high-speed-encoding-with-azure-media-services/)。

除了指定保留单元类型，还可以指定通过保留单元来预配帐户。预配的保留单元数决定了给定帐户中可同时运行的媒体任务数。例如，如果帐户具有 5 个保留单元，则只要有待处理任务，就可以同时运行 5 个媒体任务。其余任务将排队等待，正在运行的任务完成后才会选择它们以按顺序进行处理。如果帐户未预配任何保留单元，则按顺序选择任务进行处理。在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

## 在不同的保留单元类型之间进行选择

下表可帮助你在不同的编码速度之间进行选择时做出决定。它还提供了几个基准案例，并提供可用于下载视频的 SAS URL，你可以对该视频执行自己的测试：

方案|**S1**|**S2**|**S3**|
----------|------------|----------|------------
预期的用例| 单比特率编码。<br/>处于 SD 或较低分辨率的文件，对时间不敏感，成本低。|单比特率和多比特率编码。<br/>针对 SD 和 HD 编码的正常使用情况。 |单比特率和多比特率编码。<br/>全高清和 4K 分辨率视频。对时间敏感，更快的编码周转。 
基准|输入文件：5 分钟、640x360p、29.97 帧/秒。<br/><br/>编码为具有相同分辨率的单比特率 MP4 文件大约需要 11 分钟。|输入文件：5 分钟、1280x720p、29.97 帧/秒<br/><br/>使用“H264 单比特率 720p”预设值进行编码大约需要 5 分钟。<br/><br/>使用“H264 多比特率 720p”预设值进行编码大约需要 11.5 分钟。|输入文件：5 分钟、1920x1080p、29.97 帧/秒。<br/><br/>使用“H264 单比特率 1080p”预设值进行编码大约需要 2.7 分钟。<br/><br/>使用“H264 多比特率 1080p”预设值进行编码大约需要 5.7 分钟。

##注意事项

>[AZURE.IMPORTANT] 请注意以下事项：

- 保留单元可用于并行化所有媒体处理，其中包括使用 Azure Media Indexer 的索引编制作业。但是，与编码不同，使用更快的保留单元并不会加快索引编制作业的处理速度。

- 如果使用共享的池，即没有任何保留单元，编码任务将具有与 S1 RU 相同的性能。但是，任务在排队状态下花费的时间可能没有上限，并且在任何给定的时间内，最多会运行一项任务。



- 为 24 小时时间段指定的最大单位数将用于计算成本。

## 更改保留单元类型

若要更改保留单元类型和保留单元数目，请执行以下操作：

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中单击“媒体服务”。然后，单击媒体服务的名称。

2. 选择“编码”页。

	若要更改“保留单元类型”，请按“S1”、“S2”或“S3”。

	若要更改所选保留单元类型的保留单元数，请使用“编码”滑块。


	![“处理器”页](./media/media-services-portal-encoding-units/media-services-encoding-scale.png)

3. 按“保存”按钮保存更改。

	按“保存”后，会立即分配新的保留单元。
 

##配额和限制

有关配额和限制以及如何在线申请支持创建工单的信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。




 

<!---HONumber=Mooncake_Quality_Review_1202_2016-->