<properties
   pageTitle="如何使用 Azure PowerShell 命令创建一个空的云服务容器"
   description="本文介绍如何使用 PowerShell 脚本创建云服务容器和执行云服务相关的管理操作"
   services="cloud-services"
   documentationCenter=".net"
   authors="cawaMS"
   manager="paulyuk" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="01/13/2015"
   wacn.date="02/26/2016"/>

# 如何使用 Azure PowerShell 命令创建一个空的云服务容器
1. 从[下载 Azure PowerShell](http://aka.ms/webpi-azps) 安装 Azure PowerShell cmdlet。打开 PowerShell 命令提示符。使用 [Add-AzureAccount](https://msdn.microsoft.com/zh-cn/library/dn495128.aspx) 登录

> [AZURE.NOTE] 有关安装 Azure PowerShell cmdlet 和连接到 Azure 订阅的更多说明，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

2. **New-AzureService** 是用于创建空云服务容器的 cmdlet。

    ```
    New-AzureService [-ServiceName] <String> [-AffinityGroup] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
    New-AzureService [-ServiceName] <String> [-Location] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
	```

   调用该 cmdlet 的示例：
	```
New-AzureService -ServiceName "mytestcloudservice" -Location "China North" -Label "mytestcloudservice"
	```

3. 有关创建 Azure 云服务的详细信息，请运行：
```
Get-help New-AzureService
```

4. 后续步骤：

   - 若要管理云服务部署，请参阅 [Get-AzureService](https://msdn.microsoft.com/zh-cn/library/azure/dn495131.aspx)、[Remove-AzureService](https://msdn.microsoft.com/zh-cn/library/azure/dn495120.aspx) 和 [Set-AzureService](https://msdn.microsoft.com/zh-cn/library/azure/dn495242.aspx) 命令。另请参阅[如何配置云服务](/documentation/articles/cloud-services-how-to-configure)。

    - 若要将云服务项目发布到 Azure，请参阅[在 Azure 中持续交付云服务](/documentation/articles/cloud-services-dotnet-continuous-delivery)中的 **PublishCloudService.ps1** 代码示例
 

<!---HONumber=Mooncake_0215_2016-->