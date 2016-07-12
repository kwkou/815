<properties 
	pageTitle="在 Azure 虚拟网络中安装 Active Directory 林" 
	description="本教程介绍如何在 Azure 虚拟网络上的虚拟机 (VM) 中创建新的 Active Directory 林。" 
	services="active-directory, virtual-network"
    keywords="active directory 虚拟机, 安装 active directory 林, azure active directory 视频"
	documentationCenter="" 
	authors="markusvi" 
	manager="stevenpo" 
	tags=""/>

<tags 
	ms.service="active-directory" 
	ms.date="04/07/2016" 
	wacn.date="06/24/2016"/>


# 在 Azure 虚拟网络中安装新的 Active Directory 林

本主题说明如何在 [Azure 虚拟网络](/documentation/articles/virtual-networks-overview/)上的虚拟机 (VM) 中创建新的 Windows Server Active Directory 环境。在此情况下，Azure 虚拟网络未连接到本地网络。

你也有可能对下列相关主题感兴趣：

- 有关这些步骤的演示视频，请参阅[如何在 Azure 虚拟网络中安装新的 Active Directory 林](http://channel9.msdn.com/Series/Microsoft-Azure-Tutorials/How-to-install-a-new-Active-Directory-forest-on-an-Azure-virtual-network)
- 你可以有选择性地[配置站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/)，然后安装新林，或者将本地林扩展到 Azure 虚拟网络。有关这些步骤的说明，请参阅[在 Azure 虚拟网络中安装副本 Active Directory 域控制器](/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller/)。
-  有关在 Azure 虚拟网络上安装 Active Directory 域服务 (AD DS) 的概念性指南，请参阅[在 Azure 虚拟机中部署 Windows Server Active Directory 的准则](https://msdn.microsoft.com/library/azure/jj156090.aspx)。

## 方案示意图

在此案例中，外部用户需要访问在添加域的服务器上运行的应用程序。运行应用程序服务器的 VM 及运行域控制器的 VM 安装在 Azure 虚拟网络中其自身的云服务内。它们还会包含在可用性集内以提高容错能力。

![Azure 虚拟网络中虚拟机上的 Active Directory 林][1]

## 在 Azure 上安装与在本地安装有什么不同？

在 Azure 上安装域控制器与在本地安装域控制器并没有太大的不同。下表列出了主要差别。

配置... | 本地 | Azure 虚拟网络	
------------- | -------------  | ------------
**域控制器的 IP 地址** | 在网络适配器属性中分配静态 IP 地址 | 运行 Set-AzureStaticVNetIP cmdlet 以分配静态 IP 地址
**DNS 客户端解析器** | 在域成员的网络适配器属性中设置首选和备用 DNS 服务器地址 | 在虚拟网络属性中设置 DNS 服务器地址
**Active Directory 数据库存储** | 选择性地更改默认存储位置 C:\\ | 你需要更改默认存储位置 C:\\



## 创建 Azure 虚拟网络

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 创建虚拟网络。单击“网络”>“创建虚拟网络”。使用下表中的值来完成向导操作。 

	在此向导页上... | 指定这些值
	------------- | -------------
	**虚拟网络详细信息** | <p>名称：输入虚拟网络的名称</p><p>区域：选择最靠近的区域</p>
	**DNS 和 VPN** | <p>将 DNS 服务器保留为空</p><p>不要选择任一 VPN 选项</p>
	**虚拟网络地址空间** | <p>子网名称：输入子网的名称</p><p>起始 IP：<b>10.0.0.0</b></p><p>CIDR：<b>/24 (256)</b></p>



## 创建 VM 以运行域控制器和 DNS 服务器角色
 
重复以下步骤，根据需要创建用于托管 DC 角色的 VM。应该至少部署两个虚拟域控制器来提供容错和冗余。如果 Azure 虚拟网络包含至少两个采用类似配置的 DC（即，它们都是 GC、运行 DNS 服务器，并且都不包含任何 FSMO 角色，等等），那么，你可将运行这些 DC 的 VM 放在可用性集中，以获得更高的容错能力。

若要使用 Windows PowerShell 而不是 UI 创建 VM，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms/)

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

## 安装 Windows Server Active Directory

使用你在本地使用的相同例程[安装 AD DS](https://technet.microsoft.com/zh-cn/library/jj574166.aspx)（也就是说，你可以使用 UI、应答文件或 Windows PowerShell）。需要提供管理员凭据来安装新林。若要指定 Active Directory 数据库、日志和 SYSVOL 的位置，请将默认存储位置从操作系统驱动器更改为你已附加到 VM 的额外数据磁盘。

完成 DC 安装后，请再次连接到 VM 并登录到 DC。请记得指定域凭据。

## 重置 Azure 虚拟网络的 DNS 服务器

1. 重置新 DC/DNS 服务器上的 DNS 转发器设置。 
  1. 在服务器管理器中，单击“工具”>“DNS”。 
  2. 在“DNS 管理器”中，右键单击 DNS 服务器的名称，然后单击“属性”。 
  3. 在“转发器”选项卡上，单击该转发器的 IP 地址，然后单击“编辑”。选择 IP 地址，然后单击“删除”。 
  4. 单击“确定”关闭编辑器，然后再次单击“确定”关闭 DNS 服务器属性。 
2. 更新虚拟网络的 DNS 服务器设置。 
  1. 单击“虚拟网络”，双击你创建的虚拟网络，单击“配置”>“DNS 服务器”，键入运行 DC/DNS 服务器角色的某个 VM 的名称和 DIP，然后单击“保存”。 
  2. 选择 VM 并单击“重新启动”触发该 VM，以便使用新 DNS 服务器的 IP 地址配置 DNS 解析器设置。 


## 为域成员创建 VM

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


## 另请参阅

-  [如何在 Azure 虚拟网络中安装新的 Active Directory 林](http://channel9.msdn.com/Series/Microsoft-Azure-Tutorials/How-to-install-a-new-Active-Directory-forest-on-an-Azure-virtual-network)
-  [在 Azure 虚拟机中部署 Windows Server Active Directory 的准则](https://msdn.microsoft.com/library/azure/jj156090.aspx)
-  [配置站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/)
-  [在 Azure 虚拟网络中安装副本 Active Directory 域控制器](/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller/)
-  [Microsoft Azure IT Pro IaaS：(01) 虚拟机基础知识](http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/01)
-  [Microsoft Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接](http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/05)
-  [虚拟网络概述](/documentation/articles/virtual-networks-overview/)
-  [如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)

-  [Azure PowerShell](http://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)

-  [Azure Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx)

-  [设置 Azure VM 静态 IP 地址](http://windowsitpro.com/windows-azure/set-azure-vm-static-ip-address)

-  [如何向 Azure VM 分配静态 IP](http://www.bhargavs.com/index.php/2014/03/13/how-to-assign-static-ip-to-azure-vm/)
-  [安装新的 Active Directory 林](https://technet.microsoft.com/library/jj574166.aspx)
-  [介绍 Active Directory 域服务 (AD DS) 虚拟化（级别 100）](https://technet.microsoft.com/library/hh831734.aspx)

<!--Image references-->
[1]: ./media/active-directory-new-forest-virtual-machine/AD_Forest.png

<!---HONumber=Mooncake_0620_2016-->