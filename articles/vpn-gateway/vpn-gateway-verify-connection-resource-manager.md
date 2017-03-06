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
    ms.date="01/30/2017"
    wacn.date="03/03/2017"
    ms.author="cherylmc" />  


# 验证 VPN 网关连接
可通过使用门户和 PowerShell 来验证虚拟网络 VPN 网关连接。本文所含步骤适用于 Resource Manager 部署模型和经典部署模型。

## 使用 Azure 门户预览进行验证

[AZURE.INCLUDE [Azure 门户预览](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## 使用 PowerShell 验证

若要使用 PowerShell 进行验证，请安装最新版本的 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。有关使用 Resource Manager cmdlet 的详细信息，请参阅[将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。

### 登录到 Azure 帐户
1. 使用提升的权限打开 PowerShell 控制台，然后连接到帐户。
   
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
2. 检查该帐户的订阅。
   
        Get-AzureRmSubscription 
3. 指定要使用的订阅。
   
        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

### 验证连接

[AZURE.INCLUDE [Powershell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## 使用 Azure 门户预览进行验证（经典）
[AZURE.INCLUDE [Azure 门户预览](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

## 使用 PowerShell 验证（经典）
若要使用 PowerShell 进行验证，请安装最新版本的 Azure PowerShell cmdlet。请务必下载并安装 Resource Manager 版本和 Service Management (SM) 版本。有关安装 PowerShell cmdlet 的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

### 登录到 Azure 帐户
1. 使用提升的权限打开 PowerShell 控制台，然后连接到帐户。
   
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
2. 检查该帐户的订阅。
   
        Get-AzureRmSubscription 
3. 指定要使用的订阅。
   
        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
4. 登录以便使用经典部署模型的 Service Management cmdlet。

		Add-AzureAccount -Environment AzureChinaCloud

### 验证连接
[AZURE.INCLUDE [经典 PowerShell](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

## 后续步骤
* 可以将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add verification for the classic model-->