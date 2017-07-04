<properties
    pageTitle="通过添加编码单位调整媒体处理的规模 - Azure | Azure"
    description="了解如何使用 .NET 添加编码单位"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="33f7625a-966a-4f06-bc09-bccd6e2a42b5"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/10/2017"
    ms.author="juliako;milangada;" />

#如何使用 .NET SDK 缩放编码

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-portal-encoding-units/)
- [.NET](/documentation/articles/media-services-dotnet-encoding-units/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/media/operations/encodingreservedunittype)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

##概述

>[AZURE.IMPORTANT] 请确保查看[概述](/documentation/articles/media-services-scale-media-processing-overview/)主题，获取有关缩放媒体处理主题的详细信息。
 
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

<!---HONumber=Mooncake_0306_2017-->