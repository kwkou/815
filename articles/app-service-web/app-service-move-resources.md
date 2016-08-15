<properties
	pageTitle="将 Web 应用资源移到另一个资源组"
	description="介绍将 Web 应用从一个资源组移到另一个资源组的方案。"
	services="app-service"
	documentationCenter=""
	authors="ZainRizvi"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="01/04/2016"
	wacn.date="03/03/2016"/>
	
# 受支持的迁移配置

你可以使用 [ARM 迁移资源 API](/documentation/articles/resource-group-move-resources/) 迁移 Azure Web 应用资源。

Azure Web 应用当前支持以下迁移方案：

* 将一个资源组的整个内容（ Web 应用、App Service 计划和证书）移到另一个资源组 
	* 请注意：在此方案中，目标资源组不能包含任何 Microsoft.Web 资源
* 将单独 Web 应用移到不同的资源组，同时仍然在其当前 App Service 计划中托管这些 Web 应用（该 App Service 计划保留在旧资源组中）

<!---HONumber=Mooncake_0118_2016-->