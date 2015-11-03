<properties 
	pageTitle="如何缩放编码保留单元" 
	description="了解如何通过指定要为帐户设置的“按需流式处理保留单位”和“编码保留单位”数，缩放媒体服务。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015"
	wacn.date="11/02/2015"/>


#如何缩放编码

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encoding-units)
- [Portal](/documentation/articles/media-services-portal-encoding-units)
- [REST](https://msdn.microsoft.com/zh-cn/library/azure/dn859236.aspx)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)

##概述

媒体服务帐户与保留单元类型相关联，后者决定了编码作业的处理速度。你可以在以下保留单位类型之间进行选择：**基本**或**标准**。例如，与**基本**保留单位类型相比，使用**标准**类型时，相同的编码作业运行速度更快。有关详细信息，请参阅[编码保留单位类型](http://azure.microsoft.com/blog/author/milanga)。

除了指定保留单元类型，你还可以指定如何通过编码保留单元来设置帐户。设置的编码保留单元数决定了给定帐户中可并发处理的媒体任务数。例如，如果你的帐户具有 5 个保留单元，则只要有任务要处理，就可以同时运行 5 个媒体任务。其余任务将排队等待，运行的任务完成后才选择它们以按顺序进行处理。如果帐户未设置任何保留单元，则按顺序选择任务进行处理。在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

若要更改保留单元类型和编码保留单元数目，请执行以下操作：

1. 在[管理门户](https://manage.windowsazure.cn/)中单击 **媒体服务**。然后，单击媒体服务的名称。

2. 选择“编码”页。

	若要更改“保留单位类型”，请按“基本”或“标准”。

	若要更改所选保留单位类型的保留单位数，请使用“编码”滑块。
	
	
	![“处理器”页](./media/media-services-portal-encoding-units/media-services-encoding-scale.png)

	  

3. 按“保存”按钮保存更改。

	按“保存”后，会立即分配新的编码保留单元。

	>[AZURE.NOTE]为 24 小时期间指定的最大单位数将用于计算成本。

##配额和限制

有关配额和限制以及如何开具支持票证的信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations)。

<!---HONumber=76-->