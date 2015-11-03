## 如何使用 PowerShell 创建 VNet

若要使用 PowerShell 创建 VNet，请执行下面的步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)，并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。
2. 在 Azure PowerShell 提示符下运行 **Switch-AzureMode** cmdlet 以切换到资源管理器模式，如下所示。

		Switch-AzureMode AzureResourceManager
	
	预期输出：

		WARNING: The Switch-AzureMode cmdlet is deprecated and will be removed in a future release.

	>[AZURE.WARNING]Switch-AzureMode cmdlet 将在不久后弃用。如果弃用，所有资源管理器 cmdlet 都将重命名。
	
3. 如有必要，请运行 **New-AzureResourceGroup** cmdlet 来创建新资源组，如下所示。对于我们的方案，创建一个名为 *TestRG* 的资源组。有关资源组的详细信息，请访问 [Azure 资源管理器概述](/documentation/articles/resource-group-overview#resource-groups)。

		New-AzureResourceGroup -Name TestRG -Location centralus

	预期输出：
	
		ResourceGroupName : TestRG
		Location          : centralus
		ProvisioningState : Succeeded
		Tags              :
		Permissions       :
		                    Actions  NotActions
		                    =======  ==========
		                    *
		
		ResourceId        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG	

4. 运行 **New-AzureVirtualNetwork** cmdlet 以创建新 VNet，如下所示。

		New-AzureVirtualNetwork -ResourceGroupName TestRG -Name TestVNet `
			-AddressPrefix 192.168.0.0/16 -Location centralus	
		
	预期输出：

		Name              : TestVNet
		ResourceGroupName : TestRG
		Location          : centralus
		Id                : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		Etag              : W/"5b89894f-db7f-4634-9870-f9b97e209510"
		ProvisioningState : Succeeded
		Tags              :
		AddressSpace      : {
		                      "AddressPrefixes": [
		                        "192.168.0.0/16"
		                      ]
		                    }
		DhcpOptions       : {
		                      "DnsServers": null
		                    }
		NetworkInterfaces : null
		Subnets           : []

5. 运行 **Get-AzureVirtualNetwork** cmdlet 以将虚拟网络对象存储在变量中，如下所示。

		$vnet = Get-AzureVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
	
	>[AZURE.TIP]可以通过运行 **$vnet = New-AzureVirtualNetwork -ResourceGroupName TestRG -Name TestVNet -AddressPrefix 192.168.0.0/16 -Location centralus** 合并步骤 4 和步骤 5。

6. 运行 **Add-AzureVirtualNetworkSubnetConfig** cmdlet 将一个子网添加到新 VNet，如下所示。

		Add-AzureVirtualNetworkSubnetConfig -Name FrontEnd `
			-VirtualNetwork $vnet -AddressPrefix 192.168.1.0/24
		
	预期输出：

		Name              : TestVNet
		ResourceGroupName : TestRG
		Location          : centralus
		Id                : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		Etag              : W/"5b89894f-db7f-4634-9870-f9b97e209510"
		ProvisioningState : Succeeded
		Tags              :
		AddressSpace      : {
		                      "AddressPrefixes": [
		                        "192.168.0.0/16"
		                      ]
		                    }
		DhcpOptions       : {
		                      "DnsServers": null
		                    }
		NetworkInterfaces : null
		Subnets           : [
		                      {
		                        "Name": "FrontEnd",
		                        "Etag": null,
		                        "Id": null,
		                        "AddressPrefix": "192.168.1.0/24",
		                        "IpConfigurations": null,
		                        "NetworkSecurityGroup": null,
		                        "RouteTable": null,
		                        "ProvisioningState": null
		                      }
		                    ]

7. 对要创建的每个子网重复执行上面的步骤 6。对于我们的方案，以下命令将创建 *BackEnd* 子网。

		Add-AzureVirtualNetworkSubnetConfig -Name BackEnd `
			-VirtualNetwork $vnet -AddressPrefix 192.168.2.0/24

8. 尽管你创建了子网，但它们当前仅存在于用于检索你在上面的步骤 4 中创建的 VNet 的本地变量中。若要将所做的更改保存到 Azure，请运行 **Set-AzureVirtualNetwork** cmdlet，如下所示。

		Set-AzureVirtualNetwork -VirtualNetwork $vnet	
		
	预期输出：

		Name              : TestVNet
		ResourceGroupName : TestRG
		Location          : centralus
		Id                : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
		Etag              : W/"2d3496d8-2b85-4238-bde2-377fe660aa4a"
		ProvisioningState : Succeeded
		Tags              :
		AddressSpace      : {
		                      "AddressPrefixes": [
		                        "192.168.0.0/16"
		                      ]
		                    }
		DhcpOptions       : {
		                      "DnsServers": []
		                    }
		NetworkInterfaces : null
		Subnets           : [
		                      {
		                        "Name": "FrontEnd",
		                        "Etag": "W/"2d3496d8-2b85-4238-bde2-377fe660aa4a"",
		                        "Id": "/subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
		                        "AddressPrefix": "192.168.1.0/24",
		                        "IpConfigurations": [],
		                        "NetworkSecurityGroup": null,
		                        "RouteTable": null,
		                        "ProvisioningState": "Succeeded"
		                      },
		                      {
		                        "Name": "BackEnd",
		                        "Etag": "W/"2d3496d8-2b85-4238-bde2-377fe660aa4a"",
		                        "Id": "/subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
		                        "AddressPrefix": "192.168.2.0/24",
		                        "IpConfigurations": [],
		                        "NetworkSecurityGroup": null,
		                        "RouteTable": null,
		                        "ProvisioningState": "Succeeded"
		                      }
		                    ]

<!---HONumber=76-->