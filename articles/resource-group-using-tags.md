<!-- Remove Resource-group-portal -->
<properties
	pageTitle="使用标记组织 Azure 资源 | Azure"
	description="演示如何应用标记来组织资源进行计费和管理。"
	services="azure-resource-manager"
	documentationCenter=""
	authors="tfitzmac"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="azure-resource-manager"
	ms.date="04/18/2016"
	wacn.date="06/27/2016"/>


# 使用标记来组织 Azure 资源

资源管理器使您可以通过应用标记来按照逻辑组织资源。标记由通过您所定义的属性标识资源的键值对组成。若要将资源标记为属于同一类别，请将相同的标记应用到这些资源。

查看具有特定标记的资源时，您会看到所有资源组中的资源。您并不一定只能在相同资源组中组织资源，还能够以独立于部署关系的方式组织资源。当您需要组织资源以进行计费或管理时，标记会特别有用。

您添加到资源或资源组的每个标记都会自动添加到订阅范围的分类。您也可以将标记名称预先填入订阅的分类，而且您想要作为资源使用的值会在未来加以标记。

每个资源或资源组最多可以有 15 个标记。标记名称不能超过 512 个字符，标记值不能超过 256 个字符。

> [AZURE.NOTE] 您只能将标记应用到支持资源管理器操作的资源。如果通过经典部署模型（如通过 Azure 门户或服务管理 API）创建虚拟机、虚拟网络或存储器，则无法向该资源应用标记。您必须通过资源管理器重新部署这些资源才能支持标记。所有其他资源均支持标记。

## 模板中的标记

若要在部署过程中标记资源，只需将 **tags** 元素添加到正在部署的资源，然后提供标记名称和值即可。订阅中不需要预先存在标记名称和值。最多可为每个资源提供 15 个标记。

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

Resource Manager 当前不支持处理标记名称和值对象。但可以传递标记值对象，不过仍然必须指定标记名称（如下所示）。

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "tagvalues": {
          "type": "object",
          "defaultValue": {
            "dept": "Finance",
            "project": "Test"
          }
        }
      },
      "resources": [
      {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Storage/storageAccounts",
        "name": "examplestorage",
        "tags": {
          "dept": "[parameters('tagvalues').dept]",
          "project": "[parameters('tagvalues').project]"
        },
        "location": "[resourceGroup().location]",
        "properties": {
          "accountType": "Standard_LRS"
        }
      }]
    }


## 标记和 PowerShell

标记直接存在于资源和资源组中，因此，若要查看已应用了哪些标记，只需使用 **Get-AzureRmResource** 或 **Get-AzureRmResourceGroup** 获取资源或资源组。让我们从一个资源组着手。

    Get-AzureRmResourceGroup -Name tag-demo-group

此 cmdlet 将返回有关资源组的元数据的多个片段，包括已应用了哪些标记（如果有）。

    ResourceGroupName : tag-demo-group
    Location          : chinaeast
    ProvisioningState : Succeeded
    Tags              :
                    Name         Value
                    ===========  ==========
                    Dept         Finance
                    Environment  Production

获取某个资源的元数据时，不会直接显示标记。

    Get-AzureRmResource -ResourceName tfsqlserver -ResourceGroupName tag-demo-group

你会在结果中看到，标记只作为 Hashtable 对象显示。

    Name              : tfsqlserver
    ResourceId        : /subscriptions/{guid}/resourceGroups/tag-demo-group/providers/Microsoft.Sql/servers/tfsqlserver
    ResourceName      : tfsqlserver
    ResourceType      : Microsoft.Sql/servers
    Kind              : v12.0
    ResourceGroupName : tag-demo-group
    Location          : chinaeast
    SubscriptionId    : {guid}
    Tags              : {System.Collections.Hashtable}

可以通过检索 **Tags** 属性来查看实际标记。

    (Get-AzureRmResource -ResourceName tfsqlserver -ResourceGroupName tag-demo-group).Tags | %{ $_.Name + ": " + $_.Value }
   
该操作返回格式化结果：
    
    Dept: Finance
    Environment: Production

建议检索具有特定标记和值的所有资源或资源组，而不是查看特定资源组或资源的标记。若要获取具有特定标记的资源组，请结合 **-Tag** 参数使用 **Find-AzureRmResourceGroup** cmdlet。

    Find-AzureRmResourceGroup -Tag @{ Name="Dept"; Value="Finance" } | %{ $_.Name }
    
该操作返回带有该标记值的资源组的名称。
   
    tag-demo-group
    web-demo-group

若要获取具有特定标记和值的所有资源，请使用 **Find-AzureRmResource** cmdlet。

    Find-AzureRmResource -TagName Dept -TagValue Finance | %{ $_.ResourceName }
    
该操作返回带有该标记值的资源的名称。
    
    tfsqlserver
    tfsqldatabase

若要向当前没有标记的资源组添加标记，只需使用 **Set-AzureRmResourceGroup** 命令并指定一个标记对象。

    Set-AzureRmResourceGroup -Name test-group -Tag @( @{ Name="Dept"; Value="IT" }, @{ Name="Environment"; Value="Test"} )

该操作返回带有新的标记值的资源组。

    ResourceGroupName : test-group
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                    Name          Value
                    =======       =====
                    Dept          IT
                    Environment   Test
                    
可以使用 **Set-AzureRmResource** 命令向当前没有标记的资源添加标记。

    Set-AzureRmResource -Tag @( @{ Name="Dept"; Value="IT" }, @{ Name="Environment"; Value="Test"} ) -ResourceId /subscriptions/{guid}/resourceGroups/test-group/providers/Microsoft.Web/sites/examplemobileapp

标记作为一个整体进行更新，因此，如果要将一个标记添加到已标记的资源，需要使用一个数组，其中包含您要保留的所有标记。若要执行此操作，可以先选择现有标记，将新标记添加到该标记集，然后重新应用所有标记。

    $tags = (Get-AzureRmResourceGroup -Name tag-demo).Tags
    $tags += @{Name="status";Value="approved"}
    Set-AzureRmResourceGroup -Name test-group -Tag $tags

若要删除一个或多个标记，只需保存不包含您要删除的标记的数组。

该过程对于资源是相同，不过，使用的是 **Get-AzureRmResource** 和 **Set-AzureRmResource** cmdlet。

若要使用 PowerShell 获取订阅中所有标记的列表，请使用 **Get-AzureRmTag** cmdlet。

    Get-AzureRmTag
    Name                      Count
    ----                      ------
    env                       8
    project                   1

你可能会看到，有些标记以“hidden-”和“link:”开头。这属于内部标记，你应忽略这些标记并避免更改。

使用 **New-AzureRmTag** cmdlet 可将新标记添加到分类。即使这些标记尚未应用到任何资源或资源组，也会包含在自动填充内容中。若要删除某个标记名称/值，请先从它可能已应用到的所有资源中将它删除，然后使用 **Remove-AzureRmTag** cmdlet 从分类中将它删除。

## 标记和 Azure CLI

标记直接存在于资源和资源组中，因此，若要查看已应用了哪些标记，只需使用 **azure group show** 获取资源组及其资源。

    azure group show -n tag-demo-group
    
该操作返回有关资源组的元数据，包括任何应用到其中的标记。
    
    info:    Executing command group show
    + Listing resource groups
    + Listing resources for the group
    data:    Id:                  /subscriptions/{guid}/resourceGroups/tag-demo-group
    data:    Name:                tag-demo-group
    data:    Location:            westus
    data:    Provisioning State:  Succeeded
    data:    Tags: Dept=Finance;Environment=Production
    data:    Resources:
    data:
    data:      Id      : /subscriptions/{guid}/resourceGroups/tag-demo-group/providers/Microsoft.Sql/servers/tfsqlserver
    data:      Name    : tfsqlserver
    data:      Type    : servers
    data:      Location: eastus2
    data:      Tags    : Dept=Finance;Environment=Production
    ...

若要仅获取资源组的标记，请使用 JSON 实用工具，例如 [jq](http://stedolan.github.io/jq/download/)。

    azure group show -n tag-demo-group --json | jq ".tags"
    
该操作返回该资源组的标记。
    
    {
      "Dept": "Finance",
      "Environment": "Production" 
    }

使用 **azure resource show** 可以查看特定资源的标记。

    azure resource show -g tag-demo-group -n tfsqlserver -r Microsoft.Sql/servers -o 2014-04-01-preview --json | jq ".tags"
    
该操作返回该资源的标记。
    
    {
      "Dept": "Finance",
      "Environment": "Production"
    }
    
如下所示，可以检索具有特定标记和值的所有资源。

    azure resource list --json | jq ".[] | select(.tags.Dept == "Finance") | .name"
    
该操作返回带有该标记的资源的名称。
    
    "tfsqlserver"
    "tfsqlserver/tfsqldata"

标记作为一个整体进行更新，因此，如果要向已标记的资源添加一个标记，需要检索要保留的现有全部标记。若要为资源组设置标记值，请使用 **azure group set** 并提供该资源组的所有标记。

    azure group set -n tag-demo-group -t Dept=Finance;Environment=Production;Project=Upgrade
    
将返回带新标记的资源组的摘要。
    
    info:    Executing command group set
    ...
    data:    Name:                tag-demo-group
    data:    Location:            westus
    data:    Provisioning State:  Succeeded
    data:    Tags: Dept=Finance;Environment=Production;Project=Upgrade
    ...
    
可以使用 **azure tag list** 列出订阅中的现有标记，并使用 **azure tag create** 添加新标记。若要从订阅的分类中删除某个标记，请先从它可能已应用到的所有资源中将它删除，然后使用 **azure tag delete** 删除标记。

## 标记和 REST API

门户和 PowerShell 在幕后都使用[资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn848368.aspx)。如果需要将标记集成到另一个环境中，可以对资源 ID 使用 GET 以获取标记，并使用 PATCH 调用更新标记集。


## 标记和计费

对于受支持的服务，您可以使用标记对计费数据进行分组。例如，[与 Azure Resource Manager 集成的虚拟机](/documentation/articles/virtual-machines-windows-compare-deployment-models/)可让你定义并应用标签来组织虚拟机的计费使用情况。如果您针对不同组织运行多个虚拟机，可以使用标记根据成本中心对使用情况进行分组。您还可以使用标记根据运行时环境对成本进行分类；例如，在生产环境中运行的虚拟机的计费使用情况。

你可以从 [Azure 帐户门户](https://account.windowsazure.cn/)或 [EA 门户](https://ea.azure.com)下载的使用情况逗号分隔值 (CSV) 文件来检索有关标记的信息。有关以编程方式访问计费信息的详细信息，请参阅[深入了解你的 Microsoft Azure 资源消耗](/documentation/articles/billing-usage-rate-card-overview/)。有关 REST API 操作，请参阅 [Azure 计费 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)。

在你为支持标记和计费的服务下载使用情况 CSV 时，标记将显示在“标记”列中。有关更多详细信息，请参阅[了解 Azure 的计费](/documentation/articles/billing-understand-your-bill/)。

![在计费中查看标记](./media/resource-group-using-tags/billing_csv.png)

## 后续步骤

- 有关部署资源时使用 Azure PowerShell 的说明，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。
<!-- - 有关使用门户的说明，请参阅[使用 Azure 门户管理 Azure 资源](/documentation/articles/resource-group-portal/) -->

<!---HONumber=Mooncake_0620_2016-->