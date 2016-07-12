<properties
   pageTitle="使用 PowerShell 创建云服务容器 | Azure"
   description="本文说明如何使用 PowerShell 创建云服务容器。该容器承载 Web 角色和辅助角色。"
   services="cloud-services"
   documentationCenter=".net"
   authors="cawaMS"
   manager="timlt"
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="04/25/2016"
   wacn.date="05/31/2016"/>

# 使用 Azure PowerShell 命令可创建一个空的云服务容器
本文介绍如何使用 Azure PowerShell cmdlet 快速创建云服务容器。请执行以下步骤：

1. 从 [Azure PowerShell 下载](http://aka.ms/webpi-azps)页安装 Azure PowerShell cmdlet。
2. 打开 PowerShell 命令提示符。
3. 使用 [Add-AzureAccount](https://msdn.microsoft.com/zh-cn/library/dn495128.aspx) 登录。

    > [AZURE.NOTE] 有关安装 Azure PowerShell cmdlet 和连接到 Azure 订阅的更多说明，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

4. 使用 **New-AzureService** cmdlet 创建一个空的 Azure 云服务容器。

    ```
    New-AzureService [-ServiceName] <String> [-AffinityGroup] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
    New-AzureService [-ServiceName] <String> [-Location] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
```

5. 请按照本示例操作以调用 cmdlet：
```
New-AzureService -ServiceName "mytestcloudservice" -Location "China North" -Label "mytestcloudservice"
```

有关创建 Azure 云服务的详细信息，请运行：
```
Get-help New-AzureService
```

### 后续步骤

 * 若要管理云服务部署，请参阅 [Get-AzureService](https://msdn.microsoft.com/zh-cn/library/azure/dn495131.aspx)、[Remove-AzureService](https://msdn.microsoft.com/zh-cn/library/azure/dn495120.aspx) 和 [Set-AzureService](https://msdn.microsoft.com/zh-cn/library/azure/dn495242.aspx) 命令。有关更多信息，你还可以参阅[如何配置云服务](/documentation/articles/cloud-services-how-to-configure/)。

 * 若要将云服务项目发布到 Azure，请参阅[在 Azure 中持续交付云服务](/documentation/articles/cloud-services-dotnet-continuous-delivery/)中的 **PublishCloudService.ps1** 代码示例。

<!---HONumber=Mooncake_0523_2016-->
