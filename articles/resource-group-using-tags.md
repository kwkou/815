<properties
	pageTitle="使用标记来组织 Azure 资源 | Microsoft Azure"
	description="演示如何应用标记来组织资源进行计费和管理。"
	services="azure-resource-manager"
	documentationCenter=""
	authors="tfitzmac"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="azure-resource-manager"
	ms.date="11/11/2015"
	wacn.date="12/31/2015"/>


# 使用标记来组织 Azure 资源

资源管理器使您可以通过应用标记来按照逻辑组织资源。标记由通过您所定义的属性标识资源的键值对组成。若要将资源标记为属于同一类别，请将相同的标记应用到这些资源。

查看具有特定标记的资源时，您会看到所有资源组中的资源。您并不一定只能在相同资源组中组织资源，还能够以独立于部署关系的方式组织资源。当您需要组织资源以进行计费或管理时，标记会特别有用。

您添加到资源或资源组的每个标记都会自动添加到订阅范围的分类。您也可以将标记名称预先填入订阅的分类，而且您想要作为资源使用的值会在未来加以标记。

每个资源或资源组最多可以有 15 个标记。标记名称不能超过 512 个字符，标记值不能超过 256 个字符。

> [AZURE.NOTE]您只能将标记应用到支持资源管理器操作的资源。如果你是通过经典部署模型（如通过 Azure 门户创建的虚拟机、虚拟网络或存储，则无法将标记应用到该资源。您必须通过资源管理器重新部署这些资源才能支持标记。所有其他资源均支持标记。

## 模板中的标记

在部署期间，可以十分方便地将标记添加到资源。只需将 **tags** 元素添加到你要部署的资源，然后提供标记名称和值即可。订阅中不需要预先存在标记名称和值。最多可为每个资源提供 15 个标记。

以下示例显示了一个包含标记的存储帐户。

    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2015-06-15",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[resourceGroup().location]",
            "tags": {
                "dept": "Finance"
            },
            "properties": 
            {
                "accountType": "Standard_LRS"
            }
        }
    ]


## 使用 PowerShell 进行标记

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]

标记直接存在于资源和资源组中，因此，若要查看已应用了哪些标记，只需使用 **Get-AzureRmResource** 或 **Get-AzureRmResourceGroup** 获取资源或资源组。让我们从一个资源组着手。

    PS C:\> Get-AzureRmResourceGroup tag-demo

    ResourceGroupName : tag-demo
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
    Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *

    Resources         :
                    Name                             Type                                  Location
                    ===============================  ====================================  ==============
                    CPUHigh ExamplePlan              microsoft.insights/alertrules         chinaeast
                    ForbiddenRequests tag-demo-site  microsoft.insights/alertrules         chinaeast
                    LongHttpQueue ExamplePlan        microsoft.insights/alertrules         chinaeast
                    ServerErrors tag-demo-site       microsoft.insights/alertrules         chinaeast
                    ExamplePlan-tag-demo             microsoft.insights/autoscalesettings  chinaeast
                    tag-demo-site                    microsoft.insights/components         chinaeast
                    ExamplePlan                      Microsoft.Web/serverFarms             chinaeast
                    tag-demo-site                    Microsoft.Web/sites                   chinaeast


此 cmdlet 将返回有关资源组的元数据的多个片段，包括已应用了哪些标记（如果有）。若要标记资源组，只需使用 **Set-AzureRmResourceGroup** 命令并指定标记名称和值。

    PS C:\> Set-AzureRmResourceGroup tag-demo -Tag @( @{ Name="project"; Value="tags" }, @{ Name="env"; Value="demo"} )

    ResourceGroupName : tag-demo
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                    Name     Value
                    =======  =====
                    project  tags
                    env      demo

标记作为一个整体进行更新，因此，如果要将一个标记添加到已标记的资源，需要使用一个数组，其中包含您要保留的所有标记。为此，可以首先选择现有标记，然后添加一个新标记。

    PS C:\> $tags = (Get-AzureRmResourceGroup -Name tag-demo).Tags
    PS C:\> $tags += @{Name="status";Value="approved"}
    PS C:\> Set-AzureRmResourceGroup tag-demo -Tag $tags

    ResourceGroupName : tag-demo
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                    Name     Value
                    =======  ========
                    project  tags
                    env      demo
                    status   approved


若要删除一个或多个标记，只需保存不包含您要删除的标记的数组。

该过程对于资源是相同，不过，使用的是 **Get-AzureRmResource** 和 **Set-AzureRmResource** cmdlet。

若要获取具有特定标记的资源组，请结合 **-Tag** 参数使用 **Find-AzureRmResourceGroup** cmdlet。

    PS C:\> Find-AzureRmResourceGroup -Tag @{ Name="env"; Value="demo" } | %{ $_.ResourceGroupName }
    rbacdemo-group
    tag-demo

对于版本低于 1.0 预览版的 Azure PowerShell，请结合特定的标记使用以下命令来获取资源。

    PS C:\> Get-AzureResourceGroup -Tag @{ Name="env"; Value="demo" } | %{ $_.ResourceGroupName }
    rbacdemo-group
    tag-demo
    PS C:\> Get-AzureResource -Tag @{ Name="env"; Value="demo" } | %{ $_.Name }
    rbacdemo-web
    rbacdemo-docdb
    ...    

若要使用 PowerShell 获取订阅中所有标记的列表，请使用 **Get-AzureRmTag** cmdlet。

    PS C:/> Get-AzureRmTag
    Name                      Count
    ----                      ------
    env                       8
    project                   1

你可能会看到，有些标记以“hidden-”和“link:”开头。这属于内部标记，你应忽略这些标记并避免更改。

使用 **New-AzureRmTag** cmdlet 可将新标记添加到分类。即使这些标记尚未应用到任何资源或资源组，也会包含在自动填充内容中。若要删除某个标记名称/值，请先从它可能已应用到的所有资源中将它删除，然后使用 **Remove-AzureRmTag** cmdlet 从分类中将它删除。

## 使用 REST API 进行标记

门户和 PowerShell 在幕后都使用[资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn848368.aspx)。如果需要将标记集成到另一个环境中，可以对资源 ID 使用 GET 以获取标记，并使用 PATCH 调用更新标记集。


## 标记和计费

对于受支持的服务，您可以使用标记对计费数据进行分组。如果您针对不同组织运行多个虚拟机，可以使用标记根据成本中心对使用情况进行分组。您还可以使用标记根据运行时环境对成本进行分类；例如，在生产环境中运行的虚拟机的计费使用情况。

您还可以使用标记根据运行时环境对成本进行分类；例如，在生产环境中运行的虚拟机的计费使用情况。

你可以通过 [Azure 资源使用情况与费率卡 API](/documentation/articles/billing-usage-rate-card-overview) 或者可从 [Azure 帐户门户](https://account.windowsazure.cn/)下载的使用情况逗号分隔值 (CSV) 文件来检索有关标记的信息。有关以编程方式访问计费信息的详细信息，请参阅[深入了解你的 Microsoft Azure 资源消耗](/documentation/articles/billing-usage-rate-card-overview)。有关 REST API 操作，请参阅 [Azure 计费 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)。

在你为支持标记和计费的服务下载使用情况 CSV 时，标记将显示在“标记”列中。有关更多详细信息，请参阅[了解 Microsoft Azure 的计费](/documentation/articles/billing-understand-your-bill)。

![在计费中查看标记](./media/resource-group-using-tags/billing_csv.png)

## 后续步骤

- 你可以使用自定义策略对订阅应用限制和约定。你定义的策略可能要求为所有资源设置特定的标记。有关详细信息，请参阅[使用策略来管理资源和控制访问](/documentation/articles/resource-manager-policy)。
- 有关部署资源时使用 Azure PowerShell 的说明，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。
- 有关部署资源时使用 Azure CLI 的说明，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager)。

<!---HONumber=Mooncake_1221_2015-->