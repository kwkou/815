<properties
   pageTitle="如何使用 PowerShell 在 Azure Resource Manager 中创建 NSG | Azure"
   description="了解如何使用 PowerShell 在 Azure Resource Manager 中创建和部署 NSG"
   services="virtual-network"
   documentationCenter="na"
   authors="jimdial"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>  

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/23/2016"
   wacn.date="12/12/2016"
   ms.author="jdial" />  


# 如何使用 PowerShell 在 Resource Manager 中创建 NSG

[AZURE.INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[在经典部署模型中创建 NSG](/documentation/articles/virtual-networks-create-nsg-classic-ps/)。

[AZURE.INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

下面的示例 PowerShell 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先通过部署[此模板](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd)构建测试环境，单击“部署至 Azure”，如有必要替换默认参数值，然后按照门户中的说明进行操作。

## 如何为前端子网创建 NSG
若要基于上述方案创建名为 *NSG-FrontEnd* 的 NSG，请执行下面的步骤：

[AZURE.INCLUDE [powershell-preview-include.md](../../includes/powershell-preview-include.md)]

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

2. 创建允许从 Internet 访问端口 3389 的安全规则。

		$rule1 = New-AzureRmNetworkSecurityRuleConfig -Name rdp-rule -Description "Allow RDP"
		    -Access Allow -Protocol Tcp -Direction Inbound -Priority 100
		    -SourceAddressPrefix Internet -SourcePortRange *
		    -DestinationAddressPrefix * -DestinationPortRange 3389

3. 创建允许从 Internet 访问端口 80 的安全规则。

		$rule2 = New-AzureRmNetworkSecurityRuleConfig -Name web-rule -Description "Allow HTTP"
		    -Access Allow -Protocol Tcp -Direction Inbound -Priority 101
            -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix *
            -DestinationPortRange 80

4. 将上面创建的规则添加到名为 **NSG-FrontEnd** 的新 NSG。

		$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location chinanorth
        -Name "NSG-FrontEnd" -SecurityRules $rule1,$rule2

5. 检查在 NSG 中创建的规则。

		$nsg

	输出仅显示安全规则：

		SecurityRules        : [
		                         {
		                           "Name": "rdp-rule",
		                           "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
		                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/rdp-rule",
		                           "Description": "Allow RDP",
		                           "Protocol": "Tcp",
		                           "SourcePortRange": "*",
		                           "DestinationPortRange": "3389",
		                           "SourceAddressPrefix": "Internet",
		                           "DestinationAddressPrefix": "*",
		                           "Access": "Allow",
		                           "Priority": 100,
		                           "Direction": "Inbound",
		                           "ProvisioningState": "Succeeded"
		                         },
		                         {
		                           "Name": "web-rule",
		                           "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
		                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/web-rule",
		                           "Description": "Allow HTTP",
		                           "Protocol": "Tcp",
		                           "SourcePortRange": "*",
		                           "DestinationPortRange": "80",
		                           "SourceAddressPrefix": "Internet",
		                           "DestinationAddressPrefix": "*",
		                           "Access": "Allow",
		                           "Priority": 101,
		                           "Direction": "Inbound",
		                           "ProvisioningState": "Succeeded"
		                         }
		                       ]

6. 将上面创建的 NSG 与 *FrontEnd* 子网关联起来。

            		$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
            		Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd
            			-AddressPrefix 192.168.1.0/24 -NetworkSecurityGroup $nsg

            	Output showing only the *FrontEnd* subnet settings, notice the value for the **NetworkSecurityGroup** property:

            		Subnets           : [
            		                      {
            		                        "Name": "FrontEnd",
            		                        "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
            		                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
            		                        "AddressPrefix": "192.168.1.0/24",
            		                        "IpConfigurations": [
            		                          {
            		                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1"
            		                          },
            		                          {
            		                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1"
            		                          }
            		                        ],
            		                        "NetworkSecurityGroup": {
            		                          "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
            		                        },
            		                        "RouteTable": null,
            		                        "ProvisioningState": "Succeeded"
            		                      }

    >[AZURE.WARNING] 上述命令的输出显示虚拟网络配置对象的内容，该对象仅存在于运行 PowerShell 的计算机上。若要将这些设置保存到 Azure，需要运行 `Set-AzureRmVirtualNetwork` cmdlet。

7. 将新的 VNet 设置保存到 Azure。

		Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

	输出仅显示 NSG 部分：

		"NetworkSecurityGroup": {
		  "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
		}

## 如何为后端子网创建 NSG
若要基于上述方案创建名为 *NSG-BackEnd* 的 NSG，请执行下面的步骤：

1. 创建允许从前端子网访问端口 1433（SQL Server 使用的默认端口）的安全规则。

		$rule1 = New-AzureRmNetworkSecurityRuleConfig -Name frontend-rule -Description "Allow FE subnet"
		    -Access Allow -Protocol Tcp -Direction Inbound -Priority 100
		    -SourceAddressPrefix 192.168.1.0/24 -SourcePortRange *
		    -DestinationAddressPrefix * -DestinationPortRange 1433

2. 创建阻止访问 Internet 的安全规则。

		$rule2 = New-AzureRmNetworkSecurityRuleConfig -Name web-rule -Description "Block Internet"
		    -Access Deny -Protocol * -Direction Outbound -Priority 200
		    -SourceAddressPrefix * -SourcePortRange *
		    -DestinationAddressPrefix Internet -DestinationPortRange *

3. 将上面创建的规则添加到名为 **NSG-BackEnd** 的新 NSG。

		$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location chinanorth -Name "NSG-BackEnd"
			-SecurityRules $rule1,$rule2

4. 将上面创建的 NSG 与 *BackEnd* 子网关联起来。

		Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name BackEnd
			-AddressPrefix 192.168.2.0/24 -NetworkSecurityGroup $nsg

	输出只显示 *BackEnd* 子网设置，注意 **NetworkSecurityGroup** 属性值：

		Subnets           : [
                      {
                        "Name": "BackEnd",
                        "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
                        "AddressPrefix": "192.168.2.0/24",
                        "IpConfigurations": [...],
                        "NetworkSecurityGroup": {
                          "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd"
                        },
                        "RouteTable": null,
                        "ProvisioningState": "Succeeded"
                      }

5. 将新的 VNet 设置保存到 Azure。

		Set-AzureRmVirtualNetwork -VirtualNetwork $vnet


## 如何删除 NSG

若要删除现有的 NSG（在本例中名为 *NSG-Frontend*），请执行以下步骤：

如下所示运行 **Remove-AzureRmNetworkSecurityGroup**，请务必包含 NSG 所在的资源组。

            Remove-AzureRmNetworkSecurityGroup -Name "NSG-FrontEnd" -ResourceGroupName "TestRG"

<!---HONumber=Mooncake_Quality_Review_1118_2016-->