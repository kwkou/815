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
	ms.date="06/28/2016"
	wacn.date="08/15/2016"/>

# 从现有资源导出 Azure Resource Manager 模板

Resource Manager 允许你从订阅的现有资源中导出 Resource Manager 模板。你可以使用该生成的模板了解模板语法，或根据需要自动重新部署解决方案。

必须注意的是，可以使用两种不同的方式来导出模板：

- 可以导出部署时使用过的实际模板。导出的模板中包括的所有参数和变量与原始模板中定义的完全一样。如果你已通过门户部署资源，而现在想要了解如何通过构建模板来创建这些资源，则此方法特别有用。
- 你可以导出表示资源组当前状态的模板。导出的模板不基于任何已用于部署的模板。与之相反，它所创建的模板为资源组的快照。导出的模板会有许多硬编码的值，其参数可能没有你通常定义的那么多。如果你已通过门户或脚本修改资源组，并且现在需要捕获资源组作为模板，则可使用此方法。

本主题对这两种方法都进行了说明。在[自定义导出的 Azure Resource Manager 模板](resource-manager-customize-template.md)一文中，你将了解如何获取从资源组的当前状态生成的模板并使之在重新部署你的解决方案时更有用。

在本教程中，你将登录到 Azure 门户，创建存储帐户，并导出该存储帐户的模板。你将添加虚拟网络以修改资源组。最后，你将导出表示其当前状态的新模板。尽管本文重点介绍简化的基础结构，但你也可以使用这些相同的步骤为更复杂的解决方案导出模板。

## 创建存储帐户

1. 在 [Azure 门户预览](https://portal.azure.cn)中，选择“新建”>“数据 + 存储”>“存储帐户”。

      ![创建存储](./media/resource-manager-export-template/create-storage.png)

2. 使用名称 **storage**、你的姓名缩写和日期创建存储帐户。存储帐户名称在 Azure 中必须是唯一的。如果你最初尝试的名称已在使用，请尝试对其进行更改。对于资源组，使用 **ExportGroup**。可以对其他属性使用默认值。选择“创建”。

      ![提供存储的值](./media/resource-manager-export-template/provide-storage-values.png)

在部署完成后，你的订阅将包含存储帐户。

## 从部署历史记录导出模板

1. 转到新资源组的资源组边栏选项卡。你将发现列出了上次部署的结果。选择此链接。

      ![资源组边栏选项卡](./media/resource-manager-export-template/resource-group-blade.png)

2. 你将看到该组的部署历史记录。在你的情况下，有可能只列出了一个部署。选择此部署。

     ![上次部署](./media/resource-manager-export-template/last-deployment.png)

3. 将显示此部署的摘要。摘要包括此部署及其操作的状态，以及为参数提供的值。若要查看用于部署的模板，请选择“查看模板”。

     ![查看部署摘要](./media/resource-manager-export-template/deployment-summary.png)

4. Resource Manager 为你检索以下五个文件：

   1. **模板** - 定义解决方案基础结构的模板。通过门户创建存储帐户时，Resource Manager 使用模板来部署该存储帐户，并保存该模板供将来参考。
   2. **参数** - 可用于在部署过程中传入值的参数文件。它包含在首次部署期间提供的值，但你可以在重新部署该模板时更改其中任何值。
   3. **CLI** - 可用于部署该模板的 Azure 命令行界面 (CLI) 脚本文件。
   4. **PowerShell** - 可用于部署该模板的 Azure PowerShell 脚本文件。
   5. **.NET** - 可用于部署该模板的 .NET 类。

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
 
这是用于创建你的存储帐户的实际模板。请注意，该模板包含的参数可用于部署不同类型的存储帐户。若要详细了解模板的结构，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。有关可在模板中使用的函数的完整列表，请参阅 [Azure Resource Manager 模板函数](/documentation/articles/resource-group-template-functions/)。


## 添加虚拟网络

上一节中下载的模板表示该原始部署的基础结构，但它将不会记录在部署之后进行的任何更改。
为了说明此问题，让我们通过门户添加虚拟网络来修改资源组。

1. 在资源组边栏选项卡中，选择“添加”。

      ![添加资源](./media/resource-manager-export-template/add-resource.png)

2. 从可用资源中选择“虚拟网络”。

      ![选择虚拟网络](./media/resource-manager-export-template/select-vnet.png)

2. 将虚拟网络命名为 **VNET**，并对其他属性使用默认值。选择“创建”。

      ![设置警报](./media/resource-manager-export-template/create-vnet.png)

3. 在虚拟网络已成功部署到资源组后，再次查看部署历史记录。现在，你将看到两个部署。如果看不到第二个部署，则可能需要关闭资源组边栏选项卡，然后重新打开它。选择更新的部署。

      ![部署历史记录](./media/resource-manager-export-template/deployment-history.png)

4. 查看该部署的模板。请注意，它只定义了添加虚拟网络时所做的更改。

通常使用模板的最佳做法是在单个操作中部署解决方案的所有基础结构，而不是记住要部署的多个不同模板。


## 从资源组导出模板

虽然每个部署只显示已对资源组所做的更改，但你随时可以导出模板以显示整个资源组的属性。

1. 若要查看资源组的模板，请选择“导出模板”。

      ![导出资源组](./media/resource-manager-export-template/export-resource-group.png)

     并非所有资源类型都支持导出模板功能。如果你的资源组仅包含本文中显示的存储帐户和虚拟网络，则不会显示错误。不过，如果你已经创建其他资源类型，则可能会显示一个错误，指出导出存在问题。你将在[修复导出问题](#fix-export-issues)部分了解如何处理这些问题。

      

2. 你将再次看到可用于重新部署解决方案的五个文件，但这一次该模板稍有不同。该模板只有两个参数：一个用于存储帐户名称，一个用于虚拟网络名称。

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

     Resource Manager 未检索到在部署期间使用的模板。但是，它将基于资源的当前配置生成新模板。例如，存储帐户位置和复制值将设置为：

        "location": "northeurope",
        "tags": {},
        "properties": {
            "accountType": "Standard_RAGRS"
        },

3. 下载该模板，以便可以在本地处理它。

      ![下载模板](./media/resource-manager-export-template/download-template.png)

4. 找到你下载的 .zip 文件并将内容解压缩。可以使用这个下载的模板重新部署你的基础结构。

## 修复导出问题

并非所有资源类型都支持导出模板功能。某些资源类型特意没有导出，目的是防止公开敏感数据。例如，如果你的站点配置中有一个连接字符串，你可能不希望其显式显示在导出的模板中。你可以手动将缺失的资源添加回模板中，从而解决这个问题。

> [AZURE.NOTE] 只有在你是从资源组而不是从部署历史记录中导出时，才会遇到导出问题。如果你的上一个部署能够准确地代表资源组的当前状态，则应从部署历史记录而非资源组中导出模板。只有在你已对资源组进行更改且该更改未在单个模板中定义的情况下，才应从资源组导出。

例如，如果你导出的模板所对应的资源组包含一个 Web 应用（SQL 数据库），且在站点配置中包含一个连接字符串，则会显示以下消息。

![show error](./media/resource-manager-export-template/show-error.png)

选择该消息时，将会具体显示哪些资源类型未导出。
     
![show error](./media/resource-manager-export-template/show-error-details.png)

下面显示了一些常用修补程序。若要实现这些资源，需要将参数添加到模板。有关详细信息，请参阅[自定义和重新部署导出的模板](resource-manager-customize-template.md)。

### 连接字符串

在网站资源中，添加数据库连接字符串的定义：

```
{
  "type": "Microsoft.Web/sites",
  ...
  "resources": [
    {
      "apiVersion": "2015-08-01",
      "type": "config",
      "name": "connectionstrings",
      "dependsOn": [
          "[concat('Microsoft.Web/Sites/', parameters('<site-name>'))]"
      ],
      "properties": {
          "DefaultConnection": {
            "value": "[concat('Data Source=tcp:', reference(concat('Microsoft.Sql/servers/', parameters('<database-server-name>'))).fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('<database-name>'), ';User Id=', parameters('<admin-login>'), '@', parameters('<database-server-name>'), ';Password=', parameters('<admin-password>'), ';')]",
              "type": "SQLServer"
          }
      }
    }
  ]
}
```    

### 网站扩展

在网站资源中，添加要安装的代码的定义：

```
{
  "type": "Microsoft.Web/sites",
  ...
  "resources": [
    {
      "name": "MSDeploy",
      "type": "extensions",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "dependsOn": [
        "[concat('Microsoft.Web/sites/', parameters('<site-name>'))]"
      ],
      "properties": {
        "packageUri": "[concat(parameters('<artifacts-location>'), '/', parameters('<package-folder>'), '/', parameters('<package-file-name>'), parameters('<sas-token>'))]",
        "dbType": "None",
        "connectionString": "",
        "setParameters": {
          "IIS Web Application Name": "[parameters('<site-name>')]"
        }
      }
    }
  ]
}
```

### 虚拟机扩展

如需虚拟机扩展的示例，请参阅 [Azure Windows VM 扩展配置示例](./virtual-machines/virtual-machines-windows-extensions-configuration-samples.md)。

### 虚拟网络网关

添加虚拟网络网关资源类型。

```
{
  "type": "Microsoft.Network/virtualNetworkGateways",
  "name": "[parameters('<gateway-name>')]",
  "apiVersion": "2015-06-15",
  "location": "[resourceGroup().location]",
  "properties": {
    "gatewayType": "[parameters('<gateway-type>')]",
    "ipConfigurations": [
      {
        "name": "default",
        "properties": {
          "privateIPAllocationMethod": "Dynamic",
          "subnet": {
            "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('<vnet-name>'), parameters('<new-subnet-name>'))]"
          },
          "publicIpAddress": {
            "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('<new-public-ip-address-Name>'))]"
          }
        }
      }
    ],
    "enableBgp": false,
    "vpnType": "[parameters('<vpn-type>')]"
  },
  "dependsOn": [
    "Microsoft.Network/virtualNetworks/codegroup4/subnets/GatewaySubnet",
    "[concat('Microsoft.Network/publicIPAddresses/', parameters('<new-public-ip-address-Name>'))]"
  ]
},
```

### 本地网络网关

添加本地网络网关资源类型。

```
{
    "type": "Microsoft.Network/localNetworkGateways",
    "name": "[parameters('<local-network-gateway-name>')]",
    "apiVersion": "2015-06-15",
    "location": "[resourceGroup().location]",
    "properties": {
      "localNetworkAddressSpace": {
        "addressPrefixes": "[parameters('<address-prefixes>')]"
      }
    }
}
```

### 连接

添加连接资源类型。

```
{
    "apiVersion": "2015-06-15",
    "name": "[parameters('<connection-name>')]",
    "type": "Microsoft.Network/connections",
    "location": "[resourceGroup().location]",
    "properties": {
        "virtualNetworkGateway1": {
        "id": "[resourceId('Microsoft.Network/virtualNetworkGateways', parameters('<gateway-name>'))]"
      },
      "localNetworkGateway2": {
        "id": "[resourceId('Microsoft.Network/localNetworkGateways', parameters('<local-gateway-name>'))]"
      },
      "connectionType": "IPsec",
      "routingWeight": 10,
      "sharedKey": "[parameters('<shared-key>')]"
    }
},
```


## 后续步骤

祝贺你！ 你已学习如何从门户中创建的资源导出模板。

- 在本教程的第二部分中，你将通过向刚下载的模板添加更多参数来自定义该模板，并通过脚本重新部署该模板。请参阅[自定义和重新部署导出的模板](/documentation/articles/resource-manager-customize-template/)。
- 若要了解如何通过 PowerShell 导出模板，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。
- 若要了解如何通过 Azure CLI 导出模板，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure Resource Manager 配合使用](/documentation/articles/xplat-cli-azure-resource-manager/)。

<!---HONumber=Mooncake_0808_2016-->