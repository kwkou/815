<properties
   pageTitle="资源管理器支持的服务和支持的区域 | Windows Azure"
   description="介绍支持可托管资源的资源管理器和区域的资源提供程序。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="10/19/2015"
   wacn.date="11/27/2015"/>

# 服务、区域和 API 版本对资源管理器的支持

Azure 资源管理器为你提供了一种新的方式来部署和管理构成应用程序的服务。大多数（但并非所有）服务都支持资源管理器，有些服务仅部分支持资源管理器。Microsoft 将为每个服务启用资源管理器，这对于未来的解决方案而言很重要，但在全面提供支持之前，你需要了解每个服务的当前支持状态。本主题提供支持 Azure 资源管理器的资源提供程序列表。

部署资源时，你还需要知道哪些区域支持这些资源，以及哪些 API 版本可用于资源。[支持的区域](#supported-regions)部分说明了如何找出哪些区域支持你的订阅和资源。[支持的 API 版本](#supported-api-versions)部分说明了如何判断可以使用哪些 API 版本。

下表列出哪些服务可通过资源管理器支持部署和管理，哪些则不可以。标题为**移动资源**的列表示这种类型的资源是否可以移到新的资源组和新的订阅。标题为**预览门户**的列表示是否可以通过预览门户创建服务。


## 计算

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------------------------ | -------------- | -------------- |-------- | ------ |
| 虚拟机 | 是 | 是 | 否 | [创建 VM](https://msdn.microsoft.com/zh-cn/library/azure/mt163591.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Compute.json) |
| 批处理 | 是 | 否 | | [Batch REST](https://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx) | |
| 动态生命周期服务 | 是 | 否 | | | |
| 虚拟机（经典）| 有限 | 是 | 部分（参阅下文）| - | - |
| 远程应用 | 否 | - | - | - | - |
| Service Fabric | 否 | - | - | - | - |

虚拟机（经典）是指已通过经典部署模型部署的资源，而不是通过资源管理器部署模型部署的资源。一般而言，这些资源不支持资源管理器操作，但已启用某些操作。有关这些部署模型的详细信息，请参阅[了解资源管理器部署和经典部署](/documentation/articles/resource-manager-deployment-model)。

可以将虚拟机（经典）资源移到新的资源组，但不能移到新的订阅。

## Web 和移动

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | -------- | -------------- | -------- | ------ |
| API 管理| 是 | 否 | 是 | [创建 API](https://msdn.microsoft.com/zh-cn/library/azure/dn781423.aspx#CreateAPI) | |
| API Apps | 是 | 是 | | | [2015-03-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-03-01-preview/Microsoft.AppService.json) |
|网站| 是 | 是 | 是，但有限（参阅下文） | | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Web.json) |
| 通知中心 | 是 | 是 | | [创建通知中心](https://msdn.microsoft.com/zh-cn/library/azure/dn223269.aspx) | [2015-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-04-01/Microsoft.NotificationHubs.json) |
| Logic Apps | 是 | 是 | | | |
| Mobile Engagement | 是 | 否 | 是 | | |

当使用网站时，不能仅移动 App Service 计划。若要移动网站，您的选项包括：

- 如果目标资源组不具有 Microsoft.Web 资源，则将所有资源从一个资源组移到另一个资源组中。
- 将网站移到另一个资源组中，但保留原始资源组中的 App Service 计划。


## 数据和存储

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | ------- | -------------- | -------- | ------ |
| DocumentDB | 是 | 是 | 是 | [DocumentDB REST](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) | |
| 存储 | 是 | 是 | | [创建存储](https://msdn.microsoft.com/zh-cn/library/azure/mt163564.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Storage.json) |
| Redis Cache | 是 | 是 | 是 | | [2014-04-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01-preview/Microsoft.Cache.json) |
| SQL 数据库 | 是 | 是 | 是 | [创建数据库](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx) | [2014-04-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01-preview/Microsoft.Sql.json) |
| 搜索 | 是 | 是 | 是 | [搜索 REST](https://msdn.microsoft.com/zh-cn/library/azure/dn798935.aspx) | |
| SQL 数据仓库 | 是 | 是 | | | |
| StorSimple | 否 | 否 | - | - | - |
| 备份 | 否 | 否 | - | - | - |
| 站点恢复 | 否 | 否 | - | - | - |
| 托管缓存 | 否 | 否 | - | - | - |
| 数据目录 | 否 | 否 | - | - | - |

## 分析

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | --------- | -------------- | -------- | ------ |
| 事件中心 | 是 | 否 | | [创建事件中心](https://msdn.microsoft.com/zh-cn/library/azure/dn790676.aspx) | |
| 流分析 | 是 | 是 | | | |
| HDInsights | 是 | 是 | | | |
| Data Factory | 是 | 是 | 是 | [创建数据工厂](https://msdn.microsoft.com/zh-cn/library/azure/dn906717.aspx) | |
| 机器学习 | 否 | 否 | - | - | - |

## 联网

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | -------- | -------------- | -------- | ------ |
| 应用程序网关 | 是 | | | | |
| DNS | 是 | | | [创建 DNS 区域](https://msdn.microsoft.com/zh-cn/library/azure/mt130622.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) |
| 负载平衡器 | 是 | | | [创建负载平衡器](https://msdn.microsoft.com/zh-cn/library/azure/mt163574.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) |
| 虚拟网络 | 是 | 是 | 否 | [创建虚拟网络](https://msdn.microsoft.com/zh-cn/library/azure/mt163661.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) |
| 流量管理器 | 是 | 否 | | [创建流量管理器配置文件](https://msdn.microsoft.com/zh-cn/library/azure/mt163581.aspx) | |
| ExpressRoute | 是 | 否 | 否 | [ExpressRoute REST](https://msdn.microsoft.com/zh-cn/library/azure/mt586720.aspx) | |

## 媒体和 CDN

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | -------- | -------------- | -------- | ------ |
| 媒体服务 | 否 | 否 | | | |
| CDN | 否 | 否 | | | |

## 混合集成

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | -------------- | -------------- | -------- | ------ |
| BizTalk 服务 | 是 | 否 | | | [2014-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01/Microsoft.BizTalkServices.json) |
| 服务总线 | 是 | 否 | | [服务总线 REST](https://msdn.microsoft.com/zh-cn/library/azure/hh780717.aspx) | |

## 标识和访问管理 

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | -------------- | -------------- | -------- | ------ |
| Azure Active Directory | 否 | 否 | - | - | - | | Azure Actice Directory B2C | 否 | 否 | - | - | - | | Multi-Factor Authentication | 否 | 否 | - | - | - |

## 开发人员服务 

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | ---------- | -------------- | -------- | ------ |
| Application Insights | 是 | 是 | | | [2014-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01/Microsoft.Insights.json) |
| 必应地图 | 是 | 是 | | | |
| Visual Studio 帐户 | 是 | | | | [2014-02-26](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-02-26/microsoft.visualstudio.json) |

## 管理 

| 服务 | 已启用资源管理器 | 预览门户 | 移动资源 | REST API | 架构 |
| ------- | ------- | --------- | -------------- | -------- | ------ |
| 自动化 | 是 | 是 | | | |
| 密钥保管库 | 是 | 否 | 是 | [密钥保管库 REST](https://msdn.microsoft.com/zh-cn/library/azure/dn903609.aspx) | |
| 计划程序 | 是 | 否 | | | [2014-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-08-01/Microsoft.Scheduler.json) |
| 操作见解 | 是 | 否 | 是 | | |
| IoTHubs | 是 | 是 | | | |


## 支持的区域

部署资源时，通常需要指定资源的区域。所有区域都支持资源管理器，但部署的资源可能无法在所有区域中受到支持。此外，订阅上可能有一些限制，以防止使用某些支持该资源的区域。这些限制可能与所在国家/地区的税务问题有关，或者与由订阅管理员所放置，只能使用特定区域的策略结果有关。

部署资源之前，请运行以下命令之一，以检查你的资源类型支持的区域。

### REST API

若要发现哪些区域可供订阅中的特定资源类型使用，请使用[列出所有资源提供程序](https://msdn.microsoft.com/zh-cn/library/azure/dn790524.aspx)操作。

### PowerShell

以下示例演示如何使用 Azure PowerShell 1.0 预览版来获取支持网站的区域。有关 1.0 预览版的详细信息，请参阅 [Azure PowerShell 1.0 预览版](https://azure.microsoft.com/blog/azps-1-0-pre/)

    PS C:\> ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
    
输出结果将会类似于：

    China East
    China North

### Azure CLI

以下示例返回每个资源类型支持的所有位置。

    azure location list

你也可以使用 **jq** 之类的工具来筛选位置结果。

    azure location list --json | jq '.[] | select(.name == "Microsoft.Web/sites")'

将返回：

    {
      "name": "Microsoft.Web/sites",
      "location": "China North, China East"
    }

## 支持的 API 版本

部署模板时，必须指定要用于创建每个资源的 API 版本。API 版本对应于资源提供程序发布的 REST API 操作版本。资源提供程序启用新功能时，将会发布 REST API 的新版本。因此，在模板中指定的 API 版本会影响你可以在模板中指定的属性。通常，在创建新模板时，你需要选择最新的 API 版本。对于现有模板，你可以决定是要继续使用以前的 API 版本，还是要选择最新版本来更新模板以利用新功能。

### REST API

若要发现哪些 API 版本可供资源类型使用，请使用[列出所有资源提供程序](https://msdn.microsoft.com/zh-cn/library/azure/dn790524.aspx)操作。

### PowerShell

以下示例演示如何使用 Azure PowerShell 1.0 预览版来获取特定资源类型可用的 API 版本。

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).ApiVersions
    
输出结果将会类似于：
    
    2015-08-01
    2015-07-01
    2015-06-01
    2015-05-01
    2015-04-01
    2015-02-01
    2014-11-01
    2014-06-01
    2014-04-01-preview
    2014-04-01

### Azure CLI

可以使用以下命令将资源提供程序的信息（包括可用的 API 版本）保存到文件。

    azure provider show Microsoft.Web -vv --json > c:\temp.json

你可以打开该文件并查找 **apiVersions** 元素

## 后续步骤

- 若要了解如何创建资源管理器模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。

<!---HONumber=82-->