<!-- ARM: tested -->

<properties 
   pageTitle="在 Resource Manager 中使用 PowerShell 管理 NSG | Azure"
   description="了解如何在 Resource Manager 中使用 PowerShell 管理现有 NSG"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/14/2016"
	wacn.date="06/06/2016"/>

# 使用 PowerShell 管理 NSG

[AZURE.INCLUDE [virtual-network-manage-arm-selectors-include.md](../includes/virtual-network-manage-nsg-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

[AZURE.INCLUDE [virtual-network-manage-nsg-intro-include.md](../includes/virtual-network-manage-nsg-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [virtual-network-manage-nsg-arm-scenario-include.md](../includes/virtual-network-manage-nsg-arm-scenario-include.md)]

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]

## 检索信息

可以查看你的现有 NSG、检索现有 NSG 的规则和查找与 NSG 关联的资源。

### 查看现有 NSG
若要查看订阅中的全部现有 NSG，请运行 `Get-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

	Get-AzureRmNetworkSecurityGroup

预期结果：

	Name                 : NSG-BackEnd
	ResourceGroupName    : RG-NSG
	Location             : chinanorth
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/
	                       Microsoft.Network/networkSecurityGroups/NSG-BackEnd
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ResourceGuid         : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	ProvisioningState    : Succeeded
	Tags                 : 	                       
	SecurityRules        : [...]
	DefaultSecurityRules : [...]
	NetworkInterfaces    : [...]
	Subnets              : [...]
	
	Name                 : NSG-FrontEnd
	ResourceGroupName    : RG-NSG
	Location             : chinaeast
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/NRP-RG/providers/
	                       Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ResourceGuid         : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	ProvisioningState    : Succeeded
	Tags                 : 
	SecurityRules        : [...]
	DefaultSecurityRules : [...]
	NetworkInterfaces    : [...]
	Subnets              : [...]
	                     	
	Name                 : WEB1
	ResourceGroupName    : RG101
	Location             : chinaeast2
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG101/providers/M
	                       icrosoft.Network/networkSecurityGroups/WEB1
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ResourceGuid         : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	ProvisioningState    : Succeeded
	Tags                 : 
	SecurityRules        : [...]
	DefaultSecurityRules : [...]
	NetworkInterfaces    : [...]
	Subnets              : [...]


要查看特定资源组中 NSG 的列表，请运行 `Get-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

	Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG

预期输出：

	Name                 : NSG-BackEnd
	ResourceGroupName    : RG-NSG
	Location             : chinanorth
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/
	                       Microsoft.Network/networkSecurityGroups/NSG-BackEnd
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ResourceGuid         : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	ProvisioningState    : Succeeded
	Tags                 : 	                       
	SecurityRules        : [...]
	DefaultSecurityRules : [...]
	NetworkInterfaces    : [...]
	Subnets              : [...]
	
	Name                 : NSG-FrontEnd
	ResourceGroupName    : RG-NSG
	Location             : chinaeast
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/NRP-RG/providers/
	                       Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ResourceGuid         : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	ProvisioningState    : Succeeded
	Tags                 : 
	SecurityRules        : [...]
	DefaultSecurityRules : [...]
	NetworkInterfaces    : [...]
	Subnets              : [...]
		 
### 列出 NSG 的所有规则

若要查看名为 **NSG-FrontEnd** 的 NSG 的规则，请运行 `Get-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

	Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd | Select SecurityRules -ExpandProperty SecurityRules

预期输出：
	
	Name                     : rdp-rule
	Id                       : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/						   Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/rdp-rule
	Etag                     : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ProvisioningState        : Succeeded
	Description              : Allow RDP
	Protocol                 : Tcp
	SourcePortRange          : *
	DestinationPortRange     : 3389
	SourceAddressPrefix      : Internet
	DestinationAddressPrefix : *
	Access                   : Allow
	Priority                 : 100
	Direction                : Inbound
	
	Name                     : web-rule
	Id                       : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/						   Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/web-rule
	Etag                     : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ProvisioningState        : Succeeded
	Description              : Allow HTTP
	Protocol                 : Tcp
	SourcePortRange          : *
	DestinationPortRange     : 80
	SourceAddressPrefix      : Internet
	DestinationAddressPrefix : *
	Access                   : Allow
	Priority                 : 101
	Direction                : Inbound

>[AZURE.NOTE] 还可以使用 `Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name "NSG-FrontEnd" | Select DefaultSecurityRules -ExpandProperty DefaultSecurityRules` 从 **NSG-FrontEnd** NSG 列出默认规则。

###<a name="View-NSGs-associations"></a> 查看 NSG 关联项

若要查看与 **NSG-FrontEnd** NSG 相关联的资源，请运行 `Get-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

	Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

查找 **NetworkInterfaces** 和 **Subnets** 属性，如下所示：

	NetworkInterfaces    : []
	Subnets              : [
	                         {
	                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
	                           "IpConfigurations": []
	                         }
	                       ]

在上述示例中，NSG 不与任何网络接口 (NIC) 关联，而是与名为 **FrontEnd** 的子网关联。

## 管理规则

可向现有 NSG 添加规则、编辑现有规则和删除规则。

### 添加规则

若要向 **NSG-FrontEnd** NSG 添加规则，这一规则允许来自任何计算机的**入站**流量流入端口 **443**，请按照以下步骤进行操作。

1. 运行 `Get-AzureRmNetworkSecurityGroup` cmdlet 来检索现有 NSG 并将其存储在变量中，如下所示。

		$nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG `
		    -Name NSG-FrontEnd

2. 运行 `Add-AzureRmNetworkSecurityRuleConfig` cmdlet，如下所示。

		Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
		    -Name https-rule `
		    -Description "Allow HTTPS" `
		    -Access Allow `
		    -Protocol Tcp `
		    -Direction Inbound `
		    -Priority 102 `
		    -SourceAddressPrefix * `
		    -SourcePortRange * `
		    -DestinationAddressPrefix * `
		    -DestinationPortRange 443

3. 若要保存对 NSG 所做的更改，请运行 `Set-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

		Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg

	预期输出仅显示安全规则：

		Name                 : NSG-FrontEnd
		...
		SecurityRules        : [
		                         {
		                           "Name": "rdp-rule",
		                           ...
		                         },
		                         {
		                           "Name": "web-rule",
		                           ...
		                         },
		                         {
		                           "Name": "https-rule",
		                           "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
		                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/https-rule",
		                           "Description": "Allow HTTPS",
		                           "Protocol": "Tcp",
		                           "SourcePortRange": "*",
		                           "DestinationPortRange": "443",
		                           "SourceAddressPrefix": "*",
		                           "DestinationAddressPrefix": "*",
		                           "Access": "Allow",
		                           "Priority": 102,
		                           "Direction": "Inbound",
		                           "ProvisioningState": "Succeeded"
		                         }
		                       ]

### 更改规则

若要将上面创建的规则更改为仅允许来自 **Internet** 的入站流量，请按照以下步骤进行操作。

1. 运行 `Get-AzureRmNetworkSecurityGroup` cmdlet 来检索现有 NSG 并将其存储在变量中，如下所示。

		$nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG `
		    -Name NSG-FrontEnd

2. 运行 `Set-AzureRmNetworkSecurityRuleConfig` cmdlet，如下所示。

		Set-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
		    -Name https-rule `
		    -Description "Allow HTTPS" `
		    -Access Allow `
		    -Protocol Tcp `
		    -Direction Inbound `
		    -Priority 102 `
		    -SourceAddressPrefix * `
		    -SourcePortRange Internet `
		    -DestinationAddressPrefix * `
		    -DestinationPortRange 443

3. 若要保存对 NSG 所做的更改，请运行 `Set-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

		Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg

	预期输出仅显示安全规则：

		Name                 : NSG-FrontEnd
		...
		SecurityRules        : [
		                         {
		                           "Name": "rdp-rule",
		                           ...
		                         },
		                         {
		                           "Name": "web-rule",
		                           ...
		                         },
		                         {
		                           "Name": "https-rule",
		                           "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
		                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/https-rule",
		                           "Description": "Allow HTTPS",
		                           "Protocol": "Tcp",
		                           "SourcePortRange": "*",
		                           "DestinationPortRange": "443",
		                           "SourceAddressPrefix": "Internet",
		                           "DestinationAddressPrefix": "*",
		                           "Access": "Allow",
		                           "Priority": 102,
		                           "Direction": "Inbound",
		                           "ProvisioningState": "Succeeded"
		                         }
		                       ]

### 删除规则

1. 运行 `Get-AzureRmNetworkSecurityGroup` cmdlet 来检索现有 NSG 并将其存储在变量中，如下所示。

		$nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG `
		    -Name NSG-FrontEnd

2. 运行 `Remove-AzureRmNetworkSecurityRuleConfig` cmdlet，如下所示。

		Remove-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
		    -Name https-rule

3. 若要保存对 NSG 所做的更改，请运行 `Set-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

		Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg

	预期输出仅显示安全规则，请注意不再列出 **https-rule**：

		Name                 : NSG-FrontEnd
		...
		SecurityRules        : [
		                         {
		                           "Name": "rdp-rule",
		                           ...
		                         },
		                         {
		                           "Name": "web-rule",
		                           ...
		                         }
		                       ]

## 管理关联项

可将 NSG 关联到子网和 NIC。还可取消 NSG 与任何相关联的资源之间的关联。

### 将 NSG 关联到 NIC

若要将 **NSG-FrontEnd** NSG 关联到 **TestNICWeb1** NIC，请按照以下步骤进行操作。

1. 运行 `Get-AzureRmNetworkSecurityGroup` cmdlet 来检索现有 NSG 并将其存储在变量中，如下所示。

		$nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG `
		    -Name NSG-FrontEnd

2. 运行 `Get-AzureRmNetworkInterface` cmdlet 来检索现有 NIC 并将其存储在变量中，如下所示。

		$nic = Get-AzureRmNetworkInterface -ResourceGroupName RG-NSG `
		    -Name TestNICWeb1

3. 将 **NIC** 变量的 **NetworkSecurityGroup** 属性设置为 **NSG** 变量的值，如下所示。

		$nic.NetworkSecurityGroup = $nsg

4. 若要保存对 NIC 所做的更改，请运行 `Set-AzureRmNetworkInterface` cmdlet，如下所示。

		Set-AzureRmNetworkInterface -NetworkInterface $nic

	预期输出仅显示 **NetworkSecurityGroup** 属性：

		NetworkSecurityGroup : {
		                         "SecurityRules": [],
		                         "DefaultSecurityRules": [],
		                         "NetworkInterfaces": [],
		                         "Subnets": [],
		                         "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
		                       }

###<a name="Dissociate-an-NSG-from-a-NIC"></a> 取消 NSG 与 NIC 之间的关联

若要取消 **NSG-FrontEnd** NSG 与 **TestNICWeb1** NIC 之间的关联，请按照以下步骤进行操作。

1. 运行 `Get-AzureRmNetworkSecurityGroup` cmdlet 来检索现有 NSG 并将其存储在变量中，如下所示。

		$nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG `
		    -Name NSG-FrontEnd

2. 运行 `Get-AzureRmNetworkInterface` cmdlet 来检索现有 NIC 并将其存储在变量中，如下所示。

		$nic = Get-AzureRmNetworkInterface -ResourceGroupName RG-NSG `
		    -Name TestNICWeb1

3. 将 **NIC** 变量的 **NetworkSecurityGroup** 属性设置为 **$null**，如下所示。

		$nic.NetworkSecurityGroup = $null

4. 若要保存对 NIC 所做的更改，请运行 `Set-AzureRmNetworkInterface` cmdlet，如下所示。

		Set-AzureRmNetworkInterface -NetworkInterface $nic

	预期输出仅显示 **NetworkSecurityGroup** 属性：

		NetworkSecurityGroup : null

###<a name="Dissociate-an-NSG-from-a-subnet"></a> 取消 NSG 与子网之间的关联

若要取消 **NSG-FrontEnd** NSG 与 **FrontEnd** 子网之间的关联，请按照以下步骤进行操作。

1. 运行 `Get-AzureRmVirtualNetwork` cmdlet 来检索现有 VNet 并将其存储在变量中，如下所示。

		$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName RG-NSG `
		    -Name TestVNet

2. 运行 `Get-AzureRmVirtualNetworkSubnetConfig` cmdlet 来检索 **FrontEnd** 子网并将其存储在变量中，如下所示。

		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet `
		    -Name FrontEnd 

3. 将 **subnet** 变量的 **NetworkSecurityGroup** 属性设置为 **$null**，如下所示。

		$subnet.NetworkSecurityGroup = $null

4. 若要保存对子网所做的更改，请运行 `Set-AzureRmVirtualNetwork` cmdlet，如下所示。

		Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

	预期输出仅显示 **FrontEnd** 子网的属性。请注意 **NetworkSecurityGroup** 没有属性：

			...
			Subnets           : [
			                      {
			                        "Name": "FrontEnd",
			                        "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
			                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
			                        "AddressPrefix": "192.168.1.0/24",
			                        "IpConfigurations": [
			                          {
			                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1"
			                          },
			                          {
			                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1"
			                          }
			                        ],
			                        "ProvisioningState": "Succeeded"
			                      },
									...
			                    ]

### 将 NSG 关联到子网

若要再次将 **NSG-FrontEnd** NSG 关联到 **FronEnd** 子网，请按照以下步骤进行操作。

1. 运行 `Get-AzureRmVirtualNetwork` cmdlet 来检索现有 VNet 并将其存储在变量中，如下所示。

		$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName RG-NSG `
		    -Name TestVNet

2. 运行 `Get-AzureRmVirtualNetworkSubnetConfig` cmdlet 来检索 **FrontEnd** 子网并将其存储在变量中，如下所示。

		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet `
		    -Name FrontEnd 

1. 运行 `Get-AzureRmNetworkSecurityGroup` cmdlet 来检索现有 NSG 并将其存储在变量中，如下所示。

		$nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG `
		    -Name NSG-FrontEnd

3. 将 **subnet** 变量的 **NetworkSecurityGroup** 属性设置为 **$null**，如下所示。

		$subnet.NetworkSecurityGroup = $nsg

4. 若要保存对子网所做的更改，请运行 `Set-AzureRmVirtualNetwork` cmdlet，如下所示。

		Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

	预期输出仅显示 **FrontEnd** 子网的 **NetworkSecurityGroup** 属性：

		...
		"NetworkSecurityGroup": {
		                          "SecurityRules": [],
		                          "DefaultSecurityRules": [],
		                          "NetworkInterfaces": [],
		                          "Subnets": [],
		                          "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
		                        }
		...

## 删除 NSG

仅当 NSG 不与任何资源关联时，才能删除 NSG。若要删除 NSG，请按照以下步骤进行操作。

1. 若要检查与 NSG 关联的资源，请运行 `azure network nsg show`，如[查看 NSG 关联项](#View-NSGs-associations)中所示。
2. 如果 NSG 关联到任意 NIC，请为每个 NIC 运行 `azure network nic set`，如[取消 NSG 与 NIC 之间的关联](#Dissociate-an-NSG-from-a-NIC)中所示。 
3. 如果 NSG 关联到任意子网，请为每个子网运行 `azure network vnet subnet set`，如[取消 NSG 与子网之间的关联](#Dissociate-an-NSG-from-a-subnet)中所示。
4. 若要删除 NSG，请运行 `Remove-AzureRmNetworkSecurityGroup` cmdlet，如下所示。

		Remove-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd -Force

	>[AZURE.NOTE] **-Force** 参数可以确保无需确认删除。

<!---HONumber=Mooncake_0516_2016-->