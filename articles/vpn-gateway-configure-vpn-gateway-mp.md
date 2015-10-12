<properties 
   pageTitle="在管理门户中配置 VPN 网关 | Windows Azure"
   description="本文指导您配置虚拟网络 VPN 网关，以及从静态到动态或从动态到静态更改 VPN 网关路由类型。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="jdial"
   editor="tysonn" />
<tags 
   ms.service="vpn-gateway"
   ms.date="06/12/2015"
   wacn.date="09/02/2015" />

# 在管理门户中配置 VPN 网关

如果要在 Azure 和本地位置之间创建一个安全跨界连接，则需要配置一个 VPN 网关。有不同类型的网关，要创建的网关类型取决于您的网络设计规划和要使用的本地 VPN 设备。例如，某些连接选项如点到站点连接，需要一个动态路由网关。如果要将您的网关配置为同时支持点到站点 (P2S) 连接和站点到站点 (S2S) 连接，则必须配置一个动态路由网关，即使站点到站点连接可以配置为任何一种网关路由类型。此外，还必须确保您的站点到站点连接将使用的设备支持要创建的网关类型。请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways)。

## 配置概述

配置网关之前，您首先需要创建虚拟网络。有关创建虚拟网络进行跨界连接的步骤，请参阅[配置具有站点到站点 VPN 的虚拟网络](/documentation/articles/vpn-gateway-site-to-site-create)或[配置具有点到站点 VPN 的虚拟网络](/documentation/articles/vpn-gateway-point-to-site-create)。然后，使用下面的步骤配置 VPN 网关并收集配置 VPN 设备所需的信息。

如果已经拥有一个 VPN 网关并且想更改路由类型，请参阅[如何更改 VPN 网关类型](#how-to-change-your-vpn-gateway-type)。

1. [创建 VPN 网关](#create-a-vpn-gateway)

1. [为 VPN 设备配置收集信息](#gather-information-for-your-vpn-device-configuration)

1. [配置 VPN 设备](#configure-your-vpn-device)

1. [验证您的本地网络范围和 VPN 网关 IP 地址](#verify-your-local-network-ranges-and-vpn-gateway-ip-address)

## 创建 VPN 网关

1. 在“网络”页上，验证虚拟网络的“状态”列是否为“已创建”。

1. 在“名称”列中，单击您的虚拟网络的名称。

1. 在“仪表板”页上，请注意此 VNet 尚未配置网关。当完成配置网关的步骤时，您将会看到此状态。

![未创建网关](./media/vpn-gateway-configure-vpn-gateway-mp/IC717025.png)


接下来，在页面底部，单击“创建网关”。您可以选择“静态路由”或“动态路由”。选择的路由类型取决于多种因素。例如，您的 VPN 设备支持什么以及您是否需要支持点到站点连接。请检查[关于用于虚拟网络连接的 VPN 设备](http://go.microsoft.com/fwlink/p/?LinkId=615934)以验证您需要的路由类型。一旦创建了网关，在不删除并重新创建网关的情况下，您无法更改网关类型。系统提示您确认要创建网关时，请单击“是”。

![网关类型](./media/vpn-gateway-configure-vpn-gateway-mp/IC717026.png)

正在创建网关时，请注意页面上的网关图形将更改为黄色，并显示“正在创建网关”。创建网关最多可能需要 25 分钟。您必须等到网关完成之后，才能继续进行其他配置设置。

![正在创建网关](./media/vpn-gateway-configure-vpn-gateway-mp/IC717027.png)

当网关更改为“正在连接”时，您可以收集 VPN 设备需要的信息。

![正在连接网关](./media/vpn-gateway-configure-vpn-gateway-mp/IC717028.png)

## 为 VPN 设备配置收集信息

创建网关后，为 VPN 设备配置收集信息。此信息位于您的虚拟网络的“仪表板”页上：

1. **网关 IP 地址 -** 可以在“仪表板”页上找到 IP 地址。在网关创建完成之前，您将无法看到该 IP 地址。

1. **共享密钥 -** 单击屏幕底部的“管理密钥”。单击密钥旁边的图标将其复制到您的剪贴板中，然后粘贴并保存密钥。请注意，此按钮仅适用于只有单独一个 S2S VPN 隧道的情况。如果有多个 S2S VPN 隧道，请使用 Get Virtual Network Gateway Shared Key API 或 PowerShell cmdlet。

![管理密钥](./media/vpn-gateway-configure-vpn-gateway-mp/IC717029.png)


## 配置 VPN 设备

完成前面的步骤后，您或您的网络管理员需要配置 VPN 设备以创建连接。请参阅[关于用于虚拟网络连接的 VPN 设备](http://go.microsoft.com/fwlink/p/?LinkID=615934)以获取有关 VPN 设备的更多信息。

VPN 设备配置完成后，您可以在 VNet 的“仪表板”页上查看更新后的连接信息。

还可以运行以下命令之一以测试您的连接：

| | Cisco ASA | Cisco ISR/ASR | Juniper SSG/ISG | Juniper SRX/J |
|----------------------|-----------------------|-----------------------|-----------------|------------------------------------------|
| **Check main mode SAs** | show crypto isakmp sa | show crypto isakmp sa | get ike cookie | show security ike security-association |
| **Check quick mode SAs** | show crypto ipsec sa | show crypto ipsec sa | get sa | show security ipsec security-association |


## 验证您的本地网络范围和 VPN 网关 IP 地址

### 验证您的 VPN 网关 IP 地址

为了正确连接网关，必须针对“本地网络”正确配置 VPN 设备的 IP 地址，在您的跨界配置中指定了“本地网络”。一般情况下，这在站点到站点配置过程中配置。但是，如果您以前将这个本地网络与另一台设备配合使用，或者已经更改了这个本地网络的 IP 地址，您会希望编辑设置以指定正确的网关 IP 地址。

1. 若要验证您的网关 IP 地址，请单击门户左窗格中的“网络”，然后在页面顶部选择“本地网络”。您将看到已经创建的每个本地网络的“VPN 网关地址”。若要编辑 IP 地址，请选择 VNet，然后单击页面底部的“编辑”。

1. 在“指定你的本地网络详细信息”页上，编辑 IP 地址，然后单击页面底部的“下一步”按钮。

1. 在“指定地址空间”页上，单击右下角的复选标记以保存您的设置。

### 验证您的本地网络的地址范围

要使流量通过网关正确流向您的本地位置，需要验证您已经列出的要包含在本地网络配置中的每个 IP 地址范围。视您的本地位置的网络配置而定，这可能是一项相当艰巨的任务，因为必须在您的 Azure“本地网络”配置中列出每个范围。然后，与包含在所列范围内的 IP 地址绑定的流量将通过虚拟网络 VPN 网关发送。您列出的 IP 地址范围无需是专有范围，尽管您需要验证本地配置能够接收入站流量。

若要添加或编辑本地网络的范围，请按下面的步骤操作。

1. 若要编辑本地网络的 IP 地址，请单击门户左窗格中的“网络”，然后在页面顶部选择“本地网络”。在门户中，查看已列出范围的最简单方式是通过“编辑”页。要查看您的范围，请选择 VNet，然后单击页面底部的“编辑”。

1. 在“指定你的本地网络详细信息”页上，不做任何更改。单击页面底部的“下一步”箭头。

1. 在“指定地址空间”页上，修改网络地址空间。然后单击复选标记以保存您的配置。

## 如何查看网关流量

您可以从虚拟网络“仪表板”页查看您的网关和网关流量。

在“仪表板”页上，您可以查看以下内容：

- 流经网关的数据量，包括数据流入和数据流出。

- 为您的虚拟网络指定的 DNS 服务器名称。

- 您的网关和 VPN 设备之间的连接。

- 用于配置您的网关与 VPN 设备连接的共享密钥。


## 如何更改您的 VPN 网关类型

因为某些连接配置仅适用于某些网关类型，您可能会发现需要更改现有 VPN 网关的网关类型。例如，您可能需要将点到站点连接添加到一个具有静态网关的现有站点到站点连接。点到站点连接需要动态网关，这意味着要对其进行配置，您必须将网关类型从静态改为动态。

如果需要更改 VPN 网关路由类型，您应删除现有网关，然后用新的路由类型重新创建网关。您不需要为了更改网关路由类型而删除整个虚拟网络。

在更改网关类型之前，请务必验证您的 VPN 设备能够支持要使用的路由类型。若要下载新的路由配置示例并检查 VPN 设备要求，请参阅[关于用于虚拟网络连接的 VPN 设备](http://go.microsoft.com/fwlink/p/?LinkID=615934)。

>[AZURE.IMPORTANT]当删除一个虚拟网络 VPN 网关时，分配到该网关的 VIP 被释放。当重新创建网关时，将为其分配新的 VIP。

1. **删除现有 VPN 网关。**

	在您的虚拟网络的“仪表板”页上，导航到页面底部，然后单击“删除网关”。等待网关已被删除的通知。在屏幕上接收到网关已被删除的通知后，您可以创建新的网关。

1. **创建新的 VPN 网关。**

	使用页面顶部的步骤创建新网关：[创建 VPN 网关](#create-a-vpn-gateway)。


## 后续步骤

您可以在下文中了解有关虚拟网络跨界连接的更多信息：[关于虚拟网络安全跨界连接](/documentation/articles/vpn-gateway-cross-premises-options)。

您可以将虚拟机添加到虚拟网络。请参阅[如何创建自定义虚拟机](/documentation/articles/virtual-machines-create-custom)。

如果您想要配置点到站点 VPN 连接，请参阅[配置点到站点 VPN 连接](/documentation/articles/vpn-gateway-point-to-site-create)。

 

<!---HONumber=67-->