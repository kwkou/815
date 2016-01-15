<properties
   pageTitle="在 Azure 中实施公共 IP 和专用 IP 寻址（经典）| Windows Azure"
   description="了解如何在 Azure 中实施公共 IP 和专用 IP 寻址"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" 
   tags="azure-service-management" />
<tags
	ms.service="virtual-network"
	ms.date="12/14/2015"
	wacn.date="01/14/2016"/>

# Azure 中的 IP 地址（经典）
可以将 IP 地址分配给与其他 Azure 资源通信的 Azure 资源，也可以将其分配给本地网络和 Internet。可以在 Azure 中使用两种类型的 IP 地址：公共地址和专用地址。

公共 IP 地址用于与 Internet 通信，这些通信中包括面向公众的 Azure 服务。

专用 IP 地址用于在 Azure 虚拟网络 (VNet) 或云服务中通信，以及在本地网络中通信（当你使用 ExpressRoute 线路将网络扩展到 Azure 时）。

[AZURE.INCLUDE [azure-arm-classic-important-include](../includes/learn-about-deployment-models-rm-include.md)] [resource manager deployment model](/documentation/articles/virtual-network-ip-addresses-overview-arm)。

## 公共 IP 地址
分配公共 IP 地址可以让 Azure 资源与 Internet 服务（其中包括面向公众的 Azure 服务，例如 [Azure Redis 缓存](/home/features/cache)、[Azure 事件中心](/home/features/event-hubs)、[SQL 数据库](/documentation/articles/sql-database-technical-overview)和 [Azure 存储空间](/documentation/articles/storage-introduction)）通信。

你可以将公共 IP 地址分配给以下任何资源：

- 云服务
- VM
- PaaS 角色实例
- 应用程序网关数

### 分配方法
你可以使用*动态* 或*静态* 公共 IP 地址。在属于*动态* 方法的默认分配方法中，IP 地址会根据资源所属的 [Azure 位置](https://www.microsoft.com/download/details.aspx?id=41653)自动分配给资源。但是，资源所用的 IP 地址可能会在删除或停止该资源时发生变化。

如果你想确保资源的 IP 地址保持不变，则需将分配方法更改为*静态*。与专用 IP 地址不同，你不能指定所要使用的 IP 地址。静态 IP 地址是按 Azure 所用地址范围中提供的地址分配的。静态 IP 地址称为[保留 IP](/documentation/articles/virtual-networks-reserved-public-ip)。

如果你想确保资源的 IP 地址保持不变，则需将分配方法更改为*静态*。以下情况通常使用静态公共 IP 地址：

- 使用链接到 IP 地址的 SSL 证书。
- 相关资源需要在 Azure 外部设置防火墙规则。
- 相关资源依赖外部 DNS 名称解析，其中的动态 IP 需要更新 A 记录。
- 相关资源运行其他应用通过 IP 地址访问的服务。

>[AZURE.NOTE]即使配置了静态公共 IP（保留 IP），也无法指定资源所使用的实际 IP 地址。你的资源将根据其创建时所在的 [Azure 位置](https://www.microsoft.com/download/details.aspx?id=41653)获取 IP 地址。

### DNS 主机名解析
你可以将公共 IP 地址与 DNS 域名标签相关联，以便在 Azure DNS 服务器中创建相应的 DNS 条目。相应的 FQDN 将采用 *domainnamelabel*.*location*.chinacloudapp.cn 格式，并将关联到与你的资源链接的公共 IP。例如，如果你在名为*中国北方* 的 Azure 区域中创建 *domainnamelabel* 为 **azuretest** 的公共 IP，则关联到该公共 IP 的资源的 FQDN 将为 **azuretest.chinanorth.chinacloudapp.cn**。

>[AZURE.IMPORTANT]所创建的每个域名标签在其 Azure 位置中必须是唯一的。

你可以稍后使用自己的自定义域名（指向在 Azure 中创建的域标签名称）创建 CNAME 记录。

###云服务 
云服务始终有一个面向公众的 IP 地址，称为 VIP。你可以在云服务中创建终结点，以便将 VIP 中的不同端口关联到 VM 中的内部端口以及云服务中的角色实例。

###VM 和 PasS 角色实例
你可以将公共 IP 地址关联到云服务中的单个 [VM](/documentation/articles/virtual-machines-about) 或 PaaS 角色实例。该类公共 IP 称为实例级公共 IP 地址 ([ILPIP](/documentation/articles/virtual-networks-instance-level-public-ip))。

只能将动态公共 IP 地址分配给 VM 和 PaaS 实例角色。

###应用程序网关数
你可以使用[应用程序网关](/documentation/articles/application-gateway-introduction)通过单个公共 IP 地址跨多个 VM 来平衡应用程序的负载，所用方式与负载平衡器的使用方式类似。但是，应用程序网关也允许你创建路由规则，以便根据 URL 用户请求进一步地跨 VM 分配流量，同时还允许你使用常规的第 7 级负载平衡器所提供的其他功能。

可以将公共 IP 分配到应用程序网关的前端。

### 速览
下表显示了哪些资源可以使用静态和动态公共 IP 地址。

|资源|动态|静态|多个 IP 地址|
|---|---|---|---|
|VM|是|否|否|
|云服务（包括负载平衡器前端）|是|是|是|
|应用程序网关（在云服务中）|是|否|否|

## 专用 IP 地址
分配专用 IP 地址即可让 Azure 资源与 VNet 或云服务中的其他资源通信，甚至通过 ExpressRoute 线路与本地网络通信。你创建的每个 VNet 或云服务将使用一系列预定义的专用 IP 地址。VNet 或云服务中的资源必须使用该范围内的专用 IP 地址。

在 Azure 中，专用 IP 地址可以分配给不同的资源。

- VM 和角色实例
- 内部负载平衡器
- 应用程序网关 

>[AZURE.NOTE]你可以创建与你所拥有的任何其他 Azure 资源隔离的云服务，也可以将云服务添加到现有的 VNet 中，以便与其他 Azure 资源通信。

### 分配方法
你可以使用*动态* 或*静态* 专用 IP 地址。在属于*动态* 方法的默认分配方法中，IP 地址会根据资源所属的子网或云服务自动分配给资源。但是，资源所用的 IP 地址可能会在停止和重新启动该资源时发生变化。

如果你想确保资源的 IP 地址不变，则需将分配方法更改为*静态*，并指定一个分配给资源所属子网的地址范围内的有效 IP 地址。

静态专用 IP 地址通常用于：

- 充当 DNS 服务器的 VM。
- 充当域控制器的 VM。
- 需要使用 IP 地址的防火墙规则的 VM。
- 运行其他应用通过 IP 地址进行访问的服务的 VM。

>[AZURE.IMPORTANT]你只能为分配给 VNet 的资源设置静态专用 IP 地址。

### 内部 DNS 主机名解析
在 Azure 资源之间进行的大多数通信会使用可以人工读取的名称而非 IP 地址来代表资源。该名称称为*主机名*，这是一个网络专业人员通常都会懂的术语。当某个资源尝试使用主机名访问其他资源时，必须将该主机名解析成 IP 地址。此工作通常由 [DNS 服务器](https://technet.microsoft.com/magazine/2005.01.howitworksdns.aspx)来完成。

所有 Azure VM 都使用 [Azure 托管的 DNS 服务器](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances#azure-provided-name-resolution)，除非你创建自己的 DNS 服务器并将 VM 配置为使用该服务器。使用 Azure 托管的 DNS 服务器时，将自动创建 DNS 记录，以便将 VM 的主机名解析成 VM 的专用 IP 地址。使用多 NIC VM 时，主机名将解析成主 NIC 的专用 IP 地址。

###VM 和角色实例
你可以通过一个或多个 NIC 来创建 VM 和角色实例。每个 NIC 都有自己的专用 IP 地址。

###内部负载平衡器 (ILB)
你可以使用[内部负载平衡器](/documentation/articles/load-balancer-internal-overview)通过子网中的单个专用 IP 地址跨多个 VM 来平衡应用程序的负载。例如，你可以让多个 VM 托管同一数据库，然后再让来自 VNet 的数据库连接传播到这些 VM。你可以使用内部负载平衡器来执行此操作。

你可以将专用 IP 地址关联到内部负载平衡器的后端池，以便将关联到这些 IP 地址的 VM 或角色实例用作经过负载平衡处理的流量的目标。你还可以将专用 IP 地址关联到 ILB 的前端。

你可以将动态或静态专用 IP 地址分配到内部负载平衡器的前端和后端池。

###应用程序网关数
你可以使用[应用程序网关](/documentation/articles/application-gateway-introduction)通过单个专用 IP 地址跨多个 VM 来平衡应用程序的负载，从而创建一个内部应用程序网关。

你可以将专用 IP 地址关联到应用程序网关的后端地址池，以便将关联到这些 IP 地址的 VM 用作经过负载平衡处理的流量的目标。你可以将动态或静态专用 IP 地址分配给应用程序网关的后端地址池。

你可以将专用 IP 地址关联到应用程序网关的前端。你可以将动态或静态专用 IP 地址分配给应用程序网关。

### 速览
下表显示了哪些资源可以使用静态和动态专用 IP 地址。

|资源|动态|静态|多个 IP 地址|
|---|---|---|---|
|VM|是|是|是|
|PaaS 角色实例|是|否|否|
|内部负载平衡器前端|是|是|是|
|应用程序网关前端|是|是|否|
|应用程序网关后端|否|是|是|

## 后续步骤

- [使用静态公共 IP 部署 VM](/documentation/articles/virtual-network-deploy-static-pip-classic-ps)
- [使用静态专用 IP 地址部署 VM](/documentation/articles/virtual-networks-static-private-ip-classic-pportal)
- [使用 PowerShell 创建内部负载平衡器](/documentation/articles/load-balancer-get-started-ilb-classic-ps)
- [使用 PowerShell 创建内部应用程序网关](/documentation/articles/application-gateway-create-gateway)

<!---HONumber=Mooncake_0104_2016-->