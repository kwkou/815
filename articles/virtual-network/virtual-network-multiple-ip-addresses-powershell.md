<properties
    pageTitle="虚拟机的多个 IP 地址 - PowerShell | Azure"
    description="了解如何使用 PowerShell 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="c44ea62f-7e54-4e3b-81ef-0b132111f1f8"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/16/2016"
    wacn.date="01/05/2017"
    ms.author="jdial;annahar" />  


# 使用 PowerShell 将多个 IP 地址分配给虚拟机
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/virtual-network-multiple-ip-addresses-portal/)
- [PowerShell](/documentation/articles/virtual-network-multiple-ip-addresses-powershell/)
- [CLI](/documentation/articles/virtual-network-multiple-ip-addresses-cli/)

一个 Azure 虚拟机 (VM) 具有一个或多个附加的网络接口 (NIC)。可为任何 NIC 分配一个或多个静态/动态的公共和专用 IP 地址。将多个 IP 地址分配给 VM 可实现以下功能：

* 在一台服务器上托管具有不同 IP 地址和 SSL 证书的多个网站或服务。
* 用作网络虚拟设备，例如防火墙或负载均衡器。
* 能够将任何 NIC 的任意专用 IP 地址添加到 Azure Load Balancer 后端池。过去，只有主 NIC 的主 IP 地址才能添加到后端池。若要了解有关如何对多个 IP 配置进行负载均衡的详细信息，请阅读[对多个 IP 配置进行负载均衡](/documentation/articles/load-balancer-multiple-ip/)一文。

每个附加到 VM 的 NIC 都具有一个或多个与之关联的 IP 配置。系统会为每个配置分配一个静态或动态专用 IP 地址。每个配置还可拥有一个与之关联的公共 IP 地址资源。系统会为公共 IP 地址资源分配一个动态或静态 IP 地址。如果你不熟悉 Azure 中的 IP 地址，请参阅 [IP addresses in Azure](/documentation/articles/virtual-network-ip-addresses-overview-arm/)（Azure 中的 IP 地址）一文获取详细信息。

本文说明如何使用 PowerShell 将多个 IP 地址分配给通过 Azure Resource Manager 部署模型创建的 VM。无法将多个 IP 地址分配给通过经典部署模型创建的资源。若要了解有关 Azure 部署模型的详细信息，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

## <a name="Scenario"></a> 方案
创建具有一个 NIC 的 VM，并连接到虚拟网络。VM 需要三个不同的 *专用* IP 地址和两个 *公共* IP 地址。IP 地址会分配给以下 IP 配置：

* **IPConfig-1**：分配一个 *动态* 专用 IP 地址（默认）和一个 *静态* 公共 IP 地址。
* **IPConfig-2**：分配一个 *静态* 专用 IP 地址和一个 *静态* 公共 IP 地址。
* **IPConfig-3**：分配一个 *动态* 专用 IP 地址，不分配公共 IP 地址。
  
	![多个 IP 地址](./media/virtual-network-multiple-ip-addresses-powershell/OneNIC-3IP.png)  


IP 配置在创建 NIC 时关联到 NIC，NIC 在创建 VM 时附加到 VM。本方案使用的 IP 地址类型用于演示目的。可分配需要的任何 IP 地址和分配类型。

## <a name = "create"></a>创建具有多个 IP 地址的 VM

以下步骤说明如何根据本方案所述，创建具有多个 IP 地址的示例 VM。根据实现的需要，更改变量名称和 IP 地址类型。

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。如果尚未安装并配置 PowerShell，请先完成[How to install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell）一文中所述的步骤。
3. 完成[创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/) 一文中的步骤 1-4。请不要完成步骤 5（创建公共 IP 资源和网络接口）。如果更改该文中使用的任何变量的名称，请同样更改剩余步骤中的变量名称。若要创建 Linux VM，请选择 Linux 操作系统，而不要选择 Windows。
4. 键入以下命令，创建一个变量，用于存储“创建 Windows VM”一文的步骤 4（创建 VNet）中创建的子网对象：

        $SubnetName = $mySubnet.Name
        $Subnet = $myVnet.Subnets | Where-Object { $_.Name -eq $SubnetName }

5. 定义要分配给 NIC 的 IP 配置。可根据需要添加、删除或更改配置。方案所述的配置如下所示：

	**IPConfig-1**

	输入以下命令，创建：
	- 一个具有静态公共 IP 地址的公共 IP 地址资源
	- 一个具有公共 IP 地址资源和动态专用 IP 地址的 IP 配置

        $myPublicIp1     = New-AzureRmPublicIpAddress -Name "myPublicIp1" -ResourceGroupName $myResourceGroup -Location $location -AllocationMethod Static
        $IpConfigName1  = "IPConfig-1"
        $IpConfig1      = New-AzureRmNetworkInterfaceIpConfig -Name $IpConfigName1 -Subnet $Subnet -PublicIpAddress $myPublicIp1 -Primary

	请注意以上命令中的 `-Primary` 开关。为 NIC 分配多个 IP 配置时，必须将一个配置指定为 *Primary*。

	> [AZURE.NOTE]
	公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
	>

	**IPConfig-2**

	更改所创建子网上的可用有效地址后的 **$IPAddress** 变量的值。若要检查地址 10.0.0.5 在子网上是否可用，请输入命令 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.5 -VirtualNetwork $myVnet`。如果该地址可用，输出会返回 *True* 。如果不可用，输出会返回 *False* 以及可用地址的列表。输入以下命令，创建具有一个静态公共 IP 地址和一个静态专用 IP 地址的新公共 IP 地址资源和新 IP 配置：

        $IpConfigName2 = "IPConfig-2"
        $IPAddress     = 10.0.0.5
        $myPublicIp2   = New-AzureRmPublicIpAddress -Name "myPublicIp2" -ResourceGroupName $myResourceGroup `
        -Location $location -AllocationMethod Static
        $IpConfig2     = New-AzureRmNetworkInterfaceIpConfig -Name $IpConfigName2 `
        -Subnet $Subnet -PrivateIpAddress $IPAddress -PublicIpAddress $myPublicIp2

	**IPConfig-3**

	输入以下命令，创建具有一个动态专用 IP 地址且没有公共 IP 地址的 IP 配置：

        $IpConfigName3 = "IpConfig-3"
        $IpConfig3 = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName3 -Subnet $Subnet

6. 输入以下命令，使用上一步骤中定义的 IP 配置创建 NIC：

        $myNIC = New-AzureRmNetworkInterface -Name myNIC -ResourceGroupName $myResourceGroup `
        -Location $location -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3

	> [AZURE.NOTE]
	尽管本文将所有 IP 配置都分配给一个 NIC，但还可将多个 IP 配置分配给 VM 中的任何 NIC。若要了解如何创建具有多个 NIC 的 VM，请阅读[创建具有多个 NIC 的 VM](/documentation/articles/virtual-network-deploy-multinic-arm-ps/) 一文。

7. 完成[创建 VM](/documentation/articles/virtual-machines-windows-ps-create/) 一文的步骤 6。

	> [AZURE.WARNING]
	在以下情况中，“创建 VM”一文中的步骤 6 会失败：
	> - 在本文的步骤 6 中将名为 $myNIC 的变量更改为其他名称。
	> - 尚未完成本文以及“创建 VM”一文中的之前步骤。
	>
8. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        $myNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

## <a name="OsConfig"></a>将 IP 地址添加到 VM 操作系统

连接并登录所创建的具有多个专用 IP 地址的 VM。必须手动添加所有添加到 VM 的专用 IP 地址（包括主地址）。针对 VM 操作系统完成以下步骤：

### Windows

1. 在命令提示符下键入 *ipconfig /all* 。只能看到 *Primary* 专用 IP 地址（通过 DHCP）。
2. 在命令提示符窗口中键入 *ncpa.cpl* ，打开“网络连接”窗口。
3. 打开“本地连接”的属性。
4. 双击“Internet 协议版本 4 (IPv4)”。
5. 单击“使用下面的 IP 地址”并输入以下值：

	* **IP 地址**：输入 *Primary* 专用 IP 地址
	* **子网掩码**：根据子网设置此值。例如，如果子网为 /24 子网，则子网掩码为 255.255.255.0。
	* **默认网关**：子网中的第一个 IP 地址。如果子网为 10.0.0.0/24，则网关 IP 地址为 10.0.0.1。
	* 单击“使用下面的 DNS 服务器地址”并输入以下值：
		* **首选 DNS 服务器**：如果不使用自己的 DNS 服务器，请输入 168.63.129.16。如果使用自己的 DNS 服务器，请输入服务器的 IP 地址。
	* 单击“高级”按钮，然后添加其他 IP 地址。使用为主 IP 地址指定的相同子网，为 NIC 添加步骤 8 中列出的每个辅助专用 IP 地址。
	* 单击“确定”关闭“TCP/IP 设置”，然后再次单击“确定”关闭适配器设置。此时会重新建立 RDP 连接。
6. 在命令提示符下键入 *ipconfig /all* 。此时将显示添加的所有 IP 地址，DHCP 已关闭。
	
### Linux (Ubuntu)

1. 打开终端窗口。
2. 请确保以 root 用户身份操作。如果不是，请输入以下命令：

        sudo -i

3. 更新网络接口（假设为“eth0”）的配置文件。

	* 保留 dhcp 的现有行项。主 IP 地址会保留之前的配置。
	* 使用以下命令添加其他静态 IP 地址的配置：

            cd /etc/network/interfaces.d/
            ls

	应会看到一个 .cfg 文件。
4. 打开该文件：vi *文件名* 。

	该文件的末尾应会显示以下命令行：

        auto eth0
        iface eth0 inet dhcp

5. 在此文件包含的命令行后面添加以下命令行：

        iface eth0 inet static
        address <your private IP address here>

6. 使用以下命令保存该文件：

        :wq

7. 使用以下命令重置网络接口：

        sudo ifdown eth0 && sudo ifup eth0

	> [AZURE.IMPORTANT]
	如果使用远程连接，请在同一行中同时运行 ifdown 和 ifup。
	>

8. 使用以下命令验证 IP 地址是否已添加到网络接口：

        Ip addr list eth0

	应会在列表中看到添加的 IP 地址。
	
### Linux（Redhat、CentOS 和其他操作系统）

1. 打开终端窗口。
2. 请确保以 root 用户身份操作。如果不是，请输入以下命令：

        sudo -i

3. 输入密码，根据提示的说明操作。切换为 root 用户后，使用以下命令导航到网络脚本文件夹：

        cd /etc/sysconfig/network-scripts

4. 使用以下命令列出相关的 ifcfg 文件：

        ls ifcfg-*

	应会看到其中一个文件是 *ifcfg-eth0* 。

5. 使用以下命令复制 *ifcfg-eth0* 文件并将它命名为 *ifcfg-eth0:0* ：

        cp ifcfg-eth0 ifcfg-eth0:0

6. 使用以下命令编辑 *ifcfg-eth0:0* 文件：

        vi ifcfg-eth1

7. 使用以下命令在文件中将设备更改为适当的名称，在本例中为 *eth0:0* ：

        DEVICE=eth0:0

8. 更改 *IPADDR = YourPrivateIPAddress* 行以反映 IP 地址。
9. 使用以下命令保存该文件：

        :wq

10. 运行以下命令重新启动网络服务，确保更改成功：

        /etc/init.d/network restart
        Ipconfig

应会在返回的列表中看到添加的 IP 地址 *eth0:0* 。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤，可将其他专用和公共 IP 地址添加到现有 NIC。示例根据本文所述的[方案](#Scenario)生成。

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。如果尚未安装并配置 PowerShell，请先完成[How to install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell）一文中所述的步骤。
3. 将以下 $Variable 的“值”更改为要向其添加 IP 地址的 NIC 的名称，以及 NIC 所在的资源组和位置：

        $NICname         = "myNIC"
        $myResourceGroup = "myResourceGroup"
        $location        = "westchinaeast"

	如果不知道要更改的 NIC 名称，请输入以下命令，然后更改上述变量的值：

        Get-AzureRmNetworkInterface | Format-Table Name, ResourceGroupName, Location

4. 键入以下命令创建变量，并将它设置为现有的 NIC：

        $myNIC = Get-AzureRmNetworkInterface -Name $NICname -ResourceGroupName $myResourceGroup

5. 在以下命令中，将 *myVNet* 和 *mySubnet* 更改为 NIC 连接到的 VNet 和子网的名称。输入以下命令，检索 NIC 连接到的 VNet 和子网对象：

        $myVnet = Get-AzureRMVirtualnetwork -Name myVNet -ResourceGroupName $myResourceGroup
        $Subnet = $myVnet.Subnets | Where-Object { $_.Name -eq "mySubnet" }

	如果不知道 NIC 连接到的 VNet 或子网的名称，请输入以下命令：

        $mynic.IpConfigurations

	在返回的输出中查找类似如下所示的文本：

		Subnet   : {
					 "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"

	在此输出中，*myVnet* 和 *mySubnet* 分别是 NIC 连接到的 VNet 和子网。

6. 根据要求，完成以下任一部分中的步骤：

	**添加专用 IP 地址**
	
	若要将专用 IP 地址添加到 NIC，必须创建 IP 配置。以下命令创建具有静态 IP 地址 10.0.0.7 的配置。如果想要添加动态专用 IP 地址，请在输入命令前删除 `-PrivateIpAddress 10.0.0.7`。指定的静态 IP 地址必须是子网的未使用地址。建议首先输入 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet` 命令测试地址，确保地址可用。如果 IP 地址可用，输出会返回 *True* 。如果不可用，输出会返回 *False* 以及可用地址的列表。

        Add-AzureRmNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
         $myNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7

	使用唯一配置名称和专用 IP 地址（用于具有静态 IP 地址的配置），根据需要创建多个配置。

	**添加公共 IP 地址**
	
	将公共 IP 地址关联到新 IP 配置或现有 IP 配置即可添加它。根据需要，完成以下任一部分中的步骤。

	> [AZURE.NOTE]
	公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
	>

	**将资源关联到新 IP 配置**
	
	每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。若要新建，请输入以下命令：

        $myPublicIp3   = New-AzureRmPublicIpAddress -Name "myPublicIp3" -ResourceGroupName $myResourceGroup `
        -Location $location -AllocationMethod Static

 	若要创建具有动态专用 IP 地址和关联的 *myPublicIP3* 公共 IP 地址资源的新 IP 配置，请输入以下命令：

        Add-AzureRmNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
         $myNIC -Subnet $Subnet -PublicIpAddress $myPublicIp3

	**将资源关联到现有 IP 配置**
	公共 IP 地址资源仅可关联到尚未关联公共 IP 地址资源的 IP 配置。可输入以下命令，确定某个 IP 配置是否具有关联的公共 IP 地址：

        $myNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

	在返回的输出中查找类似如下所示的行：

		Name       PrivateIpAddress PublicIpAddress                                           Primary
		----       ---------------- ---------------                                           -------
		IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
		IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
		IpConfig-3 10.0.0.6                                                                     False

	*IpConfig-3* 的 **PublicIpAddress** 列为空白，因此，当前没有公共 IP 地址资源与其关联。可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

        $myPublicIp3   = New-AzureRmPublicIpAddress -Name "myPublicIp3" -ResourceGroupName $myResourceGroup `
        -Location $location -AllocationMethod Static

	输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3* 的现有 IP 配置：

        Set-AzureRmNetworkInterfaceIpConfig -Name IpConfig-3 -NetworkInterface $mynic -Subnet $Subnet -PublicIpAddress $myPublicIp3

7. 输入以下命令，为 NIC 设置新 IP 配置：

        Set-AzureRmNetworkInterface -NetworkInterface $myNIC

8. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        $myNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

9. 根据本文[将 IP 地址添加到 VM 操作系统](#OsConfig)部分中的说明，将添加到 NIC 的 IP 地址添加到 VM 操作系统。

<!---HONumber=Mooncake_1219_2016-->