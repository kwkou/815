<properties 
	pageTitle="导出 Azure Resource Manager 模板 | Azure" 
	description="使用 Azure Resource Manager 从现有资源组导出模板。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="timlt" 
	editor="tysonn"/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="05/10/2016" 
	wacn.date="07/11/2016"/>

# 从现有资源导出 Azure Resource Manager 模板

了解构造 Azure Resource Manager 模板如何艰巨，但 Resource Manager 可通过允许你从订阅中的现有资源导出模板来帮助你完成该任务。你可以使用该生成的模板了解模板语法，或根据需要自动重新部署解决方案。

在本教程中，你将通过门户创建存储帐户，并导出该存储帐户的模板。然后，你将通过向其添加虚拟网络来修改该资源组，并导出表示其当前状态的新模板。尽管本主题重点介绍简化的基础结构，但你也可以使用这些相同的步骤为更复杂的解决方案导出模板。

## 创建存储帐户

1. 在 [Azure 门户](https://portal.azure.cn)中，选择“新建”、“数据 + 存储”和“存储帐户”。

      ![创建存储](./media/resource-manager-export-template/create-storage.png)

2. 使用名称 **storage**、你的姓名缩写和日期创建存储帐户。存储帐户名称必须在 Azure 中唯一，因此，如果该名称已在使用，请尝试其他名称。对于资源组，使用 **ExportGroup**。可以对其他属性使用默认值。选择“创建”。

      ![提供存储的值](./media/resource-manager-export-template/provide-storage-values.png)

在部署完成后，你的订阅包含存储帐户。

## 导出部署模板
   
1. 导航到新资源组的资源组边栏选项卡。你将发现列出了上次部署的结果。选择此链接。

      ![资源组边栏选项卡](./media/resource-manager-export-template/resource-group-blade.png)
   
2. 你将看到该组的部署历史记录。在你的情况下，有可能只列出了一个部署。选择此部署。

     ![上次部署](./media/resource-manager-export-template/last-deployment.png)

3. 将显示此部署的摘要。摘要包括此部署及其操作的状态，以及为参数提供的值。若要查看用于部署的模板，请选择“查看模板”。

     ![查看部署摘要](./media/resource-manager-export-template/deployment-summary.png)

4. Resource Manager 将为你检索 5 个文件。它们具有以下特点：

   1. 定义解决方案基础结构的模板。通过门户创建存储帐户时，Resource Manager 使用模板来部署该存储帐户，并保存该模板供将来参考。

   2. 可用于在部署过程中传入值的参数文件。它包含在首次部署期间提供的值，但你可以在重新部署该模板时更改其中任何值。

   3. 可用于部署该模板的 Azure PowerShell 脚本文件。

   4. 可用于部署该模板的 Azure CLI 脚本文件。
   
   5. 可用于部署该模板的 .NET 类。

     可通过边栏选项卡上的链接获取这些文件。默认情况下，选中该模板。
     
       ![查看模板](./media/resource-manager-export-template/view-template.png)
     
     让我们特别注意该模板。你的模板的外观应类似于：
   
        {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "name": {
              "type": "String"
            },
            "accountType": {
              "type": "String"
            },
            "location": {
              "type": "String"
            },
            "encryptionEnabled": {
              "defaultValue": false,
              "type": "Bool"
            }
          },
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "sku": {
                "name": "[parameters('accountType')]"
              },
              "kind": "Storage",
              "name": "[parameters('name')]",
              "apiVersion": "2016-01-01",
              "location": "[parameters('location')]",
              "properties": {
                "encryption": {
                  "services": {
                    "blob": {
                      "enabled": "[parameters('encryptionEnabled')]"
                    }
                  },
                  "keySource": "Microsoft.Storage"
                }
              }
            } 
          ]
        }
   
     请注意，它为存储帐户名称、类型、位置以及是否启用了加密（其默认值为 **false**）定义了参数。在 **resources** 节中，将看到要部署的存储帐户的定义。
     
方括号包含将在部署过程中计算的表达式。上面所示的括在括号中的表达式用于在部署期间获取参数值。有更多表达式你可以使用，你将在本主题的后面部分看到其他表达式的示例。有关完整列表，请参阅 [Azure Resource Manager 模板函数](/documentation/articles/resource-group-template-functions)。
   
若要详细了解模板的结构，请参阅 [Authoring Azure Resource Manager templates（创作 Azure Resource Manager 模板）](/documentation/articles/resource-group-authoring-templates)。

## 添加虚拟网络

上一节中下载的模板表示该原始部署的基础结构，但它将不会记录在部署之后进行的任何更改。为了说明此问题，让我们通过门户添加虚拟网络来修改资源组。

1. 在资源组边栏选项卡中，选择“添加”并从可用资源中选取“虚拟网络”。
   
2. 将虚拟网络命名为 **VNET**，并对其他属性使用默认值。选择“创建”。

      ![设置警报](./media/resource-manager-export-template/create-vnet.png)
   
3. 在虚拟网络已成功部署到资源组后，再次查看部署历史记录。现在，你将看到两个部署。选择最新的部署。

      ![部署历史记录](./media/resource-manager-export-template/deployment-history.png)
   
4. 查看该部署的模板。请注意，它只定义了添加虚拟网络时所做的更改。

通常使用模板的最佳做法是在单个操作中部署解决方案的所有基础结构，而不是记住要部署的多个不同模板。


## 导出资源组的模板

虽然每个部署只显示已对资源组所做的更改，但你随时可以导出模板以显示整个资源组的属性。

1. 若要查看资源组模板，请选择“导出模板”。

      ![导出资源组](./media/resource-manager-export-template/export-resource-group.png)

2. 你将再次看到可用于重新部署解决方案的 5 个文件，但这一次该模板略有不同。该模板只有 2 个参数（一个用于存储帐户名称，一个用于虚拟网络名称）。

        "parameters": {
          "virtualNetworks_VNET_name": {
            "defaultValue": "VNET",
            "type": "String"
          },
          "storageAccounts_storagetf05092016_name": {
            "defaultValue": "storagetf05092016",
            "type": "String"
          }
        },
        
     Resource Manager 未检索到在部署期间使用的模板。但是，它将基于资源的当前配置生成新模板。Resource Manager 不知道你要以参数形式传入哪些值，因此它基于资源组中的值对大多数值进行了硬编码。例如，存储帐户位置和复制值将设置为：
     
        "location": "northeurope",
        "tags": {},
        "properties": {
            "accountType": "Standard_RAGRS"
        },

3. 下载该模板，以便可以在本地处理它。

      ![下载模板](./media/resource-manager-export-template/download-template.png)

4. 找到你下载的 .zip 文件并将内容解压缩。可以使用这个下载的模板重新部署你的基础结构。

## 后续步骤

祝贺你！ 你已学习如何从门户中创建的资源导出模板。

- 在本教程的第二部分中，你将通过添加更多参数和通过脚本重新部署刚下载的模板来自定义该模板。请参阅[自定义和重新部署导出的模板](/documentation/articles/resource-manager-customize-template)。
- 若要了解如何通过 PowerShell 导出模板，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager)。
- 若要了解如何通过 Azure CLI 导出模板，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure Resource Manager 配合使用](/documentation/articles/xplat-cli-azure-resource-manager)。 


<!---HONumber=Mooncake_0620_2016-->