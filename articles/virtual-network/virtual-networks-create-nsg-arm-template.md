<properties
    pageTitle="如何使用模板在 ARM 模式下创建 NSG | Azure"
    description="了解如何使用模板在 ARM 下创建和部署 NSG"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor="tysonn"
    tags="azure-resource-manager" />  

<tags
    ms.assetid="f3e7385d-717c-44ff-be20-f9aa450aa99b"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/02/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />

# 如何使用模板创建 NSG
[AZURE.INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文介绍 Resource Manager 部署模型。你还可以[在经典部署模型中创建 NSG](/documentation/articles/virtual-networks-create-nsg-classic-ps/)。

[AZURE.INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

## 模板文件中的 NSG 资源
可查看和下载[示例模板](https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/NSGs.json)。

以下部分说明如何根据方案定义前端 NSG。

    "apiVersion": "2015-06-15",
    "type": "Microsoft.Network/networkSecurityGroups",
    "name": "[parameters('frontEndNSGName')]",
    "location": "[resourceGroup().location]",
    "tags": {
      "displayName": "NSG - Front End"
    },
    "properties": {
      "securityRules": [
        {
          "name": "rdp-rule",
          "properties": {
            "description": "Allow RDP",
            "protocol": "Tcp",
            "sourcePortRange": "*",
            "destinationPortRange": "3389",
            "sourceAddressPrefix": "Internet",
            "destinationAddressPrefix": "*",
            "access": "Allow",
            "priority": 100,
            "direction": "Inbound"
          }
        },
        {
          "name": "web-rule",
          "properties": {
            "description": "Allow WEB",
            "protocol": "Tcp",
            "sourcePortRange": "*",
            "destinationPortRange": "80",
            "sourceAddressPrefix": "Internet",
            "destinationAddressPrefix": "*",
            "access": "Allow",
            "priority": 101,
            "direction": "Inbound"
          }
        }
      ]
    }

若要将 NSG 关联到前端子网，需要更改模板中的子网定义，并使用 NSG 的引用 ID。

    "subnets": [
      {
        "name": "[parameters('frontEndSubnetName')]",
        "properties": {
          "addressPrefix": "[parameters('frontEndSubnetPrefix')]",
          "networkSecurityGroup": {
          "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('frontEndNSGName'))]"
          }
        }
      }, 

请注意，在模板中对后端 NSG 和后端子网执行相同的操作。

## 使用 PowerShell 部署 ARM 模板
若要使用 PowerShell 部署下载的 ARM 模板，请执行以下步骤。

1. 如果从未使用过 Azure PowerShell，请按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 一文中的说明进行安装和配置。
2. 运行 **`New-AzureRmResourceGroup`** cmdlet，使用模板创建资源组。

        New-AzureRmResourceGroup -Name TestRG -Location chinanorth `
        -TemplateFile 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.json' `
        -TemplateParameterFile 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.parameters.json'

    预期输出：

        ResourceGroupName : TestRG
        Location          : chinanorth
        ProvisioningState : Succeeded
        Tags              :
        Permissions       :
                            Actions  NotActions
                            =======  ==========
                            *                  
   
        Resources         :
                            Name                Type                                     Location
                            ==================  =======================================  ========
                            sqlAvSet            Microsoft.Compute/availabilitySets       chinanorth  
                            webAvSet            Microsoft.Compute/availabilitySets       chinanorth  
                            SQL1                Microsoft.Compute/virtualMachines        chinanorth  
                            SQL2                Microsoft.Compute/virtualMachines        chinanorth  
                            Web1                Microsoft.Compute/virtualMachines        chinanorth  
                            Web2                Microsoft.Compute/virtualMachines        chinanorth  
                            TestNICSQL1         Microsoft.Network/networkInterfaces      chinanorth  
                            TestNICSQL2         Microsoft.Network/networkInterfaces      chinanorth  
                            TestNICWeb1         Microsoft.Network/networkInterfaces      chinanorth  
                            TestNICWeb2         Microsoft.Network/networkInterfaces      chinanorth  
                            NSG-BackEnd         Microsoft.Network/networkSecurityGroups  chinanorth  
                            NSG-FrontEnd        Microsoft.Network/networkSecurityGroups  chinanorth  
                            TestPIPSQL1         Microsoft.Network/publicIPAddresses      chinanorth  
                            TestPIPSQL2         Microsoft.Network/publicIPAddresses      chinanorth  
                            TestPIPWeb1         Microsoft.Network/publicIPAddresses      chinanorth  
                            TestPIPWeb2         Microsoft.Network/publicIPAddresses      chinanorth  
                            TestVNet            Microsoft.Network/virtualNetworks        chinanorth  
                            testvnetstorageprm  Microsoft.Storage/storageAccounts        chinanorth  
                            testvnetstoragestd  Microsoft.Storage/storageAccounts        chinanorth  
   
        ResourceId        : /subscriptions/[Subscription Id]/resourceGroups/TestRG

## 使用 Azure CLI 部署 ARM 模板
若要使用 Azure CLI 部署 ARM 模板，请执行下列步骤。

1. 如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **`azure config mode`** 命令，切换到 Resource Manager 模式，如下所示。

        azure config mode arm

    命令的预期输出如下所示：

        info:    New mode is arm

3. 运行 **`azure group deployment create`** cmdlet，使用上面下载并修改的模板和参数文件部署新的 VNet。输出后显示的列表阐释了所用参数。

        azure group create -n TestRG -l chinanorth -f 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.json' -e 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.parameters.json'

    预期输出：
   
        info:    Executing command group create
        info:    Getting resource group TestRG
        info:    Creating resource group TestRG
        info:    Created resource group TestRG
        info:    Initializing template configurations and parameters
        info:    Creating a deployment
        info:    Created template deployment "azuredeploy"
        data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG
        data:    Name:                TestRG
        data:    Location:            chinanorth
        data:    Provisioning State:  Succeeded
        data:    Tags: null
        data:    
        info:    group create command OK
   
   * **-n（或 --name）**。要创建的资源组的名称。
   * **-l（或 --location）**。要在其中创建资源组的 Azure 区域。
   * **-f（或 --template-file）**。ARM 模板文件的路径。
   * **-e（或 --parameters-file）**。ARM 参数文件的路径。

<!---HONumber=Mooncake_1219_2016-->