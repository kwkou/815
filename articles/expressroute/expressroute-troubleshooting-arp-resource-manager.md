<properties 
   pageTitle="ExpressRoute 故障排除指南 - 获取 ARP 表 | Azure"
   description="此页说明了如何为 ExpressRoute 线路获取 ARP 表"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carolz"
   editor="tysonn"/>
<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services" 
   ms.date="10/11/2016" 
   wacn.date="12/05/2016"
   ms.author="ganesr"/>


#ExpressRoute 故障排除指南 - 在 Resource Manager 部署模型中获取 ARP 表

> [AZURE.SELECTOR]
[PowerShell - Resource Manager](/documentation/articles/expressroute-troubleshooting-arp-resource-manager/)
[PowerShell - Classic](/documentation/articles/expressroute-troubleshooting-arp-classic/)

本文将指导你完成相关步骤，以便了解 ExpressRoute 线路的 ARP 表。

>[AZURE.IMPORTANT] 本文档旨在帮助你诊断和修复简单问题。它不是为了替代 Azure 支持部门。如果你无法通过下述指南解决问题，则必须[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单。

## 地址解析协议 (ARP) 和 ARP 表
地址解析协议 (ARP) 是在 [RFC 826](https://tools.ietf.org/html/rfc826) 中定义的第二层协议。ARP 用于映射以太网地址（MAC 地址）和 IP 地址。

可以通过 ARP 表来映射 IPv4 地址和 MAC 地址，以便实现特定的对等互连。用于 ExpressRoute 线路对等互连的 ARP 表为每个接口（主接口和辅助接口）提供以下信息

1. 将本地路由器接口 IP 地址映射到 MAC 地址
2. 将 ExpressRoute 路由器接口 IP 地址映射到 MAC 地址
3. 映射的使用期限

ARP 表可帮助验证第 2 层配置，并可针对第 2 层的基本连接问题进行故障诊断。

ARP 表示例：

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           10.0.0.1 ffff.eeee.dddd
		  0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


以下部分介绍如何查看供 ExpressRoute 边缘路由器查看的 ARP 表。

## 了解 ARP 表需具备的先决条件

在继续下一步之前，请确保你已具备以下条件

 - 配置了至少一个对等互连的有效的 ExpressRoute 线路。该线路必须由连接提供商进行完整的配置。你（或你的连接提供商）必须已经在该线路上配置了 Azure 专用和 Azure 公共这二者中的至少一个对等互连。
 - 用于配置对等互连（Azure 专用和 Azure 公共）的 IP 地址范围。查看 [ExpressRoute 路由要求页](/documentation/articles/expressroute-routing/)中的 IP 地址分配示例，了解如何将 IP 地址映射到你所在的一侧和 ExpressRoute 侧的接口。你可以通过查看 [ExpressRoute 对等互连配置页](/documentation/articles/expressroute-howto-routing-arm/)了解对等互连配置。
 - 你的网络团队/连接提供商提供的有关接口（用于这些 IP 地址）的 MAC 地址的信息。
 - 必须安装 Azure 的最新 PowerShell 模块（1.50 或更高版本）。

## 获取 ExpressRoute 线路的 ARP 表
本部分说明了如何使用 PowerShell 根据对等互连来查看 ARP 表。你或你的连接提供商必须在执行下一步之前配置好对等互连。每个线路有两个路径（主路径和辅助路径）。你可以独立地检查每个路径的 ARP 表。

### Azure 专用对等互连的 ARP 表
以下 cmdlet 为 Azure 专用对等互连提供 ARP 表

		# Required Variables
		$RG = "<Your Resource Group Name Here>"
		$Name = "<Your ExpressRoute Circuit Name Here>"
		
		# ARP table for Azure private peering - Primary path
		Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Primary
		
		# ARP table for Azure private peering - Secodary path
		Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Secondary 

下面为其中一个路径显示了示例性输出

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           10.0.0.1 ffff.eeee.dddd
		  0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


### Azure 公共对等互连的 ARP 表
以下 cmdlet 为 Azure 公共对等互连提供 ARP 表

		# Required Variables
		$RG = "<Your Resource Group Name Here>"
		$Name = "<Your ExpressRoute Circuit Name Here>"
		
		# ARP table for Azure public peering - Primary path
		Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Primary
		
		# ARP table for Azure public peering - Secodary path
		Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Secondary 


下面为其中一个路径显示了示例性输出

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           64.0.0.1 ffff.eeee.dddd
		  0 Microsoft         64.0.0.2 aaaa.bbbb.cccc



## 如何使用此信息
对等互连的 ARP 表可用于确定/验证第 2 层配置和连接。本部分概述了不同方案的 ARP 表的外观。

### 当线路处于运行状态（预期状态）时的 ARP 表

 - ARP 表会有一个针对本地端且带有有效 IP 地址和 MAC 地址的条目，以及一个类似的针对 Azure 端的条目。 
 - 本地 IP 地址的最后一个八位字节将始终是奇数。
 - Microsoft IP 地址的最后一个八位字节将始终是偶数。
 - 所有 3 种对等互连（主/辅助）在 Azure 端都会显示相同的 MAC 地址。 


		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           65.0.0.1 ffff.eeee.dddd
		  0 Microsoft         65.0.0.2 aaaa.bbbb.cccc

### 当本地端/连接提供商端出现问题时的 ARP 表

 - 只有一个条目会出现在 ARP 表中。此时会显示在 Azure 端使用的 MAC 地址与 IP 地址之间的映射。 

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		  0 Microsoft         65.0.0.2 aaaa.bbbb.cccc

>[AZURE.NOTE] 通过你的连接提供商提出支持请求，以便进行此类问题的调试。


### 当 Azure 端出现问题时的 ARP 表

 - 如果 Azure 端存在问题，则不会为对等互连显示 ARP 表。 
 -  [在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单。指出你的第 2 层连接有问题。 

## 后续步骤

 - 验证 ExpressRoute 线路的第 3 层配置
	 - 获取路由摘要以确定 BGP 会话的状态 
	 - 获取路由表以确定哪些前缀跨 ExpressRoute 播发
 - 通过查看输入/输出中的字节数来验证数据传输
 - 如果仍然存在问题，请[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单。

<!---HONumber=Mooncake_0704_2016-->