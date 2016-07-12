<properties
	pageTitle="在 Azure 中安装副本域控制器 | Azure"
	description="本教程说明如何从 Azure 虚拟机上的本地 Active Directory 林安装域控制器。"
	services="virtual-network"
	documentationCenter=""
	authors="curtand"
	manager="swadwha"
	editor=""
	tags="azure-classic-portal"/>

<tags
	ms.service="virtual-network"
	ms.date="02/01/2016"
	wacn.date="03/28/2016"/>


# 在 Azure 虚拟网络中安装副本 Active Directory 域控制器

此主题说明如何在 Azure 虚拟网络中的 Azure 虚拟机 (VM) 上为本地 Active Directory 域安装附加的域控制器（也称副本 DC）。

你也有可能对下列相关主题感兴趣：

- 你可以选择性地在 Azure 虚拟网络中安装新的 Active Directory 林。如需相关步骤，请参阅[在 Azure 虚拟网络中安装新的 Active Directory 林](/documentation/articles/active-directory-new-forest-virtual-machine/)。
-  有关在 Azure 虚拟网络上安装 Active Directory 域服务 (AD DS) 的概念性指南，请参阅[在 Azure 虚拟机中部署 Windows Server Active Directory 的准则](https://msdn.microsoft.com/zh-cn/library/azure/jj156090.aspx)。


## 方案示意图

在此案例中，外部用户需要访问在添加域的服务器上运行的应用程序。运行应用程序服务器和副本 DC 的 VM 安装在 Azure 虚拟网络中。虚拟网络可通过[站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/) 连接方式连接到本地网络，如下图中所示。
想要连接更快，你也可以使用 [ExpressRoute](/home/features/expressroute/)。

应用程序服务器和 DC 将部署在独立的云服务中以分散计算处理，并在[可用性集](/documentation/articles/virtual-machines-windows-manage-availability/)中改进容错功能。
DC 将使用 Active Directory 复制功能在彼此之间以及与本地 DC 相互复制。不需要任何同步工具。

![][1]

## 创建 Azure 虚拟网络的 Active Directory 站点

一个不错的想法是在 Active Directory 中创建一个站点来表示对应于虚拟网络的网络区域。这有助于优化身份验证、复制及其他 DC 位置操作。以下步骤说明了如何创建站点。有关更多背景信息，请参阅[添加新站点](http://technet.microsoft.com/zh-cn/library/cc781496.aspx)。

1. 打开 Active Directory 站点和服务：“服务器管理器”>“工具”>“Active Directory 站点和服务”。
2. 创建一个站点用于表示创建的 Azure 虚拟网络所在的区域：单击“站点”>“操作”>“添加站点”> 键入新站点的名称（例如“Azure 美国西部”）> 选择站点链接 >“确定”。
3. 创建子网并将其与新站点关联：双击“站点”> 右键单击“子网”>“添加子网”> 键入虚拟网络的 IP 地址范围（例如方案示意图中的 10.1.0.0/16）> 选择新的 Azure 站点 >“确定”。

## 创建 Azure 虚拟网络

1. 在 Azure 经典管理门户中，单击“添加”>“网络服务”>“虚拟网络”>“自定义创建”，然后使用以下值来完成向导。

    在此向导页上... | 指定这些值
	------------- | -------------
	**虚拟网络详细信息** | <p>名称：键入虚拟网络的名称，例如 WestUSVNet。</p><p>区域：选择最靠近的区域。</p>
	**DNS 和 VPN 连接** | <p>DNS 服务器：指定一个或多个本地 DNS 服务器的名称与 IP 地址。</p><p>连接：选择“配置站点到站点 VPN”。</p><p>局域网：指定新的局域网。</p><p>如果你使用 ExpressRoute 而不是 VPN，请参阅[通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps/)。</p>
	**站点到站点连接** | <p>名称：键入本地网络的名称。</p><p>VPN 设备 IP 地址：指定要连接到虚拟网络的设备的公共 IP 地址。VPN 设备不能位于 NAT 的后面。</p><p>地址：指定本地网络的地址范围（例如方案示意图中的 192.168.0.0/16）。</p>
	**虚拟网络地址空间** | <p>地址空间：指定你要在 Azure 虚拟网络中运行的 VM IP 地址范围（例如方案示意图中的 10.1.0.0/16）。此地址范围不能与本地网络的地址范围重叠。</p><p>子网：指定应用程序服务器的子网名称和地址（例如 Frontend、10.1.1.0/24）以及域控制器的子网名称和地址（例如 Backend、10.1.2.0/24）。</p><p>单击“添加网关子网”。</p>

2. 接下来，你将配置虚拟网络网关，以便创建安全的站点到站点 VPN 连接。<!--有关说明，请参阅[配置虚拟网络网关](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp/)。-->
3. 在新的虚拟网络与本地 VPN 设备之间创建站点到站点 VPN 连接。<!--有关说明，请参阅[配置虚拟网络网关](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp/)。-->



## 为 DC 角色创建 Azure VM

重复以下步骤，根据需要创建用于托管 DC 角色的 VM。应该至少部署两个虚拟域控制器来提供容错和冗余。如果 Azure 虚拟网络包含至少两个采用类似配置的 DC（即，它们都是 GC、运行 DNS 服务器，并且都不包含任何 FSMO 角色，等等），那么，你可将运行这些 DC 的 VM 放在可用性集中，以获得更高的容错能力。
若要使用 Windows PowerShell 而不是 UI 创建 VM，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-classic-create-powershell/)

1. 在 Azure 经典管理门户中，单击“添加”>“计算”>“虚拟机”>“从库中”。使用以下值来完成向导。除非建议或必须使用其他值，否则请接受默认的设置值。

    在此向导页上... | 指定这些值
	------------- | -------------
	**选择映像** | Windows Server 2012 R2 Datacenter
	**虚拟机配置** | <p>虚拟机名称：键入单个标签名称（例如 AzureDC1）。</p><p>新用户名：键入用户的名称。此用户将是 VM 上本地管理员组的成员。在首次登录 VM 时，你需要使用此名称。不能使用名为 Administrator 的内置帐户。</p><p>新密码/确认：键入密码</p>
	**虚拟机配置** | <p>云服务：针对第一个 VM 选择<b>“创建新的云服务”</b>，然后在创建其他用于托管 DC 角色的 VM 时选择与此相同的云服务名称。</p><p>云服务 DNS 名称：指定全局唯一的名称</p><p>区域/地缘组/虚拟网络：指定虚拟网络名称（例如 WestUSVNet）。</p><p>存储帐户：针对第一个 VM 选择<b>“使用自动生成的存储帐户”</b>，然后在创建其他用于托管 DC 角色的 VM 时选择与此相同的存储帐户名称。</p><p>可用性集：选择<b>“创建可用性集”</b>。</p><p>可用性集名称：在创建第一个 VM 时键入可用性集的名称，然后在创建更多的 VM 时键入与此相同的名称。</p>
	**虚拟机配置** | <p>选择<b>“安装 VM 代理”</b>，以及所需的任何其他扩展。</p>
2. 将磁盘附加到要运行 DC 服务器角色的每个 VM。需要提供额外的磁盘来存储 AD 数据库、日志和 SYSVOL。指定磁盘的大小（例如 10 GB）并将“主机缓存首选项”保持设置为“无”。有关步骤，请参阅[如何将数据磁盘附加到 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)。
3. 在首次登录 VM 之后，请打开“服务器管理器”>“文件和存储服务”，以使用 NTFS 在磁盘上创建一个卷。
4. 为要运行 DC 角色的 VM 保留静态 IP 地址。若要保留静态 IP 地址，请下载 Microsoft Web 平台安装程序，[安装 Azure PowerShell](/documentation/articles/powershell-install-configure/) 并运行 Set-AzureStaticVNetIP cmdlet。例如：

    'Get-AzureVM -ServiceName AzureDC1 -Name AzureDC1 | Set-AzureStaticVNetIP -IPAddress 10.0.0.4 | Update-AzureVM

有关如何设置静态 IP 地址的详细信息，请参阅[为 VM 配置静态内部 IP 地址](/documentation/articles/virtual-networks-reserved-private-ip/)。

## 在 Azure VM 上安装 AD DS

登录 VM，并确认你已通过站点到站点 VPN 建立连接，或者与本地网络上的资源建立了 ExpressRoute 连接。然后在 Azure VM 上安装 AD DS。可以使用在本地网络上安装其他域控制器的相同过程（UI、Windows PowerShell 或应答文件）。安装 AD DS 时，请务必指定新卷的 AD 数据库、日志和 SYSVOL 的位置。如果需要在 AD DS 安装中使用刷新程序，请参阅[安装 Active Directory 域服务（级别 100）](https://technet.microsoft.com/zh-cn/library/hh472162.aspx)或[在现有域中安装副本 Windows Server 2012 域控制器（级别 200）](https://technet.microsoft.com/zh-cn/library/jj574134.aspx)。

## 重新配置虚拟网络的 DNS 服务器

1. 在 Azure 经典管理门户中，单击虚拟网络的名称，然后单击“设置”选项卡以[重新设置虚拟网络的 DNS 服务器 IP 地址](/documentation/articles/virtual-networks-manage-dns-in-vnet/)，从而使用分配到副本域控制器的静态 IP 地址，而不是本地 DNS 服务器的 IP 地址。

2. 为了确保虚拟网络上的所有副本 DC VM 都已配置为使用虚拟网络上的 DNS 服务器，请单击“虚拟机”，单击每个 VM 的状态列，然后单击“重新启动”。等到 VM 显示“正在运行”状态，然后尝试登录其中。

## 为应用程序服务器创建 VM

1. 重复以下步骤，创建作为应用程序服务器运行的 VM。除非建议或必须使用其他值，否则请接受默认的设置值。


	在此向导页上... | 指定这些值
	------------- | -------------
	**选择映像** | Windows Server 2012 R2 Datacenter
	**虚拟机配置** | <p>虚拟机名称：键入单个标签名称（例如 AppServer1）。</p><p>新用户名：键入用户的名称。此用户将是 VM 上本地管理员组的成员。在首次登录 VM 时，你需要使用此名称。不能使用名为 Administrator 的内置帐户。</p><p>新密码/确认：键入密码</p>
	**虚拟机配置** | <p>云服务：针对第一个 VM 选择“创建新的云服务”，然后在创建其他用于托管应用程序的 VM 时选择与此相同的云服务名称。</p><p>云服务 DNS 名称：指定全局唯一的名称</p><p>区域/地缘组/虚拟网络：指定虚拟网络名称（例如 WestUSVNet）。</p><p>存储帐户：针对第一个 VM 选择“使用自动生成的存储帐户”，然后在创建其他用于托管应用程序的 VM 时选择与此相同的存储帐户名称。</p><p>可用性集：选择“创建可用性集”。</p><p>可用性集名称：在创建第一个 VM 时键入可用性集的名称，然后在创建更多的 VM 时键入与此相同的名称。</p>
	**虚拟机配置** | <p>选择<b>“安装 VM 代理”</b>，以及所需的任何其他扩展。</p>

2. 预配每个 VM 之后，登录 VM 并将其加入域。在“服务器管理器”中，单击“本地服务器”>“工作组”>“更改...”，然后选择“域”并键入本地域名。提供域用户的凭据，然后重新启动 VM 以完成加入域的操作。

若要使用 Windows PowerShell 而不是 UI 创建 VM，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-classic-create-powershell/)

有关使用 Windows PowerShell 的详细信息，请参阅 [Azure Cmdlet 入门](https://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx)和 [Azure Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx)。

## 其他资源

-  [在 Azure 虚拟机中部署 Windows Server Active Directory 的准则](https://msdn.microsoft.com/zh-cn/library/azure/jj156090.aspx)
-  [如何使用 Azure PowerShell 将现有的本地 Hyper-V 域控制器上载到 Azure](http://support.microsoft.com/zh-cn/kb/2904015)
-  [在 Azure 虚拟网络中安装新的 Active Directory 林](/documentation/articles/active-directory-new-forest-virtual-machine/)
-  [Azure 虚拟网络](https://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx)
-  [Azure IT Pro IaaS：(01) 虚拟机基础知识](http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/01)
-  [Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接](http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/05)
-  [Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)
-  [Azure 管理 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/jj152841)

<!--Image references-->
[1]: ./media/virtual-networks-install-replica-active-directory-domain-controller/ReplicaDCsOnAzureVNet.png

<!---HONumber=74-->