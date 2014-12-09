<properties linkid="manage-services-create-a-virtual-network" urlDisplayName="Create a virtual network" pageTitle="Create a virtual network - Azure service management" metaKeywords="" description="Learn how to create an Azure Virtual Network." metaCanonical="" services="virtual-machines,virtual-network" documentationCenter="" title="Create a Virtual Network in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中创建虚拟网络

本教程将指导你完成使用 Azure 管理门户创建基本 Azure 虚拟网络的步骤。有关 Azure 虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述][Azure 虚拟网络概述]。

本教程假定你之前未使用过 Azure。其目的是帮助你熟悉创建虚拟网络所需的步骤。如果你要查找有关虚拟网络的设计方案和高级信息，请参见 [Azure 虚拟网络概述][Azure 虚拟网络概述]。

完成本教程后，你将拥有一个虚拟网络，你可将 Azure 服务和虚拟机部署到该网络。

<div class="dev-callout"> 
<b>说明</b> 
<p>本教程不演练创建跨界配置的过程。有关演练创建虚拟网络进行站点到站点跨界连接（即连接到位于你公司内的 Active Directory 或 SharePoint）的教程，请参阅<a href="/zh-cn/manage/services/networking/cross-premises-connectivity/">创建虚拟网络进行跨界连接</a>。</p> 
</div>

有关其他虚拟网络配置过程和设置，请参阅 [Azure 虚拟网络配置任务][Azure 虚拟网络配置任务]。

有关在 Azure 虚拟机上部署 AD DS 的指南，请参阅[在 Azure 虚拟机中部署 Windows Server Active Directory 的准则][在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]。

## 目标

在本教程中，你将学习：

-   如何设置可将 Azure 云服务和虚拟机添加到其中的基本 Azure 虚拟网络。

## 先决条件

-   至少具有一个有效的活动订阅的 Windows Live 帐户。

## 创建虚拟网络

**创建仅包含云的虚拟网络：**

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  在屏幕左下角，单击“新建”。在导航窗格中，单击“网络”，然后单击“虚拟网络”。单击“自定义创建”以启动配置向导。

    ![][0]

3.  在“虚拟网络详细信息”页上，输入以下信息，然后单击右下角的“下一步”箭头。有关详细信息页上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络][关于使用管理门户配置虚拟网络]中的“虚拟网络详细信息页”部分。

-   **名称 -** 为虚拟网络命名。键入 *YourVirtualNetwork*。

-   **地缘组 -**从下拉列表中，选择“创建新的地缘组”。地缘组是一种用于在同一数据中心以物理方式将 Azure 服务组合起来以提高性能的方法。只能向一个虚拟网络分配地缘组。

-   **区域 -** 从下拉列表中，选择所需的区域。将在位于指定区域内的数据中心创建你的虚拟网络。

-   **地缘组名称 -** 为新地缘组命名。键入 *YourAffinityGroup*。

    ![][1]

1.  在“DNS 服务器和 VPN 连接”页上，输入以下信息，然后单击右下角的“下一步”箭头。有关此页上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络][关于使用管理门户配置虚拟网络]中的“DNS 服务器和 VPN 连接”页。

    -   **DNS 服务器-可选 -** 输入要使用的 DNS 服务器名称和 IP 地址。此设置不创建 DNS 服务器，而是引用已存在的 DNS 服务器。

        <div class="dev-callout"> 
<b>说明</b> 
<p>若要使用公共 DNS 服务，你可以在此屏幕上输入该信息。否则，名称解析将默认为 Azure 服务。有关详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx">Azure 名称解析概述</a>。</p>
</div>

    -   **请勿选中对应于点到站点连接或站点到站点连接的复选框**。在本教程中创建的虚拟网络并不用于进行跨界连接。

    ![][2]

2.  在“虚拟网络地址空间”页上，输入以下信息，然后单击右下角的复选框以配置网络。地址空间必须为用 CIDR 表示法指定的专用地址范围：10.0.0.0/8、172.16.0.0/12 或 192.168.0.0/16（由 RFC 1918 指定）。有关此页面上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络][关于使用管理门户配置虚拟网络]中的“虚拟网络地址空间”页。

    -   **地址空间：**单击右上角的“CIDR”，然后输入以下信息：

        -   **起始 IP：** 10.4.0.0

        -   **CIDR：** /16

    -   **添加子网：**输入以下信息：

        -   将 Subnet-1 重命名为起始 IP 为 *10.4.2.0/24* 的 *FrontEndSubnet*，然后单击“添加子网”。

        -   创建起始 IP 为 *10.4.3.0/24* 的 *BackEndSubnet*。

        -   验证你现在已创建了两个子网，然后单击右下角的复选标记以创建虚拟网络。

    ![][3]

3.  在单击复选标记后，将开始创建虚拟网络。创建虚拟网络后，在管理门户的网络页上，你将看到“状态”下面列出了“已创建”。

    ![][4]

4.  创建虚拟网络后，你可以继续学习以下教程：

    -   [将虚拟机添加到虚拟网络][将虚拟机添加到虚拟网络] – 使用此基本教程可将虚拟机安装到虚拟网络。

    -   有关虚拟机和安装选项的详细信息，请参阅[如何创建自定义虚拟机][如何创建自定义虚拟机]和 [Azure 虚拟机][Azure 虚拟机]。

    -   [在 Azure 中安装新的 Active Directory 林][在 Azure 中安装新的 Active Directory 林] - 使用本教程可在不连接任何其他网络的情况下安装新的 Active Directory 林。本教程将介绍创建虚拟机 (VM) 以安装新林所需的特定步骤。如果你计划使用本教程，请不要通过管理门户创建任何虚拟机。

## 另请参阅

-   [Azure 虚拟网络概述][Azure 虚拟网络概述]

-   [Azure 虚拟网络常见问题解答][Azure 虚拟网络常见问题解答]

-   [Azure 虚拟网络配置任务][Azure 虚拟网络配置任务]

-   [使用网络配置文件配置虚拟网络（可能为英文页面）][使用网络配置文件配置虚拟网络（可能为英文页面）]

-   [Azure 名称解析概述][Azure 名称解析概述]

  [Azure 虚拟网络概述]: http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx
  [创建虚拟网络进行跨界连接]: /zh-cn/manage/services/networking/cross-premises-connectivity/
  [Azure 虚拟网络配置任务]: http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx
  [在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]: http://msdn.microsoft.com/zh-cn/library/azure/jj156090.aspx
  [Azure 管理门户]: http://manage.windowsazure.cn/
  [0]: ./media/create-virtual-network/createVNet_01_OpenVirtualNetworkWizard.png
  [关于使用管理门户配置虚拟网络]: http://msdn.microsoft.com/zh-cn/library/azure/jj156074.aspx
  [1]: ./media/create-virtual-network/createVNet_02_VirtualNetworkDetails.png
  [Azure 名称解析概述]: http://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx
  [2]: ./media/create-virtual-network/createVNet_03_DNSServersandVPNConnectivity.png
  [3]: ./media/create-virtual-network/createVNet_04_VirtualNetworkAddressSpaces.png
  [4]: ./media/create-virtual-network/createVNet_05_VirtualNetworkCreatedStatus.png
  [将虚拟机添加到虚拟网络]: /zh-cn/manage/services/networking/add-a-vm-to-a-virtual-network/
  [如何创建自定义虚拟机]: /zh-cn/manage/windows/how-to-guides/custom-create-a-vm/
  [Azure 虚拟机]: /zh-cn/manage/windows/
  [在 Azure 中安装新的 Active Directory 林]: /zh-cn/manage/services/networking/active-directory-forest/
  [Azure 虚拟网络常见问题解答]: http://msdn.microsoft.com/zh-cn/library/azure/dn133803.aspx
  [使用网络配置文件配置虚拟网络（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/azure/jj156097.aspx
