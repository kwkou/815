<properties
    pageTitle="删除 Azure 群集及其资源 | Azure"
    description="了解如何通过删除包含该群集的资源组或有选择地删除资源彻底删除 Service Fabric 群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="ChackDan"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="de422950-2d22-4ddb-ac47-dd663a946a7e"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/24/2017"
    wacn.date="05/15/2017"
    ms.author="chackdan"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="cfd14b3dbcabdcd4ebd43a8b0860f308872e62d2"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="delete-a-service-fabric-cluster-on-azure-and-the-resources-it-uses"></a>在 Azure 上删除 Service Fabric 群集及其所用资源
Service Fabric 群集由群集资源本身及众多其他 Azure 资源组成。 因此，若要彻底删除 Service Fabric 群集，还需删除组成该群集的所有资源。
可使用两种方法：删除该群集所在的资源组（此操作将删除该资源组中的群集资源及其他任何资源），或特定删除群集资源及其关联资源（而不删除资源组中的其他资源）。

> [AZURE.NOTE]
> 删除群集资源**不会**删除组成 Service Fabric 的其他所有资源。
> 
> 

## <a name="delete-the-entire-resource-group-rg-that-the-service-fabric-cluster-is-in"></a>删除 Service Fabric 群集所在的整个资源组 (RG)
这是确保删除与群集相关联的所有资源（包括资源组）的最简单方法。 可以通过 PowerShell 或 Azure 门户删除资源组。 如果资源组中具有与 Service Fabric 群集不相关的资源，则可以删除特定资源。

### <a name="delete-the-resource-group-using-azure-powershell"></a>使用 Azure PowerShell 删除资源组
也可通过运行以下 Azure PowerShell cmdlet 删除资源组。 请确保计算机上已安装 Azure PowerShell 1.0 或更高版本。 如果尚未安装，请按照[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 中所述的步骤进行安装

打开 PowerShell 窗口并运行以下 PS cmdlet：


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	
	Remove-AzureRmResourceGroup -Name <name of ResouceGroup> -Force


如果未使用 *-Force* 选项，则将提示确认删除。确认后，即会删除 RG 及其包含的所有资源。

### <a name="delete-a-resource-group-in-the-azure-portal-preview"></a>在 Azure 门户中删除资源组
1. 登录到 [Azure 门户](https://portal.azure.cn)。
2. 导航到要删除的 Service Fabric 群集。
3. 在群集概要页上单击资源组名称。
4. 此时将显示“资源组概要”  页。
5. 单击“删除” 。
6. 按照该页面上的说明进行操作，以完成资源组的删除。

![资源组删除][ResourceGroupDelete]

## <a name="delete-the-cluster-resource-and-the-resources-it-uses-but-not-other-resources-in-the-resource-group"></a>删除群集资源及其所用资源，但不删除资源组中的其他资源
如果资源组仅包含与要删除的 Service Fabric 群集相关的资源，那么删除整个资源组更方便。 如果想有选择地逐个删除资源组中的资源，请按照下以步骤进行操作。

如果使用门户或通过模板库中的 Service Fabric Resource Manager 模板部署群集，该群集使用的所有资源都带有以下两个标记。 可以使用它们来确定要删除的资源。

***标记 1：***键 = clusterName，值 = '群集名称'

***标记 2：***键 = resourceName，值 = ServiceFabric

### <a name="delete-specific-resources-in-the-azure-portal-preview"></a>在 Azure 门户中删除特定资源

1. 登录到 [Azure 门户](https://portal.azure.cn)。
2. 导航到要删除的 Service Fabric 群集。
3. 在“基本信息”边栏选项卡上转到“**所有设置**”。
4. 在“设置”边栏选项卡中，单击“**资源管理**”下的“**标记**”。
5. 在“标记”边栏选项卡中单击“**标记**”之一，以获取带有该标记的所有资源的列表。
   
    ![资源标记][ResourceTags]

6. 获得带标记的资源的列表后，单击每个资源并将其删除。

    ![带标记的资源][TaggedResources]

### <a name="delete-the-resources-using-azure-powershell"></a>使用 Azure PowerShell 删除资源

可通过运行以下 Azure PowerShell cmdlet 逐个删除资源。 请确保计算机上已安装 Azure PowerShell 1.0 或更高版本。 如果尚未安装，请按照[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 中所述的步骤进行安装

打开 PowerShell 窗口并运行以下 PS cmdlet：


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

对要删除的每项资源，运行以下命令：


	Remove-AzureRmResource -ResourceName "<name of the Resource>" -ResourceType "<Resource Type>" -ResourceGroupName "<name of the resource group>" -Force


若要删除群集资源，请运行以下命令：


	Remove-AzureRmResource -ResourceName "<name of the Resource>" -ResourceType "Microsoft.ServiceFabric/clusters" -ResourceGroupName "<name of the resource group>" -Force

## <a name="next-steps"></a>后续步骤
参阅以下文章以了解如何升级群集以及对服务进行分区：

* [了解群集升级](/documentation/articles/service-fabric-cluster-upgrade/)
* [了解如何为有状态服务分区以最大程度地实现缩放](/documentation/articles/service-fabric-concepts-partitioning/)

<!--Image references-->

[ResourceGroupDelete]: ./media/service-fabric-cluster-delete/ResourceGroupDelete.PNG

[ResourceTags]: ./media/service-fabric-cluster-delete/ResourceTags.png

[TaggedResources]: ./media/service-fabric-cluster-delete/TaggedResources.PNG

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description:update relative links to absolute links -->