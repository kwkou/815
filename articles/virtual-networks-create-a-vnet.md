<properties 
   pageTitle="创建虚拟网络" 
   description="指导你完成相关步骤以轻松创建基本虚拟网络。" 
   services="virtual-network" 
   documentationCenter="" 
   authors="cherylmc" 
   manager="adinah" 
   editor="tysonn"/>
<tags ms.service="virtual-network"
    ms.date="04/14/2015"
    wacn.date="04/15/2015"
    />

# 创建虚拟网络 



在创建虚拟网络时，虚拟网络中的服务和 VM 可以安全地相互通信而无需通过 Internet 发布。创建仅限云的专用虚拟网络是一个相对快速而简单的过程。由于仅限云的虚拟网络不适用于跨界连接，因此你不需要获取和配置 VPN 设备或身份验证证书。 

创建虚拟网络后，你可以向其添加新的 VM 和 PaaS 实例。请注意，如果你使用管理门户来创建 VM，请务必选择**"从库中"**，以便可以指定虚拟网络。这很重要，因为在创建虚拟机之后，你不能返回将 VM 放在虚拟网络中。

[Azure Note] **使用此过程来创建仅限云的专用虚拟网络。**由于创建跨界配置涉及更高的复杂性，因此不要使用此过程来创建以后要连接到本地网络的虚拟网络。如果要在 Azure 与本地网络之间创建安全跨界连接，请参阅[关于安全跨界连接](https://msdn.microsoft.com/zh-cn/library/azure/dn133798.aspx)。

## <a name="CreateyourVNet">创建虚拟网络</a>

1. 登录到**管理门户**。
2. 在屏幕左下角，单击**"新建"**。在导航窗格中，单击**"网络服务"**，然后单击 **"虚拟网络"**。单击**"自定义创建"**以启动配置向导。
3. 在**"虚拟网络详细信息"**页上，输入以下信息，然后单击右下角的"下一步"箭头。有关详细信息页上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络](https://msdn.microsoft.com/zh-cn/library/azure/jj156074.aspx)中的**虚拟网络详细信息**部分。
	-  **名称：**为虚拟网络命名。部署 VM 和服务时将使用此虚拟网络名称，因此最好不要让此名称太复杂。

	-  **位置:**从下拉列表中选择所需的区域。将在指定区域中的 Azure 数据中心创建你的虚拟网络。



4. 在**"DNS 服务器和 VPN 连接"**页上，不要进行任何更改。只需通过单击箭头向前移动到下一页。默认情况下，Azure 将为你的虚拟网络提供基本名称解析。你的名称解析要求的复杂度可能会超过基本 Azure 名称解析可以处理的范围。在这种情况下，你以后可能会想要将运行 DNS 的虚拟机添加到虚拟网络中。有关 Azure 名称解析和 DNS 的详细信息，请参阅[名称解析](https://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx)。 
5. 在**"虚拟网络地址空间"**页上，你将输入要用于此虚拟网络的地址空间。除非你需要将特定内部 IP 地址范围用于你的 VM 或者要为 VM 创建将接收静态 DIP 的特定子网，否则不需要在此页上进行任何更改。如果你确实想要创建多个子网，则可以在此页上通过单击 **"添加子网"** 来执行该操作。有关详细信息页上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络](https://msdn.microsoft.com/zh-cn/library/azure/jj156074.aspx)中的**虚拟网络详细信息**部分。

	-  有关详细信息页上各项设置的详细信息，请参阅[关于使用管理门户配置虚拟网络](https://msdn.microsoft.com/zh-cn/library/azure/jj156074.aspx)中的**虚拟网络详细信息**部分。
	-  由于你不会使用跨界 VPN 配置将此专用虚拟网络连接到你的本地网络，因此你不需要将这些设置与现有的本地网络 IP 地址范围进行协调。如果你考虑以后可能需要创建跨界配置，则现在需要将地址空间与本地站点中已存在的地址范围进行协调，以避免出现路由问题。以后更改范围可能会有点复杂，并且通常会导致必须重新部署


6. 单击"虚拟网络地址空间"页右下角的复选标记，此时将开始创建你的虚拟网络。创建虚拟网络后，在管理门户的网络页上，你将看到"状态"下面列出了"已创建"。
7. 创建虚拟网络后，便可以部署到你的虚拟网络。例如，如果要将 VM 部署到虚拟网络，请参阅[将虚拟机添加到虚拟网络](/documentation/articles/virtual-machines-create-custom)。在创建新 VM 时请务必选择**"从库中"**，以便显示选择虚拟网络的选项。请注意，如果你已部署现有 VM 和 PaaS 实例，则不能直接将它们移到新的虚拟网络。这是因为在部署过程中已为它们配置网络配置设置。你必须重新将它们部署到新的虚拟网络。



## 后续步骤
-  [虚拟网络技术概述](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156007.aspx)

 
-  [将虚拟机添加到虚拟网络](/documentation/articles/virtual-machines-create-custom)

-  [虚拟网络常见问题解答](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn133803.aspx)

-  [使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file)

-  [Azure 名称解析概述](https://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx)
 



<!--HONumber=50-->