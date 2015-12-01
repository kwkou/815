<properties 
   pageTitle="什么是网络安全组 (NSG)"
   description="了解网络安全组 (NSG)"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="09/22/2015"
	wacn.date="11/27/2015"/>

# 什么是网络安全组 (NSG)？

可以在虚拟网络中使用 NSG 控制流向一个或多个虚拟机 (VM) 实例的流量。NSG 包含根据流量方向、协议、源地址和端口以及目标地址和端口允许或拒绝流量的访问控制规则。可以随时更改 NSG 的规则，所做的更改适用于所有关联的实例。

>[AZURE.WARNING]NSG 只能在区域 VNet 中使用。如果你尝试不用 VNet 保护部署中的终结点，或使用与地缘组关联的 VNet，请参阅[什么是终结点访问控制列表 (ACL)？](/documentation/articles/virtual-networks-acl)。你还可以[将 VNet 迁移到区域 VNet](/documentation/articles/virtual-networks-migrate-to-regional-vnet)。

![NSGs](./media/virtual-network-nsg-overview/figure1.png)

上图显示了一个包含两个子网的虚拟网络，每个子网与一个用于控制流量的 NSG 关联。

>[AZURE.NOTE]不支持将基于终结点的 ACL 和网络安全组置于相同 VM 实例上。如果你想要使用 NSG，但已有了终结点 ACL，则请先删除该终结点 ACL。有关如何执行此操作的信息，请参阅[使用 PowerShell 管理终结点的访问控制列表 (ACL)](/documentation/articles/virtual-networks-acl-powershell)。

## 网络安全组是如何工作的？

网络安全组不同于基于终结点的 ACL。终结点 ACL 只适用于通过输入终结点公开的公共端口。NSG 适用于一个或多个 VM 实例，并可控制 VM 上的所有入站和出站流量。

网络安全组具有*名称*，与*区域*相关联，并具有描述性标签。它包含两种类型的规则，即**入站**规则和**出站**规则。入站规则适用于传入到 VM 的传入数据包，出站规则适用于从 VM 传出的传出数据包。这些规则将应用到 VM 所在的主机中。传入或传出数据包必须匹配允许自己的**允许**规则，如果没有，则会删除。

规则将按优先级顺序处理。例如，优先级编号较低的规则（例如 100）在处理时将先于优先级编号较高的规则（例如 200）。找到匹配项之后，将不再对规则进行处理。

规则会指定以下内容：

- **名称：**规则的唯一标识符

- **类型：**入站/出站

- **优先级：**<You can specify an integer between 100 and 4096>

- **源 IP 地址：**源 IP 范围的 CIDR

- **源端口范围：**<integer or range between 0 and 65536>

- **目标 IP 范围：**目标 IP 范围的 CIDR

- **目标端口范围：**<integer or range between 0 and 65536>

- **协议：**<允许 TCP、UDP 或“*”>

- **访问：**允许/拒绝

### 默认规则

NSG 包含默认规则。默认规则无法删除，但由于给它们分配的优先级最低，可以用创建的规则来重写它们。默认规则描述平台所推荐的默认设置。正如以下默认规则所阐述的那样，从方向上来说，在 VNet 中发起和结束的流量可以是入站流量，也可以是出站流量。

虽然出站方向的流量允许连接到 Internet，但默认情况下，入站方向的流量在连接到 Internet 时会被阻止。还有一个的默认规则允许 Azure 的负载平衡器 (LB) 探测 VM 的运行状况。如果 NSG 下的 VM 或 VM 集不参与负载平衡集，你可以重写此规则。

默认规则是：

**入站**

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|-----------------------------------|----------|--------------------|-------------|-----------------|------------------|----------|--------|
| ALLOW VNET INBOUND                | 65000    | VIRTUAL_NETWORK    | *           | VIRTUAL_NETWORK | *                | *        | ALLOW  |
| ALLOW AZURE LOAD BALANCER INBOUND | 65001    | AZURE_LOADBALANCER | *           | *               | *                | *        | ALLOW  |
| DENY ALL INBOUND                  | 65500    | *                  | *           | *               | *                | *        | DENY   |

**出站**

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|-------------------------|----------|-----------------|-------------|-----------------|------------------|----------|--------|
| ALLOW VNET OUTBOUND     | 65000    | VIRTUAL_NETWORK | *           | VIRTUAL_NETWORK | *                | *        | ALLOW  |
| ALLOW INTERNET OUTBOUND | 65001    | *               | *           | INTERNET        | *                | *        | ALLOW  |
| DENY ALL OUTBOUND       | 65500    | *               | *           | *               | *                | *        | DENY   |

### 默认标记

默认标记是系统提供的针对某类 IP 地址的标识符。可以在客户定义的规则中指定默认标记。默认标记如下所示：

- **VIRTUAL\_NETWORK -** 此默认标记表示你的所有网络地址空间。它包括虚拟网络地址空间（Azure 中的 IP CIDR）以及所有连接的本地地址空间（本地网络）。这还包括 VNet 到 VNet 地址空间。

- **AZURE\_LOADBALANCER -** 此默认标记表示 Azure 的基础结构负载平衡器。这将转换到 Azure 数据中心 IP，从中进行 Azure 的运行状况探测。仅当与 NSG 关联的 VM 或 VM 集参与负载平衡集的情况下，才需要此项。

- **INTERNET -** 此默认标记表示虚拟网络外部的 IP 地址空间，可以通过公共 Internet 进行访问。此范围还包括 Azure 拥有的公共 IP 空间。

### ICMP 流量

当前的 NSG 规则只允许使用协议“TCP”或“UDP”。没有“ICMP”的特定标记。但默认情况下，根据入站 VNet 规则，虚拟网络中允许 ICMP 流量，因为该规则支持 VNet 中任何端口的传入和传出流量和协议。

## 将 NSG 相关联

可以将 NSG 与 VM、NIC 和子网相关联。

- **将 NSG 关联到 VM。** 将 NSG 关联到 VM 时，NSG 中的网络访问规则将应用到传入和传出 VM 的所有流量。 

- **将 NSG 关联到 NIC。** 将 NSG 关联到 NIC 时，NSG 中的网络访问规则只会应用到该 NIC。这意味着，在包含多个 NIC 的 VM 中，如果 NSG 已应用到单个 NIC，则它不会影响已绑定到其他 NIC 的流量。

- **将 NSG 关联到子网。**将 NSG 关联到子网时，NSG 中的网络访问规则将应用到子网中的所有 VM。

可以将不同的 NSG 关联到 VM、VM 所用的 NIC，以及 NIC 所绑定到的子网。如果发生这种情况，所有网络访问规则将按以下顺序应用到流量：

- **入站流量**
	1. 子网 NSG。
	2. NIC NSG。
	3. VM NSG。
- **出站流量**
	1. VM NSG。
	2. NIC NSG。
	3. 子网 NSG。

![NSG ACL](./media/virtual-network-nsg-overview/figure2.png)

>[AZURE.NOTE]尽管你只能将一个 NSG 关联到一个子网、VM 或 NIC，但可以将同一个 NSG 关联到任意数量的资源。

## 设计注意事项

设计 NSG 时，必须了解 VM 与基础结构服务和 Azure 中托管的 PaaS 服务的通信方式。大多数 Azure PaaS 服务（例如 SQL 数据库和存储器）只能通过面向公用的 Internet 地址访问。同样适用于负载平衡探测。

Azure 中的一个常见方案是基于这些对象是否需要访问 Internet 对子网中的 VM 和 PaaS 角色进行分离。在这种方案中，子网可能包含需要访问 Azure Paas 服务的 VM 或角色实例，例如 SQL 数据库和存储器，但是，这不需要任何与公共 Internet 的入站或出站通信。

假设以下 NSG 规则用于这种方案：

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|------|----------|-----------|-------------|----------------|------------------|----------|--------|
|NO INTERNET|100| VIRTUAL_NETWORK|&#42;|INTERNET|&#42;|TCP|DENY| 

由于该规则拒绝所有从虚拟网络到 Internet 的访问，VM 将不能访问任何需要公用 Internet 终结点的 Azure PaaS 服务，例如 SQL 数据库。

不使用拒绝规则，而是请考虑使用允许从虚拟网络访问 Internet，但拒绝从 Internet 访问虚拟网络的规则，如下所示：

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|------|----------|-----------|-------------|----------------|------------------|----------|--------|
|TO INTERNET|100| VIRTUAL_NETWORK|&#42;|INTERNET|&#42;|TCP|ALLOW|
|FROM INTERNET|110| INTERNET|&#42;|VIRTUAL_NETWORK|&#42;|TCP|DENY| 

>[AZURE.WARNING]Azure 使用称为“网关”子网的特殊子网处理其他 VNet 和本地网络的 VPN 网关。将 NSG 关联到此子网将导致 VPN 网关停止按预期方式正常工作。不要将 NSG 关联到网关子网！

你还需要考虑下面所列的特殊规则。请确保不要阻止这些规则允许的流量，否则你的基础结构将无法与基本 Azure 服务通信。

- **主机节点的虚拟 IP：**基本基础结构服务（例如 DHCP、DNS 和运行状况监视）是通过虚拟化主机 IP 地址 168.63.129.16 提供的。此公用 IP 地址属于 Microsoft，并将是唯一的用于所有区域的虚拟化 IP 地址，而且没有其他用途。此 IP 地址映射到托管虚拟机的服务器计算机（主机节点）的物理 IP 地址。主机节点充当 DHCP 中继、DNS 递归解析器，以及进行负载平衡器运行状况探测和计算机运行状况探测的探测源。不应将针对此 IP 地址的通信视为一种攻击。

- **许可（密钥管理服务）：**在虚拟机中运行的 Windows 映像应该获得许可。因此，将会向处理此类查询的密钥管理服务主机服务器发送许可请求。这将始终在出站端口 1688 上进行。

## 限制

在设计 NSG 时需要考虑以下限制。

|**说明**|**限制**|
|---|---|
|可与子网、VM 或 NIC 关联的 NSG 数目|1|
|每个区域每个订阅的 NSG 数目|100|
|每个 NSG 的 NSG 规则数|200|

在设计解决方案之前，请确保查看所有[与 Azure 中的网络服务相关的限制](/documentation/articles/azure-subscription-service-limits#networking-limits)。

## 后续步骤

- [在经典部署模型中部署 NSG](/documentation/articles/virtual-networks-create-nsg-classic-ps)。
- [在资源管理器中部署 NSG](/documentation/articles/virtual-networks-create-nsg-arm-pportal)。

<!---HONumber=82-->