<properties linkid="manage-services-cross-premises-connectivity" urlDisplayName="Cross-premises Connectivity" pageTitle="Create a cross-premises virtual network - Azure" metaKeywords="" description="Learn how to create an Azure Virtual Network with cross-premises connectivity." metaCanonical="" services="virtual-network" documentationCenter="" title="Create a Virtual Network for Site-to-Site Cross-Premises Connectivity" authors="" solutions="" manager="" editor="" />

# 创建虚拟网络进行站点到站点跨界连接

本教程将指导你完成创建跨界虚拟网络的步骤。我们将创建的连接类型是站点到站点连接。如果你要使用证书和 VPN 客户端创建点到站点 VPN，请参阅[在管理门户中配置点到站点 VPN][在管理门户中配置点到站点 VPN]。

本教程假定你之前未使用过 Azure。其目的是帮助你熟悉创建站点到站点虚拟网络所需的步骤。如果你要查找有关虚拟网络的设计方案和高级信息，请参阅 [Azure 虚拟网络概述][Azure 虚拟网络概述]。

完成本教程后，你将拥有一个虚拟网络，可在其中部署 Azure 服务和虚拟机，然后它们可以直接与公司网络进行通信。

有关添加虚拟机并将本地 Active Directory 扩展到 Azure 虚拟网络的信息，请参阅以下内容：

-   [如何创建自定义虚拟机][如何创建自定义虚拟机]

-   [在 Azure 虚拟网络中安装 Active Directory 域控制器副本][在 Azure 虚拟网络中安装 Active Directory 域控制器副本]

有关在 Azure 虚拟机上部署 AD DS 的指南，请参阅[在 Azure 虚拟机中部署 Windows Server Active Directory 的准则][在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]。

有关其他虚拟网络配置过程和设置，请参阅 [Azure 虚拟网络配置任务][Azure 虚拟网络配置任务]。

## 目标

在本教程中，你将学习：

-   如何设置可将 Azure 服务添加到的基本 Azure 虚拟网络。

-   如何将虚拟网络配置为与公司网络进行通信。

## 先决条件

-   至少具有一个有效的活动订阅的 Windows Live 帐户。

-   要用于虚拟网络和子网的地址空间（使用 CIDR 表示法）。

-   DNS 服务器的名称和 IP 地址（若要使用本地 DNS 服务器进行名称解析）。

-   使用公共 IPv4 地址的 VPN 设备。你需要 IP 地址才能完成向导。VPN 设备不能位于 NAT 的后面，并且必须满足最低的设备标准。有关更多信息，请参阅[关于用于虚拟网络的 VPN 设备][关于用于虚拟网络的 VPN 设备]。

    注意：你可以将 RRAS 用作 VPN 解决方案的一部分。但是，本教程不演练 RRAS 配置步骤。

    有关 RRAS 配置信息，请参阅[路由和远程访问服务模板][路由和远程访问服务模板]。

-   有关配置路由器的经验或可帮助你执行此步骤的人员。

-   本地网络的地址空间。

## 高级步骤

1.  [创建虚拟网络][创建虚拟网络]

2.  [启动网关并为网络管理员收集信息][启动网关并为网络管理员收集信息]

3.  [配置 VPN 设备][配置 VPN 设备]

## <a name="CreateVN">创建虚拟网络</a>

**创建连接到公司网络的虚拟网络：**

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  在屏幕左下角，单击“新建”。在导航窗格中，单击“网络”，然后单击“虚拟网络”。单击“自定义创建”以启动配置向导。

    ![][0]

3.  在“虚拟网络详细信息”页上，输入以下信息，然后单击右下角的“下一步”箭头。有关此页面上各项设置的详细信息，请参阅[使用管理门户配置虚拟网络][使用管理门户配置虚拟网络]中的“虚拟网络详细信息”一节。

-   **名称：**为虚拟网络命名。键入 *YourVirtualNetwork*。

-   **地缘组：**从下拉列表中，选择“创建新的地缘组”。地缘组是一种用于在同一数据中心以物理方式将 Azure 服务组合起来以提高性能的方法。只能向一个虚拟网络分配地缘组。

-   **区域：**从下拉列表中，选择所需的区域。将在位于指定区域内的数据中心创建你的虚拟网络。

-   **地缘组名称：**为新建地缘组命名。键入 *YourAffinityGroup*。

    ![][1]

1.  在“DNS 服务器和 VPN 连接”页面上，输入以下信息，然后单击右下方的向前箭头。

	<div class="dev-callout"> 
	<b>说明</b> 
	<p>可以在此页面上同时选择 **点到站点** 和 **站点到站点** 两种配置。在本教程中，我们选择只配置&ldquo;站点到站点&rdquo;。有关此页上各设置的更多信息，请参阅<a href="http://go.microsoft.com/fwlink/?LinkID=248092">关于使用管理门户配置虚拟网络</a>中的&ldquo;DNS 服务器和 VPN 连接&rdquo;页面。</p> 
	</div>

-   **DNS 服务器：**输入要用于名称解析的 DNS 服务器名称和 IP 地址。通常，这是你将用于内部部署的名称解析的 DNS 服务器。此设置不创建 DNS 服务器。为名称键入 *YourDNS*，为 IP 地址键入 *10.1.0.4*。
-   **配置点到站点 VPN：**此字段保留为空白。
-   **配置站点到站点 VPN：**选中复选框。
-   **本地网络：**从下拉列表中选择“指定新的本地网络”。

    ![][2]

1.  在“站点到站点连接”页上输入以下信息，然后单击页面右下角的复选标记。有关此页上各设置的更多信息，请参阅[关于使用管理门户配置虚拟网络][使用管理门户配置虚拟网络]中的“站点到站点连接”页面部分。

-   **名称：**键入 *YourCorpHQ*。

-   **VPN 设备 IP 地址：**输入 VPN 设备的公共 IP 地址。如果你没有此信息，则需要先获取此信息，然后才能继续在向导中执行后续步骤。注意，你的 VPN 设备不能位于 NAT 后面。有关 VPN 设备的更多信息，请参阅[有关虚拟网络的 VPN 设备][有关虚拟网络的 VPN 设备]。

-   **地址空间：**键入 *10.1.0.0/16*。
-   **添加地址空间：**本教程不需要额外的地址空间。

    ![][3]

1.  在“虚拟网络地址空间”页上，输入以下信息，然后单击右下方的复选标记以配置你的网络。

    地址空间必须为用 CIDR 表示法指定的专用地址范围：10.0.0.0/8、172.16.0.0/12 或 192.168.0.0/16（由 RFC 1918 指定）。有关此页面上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络][使用管理门户配置虚拟网络]中的“虚拟网络地址空间”页。

-   **地址空间：**单击右上角的“CIDR”，然后输入以下信息：

    -   **起始 IP：** 10.4.0.0
    -   **CIDR：** /16
-   **添加子网：**输入以下信息：

    -   将 Subnet-1 重命名为起始 IP 为 *10.4.2.0/24* 的 *FrontEndSubnet*，然后单击“添加子网”。
    -   使用起始 IP *10.4.3.0/24* 添加名为 *BackEndSubnet* 的子网。
    -   使用起始 IP *10.4.4.0/24* 添加名为 *ADDNSSubnet* 的子网。
    -   使用起始 IP *10.4.1.0/24* 添加网关子网。
    -   **验证**你现在创建了三个子网和一个网关子网，然后单击右下角的复选标记以创建虚拟网络。

    ![][4]

1.  在单击复选标记后，将开始创建虚拟网络。创建虚拟网络后，在管理门户的网络页上，你将看到“状态”下面列出了“已创建”。

    ![][5]

## <a name="StartGateway">启动网关</a>

创建 Azure 虚拟网络之后，使用以下步骤配置虚拟网络网关以创建站点到站点 VPN。此过程要求你具有满足最低要求的 VPN 设备。有关 VPN 设备和设置配置的详细信息，请参阅[关于用于虚拟网络的 VPN 设备][关于用于虚拟网络的 VPN 设备]。

**启动网关：**

1.  在虚拟网络创建完成时，“网络”页面将显示虚拟网络状态**“已创建”**。

    在**“名称”**列中，单击**“YourVirtualNetwork”**打开仪表板。

    ![][6]

2.  单击页面顶部的“仪表板”。在“仪表板”页的底部，单击“创建网关”。针对你要创建的网关类型，选择“动态路由”或“静态路由”。

    请注意，如果你要使用此虚拟网络进行点到站点以及站点到站点的连接，则必须选择“动态路由”作为网关类型。在创建网关之前，验证你的 VPN 设备将支持你要创建的网关类型。请参阅[有关虚拟网络的 VPN 设备][关于用于虚拟网络的 VPN 设备]。系统提示你确认要创建网关时，单击**“是”**。

    ![][7]

3.  当开始网关创建时，你将看到以下消息，让你知道你网关已启动。

    创建完网关可能需要最多 15 分钟。

4.  创建网关后，你需要收集将用于配置 VPN 设备的下列信息。

-   网关 IP 地址
-   共享密钥
-   VPN 设备配置脚本模板

    后续步骤将指导你完成此过程。

1.  查找网关 IP 地址 – 网关 IP 地址位于虚拟网络的“仪表板”页上。

    ![][8]

2.  获取共享密钥 – 共享密钥位于虚拟网络的“仪表板”页上。单击屏幕底部的“管理密钥”，然后复制显示在对话框中的密钥。

    ![][9]

3.  下载 VPN 设备配置脚本模板。在仪表板上，单击“下载 VPN 设备脚本”。

4.  在“下载 VPN 设备配置脚本”对话框中，选择贵公司 VPN 设备的供应商、平台和操作系统。单击复选标记按钮，保存文件。

    ![][10]

如果在下拉列表中看不到你的 VPN 设备，请参阅 MSDN 库中的[关于用于虚拟网络的 VPN 设备][关于用于虚拟网络的 VPN 设备]以获取其他脚本模板。

## <a name="ConfigVPN">配置 VPN 设备（网络管理员）</a>

因为每个 VPN 设备都不同，所以这只是一个概要的过程。此过程应由网络管理员执行。

你可以从管理门户或从[关于用于虚拟网络的 VPN 设备][11]获取 VPN 配置脚本，其中还解释了路由类型以及与你选择要使用的路由配置兼容的设备。

有关配置虚拟网络网关的更多信息，请参阅[在管理门户中配置虚拟网络网关][在管理门户中配置虚拟网络网关]并参考 VPN 设备文档。

此过程假定：

-   配置 VPN 设备的人擅长配置选定的设备。由于与虚拟网络兼容的设备数量以及特定于每个设备系列的配置非常多，因此，这些步骤并没有详细演练设备配置。因此，配置设备的人员务必熟悉设备及其配置设置。

-   你选择要用的设备与虚拟网络兼容。查看[此处][关于用于虚拟网络的 VPN 设备]以了解设备兼容性。

**配置 VPN 设备：**

1.  修改 VPN 配置脚本。你将配置：

    a. 安全策略

    b. 传入隧道

    c. 传出隧道

2.  运行修改后的 VPN 配置脚本来配置 VPN 设备。

3.  通过运行下列命令之一来测试连接：

<table border="1">
<tr>
<th>-</th>
<th>Cisco ASA</th>
<th>Cisco ISR/ASR</th>
<th>Juniper SSG/ISG</th>
<th>Juniper SRX/J</th>
</tr>

<tr>
<td><b>Check main mode SAs</b></td>
<td><FONT FACE="courier" SIZE="-1">show crypto isakmp sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show crypto isakmp sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">get ike cookie</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show security ike security-association</FONT></td>
</tr>

<tr>
<td><b>Check quick mode SAs</b></td>
<td><FONT FACE="courier" SIZE="-1">show crypto ipsec sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show crypto ipsec sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">get sa</FONT></td>
<td><FONT FACE="courier" SIZE="-1">show security ipsec security-association</FONT></td>
</tr>
</table>

## 后续步骤

若要将本地 Active Directory 扩展到你刚创建的虚拟网络，请继续学习以下教程：

-   [如何创建自定义虚拟机][如何创建自定义虚拟机]

-   [在 Azure 虚拟网络中安装 Active Directory 域控制器副本][在 Azure 虚拟网络中安装 Active Directory 域控制器副本]

如果你要将虚拟网络设置导出到网络配置文件以便备份配置或将其用作模板，请参阅[将虚拟网络设置导出到网络配置文件][将虚拟网络设置导出到网络配置文件]。

## 另请参阅

-   [Azure 虚拟网络][Azure 虚拟网络概述]

-   [虚拟网络常见问题解答][虚拟网络常见问题解答]

-   [使用网络配置文件配置虚拟网络（可能为英文页面）][使用网络配置文件配置虚拟网络（可能为英文页面）]

-   [将虚拟机添加到虚拟网络][将虚拟机添加到虚拟网络]

-   [有关虚拟网络的 VPN 设备（可能为英文页面）][有关虚拟网络的 VPN 设备]

-   [Azure 名称解析概述][Azure 名称解析概述]

  [在管理门户中配置点到站点 VPN]: http://go.microsoft.com/fwlink/?LinkId=296653
  [Azure 虚拟网络概述]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156007.aspx
  [如何创建自定义虚拟机]: http://go.microsoft.com/fwlink/?LinkID=294356
  [在 Azure 虚拟网络中安装 Active Directory 域控制器副本]: http://go.microsoft.com/fwlink/?LinkId=299877
  [在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156090.aspx
  [Azure 虚拟网络配置任务]: http://go.microsoft.com/fwlink/?LinkId=296652
  [关于用于虚拟网络的 VPN 设备]: http://go.microsoft.com/fwlink/?LinkID=248098
  [路由和远程访问服务模板]: http://msdn.microsoft.com/library/windowsazure/dn133801.aspx
  [创建虚拟网络]: #CreateVN
  [启动网关并为网络管理员收集信息]: #StartGateway
  [配置 VPN 设备]: #ConfigVPN
  [Azure 管理门户]: http://manage.windowsazure.com/
  [0]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_01_OpenVirtualNetworkWizard.png
  [使用管理门户配置虚拟网络]: http://go.microsoft.com/fwlink/?LinkID=248092
  [1]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_02_VirtualNetworkDetails.png
  [2]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_03_DNSServersandVPNConnectivity.png
  [有关虚拟网络的 VPN 设备]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156075.aspx
  [3]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_04_SitetoSite.png
  [4]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_05_VirtualNetworkAddressSpaces.png
  [5]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_06_VirtualNetworkCreatedStatus.png
  [6]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_07_ClickYourVirtualNetwork.png
  [7]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_08_CreateGateway.png
  [8]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_09_GatewayIP.png
  [9]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVNet_10_ManageSharedKey.png
  [10]: ./media/virtual-networks-create-site-to-site-cross-premises-connectivity/CreateCrossVnet_11_DownloadVPNDeviceScript.png
  [11]: http://go.microsoft.com/fwlink/?LinkId=248098
  [在管理门户中配置虚拟网络网关]: http://go.microsoft.com/fwlink/?LinkId=299878
  [将虚拟网络设置导出到网络配置文件]: http://go.microsoft.com/fwlink/?LinkID=299880
  [虚拟网络常见问题解答]: http://msdn.microsoft.com/library/windowsazure/dn133803.aspx
  [使用网络配置文件配置虚拟网络（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156097.aspx
  [将虚拟机添加到虚拟网络]: http://www.windowsazure.com/zh-cn/manage/services/networking/add-a-vm-to-a-virtual-network/
  [Azure 名称解析概述]: http://go.microsoft.com/fwlink/?LinkId=248097
