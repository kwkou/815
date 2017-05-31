<properties
    pageTitle="验证 VPN 网关连接 | Azure"
    description="本文介绍如何验证虚拟网络 VPN 网关连接。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-service-management,azure-resource-manager" />
<tags
    ms.assetid="7e3d1043-caa9-4472-96d3-832f4e2c91ee"
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
    ms.openlocfilehash="c4ac779687c2aead10db168919d04ab0ec23e61c"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="verify-a-vpn-gateway-connection"></a>验证 VPN 网关连接

本文介绍如何验证 Resource Manager 和经典部署模型的 VPN 网关连接。

## <a name="verify-using-the-azure-portal-preview"></a>使用 Azure 门户预览进行验证

[AZURE.INCLUDE [Azure portal preview](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="verify-using-powershell"></a>使用 PowerShell 验证

若要使用 PowerShell 进行验证，请安装最新版本的 Azure Resource Manager PowerShell cmdlet。 有关安装 PowerShell cmdlet 的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azure/overview)。 有关使用 Resource Manager cmdlet 的详细信息，请参阅[将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。

### <a name="log-in-to-your-azure-account"></a>登录到 Azure 帐户
1. 使用提升的权限打开 PowerShell 控制台，然后连接到帐户。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 检查该帐户的订阅。

        Get-AzureRmSubscription

3. 指定要使用的订阅。

        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

### <a name="verify-your-connection"></a>验证连接

[AZURE.INCLUDE [PowerShell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="verify-using-the-azure-cli"></a>使用 Azure CLI 进行验证

若要使用 Azure CLI 进行验证，请安装最新版本的 CLI 命令（2.0 或更高版本）。 有关安装 CLI 命令的信息，请参阅 [Install Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)（安装 Azure CLI 2.0）。

### <a name="log-in-to-your-azure-account"></a>登录到 Azure 帐户

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

1. 使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

      az login

2. 如果有多个 Azure 订阅，请列出该帐户的订阅。

      Az account list --all

3. 指定要使用的订阅。

      Az account set --subscription
      <replace_with_your_subscription_id>

### <a name="verify-your-connection"></a>验证连接

[AZURE.INCLUDE [CLI](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="verify-using-the-azure-portal-preview-classic"></a>使用 Azure 门户预览进行验证（经典）
[AZURE.INCLUDE [Azure portal preview](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

## <a name="verify-using-powershell-classic"></a>使用 PowerShell 验证（经典）
若要使用 PowerShell 进行验证，请安装最新版本的 Azure PowerShell cmdlet。 请务必同时下载并安装 Resource Manager 和服务管理 (SM) 版本。 有关安装 PowerShell cmdlet 的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azure/overview)。 

### <a name="log-in-to-your-azure-account"></a>登录到 Azure 帐户
1. 使用提升的权限打开 PowerShell 控制台，然后连接到帐户。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 检查该帐户的订阅。

        Get-AzureRmSubscription 

3. 指定要使用的订阅。

        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

4. 登录以便使用经典部署模型的 Service Management cmdlet。

        Add-AzureAccount -Environment AzureChinaCloud

### <a name="verify-your-connection"></a>验证连接
[AZURE.INCLUDE [Classic PowerShell](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

## <a name="next-steps"></a>后续步骤
* 可以将虚拟机添加到虚拟网络。 请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。

<!--Update_Description: add "verify using the azure cli"-->