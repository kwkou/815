<properties 
   pageTitle="如何创建虚拟网络 (VNet)"
   description="了解如何创建虚拟网络 (VNet)"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn"/>
<tags 
   ms.service="virtual-network"
   ms.date="06/08/2015"
   wacn.date="08/01/2015"/>

# 如何创建虚拟网络 (VNet)

创建 VNet 后，VNet 中的服务和 VM 不必通过 Internet 就能安全地互相通信。如果不希望 VNet 连接到其他 Vnet 或你的本地网络，则创建 Azure VNet 的过程相对较为快速轻松，因为你不需要获取和配置 VPN 设备，或者与其他 Vnet 或局域网协调你选择的 IP 地址。

>[AZURE.WARNING]请不要使用此过程来创建以后将连接到其他 VNet 或本地网络的 VNet。如果要创建安全的跨界或混合连接，请参阅[关于虚拟网络安全跨界连接](https://msdn.microsoft.com/zh-cn/library/azure/dn133798.aspx)。<!--如果要创建与另一个 VNet 连接的 VNet，请参阅[配置 VNet 到 VNet 连接](https://msdn.microsoft.com/zh-cn/library/azure/dn690122.aspx)。-->

## 配置 VNet

1. 登录到 **Azure 管理门户**。

2. 在屏幕左下角，单击“新建”。在导航窗格中，单击“网络服务”，然后单击“虚拟网络”。单击“自定义创建”以启动配置向导。

3. 在“虚拟网络详细信息”页上，输入以下信息：

	![虚拟网络详细信息](./media/virtual-networks-create-vnet/IC736054.png)

	- **名称** - 为你的 VNet 命名。例如，我们创建了名称 *ChinaNorthVNet*。你可以根据自己的喜好创建任何名称。部署你的 VM 和服务时，将使用此 VNet 名称，因此最好不要让名称太复杂。

	- **位置** - 从下拉列表中选择位置（区域）。该位置直接与你在将资源 (VM) 部署到此 VNet 时希望这些资源 (VM) 驻留到的物理位置有关。例如，如果你希望 VM 的物理位置位于*中国北部*，请选择该位置区域。创建 VNet 后，将无法更改与它关联的区域。

4. 在“DNS 服务器和 VPN 连接”页上，不要进行任何更改。只需通过单击箭头向前移动到下一页。默认情况下，Azure 将为你的 VNet 提供基本名称解析。你的名称解析要求的复杂度可能会超过基本 Azure 名称解析可以处理的范围。在这种情况下，你以后可能会想要将运行 DNS 的虚拟机添加到 VNet 中。<!--有关 Azure 名称解析和 DNS 的详细信息，请参阅[名称解析 (DNS)](https://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx)。-->

5. 在“虚拟网络地址空间”页上，你将输入要用于此 VNet 的地址空间。除非你需要将特定内部 IP 地址范围用于你的 VM 或者要为 VM 创建将接收静态 DIP 的特定子网，否则不需要在此页上进行任何更改。如果你确实想要创建多个子网，则可以在此页上通过单击“添加子网”来执行该操作。

	其他信息：

	- 此页上的范围包含动态内部 IP 地址 (DIP)，当你将 VM 部署到此 VNet 时，VM 将收到该 DIP。这些 IP 地址只用于 VNet 内部通信。它们不是用于 Internet 终结点的 IP 地址。

	- 由于你不会使用跨界 VPN 配置将此专用 VNet 连接到你的本地网络，因此你不需要将这些设置与现有的本地网络 IP 地址范围进行协调。如果你考虑以后可能需要创建跨界配置，则现在需要将地址空间与本地站点中已存在的地址范围进行协调，以避免出现路由问题。以后更改范围可能会有点复杂，特别是如果你存在地址范围重叠的情况。

	![地址空间](./media/virtual-networks-create-vnet/IC716778.png)

6. 单击“虚拟网络地址空间”页右下角的复选标记，此时将会创建你的 VNet。创建 VNet 后，在管理门户的“网络”页上，你将看到“状态”下面列出了“已创建”。

7. 创建 VNet 后，你便可以在 VNet 中创建 VM 和 PaaS 实例。在创建新 VM 时请务必选择“从库中”，以便显示选择 VNet 的选项。请注意，如果你已部署现有 VM 和 PaaS 实例，则不能直接将它们移到新的虚拟网络。这是因为它们所需的网络配置设置是在部署过程中添加的。

## 将虚拟机添加到 VNet

创建 VNet 后，你可以向其添加新的 VM。请务必先创建 VNet，然后再部署 VM。在部署 VM 后，将无法在未重新部署它的情况下将它移到 VNet。如果使用管理门户来创建 VM，仅当选择“新建”/“计算”/虚拟机”/“从库中”时，将 VM 部署到 VNet 的界面才可用。当你完成用于创建 VM 的向导后，将在“虚拟机配置”页上看到“区域/地缘组/虚拟网络”。从下拉列表中，选择已创建的 VNet。有关创建虚拟机的详细信息，请参阅 [Azure 虚拟机](/documentation/articles/virtual-machines)。

## 后续步骤

[如何管理虚拟网络 (VNet) 属性](/documentation/articles/virtual-networks-settings)

[如何管理虚拟网络 (VNet) 使用的 DNS 服务器](/documentation/articles/virtual-networks-manage-dns-in-vnet)

[如何在虚拟网络中使用公共 IP 地址](/documentation/articles/virtual-networks-public-ip-within-vnet)

[如何删除虚拟网络 (VNet)](/documentation/articles/virtual-networks-delete-vnet)
 

<!---HONumber=64-->