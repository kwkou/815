<properties
    pageTitle="Azure 虚拟网络常见问题 | Azure"
    description="有关 Azure 虚拟网络的常见问题的解答。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="54bee086-a8a5-4312-9866-19a1fba913d0"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/18/2017"
    wacn.date="03/24/2017"
    ms.author="jdial" />  


# Azure 虚拟网络常见问题 (FAQ)

## 虚拟网络基础知识

### Azure 虚拟网络 (VNet) 是什么？
Azure 虚拟网络 (VNet) 是你自己的网络在云中的表示形式。它是对专用于你的订阅的 Azure 云进行的逻辑隔离。你可以使用 VNet 设置和管理 Azure 中的虚拟专用网络 (VPN)，或者链接 VNet 与 Azure 中的其他 VNet，或链接你的本地 IT 基础结构，以创建混合或跨界解决方案。创建的每个 VNet 具有其自身的 CIDR 块，只要 CIDR 块不重叠，即可链接到其他 VNet 和本地网络。还可以控制 VNet 的 DNS 服务器设置并将 VNet 分离到子网中。

使用 VNet：

* 创建私有云专用的 VNet：有时，不需要对解决方案使用跨界配置。创建 VNet 时，VNet 中的服务和 VM 可以在云中安全地互相直接通信。这可以在 VNet 内安全地保存流量，但仍允许你为需要 Internet 通信的 VM 和服务配置终结点连接，作为解决方案的一部分。
* 安全地扩展数据中心：借助 VNet，可以构建传统的站点到站点 (S2S) VPN，以便安全缩放数据中心的容量。S2S VPN 使用 IPSEC 提供企业 VPN 网关和 Azure 之间的安全连接。
* 实现混合云方案：利用 VNet 可以灵活支持一系列混合云方案。你可以安全地将基于云的应用程序连接到任何类型的本地系统，例如大型机和 Unix 系统。

### 如何知道是否需要 VNet？
[虚拟网络概述](/documentation/articles/virtual-networks-overview/)一文提供了一份决策表，可帮助你确定最佳的网络设计选项。

### 如何开始？
请访问[虚拟网络文档](/documentation/services/networking/)帮助自己入门。这些内容提供了所有 VNet 功能的概述和部署信息。

### 没有跨界连接的情况下是否可以使用 VNet？
可以。可以在不使用混合连接的情况下使用 VNet。如果想要在 Azure 中运行 Microsoft Windows Server Active Directory 域控制器和 SharePoint 场，此功能特别有用。

## 配置

### 要使用哪些工具创建 VNet？
可以使用以下工具创建或配置 VNet：

* Azure 门户（用于经典 VNet 和 Resource Manager VNet）。
* 网络配置文件（netcfg - 仅用于经典 VNet）。请参阅[使用网络配置文件配置 VNet](/documentation/articles/virtual-networks-using-network-configuration-file/) 一文。
* PowerShell（用于经典 VNet 和 Resource Manager VNet）。
* Azure CLI（用于经典 VNet 和 Resource Manager VNet）。

### 在我的 VNet 中可以使用哪些地址范围？
您可以使用 [RFC 1918](http://tools.ietf.org/html/rfc1918) 中定义的公共 IP 地址范围和任何 IP 地址范围。

### 我的 VNet 中是否可以有公共 IP 地址？
可以。有关公共 IP 地址范围的详细信息，请参阅[虚拟网络中的公共 IP 地址空间](/documentation/articles/virtual-networks-public-ip-within-vnet/)一文。无法从 Internet 直接访问公共 IP 地址。

### VNet 中的子网数量是否有限制？
是的。有关详细信息，请参阅 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。子网地址空间不能相互重叠。

### <a name="are-there-any-restrictions-on-using-ip-addresses-within-these-subnets"></a> 使用这些子网中的 IP 地址是否有任何限制？
是的。Azure 会保留每个子网中的某些 IP 地址。子网的第一个和最后一个 IP 地址仅为协议一致性而保留，其他 3 个地址用于 Azure 服务。

### VNet 和子网的最小和最大容量是多少？
我们支持的最小子网为 /29，最大为 /8（使用 CIDR 子网定义）。

### 是否可以使用 VNet 将 VLAN 引入 Azure 中？
否。VNet 是第 3 层重叠。Azure 不支持任何第 2 层语义。

### 是否可以在 VNet 和子网上指定自定义路由策略？
是的。可以使用用户定义路由 (UDR)。有关 UDR 的详细信息，请访问[用户定义的路由和 IP 转发](/documentation/articles/virtual-networks-udr-overview/)。

### VNet 是否支持多播或广播？
否。我们不支持多播或广播。

### 在 VNet 中可以使用哪些协议？
可以在 VNet 中使用 TCP、UDP 和 ICMP TCP/IP 协议。VNet 中会阻止多播、广播、在 IP 里面封装 IP 的数据包以及通用路由封装 (GRE) 数据包。

### 是否可以在 VNet 中 ping 默认路由器？
没有。

### 是否可以使用 tracert 诊断连接？
没有。

### 创建 VNet 后是否可以添加子网？
是的。只要子网地址不属于 VNet 中另一个子网的一部分，就能随时将子网添加到 VNet 中。

### 创建后是否可以修改子网的大小？
可以。如果子网中未部署任何 VM 或服务，可以添加、删除、扩展或收缩该子网。

### 创建子网后是否可以对其进行修改？
是的。可以添加、删除和修改 VNet 使用的 CIDR 块。

### 如果我在 VNet 中运行服务，是否可以连接到 Internet？
是的。VNet 中部署的所有服务都可以连接到 Internet。Azure 中部署的每个云服务都具有分配到它的可公开寻址的 VIP。必须定义 PaaS 角色的输入终结点和虚拟机的终结点，以使这些服务可以接受 Internet 的连接。

### VNet 是否支持 IPv6？
否。此词无法共同使用 IPv6 和 VNet。

### VNet 是否可以跨区域？
否。一个 VNet 限制为单个区域。

### 是否可以将 VNet 连接到 Azure 中的另一个 VNet？
是的。可使用以下方式将一个 VNet 连接到另一个 VNet：
- Azure VPN 网关。有关详细信息，请参阅[配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-howto-vnet-vnet-resource-manager-portal/)一文。
- VNet 对等互连。有关详细信息，请参阅 [VNet 对等互连概述](/documentation/articles/virtual-network-peering-overview/)一文。

## 名称解析 (DNS)

### VNet 的 DNS 选项有哪些？
使用[](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)“VM 和角色实例的名称解析”页的决策表，引导你浏览提供的所有 DNS 选项。

### 是否可以为 VNet 指定 DNS 服务器？
是的。可以在 VNet 设置中指定 DNS 服务器 IP 地址。这将作为 VNet 中的所有 VM 的默认 DNS 服务器进行应用。

### 可以指定多少 DNS 服务器？
有关详细信息，请参阅 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。

### 创建网络后是否可以修改 DNS 服务器？
是的。可以随时更改 VNet 的 DNS 服务器列表。如果更改 DNS 服务器列表，则需要重新启动 VNet 中的每个 VM，以使其拾取新的 DNS 服务器。

### 什么是 Azure 提供的 DNS？它是否适用于 VNet？
Azure 提供的 DNS 是由 Azure.cn 提供的多租户 DNS 服务。Azure 在此服务中注册所有 VM 和云服务角色实例。此服务通过主机名为相同云服务内包含的 VM 和角色实例提供名称解析，并通过 FQDN 为相同 VNet 中的 VM 和角色实例提供名称解析。请参阅 [VM 和角色实例的名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)一文，了解有关 DNS 的详细信息。

> [AZURE.NOTE]
目前，使用 Azure 提供的 DNS 进行跨租户名称解析时，VNet 中的前 100 个云服务存在限制。如果使用自己的 DNS 服务器，此限制则不适用。
> 
> 

### 是否可以基于每个 VM/服务重写 DNS 设置？
是的。您可以基于每个云服务设置 DNS 服务器，以重写默认网络设置。但是，我们建议你使用尽可能多的整个网络的 DNS。

### 是否可以引入我自己的 DNS 后缀？
否。不能为 VNet 指定自定义的 DNS 后缀。

## 连接虚拟机

### 是否可以将 VM 部署到 VNet？
是的。附加到通过 Resource Manager 部署模型部署的 VM 的所有网络接口 (NIC) 必须连接到 VNet。可以选择性地将通过经典部署模型部署的 VM 连接到 VNet。

### 可向 VM 分配哪些不同类型的 IP 地址？
* **专用：**分配到每个 VM 中的每个 NIC。使用静态或动态分配方法分配地址。应该分配 VNet 子网设置中指定的范围内的专用 IP 地址。通过经典部署模型部署的资源将接收专用 IP 地址，即使它们未连接到 VNet。动态专用 IP 地址将保留分配给某个资源，直到该资源被解除分配 (VM) 或删除（VM 或云服务部署槽）。静态专用 IP 地址将保留分配给某个资源，直到该资源被删除。
* **公共：**选择性地分配给附加到通过 Azure Resource Manager 部署模型部署的 VM 的 NIC。可以使用静态或动态分配方法分配地址。通过经典部署模型部署的所有 VM 和云服务角色实例位于分配有*动态*公共虚拟 IP (VIP) 地址的云服务中。可以选择性地将某个公共*静态* IP 地址（称为[保留 IP 地址](/documentation/articles/virtual-networks-reserved-public-ip/)）分配为 VIP。可将公共 IP 地址分配给通过经典部署模型部署的单个 VM 或云服务角色实例。这些地址称为[实例级公共 IP (ILPIP](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址，可动态分配。

### 是否可为以后创建的 VM 保留专用 IP 地址？
无法保留专用 IP 地址。如果某个专用 IP 地址可用，它将由 DHCP 服务器分配给 VM 或角色实例。该 VM 可能是要将专用 IP 地址分配到的 VM，也可能不是。但是，可将已创建的 VM 的专用 IP 地址更改为任何可用的专用 IP 地址。

### VNet 中 VM 的专用 IP 地址是否会变化？
视情况而定。在停止（解除分配）或删除 VM 之前，该 VM 将保留使用动态专用 IP 地址。只有在删除 VM 之后，才会从中释放静态专用 IP 地址。

### 是否可以在 VM 操作系统中手动将 IP 地址分配到 NIC？
可以，但不建议这样做。如果分配给 Azure VM 中的 NIC 的 IP 地址会发生变化，在 VM 的操作系统中手动更改 NIC 的 IP 地址可能会导致与 VM 断开连接。

### 如果在操作系统中停止云服务部署槽或关闭 VM，IP 地址会发生什么情况？
无变化。IP 地址（公共 VIP、公共和专用）将保留分配给该云服务部署槽或 VM。仅当停止（解除分配）或删除 VM，或者删除云服务部署槽时，才会释放动态地址。在 Azure 门户中单击 VM 对应的“停止”按钮会将其状态设置为“已停止(已解除分配)”。在这种情况下，VM 将会丢失其 IP 地址。

### 在无需重新部署的情况下，是否可以将 VM 从一个子网移动到 VNet 中的另一个子网？
可以。可在[如何将 VM 或角色实例移到其他子网](/documentation/articles/virtual-networks-move-vm-role-to-subnet/)一文中找到详细信息。

### 是否可以为我的 VM 配置静态 MAC 地址？
否。MAC 地址不能以静态方式配置。

### 创建 MAC 后，其地址是否在 VM 中保持不变？
是的，通过 Resource Manager 和经典部署模型部署的 VM 在被删除之前，其 MAC 地址将保持不变。以前，如果停止（解除分配）VM，将会释放 MAC 地址，但现在，即使 VM 处于解除分配状态，也会保留其 MAC 地址。

### 是否可以通过 VNet 中的 VM 连接到 Internet？
是的。VNet 中部署的所有 VM 和云服务角色实例都可连接到 Internet。

## 连接到 VNet 的 Azure 服务

### 是否可以在 VNet 中使用 Azure 应用服务 Web 应用？
是的。可以使用 ASE（应用服务环境）在 VNet 中部署 Web 应用。如果为 VNet 配置了点到站点连接，Web 应用可以安全地连接和访问 Azure VNet 中的资源。有关详细信息，请参阅以下文章：

* [将应用与 Azure 虚拟网络进行集成](/documentation/articles/app-service-vnet-integration-powershell/)

### 是否可以在 VNet 中部署云服务与 Web 和辅助角色 (PaaS)？
是的。（可选）可在 VNet 中部署云服务角色实例。为此，请在服务配置的网络配置部分中指定 VNet 名称和角色/子网映射。不需要更新任何二进制文件。

### 是否可将虚拟机规模集 (VMSS) 连接到 VNet？
是的。必须将 VMSS 连接到 VNet。

### 是否可以将服务移入和移出 VNet？
否。无法将服务移入和移出 VNet。必须删除并重新部署该服务，以将其移动到另一个 VNet 中。

## “安全”

### VNet 的安全模型是什么？
VNet 相互之间以及与 Azure 基础结构中托管的其他服务之间完全孤立。VNet 是一条信任边界。

### 是否可以限制入站或出站流量流向与 VNet 连接的资源？
是的。可向 VNet 中的单个子网和/或附加到 VNet 的 NIC 应用[网络安全组](/documentation/articles/virtual-networks-nsg/)。

### 你们是否提供了有关保护 VNet 的信息？
是的。有关详细信息，请参阅 [Azure 网络安全概述](/documentation/articles/security-network-overview/)一文。

## API、架构和工具

### 是否可以通过代码管理 VNet？
可以。可以在 [Azure Resource Manager](https://msdn.microsoft.com/zh-cn/library/mt163658.aspx) 和[经典（服务管理）](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)部署模型中使用适用于 VNet 的 REST API。

### 是否有 VNet 的工具支持？
是的。详细了解以下操作：
- 使用 Azure 门户通过 [Azure Resource Manager](/documentation/articles/virtual-networks-create-vnet-arm-pportal/) 和[经典](/documentation/articles/virtual-networks-create-vnet-classic-pportal/)部署模型部署 VNet。
- 使用 PowerShell 来管理通过 [Resource Manager](https://docs.microsoft.com/powershell/resourcemanager/azurerm.network/v3.1.0/azurerm.network) 和[经典](https://docs.microsoft.com/powershell/servicemanagement/azure.networking/v3.1.0/azure.networking)部署模型部署的 VNet。
- 使用 [Azure 命令行接口 (CLI)](/documentation/articles/azure-cli-arm-commands/#azure-network-commands-to-manage-network-resources) 来管理通过这两种部署模型部署的 VNet。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: wording update-->