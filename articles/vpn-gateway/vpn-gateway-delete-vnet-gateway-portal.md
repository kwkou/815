<properties
    pageTitle="删除虚拟网络网关：Azure 门户：Resource Manager | Azure"
    description="在 Resource Manager 部署模型中使用 PowerShell 删除虚拟网络网关。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic=""
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/29/2017"
    wacn.date="05/22/2017"
    ms.author="cherylmc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="c287442a5fe981eb957b25df09ac5a79a57fa922"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="delete-a-virtual-network-gateway-using-the-portal"></a>使用门户删除虚拟网络网关
> [AZURE.SELECTOR]
- [Resource Manager - Azure 门户](/documentation/articles/vpn-gateway-delete-vnet-gateway-portal/)
- [Resource Manager - PowerShell](/documentation/articles/vpn-gateway-delete-vnet-gateway-powershell/)
- [经典 - PowerShell](/documentation/articles/vpn-gateway-delete-vnet-gateway-classic-powershell/)

可以使用多种不同的方法来删除 VPN 网关配置中的虚拟网络网关。

- 如果要删除所有信息并从头开始配置（例如，在测试环境中），可以删除资源组。 删除某个资源组时，会删除该组中的所有资源。 仅当不想保留资源组中的任何资源时，才建议使用此方法。 使用这种方法时，无法做到有选择性地删除一部分资源。

- 如果想要保留资源组中的某些资源，则删除虚拟网络网关的过程会略微复杂一些。 在删除虚拟网络网关之前，必须先删除任何依赖于该网关的资源。 遵循的步骤取决于创建的连接类型，以及每个连接的依赖资源。

## <a name="deletegw"></a>删除 VPN 网关

若要删除虚拟网络网关，必须先删除与该虚拟网络网关相关的每个资源。 由于存在依赖关系，必须按特定的顺序删除资源。

### <a name="step-1-navigate-to-the-virtual-network-gateway"></a>步骤 1：导航到虚拟网络网关

在 [Azure 门户](https://portal.azure.cn)中的“所有资源”中，单击要删除的虚拟网络网关。 网关图形是：

![找到虚拟网络网关](./media/vpn-gateway-delete-vnet-gateway-portal/gw.png)

### <a name="step-2-delete-connections"></a>步骤 2：删除连接

1. 在“虚拟网络网关”边栏选项卡中，单击“连接”以查看该网关的所有连接。
2. 在连接的名称行中，单击“...”，然后从下拉列表中选择“删除”。
3. 单击“是”以确认要删除该连接。 如果有多个连接，请删除每个连接。

### <a name="step-3-delete-the-local-network-gateways"></a>步骤 3：删除本地网关
1. 在“所有资源”中，找到与每个连接相关联的本地网关。 本地网关的图形是：

    ![找到本地网关](./media/vpn-gateway-delete-vnet-gateway-portal/lng.png)
2. 在本地网关的“概述”边栏选项卡上，单击“删除”。

### <a name="step-4-delete-the-virtual-network-gateway"></a>步骤 4：删除虚拟网络网关
1. 在“所有资源”中，找到要删除的虚拟网络网关。
2. 在“概述”边栏选项卡上，单击“删除”以删除网关。

>[AZURE.NOTE]
> 除了 S2S 配置，如果你还有此 VNet 的 P2S 配置，则删除虚拟网络网关将自动断开所有 P2S 客户端且不发出警告。
>
>

### <a name="step-5-delete-the-public-ip-address-for-the-gateway"></a>步骤 5：删除网关的公共 IP 地址

1. 在“所有资源”中，找到已分配到该网关的公共 IP 地址。 如果虚拟网络网关采用主动-主动配置，将显示两个公共 IP 地址。 公共 IP 地址的图形是：

    ![公共 IP 地址](./media/vpn-gateway-delete-vnet-gateway-portal/pip.png)

2. 在公共 IP 地址的“概述”页上，单击“删除”，然后单击“是”进行确认。

### <a name="step-6-delete-the-gateway-subnet"></a>步骤 6：删除网关子网

1. 在“所有资源”中，找到虚拟网络。 
2. 在“子网”边栏选项卡上，单击“GatewaySubnet”，然后单击“删除”。 
3. 单击“是”确认要删除该网关子网。

## <a name="deleterg"></a>通过删除资源组来删除 VPN 网关

如果不关心是否要保留资源组中的任何资源，而只是要从头开始配置，则可以删除整个资源组。 这种方法可以快速删除所有信息。 以下步骤仅适用于 Resource Manager 部署模型。

1. 在“所有资源”中，找到该资源组并单击以打开边栏选项卡。
2. 单击“删除” 。 在“删除”边栏选项卡上，查看受影响的资源。 请确保要删除所有这些资源。 否则使用本文开头[删除 VPN 网关](#deletegw)中的步骤。
3. 若要继续，请键入要删除的资源组的名称，然后单击“删除”。