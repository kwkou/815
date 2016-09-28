<properties 
	pageTitle="如何添加编码单元" 
	description="了解如何使用 .NET 添加编码单元"  
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="07/26/2016"
	wacn.date="09/28/2016"/>


#如何使用 .NET SDK 缩放编码


> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-portal-encoding-units/)
- [.NET](/documentation/articles/media-services-dotnet-encoding-units/)
- [REST](https://msdn.microsoft.com/zh-cn/library/azure/dn859236.aspx)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

##概述

媒体服务帐户与保留单元类型相关联，后者决定了编码作业的处理速度。你可以在以下保留单位类型之间进行选择：S1、S2 或 S3。例如，与“基本”保留单元类型相比，使用“标准”类型时，相同的编码作业运行速度更快。有关详细信息，请参阅 [Milan Gada](https://azure.microsoft.com/blog/high-speed-encoding-with-azure-media-services/) 撰写的“编码保留单位类型”。

除了指定保留单元类型，你还可以指定如何通过媒体保留单元来设置帐户。设置的媒体保留单元数决定了给定帐户中可并发处理的媒体任务数。例如，如果你的帐户具有 5 个保留单元，则只要有任务要处理，就可以同时运行 5 个媒体任务。其余任务将排队等待，运行的任务完成后才选择它们以按顺序进行处理。如果帐户未设置任何保留单元，则按顺序选择任务进行处理。在这种情况下，完成一个任务和开始下一个任务之间的等待时间将取决于系统中资源的可用性。

若要使用 .NET SDK 更改保留单元类型和媒体保留单元数目，请执行以下操作：

	IEncodingReservedUnit encodingS1ReservedUnit = _context.EncodingReservedUnits.FirstOrDefault();
	encodingS1ReservedUnit.ReservedUnitType = ReservedUnitType.Basic; // Corresponds to S1
	encodingS1ReservedUnit.Update();
	Console.WriteLine("Reserved Unit Type: {0}", encodingS1ReservedUnit.ReservedUnitType);
	
	encodingS1ReservedUnit.CurrentReservedUnits = 2;
	encodingS1ReservedUnit.Update();
	
	Console.WriteLine("Number of reserved units: {0}", encodingS1ReservedUnit.CurrentReservedUnits);

##在线申请支持

默认情况下，每个媒体服务帐户最多可缩放到 25 个媒体保留单元和 5 个点播流保留单元。你可以通过[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单申请更高的限制值。



<!---HONumber=Mooncake_0919_2016-->