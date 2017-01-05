<properties
    pageTitle="使用模板创建具有多个 NIC 的 VM | Azure"
    description="通过 Azure Resource Manager 使用模板创建具有多个 NIC 的 VM。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="486f7dd5-cf2f-434c-85d1-b3e85c427def"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/02/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用模板创建具有多个 NIC 的 VM
[AZURE.INCLUDE [virtual-network-deploy-multinic-arm-selectors-include.md](../../includes/virtual-network-deploy-multinic-arm-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/virtual-network-deploy-multinic-classic-ps/)。
> 

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

以下步骤将名为 *IaaSStory* 的资源组用于 Web 服务器，并将名为 *IaaSStory-BackEnd* 的资源组用于数据库服务器。

## <a name="Pre-requisites"></a> 先决条件
需要先创建具有此方案需要的所有资源的 *IaaSStory* 资源组，然后才能创建数据库服务器。若要创建这些资源，请完成以下步骤：

1. 导航到[模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/IaaS-Story/11-MultiNIC)。
2. 下载模板，执行一些必要的修改。

	>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

3. 使用 PowerShell 或 CLI 部署模板。

> [AZURE.IMPORTANT]
请确保存储帐户名称是唯一的。Azure 中不能存在重复的存储帐户名称。
> 

## 了解部署模板
在部署随本文档提供的模板之前，请确保了解其用途。以下步骤概要介绍模板：

1. 导航到[模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/IaaS-Story/11-MultiNIC)。
2. 单击 **azuredeploy.json** 以打开模板文件。
3. 注意下面列出的 *osType* 参数。此参数用于选择要用于数据库服务器的 VM 映像，以及多个与操作系统相关的设置。

        "osType": {
          "type": "string",
          "defaultValue": "Windows",
          "allowedValues": [
            "Windows",
            "Ubuntu"
          ],
          "metadata": {
          "description": "Type of OS to use for VMs: Windows or Ubuntu."
          }
        },

4. 向下滚动到变量列表，并检查下面列出的 **dbVMSetting** 变量的定义。它接收 **dbVMSettings** 变量中包含的数组元素之一。如果熟悉软件开发术语，可将 **dbVMSettings** 变量视为哈希表或字典。

        "dbVMSetting": "[variables('dbVMSettings')[parameters('osType')]]"

5. 假设决定在后端部署运行 SQL 的 Windows VM。那么 **osType** 的值将为 *Windows* ，**dbVMSetting** 变量将包含下面列出的元素，它表示 **dbVMSettings** 变量中的第一个值。

        "Windows": {
          "vmSize": "Standard_DS3",
          "publisher": "MicrosoftSQLServer",
          "offer": "SQL2014SP1-WS2012R2",
          "sku": "Standard",
          "version": "latest",
          "vmName": "DB",
          "osdisk": "osdiskdb",
          "datadisk": "datadiskdb",
          "nicName": "NICDB",
          "ipAddress": "192.168.2.",
          "extensionDeployment": "",
          "avsetName": "ASDB",
          "remotePort": 3389,
          "dbPort": 1433
        },

6. 请注意，**vmSize** 包含值 *Standard\_DS3* 。只有某些 VM 大小允许使用多个 NIC。可阅读 [Windows VM 大小](/documentation/articles/virtual-machines-windows-sizes/)和 [Linux VM 大小](/documentation/articles/virtual-machines-linux-sizes/)这两篇文章，确认哪些 VM 大小支持多个 NIC。

7. 向下滚动到“资源”，注意第一个元素。它描述存储帐户。此存储帐户将用于维护每个数据库 VM 使用的数据磁盘。在此方案中，每个数据库 VM 将 OS 磁盘存储在常规存储器内，并将两个数据磁盘存储在 SSD（高级）存储器内。

        {
          "apiVersion": "2015-05-01-preview",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[parameters('prmStorageName')]",
          "location": "[variables('location')]",
          "tags": {
            "displayName": "Storage Account - Premium"
          },
          "properties": {
          "accountType": "[parameters('prmStorageType')]"
          }
        },

8. 向下滚动到下一项资源，如下所示。此资源表示用于访问每个数据库 VM 中数据库的 NIC。注意此资源中 **copy** 函数的使用。根据 **dbCount** 参数，此模板允许部署所需的任意多个 VM。因此，你需要为数据库访问创建相同数量的 NIC，每个 VM 一个。

        {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/networkInterfaces",
        "name": "[concat(variables('dbVMSetting').nicName,'-DA-', copyindex(1))]",
        "location": "[variables('location')]",
        "tags": {
          "displayName": "NetworkInterfaces - DB DA"
        },
        "copy": {
          "name": "dbniccount",
          "count": "[parameters('dbCount')]"
        },
        "properties": {
          "ipConfigurations": [
            {
              "name": "ipconfig1",
              "properties": {
                "privateIPAllocationMethod": "Static",
                "privateIPAddress": "[concat(variables('dbVMSetting').ipAddress,copyindex(4))]",
                "subnet": {
                  "id": "[variables('backEndSubnetRef')]"
                }
              }
             }
           ] 
         }
        },

9. 向下滚动到下一项资源，如下所示。此资源表示每个数据库 VM 中用于管理的 NIC。同样，需要为每个数据库 VM 创建一个此类 NIC。请注意链接 NSG 的 **networkSecurityGroup** 元素，它仅允许访问连接到此 NIC 的 RDP/SSH。

        {
          "apiVersion": "2015-06-15",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "[concat(variables('dbVMSetting').nicName, '-RA-',copyindex(1))]",
          "location": "[variables('location')]",
          "tags": {
            "displayName": "NetworkInterfaces - DB RA"
        },
        "copy": {
          "name": "dbniccount",
          "count": "[parameters('dbCount')]"
        },
        "properties": {
          "ipConfigurations": [
            {
              "name": "ipconfig1",
              "properties": {
                "networkSecurityGroup": {
                 "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('remoteAccessNSGName'))]"
                 },
                 "privateIPAllocationMethod": "Static",
                 "privateIPAddress": "[concat(variables('dbVMSetting').ipAddress,copyindex(54))]",
                 "subnet": {
                  "id": "[variables('backEndSubnetRef')]"
                 }
               }
              }
            ]
          }
        },

10. 向下滚动到下一项资源，如下所示。此资源表示要由所有数据库 VM 共享的可用性集。这样可保证，在维护期间集中始终有一个 VM 在运行。

        {
          "apiVersion": "2015-06-15",
          "type": "Microsoft.Compute/availabilitySets",
          "name": "[variables('dbVMSetting').avsetName]",
          "location": "[variables('location')]",
          "tags": {
            "displayName": "AvailabilitySet - DB"
          }
        },

11. 向下滚动到下一个资源。此资源表示数据库 VM，如下面所列的前几行所示。再次注意 **copy** 函数的使用，确保根据 **dbCount** 参数创建了多个 VM。另请注意 **dependsOn** 集合。它列出了需要在部署 VM 之前创建的两个 NIC，以及可用性集和存储帐户。

        "apiVersion": "2015-06-15",
        "type": "Microsoft.Compute/virtualMachines",
        "name": "[concat(variables('dbVMSetting').vmName,copyindex(1))]",
        "location": "[variables('location')]",
        "dependsOn": [
          "[concat('Microsoft.Network/networkInterfaces/', variables('dbVMSetting').nicName,'-DA-', copyindex(1))]",
          "[concat('Microsoft.Network/networkInterfaces/', variables('dbVMSetting').nicName,'-RA-', copyindex(1))]",
          "[concat('Microsoft.Compute/availabilitySets/', variables('dbVMSetting').avsetName)]",
          "[concat('Microsoft.Storage/storageAccounts/', parameters('prmStorageName'))]"
        ],
        "tags": {
          "displayName": "VMs - DB"
        },
        "copy": {
          "name": "dbvmcount",
          "count": "[parameters('dbCount')]"
        },

12. 在 VM 资源中向下滚动到 **networkProfile** 元素，如下所示。注意每个 VM 有两个 NIC 作为引用。为一个 VM 创建多个 NIC 时，必须将其一个 NIC 的 **primary** 属性设为 *true* ，并将其余 NIC 的该属性设为 *false* 。

        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('dbVMSetting').nicName,'-DA-',copyindex(1)))]",
              "properties": { "primary": true }
            },
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('dbVMSetting').nicName,'-RA-',copyindex(1)))]",
              "properties": { "primary": false }
            }
          ]
        }

## 使用 PowerShell 部署模板
若要使用 PowerShell 部署下载的模板，请按照[安装和配置 PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 一文中的步骤安装和配置 PowerShell，然后完成以下步骤：

运行 **`New-AzureRmResourceGroup`** cmdlet，使用模板创建资源组。

    New-AzureRmResourceGroup -Name IaaSStory-Backend -Location chinanorth `
    -TemplateFile 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/azuredeploy.json' `
    -TemplateParameterFile 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/azuredeploy.parameters.json'

预期输出：

	ResourceGroupName : IaaSStory-Backend
	Location          : chinanorth
	ProvisioningState : Succeeded
	Tags              :
	Permissions       :
						Actions  NotActions
						=======  ==========
						*
		Resources         :
						Name                 Type                                 Location
						===================  ===================================  ========
						ASDB                 Microsoft.Compute/availabilitySets   chinanorth  
						DB1                  Microsoft.Compute/virtualMachines    chinanorth  
						DB2                  Microsoft.Compute/virtualMachines    chinanorth  
						NICDB-DA-1           Microsoft.Network/networkInterfaces  chinanorth  
						NICDB-DA-2           Microsoft.Network/networkInterfaces  chinanorth  
						NICDB-RA-1           Microsoft.Network/networkInterfaces  chinanorth  
						NICDB-RA-2           Microsoft.Network/networkInterfaces  chinanorth  
						wtestvnetstorageprm  Microsoft.Storage/storageAccounts    chinanorth  
	ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend

## 使用 Azure CLI 部署模板
若要使用 Azure CLI 部署模板，请执行以下步骤。

1. 如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **`azure config mode`** 命令，切换到 Resource Manager 模式，如下所示。

        azure config mode arm

    预期的输出如下所示：

		info:    New mode is arm

3. 打开[参数文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/azuredeploy.parameters.json)，选择其内容，然后将其保存到计算机上的文件中。对于本示例，我们将参数文件保存到 *parameters.json*。
4. 运行 **`azure group deployment create`** cmdlet，使用上面下载并修改的模板和参数文件部署新的 VNet。输出后显示的列表阐释了所用参数。

        azure group create -n IaaSStory-Backend -l chinanorth --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/azuredeploy.json -e parameters.json

    预期输出：
   
        info:    Executing command group create
        + Getting resource group IaaSStory-Backend
        + Creating resource group IaaSStory-Backend
        info:    Created resource group IaaSStory-Backend
        + Initializing template configurations and parameters
        + Creating a deployment
        info:    Created template deployment "azuredeploy"
        data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend
        data:    Name:                IaaSStory-Backend
        data:    Location:            chinanorth
        data:    Provisioning State:  Succeeded
        data:    Tags: null
        data:
        info:    group create command OK

<!---HONumber=Mooncake_1219_2016-->