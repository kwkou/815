<!-- ARM: tested -->

<properties
	pageTitle="使用 Resource Manager 模板创建 VM | Azure"
	description="将 Resource Manager 模板与 PowerShell 配合使用，以轻松创建新的 Windows 虚拟机。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/18/2016"
	wacn.date="06/20/2016"/>

# 使用资源管理器模板创建 Windows 虚拟机

本文介绍一个 Azure Resource Manager 模板，并说明如何使用 PowerShell 部署该模板。此模板将在包含单个子网的新虚拟网络上部署运行 Windows Server 的单个虚拟机。

执行本文中的步骤大约需要 20 分钟时间。

> [AZURE.IMPORTANT] 如果你希望将 VM 包含在某个可用性集中，可以在创建 VM 时将其添加到该集。目前不支持在创建 VM 之后再将其添加到可用性集。

## 步骤 1：创建模板文件

你可以使用 [Authoring Azure Resource Manager templates（创作 Azure Resource Manager 模板）](/documentation/articles/resource-group-authoring-templates/)中的信息来创建自己的模板。也可以从 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/)部署创建的模板。

1. 打开偏好的文本编辑器，并将以下 JSON 信息复制到名为 *VirtualMachineTemplate.json* 的新文件中：

        {
          "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "newStorageAccountName": {
              "type": "string",
              "metadata": {
                "Description": "The name of the storage account where the VM disk is stored."
              }
            },
            "adminUsername": {
              "type": "string",
              "metadata": {
                "Description": "The name of the administrator account on the VM."
              }
            },
            "adminPassword": {
              "type": "securestring",
              "metadata": {
                "Description": "The administrator account password on the VM."
              }
            },
            "dnsNameForPublicIP": {
              "type": "string",
              "metadata": {
                "Description": "The name of the public IP address used to access the VM."
              }
            }
          },
          "variables": {
            "location": "China North",
            "imagePublisher": "MicrosoftWindowsServer",
            "imageOffer": "WindowsServer",
            "windowsOSVersion": "2012-R2-Datacenter",
            "OSDiskName": "osdisk1",
            "nicName": "nc1",
            "addressPrefix": "10.0.0.0/16",
            "subnetName": "sn1",
            "subnetPrefix": "10.0.0.0/24",
            "storageAccountType": "Standard_LRS",
            "publicIPAddressName": "ip1",
            "publicIPAddressType": "Dynamic",
            "vmStorageAccountContainerName": "vhds",
            "vmName": "vm1",
            "vmSize": "Standard_A0",
            "virtualNetworkName": "vn1",
            "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
            "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables('subnetName'))]"
          },
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "name": "[parameters('newStorageAccountName')]",
              "apiVersion": "2015-06-15",
              "location": "[variables('location')]",
              "properties": {
                "accountType": "[variables('storageAccountType')]"
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Network/publicIPAddresses",
              "name": "[variables('publicIPAddressName')]",
              "location": "[variables('location')]",
              "properties": {
                "publicIPAllocationMethod": "[variables('publicIPAddressType')]",
                "dnsSettings": {
                  "domainNameLabel": "[parameters('dnsNameForPublicIP')]"
                }
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Network/virtualNetworks",
              "name": "[variables('virtualNetworkName')]",
              "location": "[variables('location')]",
              "properties": {
                "addressSpace": {
                  "addressPrefixes": [
                    "[variables('addressPrefix')]"
                  ]
                },
                "subnets": [
                  {
                    "name": "[variables('subnetName')]",
                    "properties": {
                      "addressPrefix": "[variables('subnetPrefix')]"
                    }
                  }
                ]
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Network/networkInterfaces",
              "name": "[variables('nicName')]",
              "location": "[variables('location')]",
              "dependsOn": [
                "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
                "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
              ],
              "properties": {
                "ipConfigurations": [
                  {
                    "name": "ipconfig1",
                    "properties": {
                      "privateIPAllocationMethod": "Dynamic",
                      "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIPAddressName'))]"
                      },
                      "subnet": {
                        "id": "[variables('subnetRef')]"
                      }
                    }
                  }
                ]
              }
            },
            {
              "apiVersion": "2016-03-30",
              "type": "Microsoft.Compute/virtualMachines",
              "name": "[variables('vmName')]",
              "location": "[variables('location')]",
              "dependsOn": [
                "[concat('Microsoft.Storage/storageAccounts/', parameters('newStorageAccountName'))]",
                "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
              ],
              "properties": {
                "hardwareProfile": {
                  "vmSize": "[variables('vmSize')]"
                },
                "osProfile": {
                  "computername": "[variables('vmName')]",
                  "adminUsername": "[parameters('adminUsername')]",
                  "adminPassword": "[parameters('adminPassword')]"
                },
                "storageProfile": {
                  "imageReference": {
                    "publisher": "[variables('imagePublisher')]",
                    "offer": "[variables('imageOffer')]",
                    "sku" : "[variables('windowsOSVersion')]",
                    "version":"latest"
                  },
                  "osDisk" : {
                    "name": "osdisk",
                    "vhd": {
                      "uri": "[concat('http://',parameters('newStorageAccountName'),'.blob.core.chinacloudapi.cn/',variables('vmStorageAccountContainerName'),'/',variables('OSDiskName'),'.vhd')]"
                    },
                    "caching": "ReadWrite",
                    "createOption": "FromImage"
                  }
                },
                "networkProfile": {
                  "networkInterfaces": [
                    {
                      "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
                    }
                  ]
                }
              }
            }
          ]
        }
        
    >[AZURE.NOTE] 本文创建运行 Windows Server 操作系统版本的虚拟机。若要详细了解如何选择其他映像，请参阅 [Navigate and select Azure virtual machine images with Windows PowerShell and the Azure CLI（使用 Windows PowerShell 和 Azure CLI 来导航和选择 Azure 虚拟机映像）](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。
    
2. 保存模板文件。

## 步骤 2：创建参数文件

若要为模板中定义的资源参数指定值，请创建一个包含值的参数文件，并连同模板一起将它提交到资源管理器。

1. 在文本编辑器中，将以下 JSON 信息复制到名为 *Parameters.json* 的新文件中：

        {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "newStorageAccountName": { "value": "mytestsa1" },
            "adminUserName": { "value": "mytestacct1" },
            "adminPassword": { "value": "mytestpass1" },
            "dnsNameForPublicIP": { "value": "mytestdns1" }
          }
        }

4. 保存参数文件。

## 步骤 3：安装 Azure PowerShell

有关如何安装最新版 Azure PowerShell 的信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。选择要使用的订阅，然后登录到你的 Azure 帐户。

## 步骤 4：创建资源组

必须在资源组中部署所有资源。有关详细信息，请参阅 [Azure Resource Manager overview（Azure Resource Manager 概述）](/documentation/articles/resource-group-overview/)。

1. 获取可以创建资源的可用位置列表。

	    Get-AzureLocation | sort Name | Select Name

2. 使用列表中的位置（例如 **China North**）替换 **$locName** 的值。创建变量。

        $locName = "location name"
        
3. 使用新资源组的名称替换 **$rgName** 的值。创建变量和资源组。

        $rgName = "resource group name"
        New-AzureRmResourceGroup -Name $rgName -Location $locName
        
    您应看到与下面类似的内容：
    
        ResourceGroupName : myrg1
        Location          : chinaeast
        ProvisioningState : Succeeded
        Tags              :
        ResourceId        : /subscriptions/{subscription-id}/resourceGroups/myrg1

### 步骤 7：使用模板和参数创建资源

1. 将 **$deployName** 的值替换为部署名称。将 **$templatePath** 的值替换为模板文件的路径和名称。将 **$parameterFile** 的值替换为参数文件的路径和名称。创建变量。 

        $deployName="deployment name"
        $templatePath = "template path"
        $parameterFile = "parameter file"

4. 部署模板。

        New-AzureRmResourceGroupDeployment -ResourceGroupName "davidmurg6" -TemplateFile $templatePath -TemplateParameterFile $parameterFile

    你将看到类似于下面的内容：

        DeploymentName    : VirtualMachineTemplate
        ResourceGroupName : myrg1
        ProvisioningState : Succeeded
        Timestamp         : 4/14/2016 8:11:37 PM
        Mode              : Incremental
        TemplateLink      :
        Parameters        :
                            Name             Type                       Value
                            ===============  =========================  ==========
                            newStorageAccountName  String                     mytestsa1
                            adminUsername    String                     mytestacct1
                            adminPassword    SecureString
                            dnsNameForPublicIP  String                     mytestdns1

        Outputs           :

    >[AZURE.NOTE] 也可以从 Azure 存储帐户部署模板和参数。有关详细信息，请参阅 [Using Azure PowerShell with Azure Storage（对 Azure 存储空间使用 Azure PowerShell）](/documentation/articles/storage-powershell-guide-full/)。

## 后续步骤

- 查看 [Manage virtual machines using Azure Resource Manager and PowerShell（使用 Azure Resource Manager 和 PowerShell 管理虚拟机）](/documentation/articles/virtual-machines-windows-ps-manage/)，了解如何管理刚创建的虚拟机。

<!---HONumber=Mooncake_0613_2016-->