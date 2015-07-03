<properties pageTitle="创建具有多个 NIC 的 VM" description="如何创建具有多个 NIC 的 VM" services="virtual-network, virtual-machines" documentationCenter="na" authors="telmosampaio" manager="adinah" editor="tysonn"/>

<tags ms.service="virtual-network" ms.date="04/30/2015"  wacn.date="06/26/2015"/>

# 创建具有多个 NIC 的 VM

使用多 NIC 功能，你可以在 Azure 虚拟机 (VM) 上创建和管理多个虚拟网络接口卡 (NIC)。多 NIC 是许多网络虚拟设备（例如应用程序传送和 WAN 优化解决方案）所必需的。多 NIC 还提供更多的网络流量管理功能，包括在前端 NIC 和后端 NIC 之间进行流量隔离，或者将数据平面流量与管理平面流量进行隔离。

![适合 VM 的多 NIC](./media/virtual-networks-multiple-nics/IC757773.png)

上图显示了带三个 NIC 的 VM，每个 NIC 都连接到不同的子网。

## 要求和约束

多 NIC 目前有以下要求和约束：

- 必须在 Azure 虚拟网络 (VNet) 中创建多 NIC VM。不支持非 VNet VM。 
- 在单个云服务中，仅允许以下设置： 
	- 该云服务中的所有 VM 都必须启用多 NIC，否则 
	- 该云服务中的所有 VM 都必须使用单个 NIC 

>[AZURE.IMPORTANT]如果你尝试将多 NIC VM 添加到某个已包含单 NIC VM 的部署（云服务），则会收到以下错误（反之亦然）：同一部署中不支持同时使用带辅助网络接口的虚拟机和不带辅助网络接口的虚拟机，此外，不能将不带辅助网络接口的虚拟机更新为带辅助网络接口，反之亦然。
 
- 仅在“默认”NIC 上支持面向 Internet 的 VIP。默认 NIC 的 IP 只有一个对应的 VIP。 
- 多 NIC VM 目前不支持实例级公共 IP 地址。 
- VM 内部 NIC 的顺序将是随机的，在 Azure 基础结构更新过程中也可能会更改。不过，IP 地址和相应的以太网 MAC 地址将会保持不变。例如，假定 **Eth1** 的 IP 地址为 10.1.0.100，MAC 地址为 00-0D-3A-B0-39-0D；在进行 Azure 基础结构更新并重新启动后，它可能会更改为 Eth2，但 IP 和 MAC 配对将会保持不变。如果是客户执行的重新启动，NIC 顺序将保持不变。 
- 每个 VM 上的每个 NIC 的地址必须位于一个子网中，你可以向单个 VM 上的多个 NIC 分配同一子网中的地址。 
- VM 大小决定了可以为 VM 创建的 NIC 的数目。下表列出了与 VM 大小相对应的 NIC 数目： 

|VM 大小（标准 SKU）|NIC 数（每个 VM 允许的最大数目）|  
|---|---|  
|所有基本大小|1|  
|A0\\特小型|1| 
|A1\\小型|1|  
|A2\\中型|1|  
|A3\\大型|2|  
|A4\\特大型|4|  
|A5|1|  
|A6|2|  
|A7|4|  
|A8|2|  
|A9|4|  
|A10|2|  
|A11|4|  
|D1|1|  
|D2|2|  
|D3|4|  
|D4|8|  
|D11|2|  
|D12|4|  
|D13|8|  
|D14|16|  
|DS1|1|  
|DS2|2|  
|DS3|4|  
|DS4|8|  
|DS11|2|  
|DS12|4|  
|DS13|8|  
|DS14|16|  
|G1|1|  
|G2|2|  
|G3|4|  
|G4|8|  
|G5|16|  
|所有其他大小|1|

## 网络安全组
VM 上的任何 NIC（包括启用了多 NIC 的 VM 上的任何 NIC）都可以与网络安全组 (NSG) 相关联。如果向 NIC 分配了子网中的地址，且该子网与某个 NSG 相关联，则子网的 NSG 中的规则也适用于该 NIC。除了将子网与 NSG 相关联，你还可以将 NIC 与 NSG 相关联。

如果子网与 NSG 相关联，且该子网中的 NIC 与 NSG 单独关联，则关联的 NSG 规则将按“**流顺序**”（根据传入或传出 NIC 的流量的方向）进行应用：

- **传入流量**：其目标为本文所讨论的 NIC，在首先流经子网时，会触发子网的 NSG 规则，然后在传入到 NIC 中之前，会触发 NIC 的 NSG 规则。- **传出流量**：其源为本文所讨论的 NIC，在首先从 NIC 中流出时，会触发 NIC 的 NSG 规则，然后在通过子网之前，会触发子网的 NSG 规则。 

上图表示如何根据流量（从 VM 流向子网，或者从子网流向 VM）来应用 NSG 规则。

## 如何配置多 NIC VM

下面的说明将指导你创建包含 3 个 NIC（1 个默认 NIC，2 个其他的 NIC）的多 NIC VM。这些配置步骤将会创建一个 VM，该 VM 将根据下面的服务配置文件片段进行配置：

	<VirtualNetworkSite name="MultiNIC-VNet" Location="China East">
	<AddressSpace>
	  <AddressPrefix>10.1.0.0/16</AddressPrefix>
	    </AddressSpace>
	    <Subnets>
	      <Subnet name="Frontend">
	        <AddressPrefix>10.1.0.0/24</AddressPrefix>
	      </Subnet>
	      <Subnet name="Midtier">
	        <AddressPrefix>10.1.1.0/24</AddressPrefix>
	      </Subnet>
	      <Subnet name="Backend">
	        <AddressPrefix>10.1.2.0/23</AddressPrefix>
	      </Subnet>
	      <Subnet name="GatewaySubnet">
	        <AddressPrefix>10.1.200.0/28</AddressPrefix>
	      </Subnet>
	    </Subnets>
	… Skip over the remainder section …
	</VirtualNetworkSite>


在尝试运行示例中的 PowerShell 命令之前，你需要满足以下先决条件。

- Azure 订阅。
- 已配置虚拟网络。有关 VNet 的详细信息，请参阅[虚拟网络概述](https://msdn.microsoft.com/zh-CN/library/azure/jj156007.aspx)。
- 已下载和安装最新版本的 Azure PowerShell。请参阅[如何安装和配置 Azure PowerShell](install-configure-powershell)。

1. 从 Azure VM 映像库中选择 VM 映像。请注意，映像会经常更改，并按区域提供。以下示例中指定的映像可能会更改，也可能不在你的区域提供，因此务必指定所需的映像。 

	    $image = Get-AzureVMImage `
	    	-ImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201410.01-en.us-127GB.vhd"

1. 创建 VM 配置。

		$vm = New-AzureVMConfig -Name "MultiNicVM" -InstanceSize "ExtraLarge" `
			-Image $image.ImageName –AvailabilitySetName "MyAVSet"

1. 创建默认的管理员登录名。

		Add-AzureProvisioningConfig –VM $vm -Windows -AdminUserName "<YourAdminUID>" `
			-Password "<YourAdminPassword>"

1. 将其他 NIC 添加到 VM 配置。

		Add-AzureNetworkInterfaceConfig -Name "Ethernet1" `
			-SubnetName "Midtier" -StaticVNetIPAddress "10.1.1.111" -VM $vm 
		Add-AzureNetworkInterfaceConfig -Name "Ethernet2" `
			-SubnetName "Backend" -StaticVNetIPAddress "10.1.2.222" -VM $vm

1. 指定默认 NIC 的子网和 IP 地址。

		Set-AzureSubnet -SubnetNames "Frontend" -VM $vm Set-AzureStaticVNetIP  `
			-IPAddress "10.1.0.100" -VM $vm

1. 在虚拟网络中创建 VM。

		New-AzureVM -ServiceName "MultiNIC-CS" –VNetName "MultiNIC-VNet" –VMs $vm

>[AZURE.NOTE]你在此处指定的 VNet 必须已存在（已在先决条件中提到过）。下面的示例指定名为“MultiNIC-VNet”的虚拟网络。

## 另请参阅

[虚拟网络概述](https://msdn.microsoft.com/zh-CN/library/azure/jj156007.aspx)

[虚拟网络配置任务](https://msdn.microsoft.com/zh-CN/library/azure/jj156206.aspx)

[博客文章 - Azure 中的多个 VM NIC 和 VNet 设备](multiple-vm-nics-and-network-virtual-appliances-in-azure)

<!---HONumber=61-->