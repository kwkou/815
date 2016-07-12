<!-- ARM: tested -->

<properties
   pageTitle="在 Resource Manager 中使用模板通过静态公共 IP 部署 VM | Azure"
   description="了解如何在 Resource Manager 中使用模板通过静态公共 IP 部署 VM"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="04/27/2016"
	wacn.date="07/04/2016"/>

# 使用模板通过静态公共 IP 部署 VM

[AZURE.INCLUDE [virtual-network-deploy-static-pip-arm-selectors-include.md](../includes/virtual-network-deploy-static-pip-arm-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../includes/virtual-network-deploy-static-pip-intro-include.md)]

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。

[AZURE.INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../includes/virtual-network-deploy-static-pip-scenario-include.md)]

## 模板文件中的公共 IP 资源

你可以查看和下载[示例模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/azuredeploy.json)。

下面的部分说明如何基于上述方案定义公共 IP 资源。

      {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[variables('webVMSetting').pipName]",
        "location": "[variables('location')]",
        "properties": {
          "publicIPAllocationMethod": "Static"
        },
        "tags": {
          "displayName": "PublicIPAddress - Web"
        }
      },

请注意 **publicIPAllocationMethod** 属性，该属性已设置为“静态”。该属性可以是“动态”（默认值），也可以是“静态”。将其设置为静态可确保分配的公共 IP 地址永不更改。

以下部分说明了公共 IP 地址与网络接口的关联。

      {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/networkInterfaces",
        "name": "[variables('webVMSetting').nicName]",
        "location": "[variables('location')]",
        "tags": {
          "displayName": "NetworkInterface - Web"
        },
        "dependsOn": [
          "[concat('Microsoft.Network/publicIPAddresses/', variables('webVMSetting').pipName)]",
          "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
        ],
        "properties": {
          "ipConfigurations": [
            {
              "name": "ipconfig1",
              "properties": {
                "privateIPAllocationMethod": "Static",
                "privateIPAddress": "[variables('webVMSetting').ipAddress]",
                "publicIPAddress": {
                  "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('webVMSetting').pipName)]"
                },
                "subnet": {
                  "id": "[variables('frontEndSubnetRef')]"
                }
              }
            }
          ]
        }
      },

请注意，**publicIPAddress** 属性指向名为 **variables('webVMSetting').pipName** 的资源的 **Id**。该名称是上面所示的公共 IP 资源的名称。

最后，上述网络接口列在要创建的 VM 的 **networkProfile** 属性中。

      "networkProfile": {
        "networkInterfaces": [
          {
            "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('webVMSetting').nicName)]"
          }
        ]
      }

## 使用 PowerShell 部署模板

若要使用 PowerShell 部署下载的模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)，并按步骤 1 到 3 中的说明进行操作。

2. 在 PowerShell 控制台中，根据需要运行 **New-AzureRmResourceGroup** cmdlet 以创建新资源组。如果已创建资源组，请转到步骤 3。

		New-AzureRmResourceGroup -Name PIPTEST -Location chinanorth

	预期输出：

		ResourceGroupName : PIPTEST
		Location          : chinanorth
		ProvisioningState : Succeeded
		Tags              :
		ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/StaticPublicIP

3. 在 PowerShell 控制台中，运行 **New-AzureRmResourceGroupDeployment** cmdlet 以部署模板。

		New-AzureRmResourceGroupDeployment -Name DeployVM -ResourceGroupName PIPTEST `
		    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/azuredeploy.json `
		    -TemplateParameterUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/azuredeploy.parameters.json

	预期输出：

		DeploymentName    : DeployVM
		ResourceGroupName : PIPTEST
		ProvisioningState : Succeeded
		Timestamp         : <Deployment date> <Deployment time>
		Mode              : Incremental
		TemplateLink      :
		                    Uri            : https://raw.githubusercontent.com/Azure/azure-quickstart-templates/mas
		                    ter/IaaS-Story/03-Static-public-IP/azuredeploy.json
		                    ContentVersion : 1.0.0.0

		Parameters        :
		                    Name                      Type                       Value     
		                    ========================  =========================  ==========
		                    vnetName                  String                     WTestVNet
		                    vnetPrefix                String                     192.168.0.0/16
		                    frontEndSubnetName        String                     FrontEnd  
		                    frontEndSubnetPrefix      String                     192.168.1.0/24
		                    storageAccountNamePrefix  String                     iaasestd  
		                    stdStorageType            String                     Standard_LRS
		                    osType                    String                     Windows   
		                    adminUsername             String                     adminUser
		                    adminPassword             SecureString                         

		Outputs           :

## 使用 Azure CLI 部署模板

若要使用 Azure CLI 部署模板，请执行以下步骤。

1. 如果你从未用过 Azure CLI，请遵循 [Install and Configure the Azure CLI（安装和配置 Azure CLI）](/documentation/articles/xplat-cli-install/)一文中的步骤，然后遵循 [Connect to an Azure subscription from the Azure Command-Line Interface (Azure CLI)（从 Azure 命令行接口 (Azure CLI) 连接到 Azure 订阅）](/documentation/articles/xplat-cli-connect/)一文的“Use azure login to authenticate interactively”（使用 azure 登录名以交互方式进行身份验证）部分中所述的，将 CLI 连接到订阅的步骤。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

3. 打开[参数文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/azuredeploy.parameters.json)，选择其内容，然后将其保存到计算机上的文件中。在本示例中，参数已保存到名为 parameters.json 的文件。如果需要，请更改文件中的参数值，但建议至少将 adminPassword 参数的值更改为唯一的复杂密码。

4. 运行 **azure group deployment create** cmdlet 以使用你在前面下载并修改的模板和参数文件部署新 VNet。在以下命令中，将 <path> 替换为文件保存到的目标路径。

		azure group create -n PIPTEST2 -l chinanorth --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/azuredeploy.json -e <path>\parameters.json

	预期输出（列出使用的参数值）：

		info:    Executing command group create
		+ Getting resource group PIPTEST2
		+ Creating resource group PIPTEST2
		info:    Created resource group PIPTEST2
		+ Initializing template configurations and parameters
		+ Creating a deployment
		info:    Created template deployment "azuredeploy"
		data:    Id:                  /subscriptions/<Subscription ID>/resourceGroups/PIPTEST2
		data:    Name:                PIPTEST2
		data:    Location:            chinanorth
		data:    Provisioning State:  Succeeded
		data:    Tags: null
		data:
		info:    group create command OK

<!---HONumber=Mooncake_0523_2016-->