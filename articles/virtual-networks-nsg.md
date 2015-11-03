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
   ms.date="08/13/2015"
   wacn.date="09/18/2015" />

# 什么是网络安全组 (NSG)？

可以在虚拟网络中使用 NSG 控制流向一个或多个虚拟机 (VM) 实例的流量。网络安全组是关联到订阅的顶级对象。NSG 包含特定的访问控制规则，可以通过这些规则来允许或拒绝流向 VM 实例的流量。可以随时更改 NSG 的规则，所做的更改适用于所有关联的实例。若要使用 NSG，你必须具有与某区域（位置）关联的 VNet。

>[AZURE.WARNING]NSG 不兼容与地缘组相关联的 VNet。如果你没有区域性 VNet 却又希望控制流向终结点的流量，请参阅[什么是网络访问控制列表 (ACL)？](/documentation/articles/virtual-networks-acl)。你还可以[将 VNet 迁移到区域 VNet](/documentation/articles/virtual-networks-migrate-to-regional-vnet)。

可以将 NSG 与 VM 相关联，也可以将其与 VNet. 中的子网相关联。与 VM 关联后，NSG 适用于 VM 实例发送和接收的所有流量。应用到 VNet 中的子网后，它将适用于子网中所有 VM 实例发送和接收的所有流量。一个 VM 或子网只能与 1 个 NSG 相关联，但每个 NSG 都可以包含多达 200 条规则。每个订阅可以有 100 个 NSG。

>[AZURE.NOTE]不支持将基于终结点的 ACL 和网络安全组置于相同 VM 实例上。如果你想要使用 NSG，但已有了终结点 ACL，则请先删除该终结点 ACL。有关如何执行此操作的信息，请参阅[使用 PowerShell 管理终结点的访问控制列表 (ACL)](/documentation/articles/virtual-networks-acl-powershell)。

## 网络安全组是如何工作的？

网络安全组不同于基于终结点的 ACL。终结点 ACL 只适用于通过输入终结点公开的公共端口。NSG 适用于一个或多个 VM 实例，并可控制 VM 上的所有入站和出站流量。

网络安全组具有*名称*，与*区域*相关联，并具有描述性标签。它包含两种类型的规则，即**入站**规则和**出站**规则。入站规则适用于传入到 VM 的传入数据包，出站规则适用于从 VM 传出的传出数据包。这些规则将应用到 VM 所在的主机中。传入或传出数据包必须匹配允许自己的**允许**规则，如果没有，则会被删除。

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
| ALLOW VNET INBOUND | 65000 | VIRTUAL\_NETWORK | * | VIRTUAL\_NETWORK | * | * | ALLOW |
| ALLOW AZURE LOAD BALANCER INBOUND | 65001 | AZURE\_LOADBALANCER | * | * | * | * | ALLOW |
| DENY ALL INBOUND | 65500 | * | * | * | * | * | DENY |

**出站**

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|-------------------------|----------|-----------------|-------------|-----------------|------------------|----------|--------|
| ALLOW VNET OUTBOUND | 65000 | VIRTUAL\_NETWORK | * | VIRTUAL\_NETWORK | * | * | ALLOW |
| ALLOW INTERNET OUTBOUND | 65001 | * | * | INTERNET | * | * | ALLOW |
| DENY ALL OUTBOUND | 65500 | * | * | * | * | * | DENY |

### 特殊的基础结构规则

NSG 规则是显式的。除了 NSG 规则中指定的情况，不会对流量进行允许或拒绝操作。不过，不管网络安全组的规范如何，始终允许的两种类型的流量。进行这些预配是为了支持基础结构。

- **主机节点的虚拟 IP：**基本基础结构服务（例如 DHCP、DNS 和运行状况监视）是通过虚拟化主机 IP 地址 168.63.129.16 提供的。此公用 IP 地址属于 Microsoft，并将是唯一的用于所有区域的虚拟化 IP 地址，而且没有其他用途。此 IP 地址映射到托管虚拟机的服务器计算机（主机节点）的物理 IP 地址。主机节点充当 DHCP 中继、DNS 递归解析器，以及进行负载平衡器运行状况探测和计算机运行状况探测的探测源。不应将针对此 IP 地址的通信视为一种攻击。

- **许可（密钥管理服务）：**在虚拟机中运行的 Windows 映像应该获得许可。因此，将会向处理此类查询的密钥管理服务主机服务器发送许可请求。这将始终在出站端口 1688 上进行。

### 默认标记

默认标记是系统提供的针对某类 IP 地址的标识符。可以在客户定义的规则中指定默认标记。默认标记如下所示：

- **VIRTUAL\_NETWORK -** 此默认标记表示你的所有网络地址空间。它包括虚拟网络地址空间（Azure 中的 IP CIDR）以及所有连接的本地地址空间（本地网络）。这还包括 VNet 到 VNet 地址空间。

- **AZURE\_LOADBALANCER -** 此默认标记表示 Azure 的基础结构负载平衡器。这将转换到 Azure 数据中心 IP，从中进行 Azure 的运行状况探测。仅当与 NSG 关联的 VM 或 VM 集参与负载平衡集的情况下，才需要此项。

- **INTERNET -** 此默认标记表示虚拟网络外部的 IP 地址空间，可以通过公共 Internet 进行访问。此范围还包括 Azure 拥有的公共 IP 空间。

### 端口和端口范围

可以针对单个源/目标端口或端口范围指定网络安全组规则。当你需要为某个应用程序（如 FTP）打开一大系列的端口时，此方法特别有用。此范围只能是连续的，不能与单个端口指定混合。

若要指定端口范围，请使用“-”签名，如下面 *DestinationPortRange* 参数中所示：

	Get-AzureNetworkSecurityGroup -Name ApptierSG `
	| Set-AzureNetworkSecurityRule -Name FTP -Type Inbound -Priority 600 -Action Allow `
		-SourceAddressPrefix INTERNET -SourcePortRange * `
		-DestinationAddressPrefix * -DestinationPortRange 100-500 -Protocol *

### ICMP 流量

当前的 NSG 规则只允许使用协议“TCP”或“UDP”。没有“ICMP”的特定标记。但默认情况下，根据入站 VNet 规则，虚拟网络中允许 ICMP 流量，因为该规则支持 VNet 中任何端口的传入和传出流量，并支持 (*) 协议。

## 将 NSG 相关联

将 NSG 关联到 VM - 当 NSG 与 VM 直接关联时，NSG 中的网络访问规则直接应用到目标为 VM 的所有流量。每当因为规则更改而更新 NSG 时，更新内容数分钟内就会体现在流量处理操作中。从 VM 去关联 NSG 时，状态将返回到关联 NSG 之前的任何情况，例如：如果使用 NSG，则返回到采用之前的系统默认值。

将 NSG 关联到子网 - 当 NSG 与子网相关联时，NSG 中的网络访问规则将应用到子网中的所有 VM。每当更新 NSG 中的访问规则时，所做的更改将在数分钟内应用到子网中的所有虚拟机。

将 NSG 关联到子网和 VM - 你有可能可以将 NSG 关联到 VM，并将不同 NSG 关联到 VM 所在位置的子网。支持这种操作，并且在这种情况下，VM 会获取两层保护。在入站流量中，数据包通过 VM 中的规则，然后通过子网中指定的规则，而在出站情况下，数据包首先经过 VM 中指定的规则，然后才经过子网中指定的规则，如下图所示。

![NSG ACL](./media/virtual-networks-nsg/figure1.png)

当 NSG 与 VM 或子网相关联时，网络访问控制规则变得很明确。该平台不会插入任何允许流量流向特定端口的隐式规则。在这种情况下，如果你在 VM 中创建一个终结点，则还必须创建一项规则，以允许 Internet 传入的流量。如果不执行该操作，则不能从外部访问 VIP:<Port>。

例如，创建新的 VM，还可以创建新的 NSG。将 NSG 与 VM 相关联。该 VM 可以通过 ALLOW VNET INBOUND 规则与虚拟网络中的其他 VM 通信。该 VM 还可通过 ALLOW INTERNET OUTBOUND 规则进行连接到 Internet 的出站连接。随后，你可以在端口 80 上创建一个终结点，以接收传入 VM 中运行的网站的流量。除非你向 NSG 添加与下表类似的规则，否则从 Internet 流向 VIP（公用虚拟 IP 地址）上端口 80 的数据包将不会到达 VM。

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|------|----------|-----------|-------------|----------------|------------------|----------|--------|
| Web | 100 | INTERNET | * | * | 80 | TCP | ALLOW |

## 设计注意事项

设计 NSG 时，必须了解 VM 与基础结构服务和 Azure 托管的 PaaS 服务的通信方式。大多数 Azure PaaS 服务（例如 SQL 数据库和存储器）只能通过面向公用的 Internet 地址访问。同样适用于负载平衡探测。

Azure 中的一个常见方案是基于这些对象是否需要访问 Internet 对子网中的 VM 和 PaaS 角色进行分离。在这种方案中，子网可能包含需要访问 Azure Paas 服务的 VM 或角色实例，例如 SQL 数据库和存储器，但是，这不需要任何与公共 Internet 的入站或出站通信。

假设以下 NSG 规则用于这种方案：

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|------|----------|-----------|-------------|----------------|------------------|----------|--------|
|NO INTERNET|100| VIRTUAL\_NETWORK|&#42;|INTERNET|&#42;|TCP|DENY| 

由于该规则拒绝所有从虚拟网络到 Internet 的访问，VM 将不能访问任何需要公用 Internet 终结点的 Azure PaaS 服务，例如 SQL 数据库。

不使用拒绝规则，而是请考虑使用允许从虚拟网络访问 Internet，但拒绝从 Internet 访问虚拟网络的规则，如下所示：

| Name | Priority | Source IP | Source Port | Destination IP | Destination Port | 协议 | Access |
|------|----------|-----------|-------------|----------------|------------------|----------|--------|
|TO INTERNET|100| VIRTUAL\_NETWORK|&#42;|INTERNET|&#42;|TCP|ALLOW|
|FROM INTERNET|110| INTERNET|&#42;|VIRTUAL\_NETWORK|&#42;|TCP|DENY| 

>[AZURE.WARNING]Azure 使用称为“网关”子网的特殊子网处理其他 VNet 和本地网络的 VPN 网关。将 NSG 关联到此子网将导致 VPN 网关停止按预期方式正常工作。不要将 NSG 关联到网关子网！

## 规划 - 网络安全组工作流

以下是使用网络安全组时的基本工作流步骤。

### 工作流 – 创建和关联 NSG

1. 创建网络安全组 (NSG)。

1. 添加网络安全规则，除非默认规则已经足够。

1. 将 NSG 关联到 VM。

1. 更新 VM。

1. 更新后，NSG 规则将会立即生效。

### 工作流 – 更新现有 NSG

1. 添加、删除或更新现有 NSG 中的规则。

1. 与 NSG 相关联的所有 VM 将在数分钟内进行更新。当 NSG 规则已与 VM 相关联时，不需要更新 VM。

### 工作流 – 更改 NSG 关联

1. 将新 NSG 关联到已与另一个 NSG 相关联的 VM。

1. 更新 VM。

1. 新 NSG 中的规则将在数分钟内生效。

## 如何创建、配置和管理网络安全组

此时，只能通过使用 PowerShell cmdlet 和 REST API 配置和修改 NSG。不能通过使用管理门户来配置 NSG。下面的 PowerShell cmdlet 将帮助你创建、配置和管理 NSG。

**创建网络安全组**

	New-AzureNetworkSecurityGroup -Name "MyVNetSG" -Location uswest `
		-Label "Security group for my Vnet in China North"

**添加或更新规则**

	Get-AzureNetworkSecurityGroup -Name "MyVNetSG" `
	| Set-AzureNetworkSecurityRule -Name WEB -Type Inbound -Priority 100 `
		-Action Allow -SourceAddressPrefix 'INTERNET'  -SourcePortRange '*' `
		-DestinationAddressPrefix '*' -DestinationPortRange '*' -Protocol TCP


**删除 NSG 中的规则**

	Get-AzureNetworkSecurityGroup -Name "MyVNetSG" `
	| Remove-AzureNetworkSecurityRule -Name WEB

**将 NSG 关联到 VM**

	Get-AzureVM -ServiceName "MyWebsite" -Name "Instance1" `
	| Set-AzureNetworkSecurityGroupConfig -NetworkSecurityGroupName "MyVNetSG" `
	| Update-AzureVM

**查看关联到 VM 的 NSG**

	Get-AzureVM -ServiceName "MyWebsite" -Name "Instance1" `
	| Get-AzureNetworkSecurityGroupAssociation

**从 VM 中删除 NSG**

	Get-AzureVM -ServiceName "MyWebsite" -Name "Instance1" `
	| Remove-AzureNetworkSecurityGroupConfig -NetworkSecurityGroupName "MyVNetSG" `
	| Update-AzureVM

**将 NSG 关联到子网**

	Get-AzureNetworkSecurityGroup -Name "MyVNetSG" `
	| Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'VNetChinsNorth' `
		-SubnetName 'FrontEndSubnet'

**查看关联到子网的 NSG**

	Get-AzureNetworkSecurityGroupForSubnet -SubnetName 'FrontEndSubnet' `
		-VirtualNetworkName 'VNetChinsNorth' 

**从子网中删除 NSG**

	Get-AzureNetworkSecurityGroup -Name "MyVNetSG" `
	| Remove-AzureNetworkSecurityGroupFromSubnet -VirtualNetworkName 'VNetChinsNorth' `
		-SubnetName 'FrontEndSubnet'

**删除 NSG**

	Remove-AzureNetworkSecurityGroup -Name "MyVNetSG"

**获取 NSG 以及规则的详细信息**

	Get-AzureNetworkSecurityGroup -Name "MyVNetSG" -Detailed
 
**查看与 NSG 相关的所有 Azure PowerShell cmdlet**

	Get-Command *azurenetworksecuritygroup*

<!---HONumber=70-->