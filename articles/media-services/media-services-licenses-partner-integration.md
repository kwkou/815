<properties 
	pageTitle="使用合作伙伴将 Widevine 许可证传送到 Azure 媒体服务 | Azure" 
	description="本文介绍如何使用 Azure 媒体服务 (AMS) 传送 AMS 通过 PlayReady 和 Widevine DRM 动态加密的流。PlayReady 许可证来自媒体服务 PlayReady 许可证服务器，而 Widevine 许可证则由 castLabs 许可证服务器传送。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>  


<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/26/2016"  
	wacn.date="12/16/2016"  
	ms.author="juliako"/>


#使用合作伙伴将 Widevine 许可证传送到 Azure 媒体服务

##概述

Azure 媒体服务允许传送使用 Widevine DRM 保护的 MPEG DASH，这些内容已根据通用加密 (CENC) 规范加密。

从媒体服务 .NET SDK 版本 3.5.2 开始，媒体服务允许配置 Widevine 许可证模板并获取 Widevine 许可证。还可以使用以下 AMS 合作伙伴来传送 Widevine 许可证：[EZDRM](http://ezdrm.com/)、[castLabs](http://castlabs.com/company/partners/azure/)。

##castLabs

可以使用 [castLabs](http://castlabs.com/company/partners/azure/) 来传送 Widevine 许可证。有关详细信息，请参阅[使用 castLabs 将 DRM 许可证传送到 Azure 媒体服务](/documentation/articles/media-services-castlabs-integration/)


##另请参阅

[使用 PlayReady 和/或 Widevine 动态通用加密](/documentation/articles/media-services-protect-with-drm/)

[Mingfei 的博客](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->