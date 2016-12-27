<properties
    pageTitle="使用标记组织 Azure 资源 | Azure"
    description="演示如何应用标记来组织资源进行计费和管理。"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />  

<tags
    ms.assetid="003a78e5-2ff8-4685-93b4-e94d6fb8ed5b"
    ms.service="azure-resource-manager"
    ms.workload="multiple"
    ms.tgt_pltfrm="AzurePortal"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/08/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />

# 使用标记来组织 Azure 资源
资源管理器使您可以通过应用标记来按照逻辑组织资源。标记由通过您所定义的属性标识资源的键值对组成。若要将资源标记为属于同一类别，请将相同的标记应用到这些资源。

查看具有特定标记的资源时，会看到所有资源组中的资源。不仅可以组织同一资源组中的资源，还能够以独立于部署关系的方式组织资源。需要为计费或管理目的组织资源时，标记可能十分有用。

添加到资源或资源组的每个标记都会自动添加到订阅范围的分类。您也可以将标记名称预先填入订阅的分类，而且您想要作为资源使用的值会在未来加以标记。

每个资源或资源组最多可以有 15 个标记。标记名称不能超过 512 个字符，标记值不能超过 256 个字符。

> [AZURE.NOTE]
您只能将标记应用到支持资源管理器操作的资源。如果通过经典部署模型（如通过经典门户）创建虚拟机、虚拟网络或存储，则无法向该资源应用标记。若要支持标记，需通过 Resource Manager 重新部署这些资源。所有其他资源均支持标记。
> 
> 

## 模板
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

Resource Manager 当前不支持处理标记名称和值对象。可以传递标记值对象，但仍需指定标记名称，如以下示例所示。

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


## 门户
[AZURE.INCLUDE [resource-manager-tag-resource](../../includes/resource-manager-tag-resources.md)]

## PowerShell
[AZURE.INCLUDE [resource-manager-tag-resources](../../includes/resource-manager-tag-resources-powershell.md)]

## Azure CLI
[AZURE.INCLUDE [resource-manager-tag-resources-cli](../../includes/resource-manager-tag-resources-cli.md)]

## REST API
门户和 PowerShell 在幕后都使用[资源管理器 REST API](https://docs.microsoft.com/rest/api/resources/)。如果需要将标记集成到另一个环境中，可以对资源 ID 使用 GET 以获取标记，并使用 PATCH 调用更新标记集。

## 标记和计费
对于受支持的服务，您可以使用标记对计费数据进行分组。例如，通过 Azure Resource Manager 部署虚拟机即可定义并应用标签来组织虚拟机的计费使用情况。如果您针对不同组织运行多个虚拟机，可以使用标记根据成本中心对使用情况进行分组。您还可以使用标记根据运行时环境对成本进行分类；例如，在生产环境中运行的虚拟机的计费使用情况。

可以通过 [Azure 资源使用情况与费率卡 API](/documentation/articles/billing-usage-rate-card-overview/) 或者使用情况逗号分隔值 (CSV) 文件检索有关标记的信息。可以从 [Azure 帐户门户](https://account.windowsazure.cn/)下载使用情况文件。有关以编程方式访问计费信息的详细信息，请参阅[深入了解你的 Azure 资源消耗](/documentation/articles/billing-usage-rate-card-overview/)。有关 REST API 操作，请参阅 [Azure 计费 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)。

在为支持标记和计费的服务下载使用情况 CSV 时，标记将显示在“标记”列中。有关更多详细信息，请参阅[了解 Azure 的计费](/documentation/articles/billing-understand-your-bill/)。

![在计费中查看标记](./media/resource-group-using-tags/billing_csv.png)

## 后续步骤
* 可以使用自定义策略对订阅应用限制和约定。定义的策略可能要求所有资源都拥有针对特定标记的值。有关详细信息，请参阅[使用策略来管理资源和控制访问](/documentation/articles/resource-manager-policy/)。
* 有关部署资源时使用 Azure PowerShell 的说明，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)。
* 有关部署资源时使用 Azure CLI 的说明，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager/)。
* 有关使用门户的说明，请参阅 [Using the Azure portal preview to manage your Azure resources](/documentation/articles/resource-group-portal/)（使用 Azure 门户预览管理 Azure 资源）
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_1219_2016-->