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
   ms.date="10/10/2016"
   ms.author="ganesr"
   wacn.date="10/31/2016"/>


#ExpressRoute 故障排除指南 - 在经典部署模型中获取 ARP 表

> [AZURE.SELECTOR]
[PowerShell - Resource Manager](/documentation/articles/expressroute-troubleshooting-arp-resource-manager/)
[PowerShell - Classic](/documentation/articles/expressroute-troubleshooting-arp-classic/)

本文将指导你完成为 Azure ExpressRoute 线路获取地址解析协议 (ARP) 表的步骤。

>[AZURE.IMPORTANT] 本文档旨在帮助你诊断和修复简单问题。它不是为了替代 Azure 支持部门。如果你使用以下指南无法解决问题，请使用 [Azure 帮助+支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)建立支持请求。

## 地址解析协议 (ARP) 和 ARP 表
ARP 是 [RFC 826](https://tools.ietf.org/html/rfc826) 中定义的第 2 层协议。ARP 用于将以太网地址（MAC 地址）映射到 IP 地址。

可以通过 ARP 表来映射 IPv4 地址和 MAC 地址，以便实现特定的对等互连。用于 ExpressRoute 线路对等互连的 ARP 表为每个接口（主接口和辅助接口）提供以下信息：

1. 将本地路由器接口 IP 地址映射到 MAC 地址
2. 将 ExpressRoute 路由器接口 IP 地址映射到 MAC 地址
3. 映射的使用期限

ARP 表可帮助验证第 2 层配置，并可针对第 2 层的基本连接问题进行故障诊断。

下面是一个 ARP 表的示例：

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           10.0.0.1 ffff.eeee.dddd
		  0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


以下部分介绍如何查看供 ExpressRoute 边缘路由器查看的 ARP 表。

## 使用 ARP 表的先决条件

在继续之前，请确保你具备以下条件：

 - 配置了至少一个对等互连的有效的 ExpressRoute 线路。该线路必须由连接提供商进行完整的配置。你（或你的连接提供商）必须在该线路上配置 Azure 专用和 Azure 公共这二者中的至少一个对等互连。
 - 用于配置对等互连（Azure 专用和 Azure 公共）的 IP 地址范围。查看 [ExpressRoute 路由要求页](/documentation/articles/expressroute-routing/)中的 IP 地址分配示例，了解如何将 IP 地址映射到你所在的一侧和 ExpressRoute 侧的接口。你可以通过查看 [ExpressRoute 对等互连配置页](/documentation/articles/expressroute-howto-routing-classic/)了解对等互连配置。
 - 你的网络团队或连接提供商提供的有关接口（用于这些 IP 地址）的 MAC 地址的信息。

 - Azure 的最新 Windows PowerShell 模块（1.50 版或更高版本）。

## ExpressRoute 线路的 ARP 表
本部分说明如何使用 PowerShell 查看每种类型的对等互连的 ARP 表。在继续之前，你或你的连接提供商必须配置对等互连。每个线路有两个路径（主路径和辅助路径）。你可以独立地检查每个路径的 ARP 表。

### Azure 专用对等互连的 ARP 表
以下 cmdlet 提供 Azure 专用对等互连的 ARP 表：

		# Required Variables
		$ckt = "<your Service Key here>

		# ARP table for Azure private peering--primary path
		Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Primary

		# ARP table for Azure private peering--secondary path
		Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Secondary

下面是其中一条路径的示例输出：

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           10.0.0.1 ffff.eeee.dddd
		  0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


### Azure 公共对等互连的 ARP 表：
以下 cmdlet 提供 Azure 公共对等互连的 ARP 表：

		# Required Variables
		$ckt = "<your Service Key here>

		# ARP table for Azure public peering--primary path
		Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Primary

		# ARP table for Azure public peering--secondary path
		Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Secondary

下面是其中一条路径的示例输出：

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           10.0.0.1 ffff.eeee.dddd
		  0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


下面是其中一条路径的示例输出：

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           64.0.0.1 ffff.eeee.dddd
		  0 Microsoft         64.0.0.2 aaaa.bbbb.cccc


## 如何使用此信息
对等互连的 ARP 表可用于验证第 2 层配置和连接。本部分概述了不同情况下的 ARP 表的外观。

### 当线路处于运行（预期）状态时的 ARP 表

 - ARP 表会有一个针对本地端且包含有效 IP 地址和 MAC 地址的条目，以及一个类似的针对 Azure 端的条目。 
 - 本地 IP 地址的最后一个八位字节始终是奇数。
 - Microsoft IP 地址的最后一个八位字节始终是偶数。
 - 所有 3 种对等互连（主/辅助）在 Azure 端都会显示相同的 MAC 地址。 


		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		 10 On-Prem           65.0.0.1 ffff.eeee.dddd
		  0 Microsoft         65.0.0.2 aaaa.bbbb.cccc

### 当 ARP 表在本地端或连接提供商端出现问题时的 ARP 表

 ARP 表中只显示一个条目。它显示在 Microsoft 端使用的 MAC 地址与 IP 地址之间的映射。

		Age InterfaceProperty IpAddress  MacAddress    
		--- ----------------- ---------  ----------    
		  0 Microsoft         65.0.0.2 aaaa.bbbb.cccc

>[AZURE.NOTE] 如果你遇到此类问题，请通过连接提供商联系建立支持请求以解决它。


### 当 Azure 端出现问题时的 ARP 表

 - 如果 Azure 端存在问题，则不会为对等互连显示 ARP 表。 
 -  使用 [Azure 帮助+支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)建立支持请求。指出你的第 2 层连接有问题。

## 后续步骤

 - 验证 ExpressRoute 线路的第 3 层配置：
	 - 获取路由摘要以确定 BGP 会话的状态。
	 - 获取路由表以确定哪些前缀跨 ExpressRoute 播发。
 - 通过查看输入/输出中的字节数来验证数据传输。
 - 如果仍然遇到问题，请使用 [Azure 帮助+支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)建立支持请求。

<!---HONumber=Mooncake_0704_2016-->