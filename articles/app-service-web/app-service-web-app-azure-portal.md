<properties
	pageTitle="有关在 Azure 门户中导航的参考"
	description="了解在管理门户与 Azure 门户中使用 App Service Web 的不同用户体验"
	services="app-service"
	documentationCenter=""
	authors="jaime-espinosa"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="app-service"
	ms.date="02/26/2016"
	wacn.date=""/>

# 有关在 Azure 门户中导航的参考

Azure 网站现在称为 [App Service Web Apps](/documentation/articles/app-service-changes-existing-services/)。我们正在更新所有文档以反应此名称更改，并为 Azure 门户提供说明。完成该过程之前，你可以使用此文档作为在新 Azure 门户中使用 Web Apps 的指南。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]
 
## Azure 经典门户的未来趋势

当你发现 Azure 经典门户中品牌改变时，该门户正处于被 Azure 门户替代的过程中。由于正在淘汰经典门户，因此新的开发工作重点将转移到 Azure 门户。所有即将发布的 Web Apps 新功能将出现在 Azure 门户中。开始使用 Azure 门户以利用 Web Apps 提供最新最全的功能。

## Azure 经典门户与 Azure 门户之间的布局差异

在经典门户中，所有 Azure 服务都列在左侧。经典门户中的导航遵循树状结构，即从服务开始导航，然后再导航到各个元素。此结构适用于管理独立组件。但是，在 Azure 上构建的应用程序是互联服务的集合，而此树状结构并不适用于服务集合。

Azure 门户则可以利用多个服务中的组件端对端地轻松构建应用程序。该门户被组织为旅程。一段旅程由一系列边栏选项卡组成，即为不同组件的容器。例如，设置 Web 应用的自动缩放即为一段旅程，该旅程包含好几个边栏选项卡，如下例所示：“网站”边栏选项卡（该边栏选项卡名尚未更新至使用新术语）、“设置”边栏选项卡和“缩放”边栏选项卡。在此示例中，自动缩放被设置为依据 CPU 使用情况而定，因此还具有一个“CPU 百分比”边栏选项卡。边栏选项卡中的组件被称为部件，形状像磁贴。

![](./media/app-service-web-app-azure-portal/AutoScaling.png)

## 浏览示例：创建 Web 应用

创建新 Web 应用的步骤如 1-2-3 一样简单。下图并排显示了经典门户和新门户，以便说明创建和运行 Web 应用所需的步骤数并未发生太大变化。

![](./media/app-service-web-app-azure-portal/CreateWebApp.png)

在新门户中，你可以从最常用的 Web 应用类型中选择，包括 WordPress 等受欢迎的库应用程序。有关可用应用程序的完整列表，请访问 [Azure 库]。

当创建 Web 应用时，你可以如在经典门户中所执行的操作一样在新门户中指定 URL、App Service 计划和位置。

![](./media/app-service-web-app-azure-portal/CreateWebAppSettings.png)

此外，你可以在新门户中定义其他通用设置。例如，通过[资源组](/documentation/articles/resource-group-overview/)可以轻松查看和管理相关 Azure 资源。

## 浏览示例：设置和功能

现在所有设置和功能都逻辑分组在单个边栏选项卡中，进而方便从中进行导航。

![](./media/app-service-web-app-azure-portal/WebAppSettings.png)

例如，你可以通过单击“设置”边栏选项卡中的“自定义域和 SSL”来创建自定义域。

![](./media/app-service-web-app-azure-portal/ConfigureWebApp.png)

若要设置监视警报，请单击“请求和错误”，然后单击“添加警报”。

![](./media/app-service-web-app-azure-portal/Monitoring.png)

若要启用诊断，请单击“设置”边栏选项卡中的“诊断日志”。

![](./media/app-service-web-app-azure-portal/Diagnostics.png)
 
若要配置应用程序设置，请单击“设置”边栏选项卡中的“应用程序设置”。

![](./media/app-service-web-app-azure-portal/AppSettingsPreview.png)

除品牌名称之外，已对门户中的一些事物进行了重命名或进行了不同的分组，以便更容易查找。例如，下面是经典门户中应用设置（“配置”）相应页的屏幕截图。

![](./media/app-service-web-app-azure-portal/AppSettings.png)

## 更多资源

[Azure Management Portal]: https://portal.azure.cn
[Azure 库]: /marketplace/

## 发生的更改
* 有关从网站更改为 App Service 的指南，请参阅：[Azure App Service 及其对现有 Azure 服务的影响](/documentation/articles/app-service-changes-existing-services/)
 

<!---HONumber=Mooncake_0328_2016-->