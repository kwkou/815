<!-- ARM: tested -->

<properties
   pageTitle="在 Azure Resource Manager 中实施公共 IP 和专用 IP 寻址 | Azure"
   description="了解如何在 Azure 资源管理器中实施公共 IP 和专用 IP 寻址"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager" />
<tags
	ms.service="virtual-network"
	ms.date="04/27/2016"
	wacn.date="06/06/2016"/>

# Azure 中的 IP 地址
可以将 IP 地址分配给与其他 Azure 资源通信的 Azure 资源，也可以将其分配给本地网络和 Internet。可以在 Azure 中使用两种类型的 IP 地址：

- **公共 IP 地址**：用来与 Internet 通信，包括与面向公众的 Azure 服务通信
- **专用 IP 地址**：用于在 Azure 虚拟网络 (VNet) 中通信，以及在本地网络中通信（当你使用 VPN 网关或 ExpressRoute 线路将网络扩展到 Azure 时）。


> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/virtual-network-ip-addresses-overview-classic/)。

## 公共 IP 地址
公共 IP 地址可允许 Azure 资源与 Internet 以及面向公众的 Azure 服务（例如 [Azure Redis 缓存](/home/features/redis-cache/)、[Azure 事件中心](/home/features/event-hubs/)、[SQL 数据库](/documentation/articles/sql-database-technical-overview/)和 [Azure 存储空间](/documentation/articles/storage-introduction/)）通信。

在 Azure Resource Manager 中，公共 IP 地址是具有其自身属性的资源。你可以将公共 IP 地址资源与以下任何资源相关联：

- 虚拟机 (VM)
- 面向 Internet 的负载平衡器
- VPN 网关
- 应用程序网关数

### 分配方法
将 IP 地址分配给公共 IP 资源有两种方法 - 动态或静态。默认分配方法为动态，即**不是**在创建时分配的 IP 地址。公共 IP 地址是在启动（或创建）关联的资源（例如 VM 或负载平衡器）时分配的。停止（或删除）该资源时，就会释放该 IP 地址。因此，停止和启动资源都会导致 IP 地址更改。

若要确保所关联资源的 IP 地址保持不变，可将分配方法显式设置为静态。在这种情况下，IP 地址是立刻分配的。仅当你删除该资源或将其分配方法更改为动态时，才会释放该地址。

>[AZURE.NOTE] 即使将分配方法设置为静态，也无法通过指定方式将实际 IP 地址分配到公共 IP 资源，而只能在创建资源时所在的 Azure 位置通过包含可用 IP 地址的池对 IP 地址进行分配。

以下情况通常使用静态公共 IP 地址：

- 最终用户必须更新防火墙规则才能与 Azure 资源通信。
- 对 DNS 名称进行解析时，如果更改了 IP 地址，则需更新 A 记录。
- 你的 Azure 资源与其他使用 IP 地址型安全模型的应用或服务通信。
- 使用链接到 IP 地址的 SSL 证书。

>[AZURE.NOTE] 将公共 IP 地址（动态/静态）分配给 Azure 资源时所依据的 IP 范围列表已在 [Azure Datacenter IP ranges（Azure 数据中心 IP 范围）](https://www.microsoft.com/download/details.aspx?id=41653)中发布。

### DNS 主机名解析
你可以为公共 IP 资源指定一个 DNS 域名标签，以便在 Azure 托管的 DNS 服务器中为 *domainnamelabel*.*location*.chinacloudapp.cn 创建目标为公共 IP 地址的映射。例如，如果你在创建公共 IP 资源时，将 *domainnamelabel* 指定为 **contoso**，将 Azure 的“位置”指定为“中国北部”，则会将完全限定域名 (FQDN) **contoso.chinanorth.chinacloudapp.cn** 解析成该资源的公共 IP 地址。可以使用此 FQDN 创建指向 Azure 中的公共 IP 地址的自定义域 CNAME 记录。

>[AZURE.IMPORTANT] 所创建的每个域名标签在其 Azure 位置中必须是唯一的。

### 虚拟机
可以将公共 IP 地址与 [Windows](/documentation/articles/virtual-machines-windows-about/) 或 [Linux](/documentation/articles/virtual-machines-linux-about/) VM 相关联，只需将它分配到其**网络接口**即可。对于多网络接口 VM，只能将它分配到主要网络接口。你可以向 VM 分配动态或静态公共 IP 地址。

### 面向 Internet 的负载平衡器
你可以将公共 IP 地址与 Azure Load Balancer 相关联，只需将其分配给负载平衡器**前端**配置即可。此公共 IP 地址充当负载平衡型虚拟 IP 地址 (VIP)。你可以向负载平衡器前端分配动态或静态公共 IP 地址。你还可以为负载平衡器前端分配多个公共 IP 地址，从而启用多 VIP 方案，如包含基于 SSL 的网站的多租户环境。

### VPN 网关
[Azure VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)用于将 Azure 虚拟网络 (VNet) 连接到其他 Azure VNet 或本地网络。必须将公共 IP 地址分配给其 **IP 配置**，才能与远程网络通信。目前只能向 VPN 网关分配动态公共 IP 地址。

### 应用程序网关数
你可以将公共 IP 地址与 Azure [应用程序网关](/documentation/articles/application-gateway-introduction/)相关联，只需将其分配给网关的**前端**配置即可。此公共 IP 地址充当负载平衡型 VIP。目前，你只能将动态公共 IP 地址分配给应用程序网关前端配置。

### 概览
下表显示了将公共 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

|顶级资源|IP 地址关联|动态|静态|
|---|---|---|---|
|虚拟机|网络接口|是|是|
|负载平衡器|前端配置|是|是|
|VPN 网关|网关 IP 配置|是|否|
|应用程序网关|前端配置|是|否|

## 专用 IP 地址
专用 IP 地址能够让 Azure 资源在不使用可访问 Internet 的 IP 地址的情况下，与[虚拟网络](/documentation/articles/virtual-networks-overview/)或本地网络中的其他资源（通过 VPN 网关或 ExpressRoute 线路）通信。

在 Azure Resource Manager 部署模型中，可将专用 IP 地址关联到以下类型的 Azure 资源：

- VM
- 内部负载平衡器 (ILB)
- 应用程序网关数

### 分配方法
可以根据资源所附加到的子网的地址范围来分配专用 IP 地址。子网本身的地址范围是 VNet 的地址范围的一部分。

分配专用 IP 地址有两种方法：动态或静态。默认分配方法为动态，即自动从资源的子网分配 IP 地址（使用 DHCP）。停止和启动该资源时，此 IP 地址可能更改。

可将分配方法设置为静态，以确保 IP 地址始终相同。在这种情况下，还需提供属于资源子网一部分的有效 IP 地址。

静态专用 IP 地址通常用于：

- 充当域控制器或 DNS 服务器的 VM。
- 需要使用 IP 地址的防火墙规则的资源。
- 其他应用/资源通过 IP 地址访问的资源。

### 虚拟机
可将专用 IP 地址分配到 [Windows](/documentation/articles/virtual-machines-windows-about/) 或 [Linux](/documentation/articles/virtual-machines-linux-about/) VM 的**网络接口**。对于多网络接口 VM，将为每个接口分配一个专用 IP 地址。可将网络接口的分配方法指定为动态或静态。

#### 内部 DNS 主机名解析（针对 VM）
默认情况下，所有 Azure VM 都配置了 [Azure 托管的 DNS 服务器](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/#azure-provided-name-resolution)（除非你显式配置了自定义 DNS 服务器）。这些 DNS 服务器为驻留在同一个 VNet 内的 VM 提供内部名称解析。

创建 VM 时，主机名到其专用 IP 地址的映射将添加到 Azure 托管的 DNS 服务器。使用多网络接口 VM 时，主机名将映射到主要网络接口的专用 IP 地址。

使用 Azure 托管的 DNS 服务器配置的 VM 可以将 VNet 中的所有 VM 的主机名解析为其专用 IP 地址。

### 内部负载平衡器 (ILB) 和应用程序网关
你可以将专用 IP 地址分配给 Azure 内部负载平衡器 (ILB) 或 [Azure 应用程序网关](/documentation/articles/application-gateway-introduction/)的**前端**配置。此专用 IP 地址将用作内部终结点，仅供其虚拟网络 (VNet) 和连接到该 VNet 的远程网络中的资源访问。你可以将动态或静态专用 IP 地址分配给前端配置。

### 概览
下表显示了将专用 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

|顶级资源|IP 地址关联|动态|静态|
|---|---|---|---|
|虚拟机|网络接口|是|是|
|负载平衡器|前端配置|是|是|
|应用程序网关|前端配置|是|是|

## 限制

Azure 中的[网络限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)全面阐述了对 IP 寻址施加的限制。这些限制是根据区域和订阅设置的。

## 定价

在大多数情况下，公共 IP 地址是免费的。使用其他和/或静态公共 IP 地址要收取少许费用。

<!---HONumber=Mooncake_0523_2016-->