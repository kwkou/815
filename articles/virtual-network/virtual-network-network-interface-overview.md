<properties 
   pageTitle="网络接口 | Azure"
   description="了解 Azure Resource Manager 中的 Azure 网络接口。"
   services="virtual-network"
   documentationCenter="na"
   authors="jimdial"
   manager="carmonm"
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
   wacn.date="11/14/2016"
   ms.author="jdial" />  


# 网络接口

网络接口 (NIC) 是虚拟机 (VM) 与基础软件网络之间互相连接的桥梁。本文解释什么是网络接口，以及在 Azure Resource Manager 部署模型中如何使用它。

Microsoft 建议使用 Resource Manager 部署模型来部署新资源，但也可以在[经典](/documentation/articles/virtual-network-ip-addresses-overview-classic/)部署模型中部署具有网络连接的 VM。如果你熟悉经典模式，请注意，Resource Manager 部署模型中的 VM 网络具有重要的差别。请阅读 [Virtual machine networking - Classic](/documentation/articles/virtual-network-ip-addresses-overview-classic/#differences-between-resource-manager-and-classic-deployments)（虚拟机网络 - 经典）一文详细了解这些差别。

在 Azure 中：

1. 网络接口是一种可以创建、删除的资源，具有自身的可配置的设置。
2. 在创建时，网络接口必须连接到 Azure 虚拟网络 (VNet) 中的一个子网。如果你不熟悉 VNet，请阅读 [Virtual network overview](/documentation/articles/virtual-networks-overview/)（虚拟网络概述）一文了解详细信息。NIC 必须连接到与 NIC 位于相同 Azure 位置和[订阅](/documentation/articles/azure-glossary-cloud-terminology/#subscription)中的 VNet。创建 NIC 之后，可以更改它连接到的子网，但无法更改它连接到的 VNet。
3. 有一个分配的名称，创建 NIC 后无法更改此名称。该名称在 Azure [资源组](/documentation/articles/resource-group-overview/#resource-groups)中必须唯一，但是在订阅中、创建所在的 Azure 位置中或者连接到的 VNet 中不必要唯一。通常会在 Azure 订阅中创建多个 NIC。建议制定一种命名约定，以便更轻松地管理多个 NIC，而不像使用默认名称时那样麻烦。
4. 可以附加到 VM，但只能附加到与 NIC 位于相同位置的单个 VM。
5. 具有 MAC 地址。只要 NIC 与 VM 保持连接，MAC 地址就会在 NIC 上保留。无论是使用 Azure 门户、Azure PowerShell 或 Azure 命令行接口将 VM 重新启动（从操作系统内部）还是停止（解除分配）再启动，MAC 地址都会保留。如果将 NIC 从 VM 分离，然后将其附加到不同的 VM，则 NIC 会收到不同的 MAC 地址。如果删除 NIC，MAC 地址将分配到其他 NIC。
6. 必须向 NIC 分配一个主要**专用** *IPv4* 静态或动态 IP 地址。
8. NIC 可以关联一个公共 IP 地址资源。
10. 如果为 NIC 启用了 IP 转发，NIC 可以接收目标不是其分配的专用 IP 地址的流量。例如，如果 VM 正在运行防火墙软件，则会路由目标不是其自身 IP 地址的数据包。VM 仍然必须运行可以路由或转发流量的软件，但为此必须对 NIC 启用 IP 转发。
11. NIC 通常在与它附加到的 VM 相同的资源组中创建，或者在它连接到的相同 VNet 中创建，但这不是强制性的要求。

可以将多个 NIC 附加到同一个 VM，但前提是 VM 大小支持这样做。有关哪些 VM 大小支持多个 NIC 的详细信息，请阅读文章 [Windows Server VM sizes](/documentation/articles/virtual-machines-windows-sizes/)（Windows Server VM 大小）或 [Linux VM sizes](/documentation/articles/virtual-machines-linux-sizes/)（Linux VM 大小）。

## 后续步骤

- 阅读 [Create a VM](/documentation/articles/virtual-machines-windows-hero-tutorial/)（创建 VM）一文，了解如何创建具有单个 NIC 的 VM。
- 阅读 [Deploy a VM with multiple NIC](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)（部署具有多个 NIC 的 VM）一文，了解如何创建具有多个 NIC 的 VM。
- 阅读 [Multiple IP addresses for Azure virtual machines](/documentation/articles/virtual-network-multiple-ip-addresses-powershell/)（Azure 虚拟机的多个 IP 地址）一文，了解如何创建具有多个 IP 配置的 NIC。

<!---HONumber=Mooncake_1107_2016-->