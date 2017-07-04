<properties
    pageTitle="IP 地址（经典）| Azure"
    description="了解 Azure 中的公共 IP 地址和专用 IP 地址（经典）。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor="tysonn"
    tags="azure-service-management" />  

<tags
    ms.assetid="2f8664ab-2daf-43fa-bbeb-be9773efc978"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/11/2016"
    wacn.date="03/28/2017"
    ms.author="jdial" />

# Azure 中的 IP 地址（经典）
可以将 IP 地址分配到与其他 Azure 资源通信的 Azure 资源，也可以将其分配到本地网络和 Internet。可以在 Azure 中使用两种类型的 IP 地址：公共地址和专用地址。

公共 IP 地址用于与 Internet 通信，这些通信中包括面向公众的 Azure 服务。

使用 VPN 网关或 ExpressRoute 线路将网络扩展到 Azure 时，专用 IP 地址用于在 Azure 虚拟网络 (VNet)、云服务以及本地网络中通信。

> [AZURE.IMPORTANT]
Azure 拥有用于创建和使用资源的两个不同的部署模型：[资源管理器和经典部署](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型的情况。Azure 建议大多数新部署使用 Resource Manager。请阅读 [IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)一文，了解 Resource Manager 中的 IP 地址。

## 公共 IP 地址
公共 IP 地址可允许 Azure 资源与 Internet 以及面向公众的 Azure 服务（例如 [Azure Redis 缓存](/home/features/redis-cache/)、[Azure 事件中心](/home/features/event-hubs/)、[SQL 数据库](/documentation/articles/sql-database-technical-overview/)和 [Azure 存储空间](/documentation/articles/storage-introduction/)）通信。

公共 IP 地址与以下资源类型相关联：

* 云服务
* IaaS 虚拟机 (VM)
* PaaS 角色实例
* VPN 网关
* 应用程序网关

### 分配方法
如果需要向 Azure 资源分配公共 IP 地址，将动态地从资源的创建位置中的可用公共 IP 地址池分配该 IP 地址。停止该资源时，将释放此 IP 地址。对于云服务而言，所有角色实例均已停止时会发生这种情况，可以使用静态（保留）IP 地址避免发生这种情况（请参阅[云服务](#Cloud-services)）。

> [AZURE.NOTE]
将公共 IP 地址分配到 Azure 资源时所依据的 IP 范围列表已在 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)中发布。
> 
> 

### DNS 主机名解析
在创建云服务或 IaaS VM 时，你需要提供在 Azure 的所有资源中唯一的云服务 DNS 名称。这在 Azure 托管的 DNS 服务器中创建 *dnsname*.chinacloudapp.cn 到资源的公共 IP 地址的映射。例如，创建云服务 DNS 名称为 **contoso** 的云服务时，完全限定域名 (FQDN) **contoso.chinacloudapp.cn** 将解析为该云服务的公共 IP 地址 (VIP)。可以使用此 FQDN 创建指向 Azure 中的公共 IP 地址的自定义域 CNAME 记录。

### <a name="Cloud-services"></a> 云服务
云服务始终具有一个称为虚拟 IP 地址 (VIP) 的公共 IP 地址。你可以在云服务中创建终结点，以便将 VIP 中的不同端口关联到 VM 中的内部端口以及云服务中的角色实例。

云服务可以包含多个 IaaS VM 或 PaaS 角色实例，通过同一云服务 VIP 公开。还可以[向云服务分配多个 VIP](/documentation/articles/load-balancer-multivip/)，这样可以启用多 VIP 方案，如包含基于 SSL 的网站的多租户环境。

可以使用称为[保留 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 的静态公共 IP 地址，确保云服务的公共 IP 地址保持不变，即使所有角色实例均已停止，也是如此。你可以在特定位置中创建静态（保留）IP 资源并将其分配到该位置中的任何云服务。不能为保留 IP 指定实际 IP 地址，该地址从创建它的位置中的可用的 IP 地址池分配。除非显式删除，否则不会释放该 IP 地址。

静态（保留）公共 IP 地址通常用于云服务满足以下条件的方案：

* 需要最终用户设置防火墙规则。
* 依赖于外部 DNS 名称解析，并且动态 IP 需要更新 A 记录。
* 使用外部 Web 服务，这些服务使用基于 IP 的安全模型。
* 使用链接到 IP 地址的 SSL 证书。

> [AZURE.NOTE]
创建经典 VM 时，Azure 创建的容器 *云服务* 具有虚拟 IP 地址 (VIP)。通过门户创建时，由门户配置默认 RDP 或 SSH *终结点*，使用户可通过云服务 VIP 连接到 VM。可以保留此云服务 VIP，它能够有效地提供保留 IP 地址以便连接到 VM。你可以通过配置多个终结点打开其他端口。
> 
> 

### IaaS VM 和 PasS 角色实例
可以将公共 IP 地址直接分配到云服务内的 IaaS [VM](/documentation/articles/virtual-machines-linux-azure-overview/) 或 PaaS 角色实例。此类 IP 地址称为实例层级公共 IP 地址 ([ILPIP](/documentation/articles/virtual-networks-instance-level-public-ip/))。此公共 IP 地址只能是动态的。

> [AZURE.NOTE]
它与云服务的 VIP（IaaS VM 或 PaaS 角色实例的容器）不同，因为云服务可包含多个 IaaS VM 或 PaaS 角色实例，通过同一云服务 VIP 公开。
> 
> 

### VPN 网关
可以使用 [VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)将 Azure VNet 连接到其他 Azure VNet 或本地网络。为 VPN 网关动态地分配公共 IP 地址，启用与远程网络的通信。

### 应用程序网关数
Azure [应用程序网关](/documentation/articles/application-gateway-introduction/)可使 Layer7 负载均衡用来根据 HTTP 路由网络流量。为应用程序网关动态地分配公共 IP 地址，该地址可作为负载均衡 VIP。

### 速览
下表显示了具有可能分配方法（动态/静态）的每种资源类型以及它是否具有分配多个公共 IP 地址的能力。

| 资源 | 动态 | 静态 | 多个 IP 地址 |
| --- | --- | --- | --- |
| 云服务 |是 |是 |是 |
| IaaS VM 或 PaaS 角色实例 |是 |否 |否 |
| VPN 网关 |是 |否 |否 |
| 应用程序网关 |是 |否 |否 |

## 专用 IP 地址
专用 IP 地址让 Azure 资源能够在不使用可访问 Internet 的 IP 地址的情况下，与云服务、[虚拟网络](/documentation/articles/virtual-networks-overview/) (VNet) 或本地网络中的其他资源（通过 VPN 网关和 ExpressRoute 线路）进行通信。

在 Azure 经典部署模型中，可以将专用 IP 地址分配到以下 Azure 资源：

* IaaS VM 和 PasS 角色实例
* 内部负载均衡器
* 应用程序网关

### IaaS VM 和 PasS 角色实例
与 PaaS 角色实例类似，使用经典部署模型创建的虚拟机 (VM) 始终放置在云服务中。因此，这些资源的专用 IP 地址的行为是类似的。

请务必注意，可以通过两种方式部署云服务：

* 作为独立云服务，在这种情况下它不在虚拟网络中。
* 作为虚拟网络的一部分。

#### 分配方法
如果是独立云服务，资源将从 Azure 数据中心专用 IP 地址范围获取动态分配的专用 IP 地址。只能使用它与同一云服务内的其他 VM 通信。停止和启动该资源时，此 IP 地址可能更改。

如果是虚拟网络中部署的云服务，资源将获取从关联子网（在其网络配置中指定）的地址范围中分配的专用 IP 地址。此专用 IP 地址可用于 VNet 中所有 VM 之间的通信。

此外，如果是 VNet 中的云服务，则默认动态（使用 DHCP）分配专用 IP 地址。停止和启动该资源时，该地址可能更改。若要确保 IP 地址保持不变，则需要将分配方法设置为静态，并提供相应地址范围内的有效 IP 地址。

静态专用 IP 地址通常用于：

* 充当域控制器或 DNS 服务器的 VM。
* 需要使用 IP 地址的防火墙规则的 VM。
* 运行服务的 VM，其他应用通过 IP 地址访问这些服务。

#### 内部 DNS 主机名解析
所有 Azure VM 和 PaaS 角色实例都默认配置了 [Azure 托管的 DNS 服务器](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/#azure-provided-name-resolution)（除非显式配置了自定义 DNS 服务器）。这些 DNS 服务器为驻留在同一个 VNet 或云服务内的 VM 和角色实例提供内部名称解析。

创建 VM 时，主机名到其专用 IP 地址的映射将添加到 Azure 托管的 DNS 服务器。使用多 NIC VM 时，主机名将映射到主 NIC 的专用 IP 地址。但是，此映射信息将限制为同一云服务或 VNet 中的资源。

如果是独立云服务，则只能解析同一云服务内的所有 VM/角色实例的主机名。如果是 VNet 中的云服务，则可以解析该 VNet 中的所有 VM/角色实例的主机名。

### 内部负载均衡器 (ILB) 和应用程序网关
可以将专用 IP 地址分配给 [Azure 内部负载均衡器](/documentation/articles/load-balancer-internal-overview/) (ILB) 或 [Azure 应用程序网关](/documentation/articles/application-gateway-introduction/)的**前端**配置。此专用 IP 地址将用作内部终结点，仅供其虚拟网络 (VNet) 和连接到该 VNet 的远程网络中的资源访问。你可以将动态或静态专用 IP 地址分配到前端配置。还可以分配多个专用 IP 地址以启用多 vip 方案。

### 速览
下表显示了具有可能分配方法（动态/静态）的每种资源类型以及它是否具有分配多个专用 IP 地址的能力。

| 资源 | 动态 | 静态 | 多个 IP 地址 |
| --- | --- | --- | --- |
| VM（在独立云服务中） |是 |是 |是 |
| PaaS 角色实例（在独立云服务中） |是 |否 |是 |
| VM 或 PaaS 角色实例（在 VNet 中） |是 |是 |是 |
| 内部负载均衡器前端 |是 |是 |是 |
| 应用程序网关前端 |是 |是 |是 |

## 限制
下表显示 Azure 中每个订阅的 IP 寻址的限制。你可以[与支持人员联系](/support/contact/)，将默认限制提高到最大限制，具体取决于业务需求。

| | 默认限制 | 最大限制 |
| --- | --- | --- |
| 公共 IP 地址 (动态) |5 |联系支持人员 |
| 保留的公共 IP 地址 |20 |联系支持人员 |
| 每个部署（云服务）的公共 VIP |5 |联系支持人员 |
| 每个部署（云服务）的专用 VIP (ILB) |1 |1 |

确保已阅读 Azure 中所有的[网络限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)。

## 定价
在大多数情况下，公共 IP 地址是免费的。使用额外和/或静态公共 IP 地址要收取少许费用。确保用户了解[公共 IP 的定价结构](/pricing/details/reserved-ip-addresses/)。

## <a name="Differences-between-Resource-Manager-and-classic-deployments" id="differences-between-resource-manager-and-classic-deployments"></a>Resource Manager 与经典部署之间的差异
下面是 Resource Manager 和经典部署模型中的 IP 寻址功能的比较。

| | 资源 | 经典 | 资源管理器 |
| --- | --- | --- | --- |
| **公共 IP 地址** |VM |称为 ILPIP（仅限动态） |称为公共 IP（动态或静态） |
| |分配到 IaaS VM 或 PaaS 角色实例 |与 VM 的 NIC 关联 | |
| 面向 Internet 的负载均衡器 |称为 VIP（动态）或保留 IP（静态） |称为公共 IP（动态或静态） | |
| |分配到云服务 |与负载均衡器的前端配置关联 | |
| | | | |
| **专用 IP 地址** |VM |称为 DIP |称为专用 IP 地址 |
| |分配到 IaaS VM 或 PaaS 角色实例 |分配到 VM 的 NIC | |
| 内部负载均衡器 (ILB) |分配到 ILB（动态或静态） |分配到 ILB 的前端配置（动态或静态） | |

## 后续步骤
* 通过经典管理门户[使用静态专用 IP 地址部署 VM](/documentation/articles/virtual-networks-static-private-ip-classic-pportal/)。

<!---HONumber=Mooncake_1219_2016-->