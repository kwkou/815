<properties 
	pageTitle="使用 PowerShell 管理 Azure 媒体服务帐户" 
	description="了解如何使用 PowerShell cmdlet 管理 Azure 媒体服务帐户。" 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"
	wacn.date="06/27/2016"/>


#使用 PowerShell 管理 Azure 媒体服务帐户

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-create-account/)
- [PowerShell](/documentation/articles/media-services-manage-with-powershell/)
- [REST](http://msdn.microsoft.com/zh-cn/library/azure/dn194267.aspx)

> [AZURE.NOTE] 若要创建 Azure 媒体服务帐户，你必须有一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/?WT.mc_id=A8A8397B5" target="_blank">Azure 试用</a>。

##概述 

本文说明如何使用 PowerShell cmdlet 管理 Azure 媒体服务帐户。

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/?WT.mc_id=A8A8397B5" target="_blank">Azure 试用</a>。

##安装 Azure PowerShell Cmdlet

若要安装最新的 Azure PowerShell cmdlet，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)

##选择 Azure 订阅

在安装并配置 PowerShell cmdlet 后，你应该指定要使用的订阅。

若要获取可用订阅的列表，请运行以下 cmdlet：

	PS C:\> Get-AzureSubscription

然后，通过执行以下操作来选择一个订阅：

	PS C:\> Select-AzureSubscription "TestSubscription"

 
##获取存储帐户名称

Azure 媒体服务使用 Azure 存储空间来存储媒体内容。当你创建新的媒体服务帐户时，必须将该帐户与存储帐户相关联。存储帐户必须与你打算用于媒体服务帐户属于同一个订阅。

此示例中使用了现有的存储帐户。[Get-AzureStorageAccount](https://msdn.microsoft.com/zh-cn/library/azure/dn495134.aspx) cmdlet 可以获取当前订阅中的存储帐户。获取你要关联到媒体帐户的存储帐户的名称 (StorageAccountName)。

	StorageAccountDescription : 
	AffinityGroup             :
	Location                  : China East
	GeoReplicationEnabled     : True
	GeoPrimaryLocation        : China East
	GeoSecondaryLocation      : China North
	Label                     : storagetest001
	StorageAccountStatus      : Created
	StatusOfPrimary           : Available
	StatusOfSecondary         : Available
	Endpoints                 : {https://storagetest001.blob.core.chinacloudapi.cn/,
	                            https://storagetest001.queue.core.chinacloudapi.cn/,
	                            https://storagetest001.table.core.chinacloudapi.cn/}
	AccountType               : Standard_GRS
	StorageAccountName        : storatetest001
	OperationDescription      : Get-AzureStorageAccount
	OperationId               : e919dd56-7691-96db-8b3c-2ceee891ae5d
	OperationStatus           : Succeeded

##创建新的媒体服务帐户

若要创建新的 Azure 媒体服务帐户，请使用 [New-AzureMediaServicesAccount](https://msdn.microsoft.com/zh-cn/library/azure/dn495286.aspx) cmdlet，它会提供媒体服务帐户名、要在其中创建该帐户的数据中心位置，以及存储帐户名称。


	PS C:\> New-AzureMediaServicesAccount -Name "amstestaccount001" -StorageAccountName "storagetest001" -Location "China East"

##获取媒体服务帐户

创建一个或多个媒体服务帐户后，你可以使用 [Get-AzureMediaServicesAccount](https://msdn.microsoft.com/zh-cn/library/azure/dn495286.aspx) 列出信息

	
	PS C:\> Get-AzureMediaServicesAccount
	
	AccountId		Name				State
	---------       ----       			 -----
	xxxxxxxxxx      amstestaccount001   Active

通过提供 Name 参数，你将会获得更多详细信息，包括帐户密钥。

	PS C:\> Get-AzureMediaServicesAccount -Name amstestaccount001

##重新生成媒体服务访问密钥

如果你想要更新媒体服务主访问密钥或辅助访问密钥，请使用 [New-AzureMediaServicesKey](https://msdn.microsoft.com/zh-cn/library/azure/dn495215.aspx)。你需要提供帐户名并指定你想要重新生成的密钥（主密钥或辅助密钥）。

如果你不希望 PowerShell 提出确认问题，请指定 -Force 开关。

	PS C:\> New-AzureMediaServicesKey -Name "amstestaccount001" -KeyType "Primary" -Force

##删除媒体服务帐户

当你准备好删除 Azure Media 帐户时，请使用 [Remove-AzureMediaServicesAccount](https://msdn.microsoft.com/zh-cn/library/azure/dn495220.aspx)。

	PS C:\> Remove-AzureMediaServicesAccount -Name "amstestaccount001" -Force


 
<!---HONumber=Mooncake_0620_2016-->