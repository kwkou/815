<properties
   pageTitle="使用 Powershell cmdlet 创建 VNet 对等互连 | Azure"
   description="了解如何在 Resource Manager 中使用 Azure 门户创建虚拟网络。"
   services="virtual-network"
   documentationCenter=""
   authors="NarayanAnnamalai"
   manager="jefco"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/14/2016"
   wacn.date="12/07/2016"
   ms.author="narayanannamalai; annahar"/>

# 使用 Powershell cmdlet 创建 VNet 对等互连

[AZURE.INCLUDE [virtual-networks-create-vnet-selectors-arm-include](../../includes/virtual-networks-create-vnetpeering-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-vnet-intro](../../includes/virtual-networks-create-vnetpeering-intro-include.md)]

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-basic-include](../../includes/virtual-networks-create-vnetpeering-scenario-basic-include.md)]

若要使用 PowerShell 创建 VNet 对等互连，请执行以下步骤：

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

    > [AZURE.NOTE] PowerShell cmdlet for managing VNet peering is shipped with [Azure PowerShell 1.6.](http://www.powershellgallery.com/packages/Azure/1.6.0)

2. 读取虚拟网络对象：

        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet1
        $vnet2 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet2

3. 若要建立 VNet 对等互连，需要创建两个链接，每个方向各一个。下一步将首先创建从 VNet1 到 VNet2 的 VNet 对等互连链接：

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet2 -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.id

    输出显示：

        Name			: LinkToVNet2
        Id: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1/virtualNetworkPeerings/LinkToVNet2
        Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName	: vnet101
        VirtualNetworkName	: vnet1
        ProvisioningState		: Succeeded
        RemoteVirtualNetwork	: {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2"
                                        }
        AllowVirtualNetworkAccess	: True
        AllowForwardedTraffic	: False
        AllowGatewayTransit	: False
        UseRemoteGateways	: False
        RemoteGateways		: null
        RemoteVirtualNetworkAddressSpace : null

4. 此步骤将创建 VNet2 到 VNet1 的 VNet 对等互连链接：

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet1 -VirtualNetwork $vnet2 -RemoteVirtualNetworkId $vnet1.id

    输出显示：

        Name			: LinkToVNet1
        Id				: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2/virtualNetworkPeerings/LinkToVNet1
        Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName	: vnet101
        VirtualNetworkName	: vnet2
        ProvisioningState		: Succeeded
        RemoteVirtualNetwork	: {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1"
                                        }
        AllowVirtualNetworkAccess	: True
        AllowForwardedTraffic	: False
        AllowGatewayTransit	: False
        UseRemoteGateways	: False
        RemoteGateways		: null
        RemoteVirtualNetworkAddressSpace : null

5. 创建 VNet 对等互连链接后，可以看到下方的链接状态：

        Get-AzureRmVirtualNetworkPeering -VirtualNetworkName vnet1 -ResourceGroupName vnet101 -Name linktovnet2

    输出显示：

		Name			: LinkToVNet2
		Id				: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1/virtualNetworkPeerings/LinkToVNet2
		Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
		ResourceGroupName	: vnet101
		VirtualNetworkName	: vnet1
		ProvisioningState		: Succeeded
		RemoteVirtualNetwork	: {
		                                     "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2"
		                                }
		AllowVirtualNetworkAccess	: True
		AllowForwardedTraffic            : False
		AllowGatewayTransit              : False
		UseRemoteGateways                : False
		RemoteGateways                   : null
		RemoteVirtualNetworkAddressSpace : null

	每个 VNet 对等互连有几个可配置属性：

	|选项|说明|默认|
	|:-----|:----------|:------|
	|允许虚拟网络访问|是否将对等 VNet 的地址空间包括为 Virtual\_network 标记的一部分|是|
	|允许转发的流量|允许接受或丢弃不是源自对等 VNet 的流量|否|
	|允许网关传输|允许对等 VNet 使用你的 VNet 网关|否|
	|使用远程网关|使用对等方的 VNet 网关。对等 VNet 必须已配置网关并且已选中“允许网关传输”。如果你已配置了网关，则无法使用此选项|否|

	VNet 对等互连中的每个链接都有上述属性集。例如，可针对 VNet1 到 VNet2 的 VNet 对等互连链接将“允许虚拟网络访问”设置为“Ture”，而针对另一方向的 VNet 对等互连链接设置为“False”。

        $LinktoVNet2 = Get-AzureRmVirtualNetworkPeering -VirtualNetworkName vnet1 -ResourceGroupName vnet101 -Name LinkToVNet2
        $LinktoVNet2.AllowForwardedTraffic = $true
        Set-AzureRmVirtualNetworkPeering -VirtualNetworkPeering $LinktoVNet2

    更改后，可以运行 Get-AzureRmVirtualNetworkPeering 仔细检查属性值。运行上述 cmdlet 后，可以在输出中看到 AllowForwardedTraffic 设置已更改为 True。

        Name			: LinkToVNet2
        Id			: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1/virtualNetworkPeerings/LinkToVNet2
        Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName	: vnet101
        VirtualNetworkName	: vnet1
        ProvisioningState	: Succeeded
        RemoteVirtualNetwork	: {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2"
                                        }
        AllowVirtualNetworkAccess	: True
        AllowForwardedTraffic		: True
        AllowGatewayTransit		: False
        UseRemoteGateways		: False
        RemoteGateways		: null
        RemoteVirtualNetworkAddressSpace : null

	此方案中的对等互连建立好后，应该能够启动两个 Vnet 中的任何虚拟机之间的连接。默认情况下，“允许虚拟网络访问”为“True”，且 VNet 对等互连将设置正确的 ACL 以允许 Vnet 之间的通信。仍可应用网络安全组 (NSG) 规则来阻止特定子网或虚拟机之间的连接，从而实现两个虚拟网络之间访问权限的细粒度控制。有关创建 NSG 规则的详细信息，请参阅此[文章](/documentation/articles/virtual-networks-create-nsg-arm-ps/)。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-crosssub-include](../../includes/virtual-networks-create-vnetpeering-scenario-crosssub-include.md)]

若要使用 PowerShell 跨订阅创建 VNet 对等互连，请执行以下步骤：

1. 使用订阅 A 的特权用户 A 帐户登录到 Azure，并运行以下 cmdlet：

        New-AzureRmRoleAssignment -SignInName <UserB ID> -RoleDefinitionName "Network Contributor" -Scope /subscriptions/<Subscription-A-ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/VirtualNetwork/VNet5

    这不是必须的，只要请求匹配，即使用户分别提出针对各个 Vnet 的对等互连请求，也可建立对等互连。添加另一个 VNet 的特权用户作为本地 VNet 用户可以更轻松地执行安装程序。

2. 使用订阅 B 的特权用户 B 帐户登录到 Azure，并运行以下 cmdlet：

        New-AzureRmRoleAssignment -SignInName <UserA ID> -RoleDefinitionName "Network Contributor" -Scope /subscriptions/<Subscription-B-ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/VirtualNetwork/VNet3

3. 在用户 A 的登录会话中，运行以下 cmdlet：

        $vnet3 = Get-AzureRmVirtualNetwork -ResourceGroupName hr-vnets -Name vnet3

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet5 -VirtualNetwork $vnet3 -RemoteVirtualNetworkId "/subscriptions/<Subscriptoin-B-Id>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/VNet5" -BlockVirtualNetworkAccess

4. 在用户 B 的登录会话中，运行以下 cmdlet：

        $vnet5 = Get-AzureRmVirtualNetwork -ResourceGroupName vendor-vnets -Name vnet5

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet3 -VirtualNetwork $vnet5 -RemoteVirtualNetworkId "/subscriptions/<Subscriptoin-A-Id>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/VNet3" -BlockVirtualNetworkAccess

5. 对等互连建立后，VNet3 中的任意虚拟机应该能够与 VNet5 中的任意虚拟机进行通信。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-transit-include](../../includes/virtual-networks-create-vnetpeering-scenario-transit-include.md)]

1. 在此方案中，可以运行以下 PowerShell cmdlet 以建立 VNet 对等互连。需要将“允许转发的流量”属性设置为“True”，并将 VNET1 链接到 HubVnet，该属性允许来自对等互连 Vnet 以外的地址空间的入站流量。

        $hubVNet = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name HubVNet
        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet1

        Add-AzureRmVirtualNetworkPeering -name LinkToHub -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $HubVNet.id -AllowForwardedTraffic

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet1 -VirtualNetwork $HubVNet -RemoteVirtualNetworkId $vnet1.id

2. 对等互连建立后，可以参考此[文章](/documentation/articles/virtual-network-create-udr-arm-ps/)，定义用户定义的路由 (UDR)，以便通过虚拟设备重定向 VNet1 流量以使用其功能。在路由中指定下一个跃点地址时，可以在对等 VNet HubVNet 中将其设置为虚拟装置的 IP 地址。下面是一个示例：

        $route = New-AzureRmRouteConfig -Name TestNVA -AddressPrefix 10.3.0.0/16 -NextHopType VirtualAppliance -NextHopIpAddress 192.0.1.5

        $routeTable = New-AzureRmRouteTable -ResourceGroupName VNet101 -Location brazilsouth -Name TestRT -Route $route

        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName VNet101 -Name VNet1

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet1 -Name subnet-1 -AddressPrefix 10.1.1.0/24 -RouteTable $routeTable

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet1

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-asmtoarm-include](../../includes/virtual-networks-create-vnetpeering-scenario-asmtoarm-include.md)]

若要通过 PowerShell 在典型虚拟网络与 Azure Resource Manager 虚拟网络之间创建 VNet 对等互连，请执行以下步骤：

1. 读取 **VNET1**、Azure Resource Manager 虚拟网络的虚拟网络对象，如下所示：
        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet1

2. 若要在此方案中建立对等互连，只需要一个链接，具体而言，是从 **VNET1** 到 **VNET2** 的链接。此步骤需要知道经典 VNet 的资源 ID。资源组 ID 格式如下所示：

        /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ClassicNetwork/virtualNetworks/VirtualNetworkName

    请务必将 SubscriptionID、ResourceGroupName 和 VirtualNetworkName 替换为相应的名称。

    可通过以下命令实现此目的：

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet2 -VirtualNetwork $vnet1 -RemoteVirtualNetworkId /subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicNetwork/virtualNetworks/VNET2

3. 创建 VNet 对等互连链接后，可以看到以下输出中所示的链接状态：

        Name                             : LinkToVNet2
        Id                               : /subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/VNET1/virtualNetworkPeerings/LinkToVNet2
        Etag                             : W/"acecbd0f-766c-46be-aa7e-d03e41c46b16"
        ResourceGroupName                : MyResourceGroup
        VirtualNetworkName               : VNET1
        PeeringState                     : Connected
        ProvisioningState                : Succeeded
        RemoteVirtualNetwork             : {
                                         "Id": "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicNetwork/virtualNetworks/VNET2"
                                       }
        AllowVirtualNetworkAccess        : True
        AllowForwardedTraffic            : False
        AllowGatewayTransit              : False
        UseRemoteGateways                : False
        RemoteGateways                   : null
        RemoteVirtualNetworkAddressSpace : null

## 删除 VNet 对等互连

1.	为了删除 VNet 对等互连，需要运行以下 cmdlet：

        Remove-AzureRmVirtualNetworkPeering  

        remove both links, as shown below:

        Remove-AzureRmVirtualNetworkPeering -ResourceGroupName vnet101 -VirtualNetworkName vnet1 -Name linktovnet2
        Remove-AzureRmVirtualNetworkPeering -ResourceGroupName vnet101 -VirtualNetworkName vnet1 -Name linktovnet2

2. 删除 VNET 对等互连中的一个链接后，该对等链接状态将为断开。在此状态下，在对等链接状态更改为已启动之前无法重新创建链接。建议先删除这两个链接，然后再重新创建 VNet 对等互连。

<!---HONumber=Mooncake_1010_2016-->