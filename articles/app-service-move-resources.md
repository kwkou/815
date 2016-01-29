<properties
	pageTitle="将网站资源移到另一个资源组"
	description="介绍将网站和 Azure 网站从一个资源组移到另一个资源组的方案。"
	services="app-service"
	documentationCenter=""
	authors="ZainRizvi"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="10/29/2015"
	wacn.date="01/29/2016"/>
	
# 受支持的迁移配置

你可以使用 [ARM 迁移资源 API](/documentation/articles/resource-group-move-resources) 迁移 Azure 网站资源。

Azure 网站当前支持以下迁移方案：

* 将一个资源组的整个内容（网站、App Service 计划和证书）移到另一个资源组 
	* 请注意：在此方案中，目标资源组不能包含任何 Microsoft.Web 资源
* 将单独网站移到不同的资源组，同时仍然在其当前 App Service 计划中托管这些网站（该 App Service 计划保留在旧资源组中）

<!---HONumber=Mooncake_0118_2016-->