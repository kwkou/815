<properties
	pageTitle="自定义导出的 Resource Manager 模板 | Azure"
	description="向导出的 Azure Resource Manager 模板添加参数，并通过 Azure PowerShell 或 Azure CLI 重新部署该模板。"
	services="azure-resource-manager"
	documentationCenter=""
	authors="tfitzmac"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="azure-resource-manager"
	ms.workload="multiple"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/01/2016"
	wacn.date="09/26/2016"
	ms.author="tomfitz"/>

# 自定义导出的 Azure Resource Manager 模板

本文演示如何修改导出的模板，以便可以以参数形式传入其他值。它基于[导出 Resource Manager 模板](/documentation/articles/resource-manager-export-template)一文中执行的步骤构建，但你不一定要先完成该文。你可以在本文中找到所需的模板和脚本。

## 查看导出的模板

如果你已完成[导出 Resource Manager 模板](/documentation/articles/resource-manager-export-template)，请打开已下载的模板。该模板名为 **template.json**。如果你有 Visual Studio 或 Visual Code，则可以使用这两种中的任一个来编辑模板。否则，可以使用任何 JSON 编辑器或文本编辑器。

如果你尚未完成前面的演练，请创建一个名为 **template.json** 的文件，并将导出模板中的以下内容添加到该文件。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
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
        "variables": {},
        "resources": [
            {
                "comments": "Generalized from resource in walkthrough.",
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[parameters('virtualNetworks_VNET_name')]",
                "apiVersion": "2015-06-15",
                "location": "northeurope",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "10.0.0.0/16"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "default",
                            "properties": {
                                "addressPrefix": "10.0.0.0/24"
                            }
                        }
                    ]
                },
                "dependsOn": []
            },
            {
                "comments": "Generalized from resource in walkthrough.",
                "type": "Microsoft.Storage/storageAccounts",
                "name": "[parameters('storageAccounts_storagetf05092016_name')]",
                "apiVersion": "2015-06-15",
                "location": "northeurope",
                "tags": {},
                "properties": {
                    "accountType": "Standard_RAGRS"
                },
                "dependsOn": []
            }
        ]
    }

如果你要在虚拟网络所在的区域中创建相同类型的存储帐户以使用相同的地址前缀和相同的子网前缀进行每个部署，则 template.json 模板将能很好地工作。但是，Resource Manager 提供相关选项，因此使用它部署模板比这具有更大的灵活性。例如，在部署期间，你可能需要指定要创建的存储帐户的类型或要用于虚拟网络地址前缀和子网前缀的值。

## 自定义模板

在此部分中，你将向导出的模板添加参数，以便在将这些资源部署到其他环境时，可以重新使用该模板。你还将向模板添加某些功能，以减少在部署模板时遇到错误的可能性。你不再需要想存储帐户的唯一名称。而是，该模板将创建唯一名称。你将限制可以为存储帐户类型指定的值，以仅匹配有效的选项。

1. 若要能够传入在部署过程中可能需要指定的值，请将 **parameters** 节替换为以下参数定义。请注意 **storageAccount\_accountType** 的 **allowedValues** 的值。如果你意外地提供了无效的值，则在部署开始之前将识别该错误。另请注意，你仅提供存储帐户名称的前缀，该前缀限制为 11 个字符。将前缀限制为 11 个字符时，可确保完整名称不会超过存储帐户的最大字符数。使用前缀，可将命名约定应用于存储帐户。在下一步中，你将了解如何创建唯一名称。

        "parameters": {
          "storageAccount_prefix": {
            "type": "string",
            "maxLength": 11
          },
          "storageAccount_accountType": {
            "defaultValue": "Standard_RAGRS",
            "type": "string",
            "allowedValues": [
              "Standard_LRS",
              "Standard_ZRS",
              "Standard_GRS",
              "Standard_RAGRS",
              "Premium_LRS"
            ]
          },
          "virtualNetwork_name": {
            "type": "string"
          },
          "addressPrefix": {
            "defaultValue": "10.0.0.0/16",
            "type": "string"
          },
          "subnetName": {
            "defaultValue": "subnet-1",
            "type": "string"
          },
          "subnetAddressPrefix": {
            "defaultValue": "10.0.0.0/24",
            "type": "string"
          }
        },

2. 模板的 **variables** 节当前为空。将此节替换为以下代码。在 **variables** 节中，你作为模板作者可以创建值，以简化模板的其余部分的语法。**StorageAccount\_name** 变量将参数中的前缀连接成一个唯一字符串，该字符串将基于资源组的标识符生成。

        "variables": {
          "storageAccount_name": "[concat(parameters('storageAccount_prefix'), uniqueString(resourceGroup().id))]"
        },

3. 若要在资源定义中使用参数和变量，请将 **resources** 节替换为以下定义。请注意，除了分配给资源属性的值外，实际上资源定义中只发生了非常少的更改。属性与导出的模板中的属性完全相同。你只需为参数值分配属性，而不是对值进行硬编码。资源的位置通过 **resourceGroup().location** 表达式设置为使用资源组所在的位置。为存储帐户名称创建的变量通过 **variables** 表达式进行引用。

        "resources": [
          {
            "type": "Microsoft.Network/virtualNetworks",
            "name": "[parameters('virtualNetwork_name')]",
            "apiVersion": "2015-06-15",
            "location": "[resourceGroup().location]",
            "properties": {
              "addressSpace": {
                "addressPrefixes": [
                  "[parameters('addressPrefix')]"
                ]
              },
              "subnets": [
                {
                  "name": "[parameters('subnetName')]",
                  "properties": {
                    "addressPrefix": "[parameters('subnetAddressPrefix')]"
                  }
                }
              ]
            },
            "dependsOn": []
          },
          {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageAccount_name')]",
            "apiVersion": "2015-06-15",
            "location": "[resourceGroup().location]",
            "tags": {},
            "properties": {
                "accountType": "[parameters('storageAccount_accountType')]"
            },
            "dependsOn": []
          }
        ]

## 修复参数文件

下载的参数文件不再与模板中的参数匹配。你不必使用参数文件，但参数文件可以简化重新部署环境时的过程。对于许多参数，你将使用模板中定义的默认值，因此，参数文件只需要两个值。

将 parameters.json 文件的内容替换为以下内容：

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccount_prefix": {
          "value": "storage"
        },
        "virtualNetwork_name": {
          "value": "VNET"
        }
      }
    }

更新后的参数文件仅为没有默认值的参数提供值。如果你需要不同于默认值的值，则可以为其他参数提供值。

## 部署模板

可以使用 Azure PowerShell 或 Azure 命令行界面 (CLI) 部署自定义的模板和参数文件。如果需要，请安装 [Azure PowerShell](/documentation/articles/powershell-install-configure) 或 [Azure CLI](/documentation/articles/xplat-cli-install)。你导出原始模板后，可以将下载的脚本与模板配合使用，也可以编写自己的脚本来部署模板。
在本文中说明了这两个选项。

2. 若要使用你自己的脚本部署，请使用以下任一项。

     对于 PowerShell，运行：

        # login
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud

        # create a new resource group
        New-AzureRmResourceGroup -Name ExportGroup -Location "China East"

        # deploy the template to the resource group
        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExportGroup -TemplateFile {path-to-file}\template.json -TemplateParameterFile {path-to-file}\parameters.json

     对于 Azure CLI，运行：

        azure login -e AzureChinaCloud

        azure group create -n ExportGroup -l "China East"

        azure group deployment create -f {path-to-file}\azuredeploy.json -e {path-to-file}\parameters.json -g ExportGroup -n ExampleDeployment

3. 如果你已下载导出的模板和脚本，请找到 **deploy.ps1** 文件（适用于 PowerShell）或 **deploy.sh** 文件（适用于 Azure CLI）。

     对于 PowerShell，运行：

        Get-Item deploy.ps1 | Unblock-File

        .\deploy.ps1

     对于 Azure CLI，运行：

        .\deploy.sh

## 后续步骤

- [Resource Manager 模板演练](/documentation/articles/resource-manager-template-walkthrough)在本文中学习的内容的基础上构建，它将为更复杂的解决方案创建模板。它可帮助你了解有关可用资源的更多信息以及如何确定要提供的值。
- 若要了解如何通过 PowerShell 导出模板，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager)。
- 若要了解如何通过 Azure CLI 导出模板，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure Resource Manager 配合使用](/documentation/articles/xplat-cli-azure-resource-manager)。
- 若要了解如何构造模板，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates)。

<!---HONumber=Mooncake_0711_2016-->