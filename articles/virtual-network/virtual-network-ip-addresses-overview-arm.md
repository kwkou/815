<properties
    pageTitle="IP 地址 | Azure"
    description="了解 Azure 中的公共 IP 地址和专用 IP 地址。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor="tysonn"
    tags="azure-resource-manager" />  

<tags
    ms.assetid="610b911c-f358-4cfe-ad82-8b61b87c3b7e"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/27/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />

# Azure 中的 IP 地址
可以将 IP 地址分配给 Azure 资源，以便与其他 Azure 资源、本地网络和 Internet 通信。Azure 中可使用两种类型的 IP 地址：

* **公共 IP 地址**：用于与 Internet（包括面向公众的 Azure 服务）通信
* **专用 IP 地址**：用于在 Azure 虚拟网络 (VNet) 中通信，以及在本地网络中通信（当你使用 VPN 网关或 ExpressRoute 线路将网络扩展到 Azure 时）。

> [AZURE.NOTE]
Azure 提供用于创建和处理资源的两种部署模型：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/virtual-network-ip-addresses-overview-classic/)。
> 

如果熟悉经典部署模型，请参阅 [经典部署与 Resource Manager IP 寻址间的差异](/documentation/articles/virtual-network-ip-addresses-overview-classic/#differences-between-resource-manager-and-classic-deployments)。

## 公共 IP 地址
公共 IP 地址允许 Azure 资源与 Internet 以及面向公众的 Azure 服务（例如 [Azure Redis 缓存](/home/features/redis-cache/)、[Azure 事件中心](/home/features/event-hubs/)、[SQL 数据库](/documentation/articles/sql-database-technical-overview/)和 [Azure 存储](/documentation/articles/storage-introduction/)）通信。

在 Azure Resource Manager 中，[公共 IP](/documentation/articles/resource-groups-networking/#public-ip-address) 地址是具有其自身属性的资源。可以将公共 IP 地址资源与以下任意资源相关联：

* 虚拟机 (VM)
* 面向 Internet 的负载均衡器
* VPN 网关
* 应用程序网关

### 分配方法
可通过两种方法将 IP 地址分配给 *公共* IP 资源： *动态* 或 *静态* 。默认分配方法为 *动态* ，即**不**在创建时分配 IP 地址。相反，在启动（或创建）关联的资源（例如 VM 或负载均衡器）时分配公共 IP 地址。停止（或删除）资源时即释放 IP 地址。因此，停止和启动资源都会导致 IP 地址更改。

若要确保所关联资源的 IP 地址保持不变，可将分配方法显式设置为 *静态* 。在这种情况下，创建时立即分配 IP 地址。只有删除资源或将其分配方法改为 *动态* 时，才会释放该地址。

> [AZURE.NOTE]
即使将分配方法设置为 *静态* ，也无法通过指定方式将实际 IP 地址分配到 *公共 IP 资源* ，而只能在创建资源时所在的 Azure 位置通过包含可用 IP 地址的池对 IP 地址进行分配。
>

以下情况通常使用静态公共 IP 地址：

* 最终用户需要更新防火墙规则才能与你的 Azure 资源通信。
* 对 DNS 名称进行解析时，如果更改了 IP 地址，则需更新 A 记录。
* 你的 Azure 资源可与使用基于 IP 地址的安全模型的其他应用或服务通信。
* 使用链接到 IP 地址的 SSL 证书。

> [AZURE.NOTE]
将公共 IP 地址（动态/静态）分配给 Azure 资源时所依据的 IP 范围列表已在 [Azure Datacenter IP ranges](https://www.microsoft.com/en-us/download/details.aspx?id=42064)（Azure 数据中心 IP 范围）中发布。
>

### DNS 主机名解析
可以为公共 IP 资源指定一个 DNS 域名标签，以便在 Azure 托管的 DNS 服务器中创建  *domainnamelabel* . *location* .chinacloudapp.cn 到公共 IP 地址的映射。例如，如果创建公共 IP 资源将 *domainnamelabel* 指定为“contoso”、将 Azure *location* 指定为“中国北部”，则完全限定域名 (FQDN) **contoso.chinanorth.chinacloudapp.cn** 将解析为该资源的公共 IP 地址。可以使用此 FQDN 创建指向 Azure 中的公共 IP 地址的自定义域 CNAME 记录。

> [AZURE.IMPORTANT]
所创建的每个域名标签在其 Azure 位置必须是唯一的。
>

### 虚拟机
将公共 IP 地址分配到其**网络接口**，可将其与 [Windows](/documentation/articles/virtual-machines-windows-about/) 或 [Linux](/documentation/articles/virtual-machines-linux-azure-overview/) VM 相关联。对于具有多个网络接口的 VM，只能将它分配给 *主要* 网络接口。可以向 VM 分配动态或静态公共 IP 地址。

### 面向 Internet 的负载均衡器
将公共 IP 地址分配给负载均衡器**前端**配置，可将其与 [Azure Load Balancer](/documentation/articles/load-balancer-overview/) 相关联。此公共 IP 地址充当负载均衡型虚拟 IP 地址 (VIP)。可以向负载均衡器前端分配动态或静态公共 IP 地址。还可以向负载均衡器前端分配多个公共 IP 地址，这样做可实现[多 VIP](/documentation/articles/load-balancer-multivip/) 方案，例如包含基于 SSL 的网站的多租户环境。

### VPN 网关
[Azure VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)用于将 Azure 虚拟网络 (VNet) 连接到其他 Azure VNet 或本地网络。必须将公共 IP 地址分配给其 **IP 配置**，才能与远程网络通信。目前只能向 VPN 网关分配 *动态* 公共 IP 地址。

### 应用程序网关
将公共 IP 地址分配给网关的**前端**配置可以将其与 Azure [应用程序网关](/documentation/articles/application-gateway-introduction/)相关联。此公共 IP 地址充当负载均衡型 VIP。目前，只能将 *动态* 公共 IP 地址分配给应用程序网关前端配置。

### 概览
下表显示了将公共 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

| 顶级资源 | IP 地址关联 | 动态 | 静态 |
| --- | --- | --- | --- |
| 虚拟机 |网络接口 |是 |是 |
| 负载均衡器 |前端配置 |是 |是 |
| VPN 网关 |网关 IP 配置 |是 |否 |
| 应用程序网关 |前端配置 |是 |否 |

## 专用 IP 地址
专用 IP 地址能够让 Azure 资源在不使用可访问 Internet 的 IP 地址的情况下，与[虚拟网络](/documentation/articles/virtual-networks-overview/)或本地网络中的其他资源（通过 VPN 网关或 ExpressRoute 线路）通信。

在 Azure Resource Manager 部署模型中，可将专用 IP 地址关联到以下类型的 Azure 资源：

* VM
* 内部负载均衡器 (ILB)
* 应用程序网关

### 分配方法
可以根据资源所附加到的子网的地址范围来分配专用 IP 地址。子网本身的地址范围是 VNet 的地址范围的一部分。

可通过两种方法分配专用 IP 地址： *动态* 或 *静态* 。默认分配方法为 *动态* ，即从资源的子网自动分配 IP 地址（使用 DHCP）。停止和启动该资源时，此 IP 地址可能更改。

可将分配方法设置为 *静态* ，以确保 IP 地址始终相同。在这种情况下，还需提供属于资源子网的有效 IP 地址。

静态专用 IP 地址通常用于：

* 充当域控制器或 DNS 服务器的 VM。
* 需要使用 IP 地址的防火墙规则的资源。
* 其他应用/资源通过 IP 地址访问的资源。

### 虚拟机
可将专用 IP 地址分配到 [Windows](/documentation/articles/virtual-machines-windows-about/) 或 [Linux](/documentation/articles/virtual-machines-linux-azure-overview/) VM 的**网络接口**。对于多网络接口 VM，将为每个接口分配一个专用 IP 地址。可将网络接口的分配方法指定为动态或静态。

#### 内部 DNS 主机名解析（针对 VM）
所有 Azure VM 都默认配置了 [Azure 托管的 DNS 服务器](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/#azure-provided-name-resolution)（除非显式配置自定义 DNS 服务器）。这些 DNS 服务器为驻留在同一个 VNet 内的 VM 提供内部名称解析。

创建 VM 时，主机名到其专用 IP 地址的映射将添加到 Azure 托管的 DNS 服务器。使用多网络接口 VM 时，主机名将映射到主要网络接口的专用 IP 地址。

使用 Azure 托管的 DNS 服务器配置的 VM 可以将 VNet 中的所有 VM 的主机名解析为其专用 IP 地址。

### 内部负载均衡器 (ILB) 和应用程序网关
可以将专用 IP 地址分配给 [Azure 内部负载均衡器](/documentation/articles/load-balancer-internal-overview/) (ILB) 或 [Azure 应用程序网关](/documentation/articles/application-gateway-introduction/)的**前端**配置。此专用 IP 地址将用作内部终结点，仅供其虚拟网络 (VNet) 和连接到该 VNet 的远程网络中的资源访问。可以将动态或静态专用 IP 地址分配给前端配置。

### 概览
下表显示了可用于将专用 IP 地址关联到顶级资源的特定属性，以及可使用的分配方法（动态或静态）。

| 顶级资源 | IP 地址关联 | 动态 | 静态 |
| --- | --- | --- | --- |
| 虚拟机 |网络接口 |是 |是 |
| 负载均衡器 |前端配置 |是 |是 |
| 应用程序网关 |前端配置 |是 |是 |

## 限制
Azure 中的全套[网络限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)详细说明了对 IP 寻址的限制。这些限制因区域和订阅而异。

## 定价
公共 IP 地址可能会产生少许费用。有关 Azure 中 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。

## 后续步骤
* [使用 Azure 门户预览部署具有静态公共 IP 的 VM](/documentation/articles/virtual-network-deploy-static-pip-arm-portal/)
* [使用模板为 VM 部署静态公共 IP](/documentation/articles/virtual-network-deploy-static-pip-arm-template/)
* [使用 Azure 门户预览部署具有静态专用 IP 地址的 VM](/documentation/articles/virtual-networks-static-private-ip-arm-pportal/)

<!---HONumber=Mooncake_1219_2016-->