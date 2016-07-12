<!-- ARM: tested -->

<properties 
   pageTitle="如何使用模板在 ARM 模式下创建 NSG | Azure"
   description="了解如何使用模板在 ARM 下创建和部署 NSG"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/02/2016"
	wacn.date="07/04/2016"/>

# 如何使用模板创建 NSG

[AZURE.INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-nsg-intro-include](../includes/virtual-networks-create-nsg-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[在经典部署模型中创建 NSG](/documentation/articles/virtual-networks-create-nsg-classic-ps/)。

[AZURE.INCLUDE [virtual-networks-create-nsg-scenario-include](../includes/virtual-networks-create-nsg-scenario-include.md)]

## 使用 PowerShell 部署 ARM 模板

若要使用 PowerShell 部署下载的 ARM 模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)，并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

3. 运行 **New-AzureRMResourceGroup** cmdlet 以使用模板创建资源组。

		New-AzureRMResourceGroup -Name TestRG -Location uswest `
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
		                    
		ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG

## 使用 Azure CLI 部署 ARM 模板

若要使用 Azure CLI 部署 ARM 模板，请执行下列步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

4. 运行 **azure group deployment create** cmdlet 以使用你在前面下载并修改的模板和参数文件部署新 VNet。在输出后显示的列表说明了所用的参数。

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

	- **-n（或 --name）**。要创建的资源组的名称。
	- **-l（或 --location）**。要创建资源组所在的 Azure 区域。
	- **-f（或 --template-file）**。ARM 模板文件的路径。
	- **-e（或 --parameters-file）**。ARM 参数文件的路径。

<!---HONumber=79-->