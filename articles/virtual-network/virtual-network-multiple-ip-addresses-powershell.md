<properties
   pageTitle="虚拟机的多个 IP 地址 - PowerShell | Azure"
   description="了解如何使用 Azure PowerShell 将多个 IP 地址分配到虚拟机。"
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
   ms.date="10/05/2016"
   wacn.date="11/14/2016"
   ms.author="jdial;annahar" />  



# 将多个 IP 地址分配到虚拟机

在一个 Azure 虚拟机 (VM) 上可以附加一个或多个网络接口 (NIC)。可为每个 NIC 分配一个或多个公共或专用 IP 地址。如果你不熟悉 Azure 中的 IP 地址，请参阅 [IP addresses in Azure](/documentation/articles/virtual-network-ip-addresses-overview-arm/)（Azure 中的 IP 地址）一文获取详细信息。本文说明如何使用 Azure PowerShell 在 Azure Resource Manager 部署模型中为 NIC 分配多个 IP 地址。

为 NIC 分配多个 IP 地址可让 VM：

- 在单个服务器上使用不同的 IP 地址和 SSL 证书托管多个网站或服务。
- 用作网络虚拟设备，例如防火墙或负载均衡器。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

## 方案

本文将三种 IP 配置关联到一个网络接口。将创建以下示例配置并将其分配到具有三个专用 IP 地址和一个公共 IP 地址的 NIC。

- IPConfig-1：一个动态专用 IP 地址（默认值）以及一个来自名为 PIP1 的公共 IP 地址资源的公共 IP 地址。
- IPConfig-2：一个静态专用 IP 地址，没有公共 IP 地址。
- IPConfig-3：一个动态专用 IP 地址，没有公共 IP 地址。

    ![Alt 图像文本](./media/virtual-network-multiple-ip-addresses-powershell/OneNIC-3IP.png)  


本方案假设有一个名为 *RG1* 的资源组，其中有一个名为 *VNet1* 的 VNet 和一个名为 *Subnet1* 的子网。此外，假设有一个名为 *VM1* 的 VM、与该 VM 关联的名为 *VM1-NIC1* 的网络接口，以及一个名为 *PIP1* 的公共 IP 地址。

[本文](/documentation/articles/virtual-machines-windows-ps-create/)逐步讲解如何创建上述资源（如果以前尚未创建这些资源）。

## <a name = "create"></a>创建具有多个 IP 地址的 VM

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。如果尚未安装并配置 PowerShell，请先完成[How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）一文中所述的步骤。

2. 将以下 $Variable 的“值”分别更改为虚拟网络所在的 Azure [位置](https://azure.microsoft.com/regions)、[资源组](/documentation/articles/resource-group-overview/#resource-groups)的名称、资源组中的 VNet、要将 NIC 连接到的子网以及 NIC 的名称。

        $Location = "chinaeast"
        $RgName   = "RG1"
        $VNetName   = "VNet1"
        $SubnetName = "Subnet1"
        $NicName     = "VM1-NIC1"
        $PIP = "PIP"

	如果不知道现有 Azure 位置或资源组的名称，请键入以下命令：

		Get-AzureRmLocation 	 | Format-Table Location
		Get-AzureRmResourceGroup | Format-Table ResourceGroupName

    <a name="subnet"></a>NIC 必须连接到现有 Azure 虚拟网络 (VNet) 中的子网。NIC、子网和 VNet 这三个组件必须位于同一个区域和[订阅](/documentation/articles/azure-glossary-cloud-terminology/#subscription)中。如果你不熟悉 VNet，请阅读 [Virtual network overview](/documentation/articles/virtual-networks-overview/)（虚拟网络概述）一文获取详细信息，或阅读 [Create a VNet](/documentation/articles/virtual-networks-create-vnet-arm-ps/)（创建 VNet）一文了解如何创建 VNet。

	如果不知道现有的 VNet 名称，请输入以下命令，并将上述变量中的 *VNet1* 替换为 VNet 的名称：

		Get-AzureRmVirtualNetwork | Format-Table Name

	如果返回的列表是空的，需要创建一个 VNet。若要了解创建方法，请阅读 [Create a virtual network](/documentation/articles/virtual-networks-create-vnet-arm-ps/)（创建虚拟网络）一文。

	键入以下命令获取 VNet 内的子网名称，将上述 *Subnet1* 替换为子网的名称：

		$VNet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RgName
		$VNet.Subnets | Format-Table Name, AddressPrefix

4. 输入以下命令检索子网，并将它分配给某个变量。

		$Subnet = $VNet.Subnets | Where-Object { $_.Name -eq $SubnetName }

5. <a name="ipconfigs"></a>定义要分配给 NIC 的 IP 配置。每个配置只能有一个静态或动态专用 IP 地址，以及一个与静态或动态地址关联的公共 IP 地址资源。

	根据要将多少个 IP 地址与 NIC 关联以及要配置的设置，添加或删除任意数目的配置。

	**IPConfig-1**

	将 *PIP1* 值更改为在创建 NIC 的位置中存在的、当前不与其他 NIC 关联的现有公共 IP 地址资源的名称。将 *RG1* 更改为公共 IP 地址资源所在的资源组的名称。将 *IPConfig-1* 更改为要提供给第一个 IP 配置的名称。输入以下命令：

		$PIP1 = Get-AzureRmPublicIPAddress -Name "PIP1" -ResourceGroupName $RgName

		$IpConfigName1 = "IPConfig-1"
		$IPConfig1     = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName1 -Subnet $Subnet -PublicIpAddress $PIP1 -Primary

	请注意 *-Primary* 开关。为 NIC 分配多个 IP 配置时，必须将一个配置指定为 *Primary*。如果不知道现有公共 IP 地址资源的名称，请输入以下命令：

		Get-AzureRMPublicIPAddress |Format-Table Name, Location, IPAddress, IpConfiguration

	如果返回的输出中 **IPConfiguration** 列没有任何值，则表示该公共 IP 地址资源不与现有的任何 NIC 关联，因此可以使用它。如果列表为空或没有可用的公共 IP 地址资源，可以使用 **New-AzureRmPublicIPAddress** 命令创建一个。

	>[AZURE.NOTE] 公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。

	**IPConfig-2**

	将 *IPConfig-2* 更改为要提供给第二个 IP 配置的名称，将 *10.0.0.5* 更改为要将 NIC 分配到有效且未使用的子网 IP 地址。输入以下命令：

		$IPConfigName2 = "IPConfig-2"
		$IPAddress = 10.0.0.5

		$IPConfig2 = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName2 -Subnet $Subnet -PrivateIpAddress $IPAddress

	如果不知道分配给子网的 IP 地址范围，请输入以下命令：

		$VNet.Subnets | Format-Table Name, AddressPrefix

	**IPConfig-3**

	将 *IPConfig-3* 更改为要提供给第三个 IP 配置的名称，并输入以下命令：

		$IPConfigName3 = "IPConfig-3"
		$IPConfig3 = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName3 -Subnet $Subnet

	>[AZURE.NOTE] 最多可为一个 NIC 分配 250 个专用 IP 地址。可在一个订阅中使用的公共 IP 地址数有限制。有关详细信息，请阅读 [Azure limits](/documentation/articles/azure-subscription-service-limits/#networking-limits---azure-resource-manager)（Azure 限制）一文。

6. 使用上一步骤中定义的 IP 配置创建 NIC。

		$nic = New-AzureRmNetworkInterface -Name $NicName -ResourceGroupName $RgName -Location $Location -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3

7. 根据 [Create a VM](/documentation/articles/virtual-machines-windows-ps-create/)（创建 VM）一文中的步骤创建 VM 时，请附加该 NIC。尽管上述文章创建的是运行 Windows Server 的 VM，但这些步骤同样适用于 Linux VM，差别只在于选择的操作系统不同。完成该文章中的步骤 1-3。略过“Create a VM”（创建 VM）一文中的步骤 4 和 5，然后完成步骤 6。

	>[AZURE.WARNING] 如果在本文的步骤 6 中将名为 $nic 的变量更改为其他设置，或者尚未完成本文的前面步骤，“Create a VM”（创建 VM）一文中的步骤 6 将会失败。

8. 输入以下命令，查看 Azure DHCP 分配给 NIC 的专用 IP 地址，以及分配给 NIC 的公共 IP 地址资源：

		$nic.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

9. <a name="os"></a>手动将所有辅助专用 IP 地址（在上一步骤中，输出的 **Primary** 列中值为 *False* 的 IP 地址）添加到操作系统中的 TCP/IP 配置。分配到 *IPConfig-1* 的专用 IP 地址将在步骤 5 中通过 Azure DHCP 自动分配到操作系统，因为它是 *Primary* 配置。


**Windows**

1. 在命令提示符下键入 *ipconfig /all*。只能看到 *Primary* 专用 IP 地址（通过 DHCP）。
2. 接下来，在命令提示窗口中键入 *ncpa.cpl*。此时将打开一个新窗口。
3. 打开“本地连接”的属性。
4. 双击“Internet 协议版本 4 (IPv4)”
5. 单击“使用下面的 IP 地址”并输入以下值：
	- **IP 地址**：输入 *Primary* 专用 IP 地址
	- **子网掩码**：根据子网设置此值。例如，如果子网为 /24 子网，则子网掩码为 255.255.255.0。
	- **默认网关**：子网中的第一个 IP 地址。如果子网为 10.0.0.0/24，则网关 IP 地址为 10.0.0.1。
	- 单击“使用下面的 DNS 服务器地址”并输入以下值：
		- **首选 DNS 服务器**：如果不使用自己的 DNS 服务器，请输入 168.63.129.16。否则请输入 DNS 服务器的 IP 地址。
	- 单击“高级”按钮，然后添加其他 IP 地址。使用为主 IP 地址指定的相同子网，为 NIC 添加步骤 8 中列出的每个辅助专用 IP 地址。
	- 单击“确定”关闭“TCP/IP 设置”，然后再次单击“确定”关闭适配器设置。随后将重新建立 RDP 连接。

6. 在命令提示符下键入 *ipconfig /all*。此时将显示添加的所有 IP 地址，DHCP 已关闭。


**Linux (Ubuntu)**

1. 打开终端窗口。
2. 请确保以 root 用户身份操作。否则，可以使用以下命令执行此操作：

			sudo -i
3. 更新网络接口（假设为“eth0”）的配置文件。
	- 保留 dhcp 的现有行项。这会像前面一样配置主 IP 地址。
	- 使用以下命令添加其他静态 IP 地址的配置：

				cd /etc/network/interfaces.d/
				ls

		应会看到一个 .cfg 文件。
4. 打开该文件：vi *文件名*。

	该文件的末尾应会显示以下命令行：

			auto eth0
			iface eth0 inet dhcp
5. 在此文件包含的命令行后面添加以下命令行：

			iface eth0 inet static
			address <your private IP address here>
6. 使用以下命令保存该文件：

			:wq
7.  使用以下命令重置网络接口：

			sudo ifdown eth0 && sudo ifup eth0

	>[AZURE.IMPORTANT] 如果使用远程连接，请在同一行中同时运行 ifdown 和 ifup。
8. 使用以下命令验证 IP 地址是否已添加到网络接口：

			ip addr list eth0

	应会在列表中看到添加的 IP 地址。

**Linux（Redhat、CentOS 和其他操作系统）**

1. 打开终端窗口。
2. 请确保以 root 用户身份操作。否则，可以使用以下命令执行此操作：

			sudo -i
3. 输入密码，根据提示的说明操作。切换为 root 用户后，使用以下命令导航到网络脚本文件夹：

			cd /etc/sysconfig/network-scripts
4. 使用以下命令列出相关的 ifcfg 文件：

			ls ifcfg-*

	应会看到其中一个文件是 *ifcfg-eth0*。
5. 使用以下命令复制 *ifcfg-eth0* 文件并将它命名为 *ifcfg-eth0:0*：

			cp ifcfg-eth0 ifcfg-eth0:0
6. 使用以下命令编辑 *ifcfg-eth0:0* 文件：

			vi ifcfg-eth1
7. 使用以下命令在文件中将设备更改为适当的名称，在本例中为 *eth0:0*：

			DEVICE=eth0:0
8. 更改 *IPADDR = YourPrivateIPAddress* 行以反映 IP 地址。
9. 使用以下命令保存该文件：

			:wq
10. 运行以下命令重新启动网络服务，确保更改成功：

			/etc/init.d/network restart
			Ipconfig

	应会在返回的列表中看到添加的 IP 地址 *eth0:0*。

## <a name="add"></a>将 IP 地址添加到现有 VM

完成以下步骤，将更多 IP 地址添加到现有 NIC：

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。如果尚未安装并配置 PowerShell，请先完成[How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）一文中所述的步骤。

2. 将以下 $Variable 的“值”分别更改为要向其添加 IP 地址的 NIC 名称，以及 NIC 所在的资源组和位置：

		$NicName     = "RG1-VM1-NIC1"
		$RgName   = "RG1"
		$NicLocation = "chinaeast"

	如果不知道要更改的 NIC 名称，请输入以下命令，然后更改上述变量的值：

		Get-AzureRmNetworkInterface | Format-Table Name, ResourceGroupName, Location

3. 键入以下命令创建变量，并将它设置为现有的 NIC：

		$nic = Get-AzureRmNetworkInterface -Name $NicName -ResourceGroupName $RgName

4. 完成本文“创建具有多个 IP 地址的 VM”部分中的[步骤 3](#subnet)，检索 NIC 连接到的子网 ID。

5. 根据本文“创建具有多个 IP 地址的 VM”部分的[步骤 5](#ipconfigs) 中的说明，创建要添加到网络的 IP 配置。

6. 将 *$IPConfigName4* 更改为在上一步骤中创建的 IP 配置名称。若要添加配置，请输入以下命令：

		Add-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName4 -NetworkInterface $nic -Subnet $Subnet1

7. 若要使用 IP 配置设置 NIC，请输入以下命令：

		Set-AzureRmNetworkInterface -NetworkInterface $nic

8. 根据本文“创建具有多个 IP 地址的 VM”部分的[步骤 9](#os) 中的说明，将添加到 NIC 的 IP 地址添加到 VM 操作系统。

<!---HONumber=Mooncake_1107_2016-->