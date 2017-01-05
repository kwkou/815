<properties
    pageTitle="使用模板控制路由和虚拟设备 | Azure"
    description="了解如何使用 Azure Resource Manager 模板控制路由和虚拟设备。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="832c7831-d0e9-449b-b39c-9a09ba051531"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/23/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用模板创建用户定义的路由 (UDR)
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/virtual-network-create-udr-arm-ps/)
- [Azure CLI](/documentation/articles/virtual-network-create-udr-arm-cli/)
- [模板](/documentation/articles/virtual-network-create-udr-arm-template/)
- [PowerShell（经典）](/documentation/articles/virtual-network-create-udr-classic-ps/)
- [CLI（经典）](/documentation/articles/virtual-network-create-udr-classic-cli/)

> [AZURE.IMPORTANT]
在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Azure Resource Manager 部署模型和经典部署模型。在使用任何 Azure 资源前，请确保了解[部署模型和工具](/documentation/articles/resource-manager-deployment-model/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍 Resource Manager 部署模型。

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../../includes/virtual-network-create-udr-scenario-include.md)]

## 模板文件中的 UDR 资源
可查看和下载[示例模板](https://github.com/telmosampaio/azure-templates/tree/master/IaaS-NSG-UDR)。

以下部分说明如何针对方案在 **azuredeploy-vnet-nsg-udr.json** 文件中定义前端 UDR：

	"apiVersion": "2015-06-15",
	"type": "Microsoft.Network/routeTables",
	"name": "[parameters('frontEndRouteTableName')]",
	"location": "[resourceGroup().location]",
	"tags": {
	  "displayName": "UDR - FrontEnd"	
	},
	"properties": {
	  "routes": [
	    {
	      "name": "RouteToBackEnd",
	      "properties": {
	        "addressPrefix": "[parameters('backEndSubnetPrefix')]",
	        "nextHopType": "VirtualAppliance",
	        "nextHopIpAddress": "[parameters('vmaIpAddress')]"
	      }
	    }
	  ]

若要将 UDR 与前端子网关联，需要更改模板中的子网定义，并使用 UDR 的引用 ID。

    "subnets": [
        "name": "[parameters('frontEndSubnetName')]",
        "properties": {
          "addressPrefix": "[parameters('frontEndSubnetPrefix')]",
          "networkSecurityGroup": {
            "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('frontEndNSGName'))]"
          },
          "routeTable": {
              "id": "[resourceId('Microsoft.Network/routeTables', parameters('frontEndRouteTableName'))]"
          }
        },

请注意，在模板中对后端 NSG 和后端子网执行相同的操作。

还需要确保 **FW1** VM 在用于接收和转发数据包的 NIC 上启用了 IP 转发属性。以下部分显示了 azuredeploy-nsg-udr.json 文件中 FW1 的 NIC 的定义（基于上述方案）。

    "apiVersion": "2015-06-15",
    "type": "Microsoft.Network/networkInterfaces",
    "location": "[variables('location')]",
    "tags": {
      "displayName": "NetworkInterfaces - DMZ"
    },
    "name": "[concat(variables('fwVMSettings').nicName, copyindex(1))]",
    "dependsOn": [
      "[concat('Microsoft.Network/publicIPAddresses/', variables('fwVMSettings').pipName, copyindex(1))]",
      "[concat('Microsoft.Resources/deployments/', 'vnetTemplate')]"
    ],
    "properties": {
      "ipConfigurations": [
        {
          "name": "ipconfig1",
          "properties": {
            "enableIPForwarding": true,
            "privateIPAllocationMethod": "Static",
            "privateIPAddress": "[concat('192.168.0.',copyindex(4))]",
            "publicIPAddress": {
              "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('fwVMSettings').pipName, copyindex(1)))]"
            },
            "subnet": {
              "id": "[variables('dmzSubnetRef')]"
            }
          }
        }
      ]
    },
    "copy": {
      "name": "fwniccount",
      "count": "[parameters('fwCount')]"
    }

## 使用 PowerShell 部署 ARM 模板

若要使用 PowerShell 部署下载的 ARM 模板，请执行以下步骤。

1. 如果从未使用过 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)，按照说明逐步完成操作，登录到 Azure 并选择订阅。
2. 运行以下命令创建资源组：

        New-AzureRmResourceGroup -Name TestRG -Location chinanorth

3. 运行以下命令，部署模板：

        New-AzureRmResourceGroupDeployment -Name DeployUDR -ResourceGroupName TestRG `
            -TemplateUri https://raw.githubusercontent.com/telmosampaio/azure-templates/master/IaaS-NSG-UDR/azuredeploy.json `
            -TemplateParameterUri https://raw.githubusercontent.com/telmosampaio/azure-templates/master/IaaS-NSG-UDR/azuredeploy.parameters.json

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
                            ASFW                Microsoft.Compute/availabilitySets       chinanorth  
                            ASSQL               Microsoft.Compute/availabilitySets       chinanorth  
                            ASWEB               Microsoft.Compute/availabilitySets       chinanorth  
                            FW1                 Microsoft.Compute/virtualMachines        chinanorth  
                            SQL1                Microsoft.Compute/virtualMachines        chinanorth  
                            SQL2                Microsoft.Compute/virtualMachines        chinanorth  
                            WEB1                Microsoft.Compute/virtualMachines        chinanorth  
                            WEB2                Microsoft.Compute/virtualMachines        chinanorth  
                            NICFW1              Microsoft.Network/networkInterfaces      chinanorth  
                            NICSQL1             Microsoft.Network/networkInterfaces      chinanorth  
                            NICSQL2             Microsoft.Network/networkInterfaces      chinanorth  
                            NICWEB1             Microsoft.Network/networkInterfaces      chinanorth  
                            NICWEB2             Microsoft.Network/networkInterfaces      chinanorth  
                            NSG-BackEnd         Microsoft.Network/networkSecurityGroups  chinanorth  
                            NSG-FrontEnd        Microsoft.Network/networkSecurityGroups  chinanorth  
                            PIPFW1              Microsoft.Network/publicIPAddresses      chinanorth  
                            PIPSQL1             Microsoft.Network/publicIPAddresses      chinanorth  
                            PIPSQL2             Microsoft.Network/publicIPAddresses      chinanorth  
                            PIPWEB1             Microsoft.Network/publicIPAddresses      chinanorth  
                            PIPWEB2             Microsoft.Network/publicIPAddresses      chinanorth  
                            UDR-BackEnd         Microsoft.Network/routeTables            chinanorth  
                            UDR-FrontEnd        Microsoft.Network/routeTables            chinanorth  
                            TestVNet            Microsoft.Network/virtualNetworks        chinanorth  
                            testvnetstorageprm  Microsoft.Storage/storageAccounts        chinanorth  
                            testvnetstoragestd  Microsoft.Storage/storageAccounts        chinanorth

        ResourceId        : /subscriptions/[Subscription Id]/resourceGroups/TestRG

## 使用 Azure CLI 部署模板

若要使用 Azure CLI 部署 ARM 模板，请完成以下步骤：

1. 如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行以下命令，切换到 Resource Manager 模式：

        azure config mode arm

    以上命令的预期输出如下：

		info:    New mode is arm

3. 从浏览器导航到 **https://raw.githubusercontent.com/telmosampaio/azure-templates/master/IaaS-NSG-UDR/azuredeploy.parameters.json** ，复制 json 文件的内容并粘贴到计算机中的新文件。对于此方案，将下面的值复制到名为 **c:\\udr\\azuredeploy.parameters.json** 的文件。

            {
              "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
              "contentVersion": "1.0.0.0",
              "parameters": {
                "fwCount": {
                  "value": 1
                },
                "webCount": {
                  "value": 2
                },
                "sqlCount": {
                  "value": 2
                }
              }
            }

4. 运行以下命令，使用上面下载并修改的模板和参数文件部署新的 VNet：

        azure group create -n TestRG -l chinanorth --template-uri 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/IaaS-NSG-UDR/azuredeploy.json' -e 'c:\udr\azuredeploy.parameters.json'

    预期输出：
   
        info:    Executing command group create
        info:    Getting resource group TestRG
        info:    Updating resource group TestRG
        info:    Updated resource group TestRG
        info:    Initializing template configurations and parameters
        info:    Creating a deployment
        info:    Created template deployment "azuredeploy"
        data:    Id:                  /subscriptions/[Subscription Id]/resourceGroups/TestRG
        data:    Name:                TestRG
        data:    Location:            chinanorth
        data:    Provisioning State:  Succeeded
        data:    Tags: null
        data:    
        info:    group create command OK

5. 运行以下命令，查看在新资源组中创建的资源：

        azure group show TestRG

    预期结果：

	        info:    Executing command group show
	        info:    Listing resource groups
	        info:    Listing resources for the group
	        data:    Id:                  /subscriptions/[Subscription Id]/resourceGroups/TestRG
	        data:    Name:                TestRG
	        data:    Location:            chinanorth
	        data:    Provisioning State:  Succeeded
	        data:    Tags: null
	        data:    Resources:
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/availabilitySets/ASFW
	        data:      Name    : ASFW
	        data:      Type    : availabilitySets
	        data:      Location: chinanorth
	        data:      Tags    : displayName=AvailabilitySet - DMZ
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/availabilitySets/ASSQL
	        data:      Name    : ASSQL
	        data:      Type    : availabilitySets
	        data:      Location: chinanorth
	        data:      Tags    : displayName=AvailabilitySet - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/availabilitySets/ASWEB
	        data:      Name    : ASWEB
	        data:      Type    : availabilitySets
	        data:      Location: chinanorth
	        data:      Tags    : displayName=AvailabilitySet - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/FW1
	        data:      Name    : FW1
	        data:      Type    : virtualMachines
	        data:      Location: chinanorth
	        data:      Tags    : displayName=VMs - DMZ
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/SQL1
	        data:      Name    : SQL1
	        data:      Type    : virtualMachines
	        data:      Location: chinanorth
	        data:      Tags    : displayName=VMs - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/SQL2
	        data:      Name    : SQL2
	        data:      Type    : virtualMachines
	        data:      Location: chinanorth
	        data:      Tags    : displayName=VMs - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/WEB1
	        data:      Name    : WEB1
	        data:      Type    : virtualMachines
	        data:      Location: chinanorth
	        data:      Tags    : displayName=VMs - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/WEB2
	        data:      Name    : WEB2
	        data:      Type    : virtualMachines
	        data:      Location: chinanorth
	        data:      Tags    : displayName=VMs - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICFW1
		        data:      Name    : NICFW1
        data:      Type    : networkInterfaces
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NetworkInterfaces - DMZ
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICSQL1
	        data:      Name    : NICSQL1
	        data:      Type    : networkInterfaces
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NetworkInterfaces - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICSQL2
	        data:      Name    : NICSQL2
	        data:      Type    : networkInterfaces
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NetworkInterfaces - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB1
	        data:      Name    : NICWEB1
	        data:      Type    : networkInterfaces
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NetworkInterfaces - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB2
	        data:      Name    : NICWEB2
	        data:      Type    : networkInterfaces
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NetworkInterfaces - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd
	        data:      Name    : NSG-BackEnd
	        data:      Type    : networkSecurityGroups
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NSG - Front End
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
	        data:      Name    : NSG-FrontEnd
	        data:      Type    : networkSecurityGroups
	        data:      Location: chinanorth
	        data:      Tags    : displayName=NSG - Remote Access
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPFW1
	        data:      Name    : PIPFW1
	        data:      Type    : publicIPAddresses
	        data:      Location: chinanorth
	        data:      Tags    : displayName=PublicIPAddresses - DMZ
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPSQL1
	        data:      Name    : PIPSQL1
		        data:      Type    : publicIPAddresses
	        data:      Location: chinanorth
	        data:      Tags    : displayName=PublicIPAddresses - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPSQL2
	        data:      Name    : PIPSQL2
	        data:      Type    : publicIPAddresses
	        data:      Location: chinanorth
	        data:      Tags    : displayName=PublicIPAddresses - SQL
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPWEB1
	        data:      Name    : PIPWEB1
	        data:      Type    : publicIPAddresses
	        data:      Location: chinanorth
	        data:      Tags    : displayName=PublicIPAddresses - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPWEB2
	        data:      Name    : PIPWEB2
	        data:      Type    : publicIPAddresses
	        data:      Location: chinanorth
	        data:      Tags    : displayName=PublicIPAddresses - Web
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-BackEnd
	        data:      Name    : UDR-BackEnd
	        data:      Type    : routeTables
	        data:      Location: chinanorth
	        data:      Tags    : displayName=Route Table - Back End
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-FrontEnd
	        data:      Name    : UDR-FrontEnd
	        data:      Type    : routeTables
	        data:      Location: chinanorth
	        data:      Tags    : displayName=UDR - FrontEnd
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
	        data:      Name    : TestVNet
	        data:      Type    : virtualNetworks
	        data:      Location: chinanorth
	        data:      Tags    : displayName=VNet
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/testvnetstorageprm
	        data:      Name    : testvnetstorageprm
	        data:      Type    : storageAccounts
	        data:      Location: chinanorth
	        data:      Tags    : displayName=Storage Account - Premium
	        data:    
	        data:      Id      : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Storage/storageAccounts/testvnetstoragestd
	        data:      Name    : testvnetstoragestd
	        data:      Type    : storageAccounts
	        data:      Location: chinanorth
	        data:      Tags    : displayName=Storage Account - Simple
	        data:    
	        data:    Permissions:
	        data:      Actions: *
	        data:      NotActions: 
	        data:
	        info:    group show command OK

> [AZURE.TIP]
如果看不到所有资源，可运行 `azure group deployment show` 命令以确保部署的预配状态为“成功”。
> 

<!---HONumber=Mooncake_1219_2016-->