<properties 
	pageTitle="如何缩放媒体服务" 
	description="了解如何通过指定要为帐户设置的“按需流式处理保留单位”和“编码保留单位”数，缩放媒体服务。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/20/2016"/>


#如何缩放媒体服务  

##概述

通过指定要为帐户预配的**串流保留单位**和**编码保留单位**的数量，可以缩放**媒体服务**。

也可以通过向媒体服务帐户添加存储帐户来缩放该帐户。每个存储帐户大小限制为 500 TB。若要在默认限制之外扩展存储，可选择将多个存储帐户附加到单个媒体服务帐户。

本主题链接到相关的主题。

##<a id="streaming_endpoins"></a>流式处理保留单位

有关详细信息，请参阅[缩放流式处理单位](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。

##<a id="encoding_reserved_units"></a>编码保留单位

有关缩放编码单位的详细信息，请参阅以下**门户**和 **.NET** 主题。

[AZURE.INCLUDE [media-services-selector-scale-encoding-units](../includes/media-services-selector-scale-encoding-units.md)]

请注意，编码和索引任务的保留单位相同。

##<a id="storage"></a>缩放存储

有关详细信息，请参阅[跨多个存储帐户管理媒体服务资产](/documentation/articles/meda-services-managing-multiple-storage-accounts/)和[使用 Azure 存储空间](/documentation/services/storage/)。




<!---HONumber=Mooncake_0613_2016-->