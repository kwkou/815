<properties
    pageTitle="修改本地网关 IP 地址前缀和 VPN 网关 IP 地址| Azure| CLI| Azure"
    description="本文介绍如何使用 Azure CLI 更改本地网络网关的 IP 地址前缀"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="8c7db48f-d09a-44e7-836f-1fb6930389df"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/24/2017"
    wacn.date="05/31/2017"
    ms.author="cherylmc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="3a36b83eb649303312f68bfbe331af19701f3a5b"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="modify-local-network-gateway-settings-using-the-azure-cli"></a>使用 Azure CLI 修改本地网络网关设置

有时本地网络网关 AddressPrefix 或 GatewayIPAddress 的设置会变更。 本文演示如何修改本地网络网关设置。 也可在 Azure 门户或 PowerShell 中修改这些设置。

## <a name="before-you-begin"></a>开始之前

安装最新版本的 CLI 命令（2.0 或更高版本）。 有关安装 CLI 命令的信息，请参阅 [Install Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)（安装 Azure CLI 2.0）。

[AZURE.INCLUDE [CLI-login](../../includes/vpn-gateway-cli-login-include.md)]

## <a name="modify-ip-address-prefixes"></a>修改 IP 地址前缀

[AZURE.INCLUDE [modify-prefix](../../includes/vpn-gateway-modify-ip-prefix-cli-include.md)]

## <a name="modify-the-gateway-ip-address"></a>修改网关 IP 地址

[AZURE.INCLUDE [modify-gateway-IP](../../includes/vpn-gateway-modify-lng-gateway-ip-cli-include.md)]

## <a name="next-steps"></a>后续步骤

可验证网关连接。 请参阅[验证网关连接](/documentation/articles/vpn-gateway-verify-connection-resource-manager/)。