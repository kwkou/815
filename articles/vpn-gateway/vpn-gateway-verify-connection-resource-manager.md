<properties
   pageTitle="验证网关连接 | Azure"
   description="本文说明如何验证 Resource Manager 部署模型中的网关连接"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>  


<tags
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/14/2016"
   wacn.date="11/07/2016"
   ms.author="cherylmc"/>

# 验证网关连接

可使用几种不同的方法验证网关连接。本文说明如何使用 Azure 门户预览和 PowerShell 验证 Resource Manager 网关连接的状态。


## 使用 PowerShell 验证

你需要安装最新版本的 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。有关使用 Resource Manager cmdlet 的详细信息，请参阅[将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。

### 步骤 1：登录到 Azure 帐户

1. 使用提升的权限打开 PowerShell 控制台，然后连接到帐户。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 检查该帐户的订阅。

		Get-AzureRmSubscription 

3. 指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

### 步骤 2：验证连接


[AZURE.INCLUDE [在 powershell 中验证连接](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]


## 使用 Azure 门户预览验证

[AZURE.INCLUDE [在门户中验证连接](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]


## 后续步骤

- 你可以将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。

<!---HONumber=Mooncake_1031_2016-->