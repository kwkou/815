<properties 
   pageTitle="排查路由问题 - PowerShell | Azure"
   description="了解如何使用 Azure PowerShell 在 Azure Resource Manager 部署模型中排查路由问题。"
   services="virtual-network"
   documentationCenter="na"
   authors="AnithaAdusumilli"
   manager="narayan"
   editor=""
   tags="azure-resource-manager"
/>  

<tags 
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/23/2016"
   wacn.date="01/05/2017"
   ms.author="anithaa" />  


# 使用 Azure PowerShell 排查路由问题

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/virtual-network-routes-troubleshoot-portal/)
- [PowerShell](/documentation/articles/virtual-network-routes-troubleshoot-powershell/)

如果与 Azure 虚拟机 (VM) 之间出现网络连接问题，路由可能会影响 VM 的流量流。本文概述路由诊断功能，帮助用户做进一步的故障排除。

路由表与子网关联，在该子网中的所有网络接口 (NIC) 上有效。可将以下类型的路由应用到每个网络接口：

- **系统路由：**默认情况下，在 Azure 虚拟网络 (VNet) 中创建的每个子网都具有系统路由表，允许传送本地 VNet 流量、通过 VPN 网关的本地流量以及 Internet 流量。对等互连的 VNet 也有系统路由。
- **BGP 路由：**通过 ExpressRoute 或站点到站点 VPN 连接传播到网络接口。阅读 [BGP with VPN gateways](/documentation/articles/vpn-gateway-bgp-overview/)（使用 VPN 网关的 BGP）和 [ExpressRoute overview](/documentation/articles/expressroute-introduction/)（ExpressRoute 概述）文章，了解有关 BGP 路由的详细信息。
- **用户定义的路由 (UDR)：**如果使用网络虚拟设备或强制隧道通过站点到站点 VPN 将流量传送到本地网络，可将用户定义的路由 (UDR) 与子网路由表相关联。如果你不熟悉 UDR，请阅读 [user-defined routes](/documentation/articles/virtual-networks-udr-overview/#user-defined-routes)（用户定义的路由）一文。

由于可将各种路由应用到网络接口，因此有时难以确定哪些聚合路由是有效的。为了帮助排查 VM 网络连接问题，可以查看 Azure Resource Manager 部署模型中某个网络接口的所有有效路由。

## 使用有效路由排查 VM 流量流问题

本文使用以下情景作为示例，演示如何排查网络接口的有效路由问题：

与 VNet（*VNet1*，前缀为 10.9.0.0/16）连接的 VM (*VM1*) 无法连接到最近对等互连的 VNet（*VNet3*，前缀为 10.10.0.0/16）中的 VM (VM3)。连接到该 VM 的 VM1-NIC1 网络接口未应用 UDR 或 BGP 路由，只应用了系统路由。

本文说明如何使用 Azure Resource Manager 部署模型中的有效路由功能确定连接失败的原因。尽管本示例只使用系统路由，但可以使用相同的步骤判断任何路由类型的入站和出站连接失败情况。

>[AZURE.NOTE] 如果 VM 附加了多个 NIC，请检查每个 NIC 的有效路由，以便诊断与 VM 之间的网络连接问题。

### 查看虚拟机的有效路由

若要查看应用到 VM 的聚合路由，请完成以下步骤：

### 查看网络接口的有效路由

若要查看应用到网络接口的聚合路由，请完成以下步骤：

1.  启动 Azure PowerShell 会话并登录到 Azure。如果不熟悉 Azure PowerShell，请阅读 [How to install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell）一文。

2.  以下命令返回对资源组 *RG1* 中名为 *VM1-NIC1* 的网络接口应用的所有路由：

		Get-AzureRmEffectiveRouteTable -NetworkInterfaceName VM1-NIC1 -ResourceGroupName RG1

	>[AZURE.TIP] 如果不知道网络接口的名称，请输入以下命令检索资源组中所有网络接口的名称。*

	    Get-AzureRmNetworkInterface -ResourceGroupName RG1 | Format-Table Name

	以下输出类似于应用到 NIC 所连接子网的每个路由的输出：

		Name :
		State : Active
		AddressPrefix : {10.9.0.0/16}
		NextHopType : VNetLocal
		NextHopIpAddress : {}

		Name :
		State : Active
		AddressPrefix : {0.0.0.0/16}
		NextHopType : Internet
		NextHopIpAddress : {}

	请注意输出中的以下信息：
	- **Name**：用户定义的路由，除非显式指定，否则有效路由的名称可能为空。
	- **State**：表示有效路由的状态。可能的值为“Active”或“Invalid”。
	- **AddressPrefixes**：以 CIDR 表示法指定有效路由的地址前缀。
	- **nextHopType**：表示给定路由的下一跃点。可能的值为 *VirtualAppliance*、*Internet*、*VNetLocal*、*VNetPeering* 或 *Null*。如果 UDR 中的 **nextHopType** 值为 *Null*，可能表示是路由无效。例如，如果 **nextHopType** 为 *VirtualAppliance*，但网络虚拟设备 VM 不处于已预配/运行中状态。如果 **nextHopType** 为 *VPNGateway*，但给定的 VNet 中没有任何网关处于已预配/运行中状态，则路由可能失效。
	- **NextHopIpAddress**：指定有效路由下一跃点的 IP 地址。
    
	以下命令在一个方便查看的表中返回路由：

		Get-AzureRmEffectiveRouteTable -NetworkInterfaceName VM1-NIC1 -ResourceGroupName RG1 | Format-Table

	以下输出是上述方案收到的一部分输出：

		Name State AddressPrefix NextHopType NextHopIpAddress
		---- ----- ------------- ----------- ----------------
		Active {10.9.0.0/16} VnetLocal {}
		Active {0.0.0.0/0} Internet {}
	

3. 上一步骤的输出没有列出*从 *ChinaNorth-VNet1*（前缀 10.9.0.0/16）* 到 *ChinaNorth-VNet3* VNet（前缀 10.10.0.0/16）的路由。如下图所示，包含 *ChinaNorth-VNet3* VNet 的 VNet 对等互连链接处于 *Disconnected* 状态。
	
	![](./media/virtual-network-routes-troubleshoot-portal/image4.png)  


	对等互连的双向链接已断开，正因如此，VM1 无法连接到 *ChinaNorth-VNet3* VNet 中的 VM3。为 *ChinaNorth-VNet1* 和 *ChinaNorth-VNet3* VNet 再次设置双向对等互连链接。正确建立 VNet 对等互连链接后，返回的输出如下所示：

		Name State AddressPrefix NextHopType NextHopIpAddress
		---- ----- ------------- ----------- ----------------
		Active {10.9.0.0/16} VnetLocal {}
		Active {10.10.0.0/16} VNetPeering {}
		Active {0.0.0.0/0} Internet {}
		
	确定问题后，可以添加、删除或更改路由和路由表。键入以下命令，查看用于执行这些操作的命令：

		Get-Help *-AzureRmRouteConfig

## 注意事项

查看返回的路由列表时，请注意以下事项：

- 路由基于 UDR、BGP 与系统路由之间的最长前缀匹配 (LPM)。如果有多个路由的 LPM 匹配情况相同，则按以下顺序根据路由源来选择路由：
	- 用户定义的路由
	- BGP 路由
	- 系统（默认）路由

	对于有效路由，只能查看基于所有可用路由匹配 LPM 的有效路由。如果显示了如何针对给定 NIC 实际评估路由，因此可以更方便地对可能影响 VM 连接的特定路由进行故障排除。

- 如果使用 UDR 并且要将流量发送到网络虚拟设备 (NVA)（*VirtualAppliance* 为 **nextHopType**），请确保接收流量的 NVA 已启用 IP 转发，否则数据包将被丢弃。
- 如果启用了强制隧道，所有出站 Internet 流量将路由到本地。根据本地处理此流量的方式，可能无法使用此设置通过 RDP/SSH 从 Internet 连接到 VM。符合以下条件时，可以启用强制隧道：
	- 使用站点到站点 VPN 时，将用户定义的路由 (UDR) 的 nextHopType 设置为 VPN 网关
	- 通过 BGP 播发默认路由时
- 要使 VNet 对等互连流量正常工作，对等互连的 VNet 的前缀范围中必须存在 **nextHopType** 为 *VNetPeering* 的系统路由。如果没有此类路由，并且 VNet 对等互连链接看起来正常：
	- 如果是新建立的对等互连链接，请等待几秒钟并重试。有时需要花费较长的时间才能将路由传播到子网中的所有网络接口。
	- 网络安全组 (NSG) 规则可能会影响流量流。有关详细信息，请参阅 [Troubleshoot Network Security Groups](/documentation/articles/virtual-network-nsg-troubleshoot-powershell/)（排查网络安全组问题）一文。

<!---HONumber=Mooncake_1107_2016-->