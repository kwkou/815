<properties
	pageTitle="使用 PowerShell 创建 SSL 证书绑定"
	description="了解如何使用 PowerShell 将 SSL 证书绑定到 Web 应用。"
	services="app-service\web"
	documentationCenter=""
	authors="ahmedelnably"
	manager="stefsch"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="01/13/2016"
	wacn.date="07/04/2016"/>

# 使用 PowerShell 创建 Azure SSL 证书绑定 #

发行的 Azure PowerShell 版本 1.1.0 中添加了新的 cmdlet，可让用户将现有的或新的 SSL 证书绑定到现有的 Web 应用。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../includes/app-service-web-to-api-and-mobile.md)]

若要了解如何使用基于 Azure Resource Manager 的 Azure PowerShell cmdlet 来管理 Web Apps，请查看[适用于 Azure Web 应用的基于 Azure Resource Manager 的 PowerShell 命令](/documentation/articles/app-service-web-app-azure-resource-manager-powershell/)

## 上载和绑定新的 SSL 证书 ##

情景：用户要将 SSL 证书绑定到某个 Web 应用。

如果知道包含 Web 应用的资源组名称、Web 应用名称、用户计算机上的证书 .pfx 文件路径、证书的密码以及自定义主机名，我们就可以使用以下 PowerShell 命令来创建 SSL 绑定：

    New-AzureRmWebAppSSLBinding -ResourceGroupName myresourcegroup -WebAppName mytestapp -CertificateFilePath PathToPfxFile -CertificatePassword PlainTextPwd -Name www.contoso.com

请注意，在将 SSL 绑定添加到 Web 应用之前，必须已经配置主机名（自定义域）。如果未配置主机名，则在运行 New-AzureRmWebAppSSLBinding 时会收到“主机名不存在”错误。你可以直接从经典管理门户或使用 Azure PowerShell 添加主机名。以下 PowerShell 代码段可以在运行 New-AzureRmWebAppSSLBinding 之前配置主机名。
  
    $webApp = Get-AzureRmWebApp -Name mytestapp -ResourceGroupName myresourcegroup  
    $hostNames = $webApp.HostNames  
    $HostNames.Add("www.contoso.com")  
    Set-AzureRmWebApp -Name mytestapp -ResourceGroupName myresourcegroup -HostNames $HostNames   
  
请务必了解，Set-AzureRmWebApp cmdlet 会覆盖 Web 应用的主机名。因此，上述 PowerShell 代码段会追加到 Web 应用现有的主机名列表。

## 上载和绑定现有的 SSL 证书 ##

情景：用户要将以前上载的 SSL 证书绑定到某个 Web 应用。

我们可以使用以下命令获取已上载到特定资源组的证书列表

	Get-AzureRmWebAppCertificate -ResourceGroupName myresourcegroup

请注意，证书位于特定位置和资源组的本地，如果配置的 Web 应用与所需的证书位于不同的位置和资源组，用户需要重新上载证书

如果知道包含 Web 应用的资源组名称、Web 应用名称、证书指纹以及自定义主机名，我们就可以使用以下 PowerShell 命令来创建该 SSL 绑定：

    New-AzureRmWebAppSSLBinding -ResourceGroupName myresourcegroup -WebAppName mytestapp -Thumbprint <certificate thumbprint> -Name www.contoso.com

## 删除现有的 SSL 绑定  ##

情景：用户想要删除现有的 SSL 绑定。

如果知道包含 Web 应用的资源组名称、Web 应用名称以及自定义主机名，我们就可以使用以下 PowerShell 命令来删除该 SSL 绑定：

    Remove-AzureRmWebAppSSLBinding -ResourceGroupName myresourcegroup -WebAppName mytestapp -Name www.contoso.com

请注意，如果删除的 SSL 绑定是在该位置使用该证书的最后一个绑定，则会按默认删除该证书；如果用户想要保留证书，可以使用 DeleteCertificate 选项来保留证书

	Remove-AzureRmWebAppSSLBinding -ResourceGroupName myresourcegroup -WebAppName mytestapp -Name www.contoso.com -DeleteCertificate $false

### 参考 ###
- [适用于 Azure Web 应用的基于 Azure Resource Manager 的 PowerShell 命令](/documentation/articles/app-service-web-app-azure-resource-manager-powershell/)
- [将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)

<!---HONumber=Mooncake_0627_2016-->