<properties 
	pageTitle="教程：创建跨界虚拟网络以建立站点到站点连接" 
	description="通过本教程了解如何创建具有跨界连接的 Azure 虚拟网络。" 
	services="virtual-network" 
	documentationCenter="" 
	authors="cherylmc" 
	manager="adinah" 
	editor="tysonn"/>

<tags 
	ms.service="virtual-network" 
	ms.date="08/17/2015" 
	wacn.date="09/18/2015"/>



# 教程：创建跨界虚拟网络以建立站点到站点连接

本教程将指导你逐步完成创建支持站点到站点连接的跨界虚拟网络的步骤。

如果你想要创建仅包含云的虚拟网络，请参阅 [教程：在 Azure 中创建仅包含云的虚拟网络](/documentation/articles/create-virtual-network)。<!-- If you want to create a point-to-site VPN by using certificates and a VPN client, see [Configure a Point-to-Site VPN in the Management Portal](http://msdn.microsoft.com/zh-cn/library/azure/dn133792.aspx).-->

本教程假定你之前未使用过 Azure。其目的是帮助你熟悉创建跨界虚拟网络所需的步骤。如果你要查找有关虚拟网络的设计方案和高级信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview)。

完成本教程后，你将获得一个示例跨界虚拟网络。下图根据本教程中的示例设置显示了详细信息。

![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_12_TutorialCrossPremVNet.png)

有关此图的副本以及可用于描述自己跨界虚拟网络的副本，请参阅[教程主题中的示例跨界虚拟网络图](http://gallery.technet.microsoft.com/Example-cross-premises-e5ecb8bb)。

请注意，本教程中使用的示例配置设置未针对你组织的网络自定义。如果你使用本主题中所述的示例配置设置来配置虚拟网络和站点到站点连接，将无法正常操作。若要配置可行的跨界虚拟网络，你必须咨询你的 IT 部门和网络管理员以获取正确的设置。有关详细信息，请参阅本主题的**先决条件**部分。

有关添加虚拟机并将本地 Active Directory 扩展到 Azure 虚拟网络的信息，请参阅以下内容：

-  [如何创建自定义虚拟机](/documentation/articles/virtual-machines-create-custom)

-  [在 Azure 虚拟网络中安装 Active Directory 域控制器副本](/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller)

有关在 Azure 虚拟机上部署 AD DS 的指南，请参阅[在 Azure 虚拟机中部署 Windows Server Active Directory 的准则](http://msdn.microsoft.com/zh-cn/library/azure/jj156090.aspx)。

<!--For additional Virtual Network configuration procedures and settings, see [Azure Virtual Network Configuration Tasks](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx).-->

##  目标

在本教程中，你将学习：

-  如何设置可在其中添加 Azure 服务的示例跨界 Azure 虚拟网络。

-  如何将虚拟网络配置为与组织网络进行通信。

##  先决条件

-  至少具有一个有效活动 Azure 订阅的 Microsoft 帐户。如果你还没有 Azure 订阅，可以在[试用 Azure](/pricing/1rmb-trial/) 中注册一个免费试用版。 

如果你要使用本教程来配置针对你组织自定义的可行的跨界虚拟网络，你需要满足以下条件：

-  虚拟网络及其子网的专用 IPv4 地址空间（采用 CIDR 表示法）。

-  本地 DNS 服务器的名称和 IP 地址。

-  使用公共 IPv4 地址的 VPN 设备。你需要 IP 地址才能完成向导。VPN 设备不能位于网络地址转换器 (NAT) 的后面，并且必须满足最低的设备标准。有关更多信息，请参阅[关于用于虚拟网络的 VPN 设备](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)。

	注意：可以使用 Windows Server 中的路由和远程访问服务 (RRAS) 作为 VPN 解决方案的一部分。但是，本教程不演练 RRAS 配置步骤。有关 RRAS 配置信息，请参阅[路由和远程访问服务模板](http://msdn.microsoft.com/zh-cn/library/azure/dn133801.aspx)。

-  有配置用于 IPsec 隧道模式连接的路由器方面的经验，或者其他某人可帮助你完成此步骤。

-  汇总了本地网络（也称为局域网）可访问位置的地址空间（采用 CIDR 表示法）的集。


## 高级步骤

1.	[创建虚拟网络](#CreateVN)

2.	[启动网关并为网络管理员收集信息](#StartGateway)

3.  [配置 VPN 设备](#ConfigVPN)

##  <a name="CreateVN"></a>创建虚拟网络

创建可连接到公司网络的示例虚拟网络：

1.	登录到 [Azure 管理门户](http://manage.windowsazure.cn/)。

2.	在屏幕左下角，单击“新建”。在导航窗格中，单击“网络”，然后单击“虚拟网络”。单击“自定义创建”以启动配置向导。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_01_OpenVirtualNetworkWizard.png)

3.	在“虚拟网络详细信息”页上，输入以下信息，然后单击右下角的“下一步”箭头。有关详细信息页上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络](/documentation/articles/virtual-networks-settings)中的**虚拟网络详细信息**部分。

	-  **名称：**为虚拟网络命名。对于本教程中的示例，请键入 **YourVirtualNetwork**。

	-  **区域：**从下拉列表中，选择所需的区域。将在位于指定区域内的 Azure 数据中心创建你的虚拟网络。

	
4.	在“DNS 服务器和 VPN 连接”页面上，输入以下信息，然后单击右下方的向前箭头。

> [AZURE.NOTE]可以在此页面上同时选择“点到站点”和“站点到站点”两种配置。在本教程中，我们选择只配置“站点到站点”。有关此页上各设置的更多信息，请参阅[关于使用管理门户配置虚拟网络](/documentation/articles/virtual-networks-settings)中的 **DNS 服务器和 VPN 连接**页。

	-  **DNS SERVERS:** Enter the DNS server name and IP address that you want to use for name resolution. Typically this would be a DNS server that you use for on-premises name resolution. This setting does not create a DNS server. For the example in this tutorial, type **YourDNS** for the name and **10.1.0.4** for the IP address.
	-  **Configure Point-To-Site VPN:** Leave this field blank. 
	-  **Configure Site-To-Site VPN:** Select checkbox.
	-  **LOCAL NETWORK:** Select **Specify a New Local Network** from the drop-down list.
 
	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_03_DNSServersandVPNConnectivity.png)

5.	在“站点到站点连接”页上输入以下信息，然后单击页面右下角的复选标记。<!--For more information about the settings on this page, see the **Site-to-Site Connectivity** page section in [About Configuring a Virtual Network using the Management Portal](http://msdn.microsoft.com/zh-cn/library/azure/jj156074.aspx). -->

	-  **名称：**对于本教程中的示例，请键入 **YourCorpHQ**。

	-  **VPN 设备 IP 地址：**对于本教程中的示例，请键入 **3.2.1.1**。否则，请输入 VPN 设备的公共 IP 地址。如果你没有此信息，则需要先获取此信息，然后才能继续在向导中执行后续步骤。请注意，VPN 设备不能在 NAT 的后面。有关 VPN 设备的详细信息，请参阅[关于用于虚拟网络连接的 VPN 设备](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)。

	-  **地址空间：**对于本教程中的示例，请键入 **10.1.0.0/16**。
	-  **添加地址空间：**本教程不需要额外的地址空间。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_04_SitetoSite.png)

6.  在“虚拟网络地址空间”页上，输入以下信息，然后单击右下方的复选标记以配置你的网络。

	地址空间必须是 10.0.0.0/8、172.16.0.0/12 或 192.168.0.0/16（由 RFC 1918 指定）中用 CIDR 表示法指定的专用地址范围。<!--有关此页面上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络](http://msdn.microsoft.com/zh-cn/library/azure/jj156074.aspx)中的**虚拟网络地址空间页**。-->

	-  **地址空间：**对于本教程中的示例，请单击右上角的“CIDR”，然后输入以下内容：
		-  **起始 IP：**10.4.0.0
		-  **CIDR：**/16
	-  **添加子网：**对于本教程中的示例，请输入以下内容：
		-  将 **Subnet-1** 重命名为起始 IP 为 **10.4.2.0/24** 的 **FrontEndSubnet**。
		-  使用起始 IP **10.4.3.0/24** 添加名为 **BackEndSubnet** 的子网。
		-  使用起始 IP **10.4.4.0/24** 添加名为 **ADDNSSubnet** 的子网。
		-  使用起始 IP **10.4.1.0/24** 添加网关子网。
	-  对于本教程中的示例，确认你现在创建了三个子网和一个网关子网，然后单击右下角的复选标记以创建虚拟网络。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_05_VirtualNetworkAddressSpaces.png)

7.	在单击复选标记后，将开始创建虚拟网络。创建虚拟网络后，在管理门户的网络页上，你将看到“状态”下面列出了“已创建”。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_06_VirtualNetworkCreatedStatus.png)

##  <a name="StartGateway"></a>启动网关

创建 Azure 虚拟网络之后，使用以下步骤配置虚拟网络网关以创建站点到站点 VPN。此过程要求你具有满足最低要求的 VPN 设备。有关 VPN 设备和设置配置的详细信息，请参阅[关于用于虚拟网络的 VPN 设备](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)。

**启动网关：**

1.	在虚拟网络创建完成时，“网络”页面将显示虚拟网络状态“已创建”。

	在“名称”列中，单击“YourVirtualNetwork”（在本教程中创建的示例）打开仪表板。
 
	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_07_ClickYourVirtualNetwork.png)

2.	单击页面顶部的“仪表板”。在“仪表板”页的底部，单击“创建网关”。针对你要创建的网关类型，选择“动态路由”或“静态路由”。

	请注意，如果你要使用此虚拟网络进行点到站点以及站点到站点的连接，则必须选择“动态路由”作为网关类型。在创建网关之前，验证你的 VPN 设备将支持你要创建的网关类型。请参阅[关于用于虚拟网络的 VPN 设备](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)。系统提示你确认要创建网关时，单击“是”。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_08_CreateGateway.png)

3.	当开始网关创建时，你将看到以下消息，让你知道你网关已启动。

	创建完网关可能需要最多 15 分钟。

4.	创建网关后，你需要收集将用于配置 VPN 设备的下列信息。

	-  网关 IP 地址
	-  共享密钥
	-  VPN 设备配置脚本模板

	后续步骤将指导你完成此过程。

5.	查找网关 IP 地址：网关 IP 地址位于虚拟网络的“仪表板”页上。下面是一个示例：

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_09_GatewayIP.png)

6.	获取共享密钥：共享密钥位于虚拟网络的“仪表板”页上。单击屏幕底部的“管理密钥”，然后复制显示在对话框中的密钥。你需要使用此密钥在公司的 VPN 设备上配置 IPsec 隧道。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_10_ManageSharedKey.png)

7.	若要下载 VPN 设备配置脚本模板，请在仪表板上，单击“下载 VPN 设备脚本”。

8.	在“下载 VPN 设备配置脚本”对话框中，选择公司 VPN 设备的供应商、平台和操作系统。单击复选标记按钮，保存文件。

	![](./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_11_DownloadVPNDeviceScript.png)

如果在下拉列表中看不到你的 VPN 设备，请参阅 MSDN 库中的[关于用于虚拟网络的 VPN 设备](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)以获取其他脚本模板。


##  <a name="ConfigVPN"></a>配置 VPN 设备（网络管理员）

因为每个 VPN 设备都不同，所以这只是一个概要的过程。此过程应由网络管理员执行。

你可以从管理门户或从[关于用于虚拟网络的 VPN 设备](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)中获取 VPN 配置脚本，其中还解释了路由类型以及与你选择要使用的路由配置兼容的设备。

<!--For additional information about configuring a virtual network gateway, see [Configure the Virtual Network Gateway in the Management Portal](http://msdn.microsoft.com/zh-cn/library/azure/jj156210) and consult your VPN device documentation.-->

此过程假定：

-  配置 VPN 设备的人擅长配置选定的设备。由于与虚拟网络兼容的设备数量以及特定于每个设备系列的配置非常多，因此，这些步骤并没有详细演练设备配置。因此，配置设备的人员务必熟悉设备及其配置设置。 

-  你选择要用的设备与虚拟网络兼容。查看[此处](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)以了解设备兼容性。


**配置 VPN 设备：**

1.	修改 VPN 配置脚本。你将配置：

	a.安全策略

	b.传入隧道

	c.传出隧道

2.	运行修改后的 VPN 配置脚本来配置 VPN 设备。

3.	通过运行下列命令之一来测试连接：

<table border="1">
<tr>
<th>-</th>
<th>Cisco ASA</th>
<th>Cisco ISR/ASR</th>
<th>Juniper SSG/ISG</th>
<th>Juniper SRX/J</th>
</tr>

<tr>
<td><b>检查主模式 SA</b></td>
<td><FONT FACE="courier" SIZE="-1">show crypto isakmp sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show crypto isakmp sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">get ike cookie</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show security ike security-association</FONT></td>
</tr>

<tr>
<td><b>检查快速模式 SA</b></td>
<td><FONT FACE="courier" SIZE="-1">show crypto ipsec sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show crypto ipsec sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">get sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show security ipsec security-association</FONT></td>
</tr>
</table>


##  后续步骤
若要将本地 Active Directory 扩展到你刚创建的虚拟网络，请继续学习以下教程：

-  [如何创建自定义虚拟机](/documentation/articles/virtual-machines-create-custom)

-  [在 Azure 虚拟网络中安装 Active Directory 域控制器副本](/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller)

如果你要将虚拟网络设置导出到网络配置文件以便备份配置或将其用作模板，请参阅[将虚拟网络设置导出到网络配置文件](/documentation/articles/virtual-networks-create-vnet-classic-portal)。

## 另请参阅

-  [Azure 虚拟网络](/documentation/articles/virtual-networks-overview)

-  [虚拟网络常见问题解答](/documentation/articles/virtual-networks-faq)

-  [使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-create-vnet-classic-portal)

-  [将虚拟机添加到虚拟网络](/documentation/articles/virtual-machines-create-custom)

-  [有关虚拟网络的 VPN 设备（可能为英文页面）](http://msdn.microsoft.com/zh-cn/library/azure/jj156075.aspx)

<!---  [Azure Name Resolution Overview](http://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx)-->

<!---HONumber=70-->