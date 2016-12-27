<properties
    pageTitle="网络接口 | Azure"
    description="了解 Azure Resource Manager 中的 Azure 网络接口。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="f58b503f-18bf-4377-aa63-22fc8a96e4be"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="09/23/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 网络接口
网络接口 (NIC) 是虚拟机 (VM) 与基础软件网络之间互相连接的桥梁。本文解释什么是网络接口，以及在 Azure Resource Manager 部署模型中如何使用它。

Microsoft 建议使用 Resource Manager 部署模型来部署新资源，但也可以在[经典](/documentation/articles/virtual-network-ip-addresses-overview-classic/)部署模型中部署具有网络连接的 VM。如果你熟悉经典模式，请注意，Resource Manager 部署模型中的 VM 网络具有重要的差别。请阅读 [Virtual machine networking - Classic](/documentation/articles/virtual-network-ip-addresses-overview-classic/#differences-between-resource-manager-and-classic-deployments)（虚拟机网络 - 经典）一文详细了解这些差别。

在 Azure 中：

1. 网络接口是一种可以创建、删除的资源，具有自身的可配置的设置。
2. 在创建时，网络接口必须连接到 Azure 虚拟网络 (VNet) 中的一个子网。如果你不熟悉 VNet，请阅读 [Virtual network overview](/documentation/articles/virtual-networks-overview/)（虚拟网络概述）一文了解详细信息。NIC 必须连接到与 NIC 位于相同 Azure [位置](https://azure.microsoft.com/regions)和[订阅](/documentation/articles/azure-glossary-cloud-terminology/#subscription)中的 VNet。创建 NIC 之后，可以更改它连接到的子网，但无法更改它连接到的 VNet。
3. 有一个分配的名称，创建 NIC 后无法更改此名称。该名称在 Azure [资源组](/documentation/articles/resource-group-overview/#resource-groups)中必须唯一，但是在订阅中、创建所在的 Azure 位置中或者连接到的 VNet 中不必要唯一。通常会在 Azure 订阅中创建多个 NIC。建议制定一种命名约定，以便更轻松地管理多个 NIC，而不像使用默认名称时那样麻烦。
4. 可以附加到 VM，但只能附加到与 NIC 位于相同位置的单个 VM。
5. 具有 MAC 地址。只要 NIC 与 VM 保持连接，MAC 地址就会在 NIC 上保留。无论是使用 Azure 门户、Azure PowerShell 或 Azure 命令行接口将 VM 重新启动（从操作系统内部）还是停止（解除分配）再启动，MAC 地址都会保留。如果将 NIC 从 VM 分离，然后将其附加到不同的 VM，则 NIC 会收到不同的 MAC 地址。如果删除 NIC，MAC 地址将分配到其他 NIC。
6. 必须向 NIC 分配一个主要**专用** *IPv4* 静态或动态 IP 地址。
7. NIC 可以关联一个公共 IP 地址资源。
8. 对于运行特定 Microsoft Windows Server 操作系统版本的特定 VM 大小，支持具有单根 I/O 虚拟化 (SR-IOV) 的加快网络功能。
9. 如果为 NIC 启用了 IP 转发，NIC 可以接收目标不是其分配的专用 IP 地址的流量。例如，如果 VM 正在运行防火墙软件，则会路由目标不是其自身 IP 地址的数据包。VM 仍然必须运行可以路由或转发流量的软件，但为此必须对 NIC 启用 IP 转发。
10. NIC 通常在与它附加到的 VM 相同的资源组中创建，或者在它连接到的相同 VNet 中创建，但这不是强制性的要求。

## 具有多个网络接口的 VM
一个 VM 可以附加多个 NIC，但这种情况需要注意的是：

* VM 大小必须支持多个 NIC。若要了解有关哪些 VM 大小支持多个 NIC 的详细信息，请阅读 [Windows Server VM 大小](/documentation/articles/virtual-machines-windows-sizes/)或 [LLinux VM 大小](/documentation/articles/virtual-machines-linux-sizes/)这两篇文章。
* 必须使用至少两个 NIC 创建 VM。如果仅使用一个 NIC 创建 VM，那么即使该 VM 大小支持多个 NIC，也无法在创建 VM 后附加其他 NIC。只要使用至少两个 NIC 创建 VM，且 VM 大小支持两个以上的 NIC，就可以在创建 VM 后附加其他 NIC。
* 如果 VM 至少附加了三个 NIC，可以从 VM 中分离次要 NIC（无法分离主要 NIC）。如果 VM 附加了两个或两个以下的 NIC，则不能分离 NIC。
* VM 内部 NIC 的顺序将是随机的，在 Azure 基础结构更新过程中也可能会更改。但是，IP 地址和对应的以太网 MAC 地址会保持不变。例如，假设操作系统将 Azure NIC1 标识为 Eth1。Eth1 的 IP 地址为 10.1.0.100，MAC 地址为 00-0D-3A-B0-39-0D。Azure 基础结构更新并重启后，操作系统现在可能会将 Azure NIC1 标识为 Eth2，但 IP 和 MAC 地址会与操作系统将 Azure NIC1 标识为 Eth1 时相同。如果客户发起重启操作，NIC 顺序会在操作系统内保持不变。
* 如果 VM 是[可用性集](/documentation/articles/azure-glossary-cloud-terminology/#availability-set)的成员，可用性集内的所有 VM 都必须具有一个 NIC 或多个 NIC。如果 VM 拥有多个 NIC，只要每个 VM 至少拥有两个 NIC 即可，不要求每个 VM 拥有的 NIC 数量相同。

## 后续步骤
* 阅读 [Create a VM](/documentation/articles/virtual-machines-windows-hero-tutorial/)（创建 VM）一文，了解如何创建具有单个 NIC 的 VM。
* 阅读 [Deploy a VM with multiple NIC](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)（部署具有多个 NIC 的 VM）一文，了解如何创建具有多个 NIC 的 VM。
* 阅读 [Multiple IP addresses for Azure virtual machines](/documentation/articles/virtual-network-multiple-ip-addresses-powershell/)（Azure 虚拟机的多个 IP 地址）一文，了解如何创建具有多个 IP 配置的 NIC。

<!---HONumber=Mooncake_1219_2016-->