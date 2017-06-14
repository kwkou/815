<properties
    pageTitle="更改存储访问密钥后更新媒体服务 | Azure"
    description="此文将提供有关在轮转存储访问密钥后如何更新媒体服务的指导。"
    services="media-services"
    documentationcenter=""
    author="Juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="a892ebb0-0ea0-4fc8-b715-60347cc5c95b"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/28/2017"
    wacn.date="03/10/2017"
    ms.author="milanga;cenkdin;juliako" />  


#更改存储访问密钥后更新媒体服务

创建新的 Azure 媒体服务 (AMS) 帐户时，系统还会要求用户选择用于存储媒体内容的 Azure 存储帐户。可以向媒体服务帐户添加多个存储帐户。本主题介绍如何轮换存储密钥，以及如何将存储帐户添加到媒体帐户。

若要执行本主题所述操作，应使用 [ARM API](https://docs.microsoft.com/rest/api/media/mediaservice) 和 [Powershell](https://docs.microsoft.com/powershell/resourcemanager/azurerm.media/v0.3.2/azurerm.media)。有关详细信息，请参阅[如何使用 PowerShell 和 Resource Manager 管理 Azure 资源](/documentation/articles/powershell-azure-resource-manager/)。

## 概述

创建新的存储帐户后，Azure 将生成两个 512 位存储访问密钥，用于对对存储帐户的访问进行身份验证。为保持存储连接更加安全，建议定期重新生成并轮转存储访问密钥。将提供两个访问密钥（主密钥和辅助密钥），以便在重新生成其中一个访问密钥时，始终能够使用另一个访问密钥连接到存储帐户。此过程也称为“轮转访问密钥”。

媒体服务依赖于向其提供的存储密钥。具体而言，用于流式处理或下载资产的定位符依赖于指定的存储访问密钥。创建 AMS 帐户后，它默认依赖于主存储访问密钥，但用户可以更新 AMS 的存储密钥。必须遵循本主题中所述的步骤，确保媒体服务知道要使用哪个密钥。

>[AZURE.NOTE]
若有多个存储帐户，请对每个存储帐户执行此过程。存储密钥的轮换顺序不是固定的。可以先轮换辅助密钥，然后再轮换主密钥，反之亦然。
>
>在对生产帐户执行本主题中所述的步骤之前，请确保对生产前帐户测试这些步骤。


## 轮换存储密钥的步骤 
 
 1. 通过 PowerShell cmdlet 或 [Azure](https://portal.azure.cn/) 门户更改存储帐户主密钥。
 2. 使用适当的参数调用 Sync-AzureRmMediaServiceStorageKeys cmdlet，强制媒体帐户选取存储帐户密钥
 
    以下示例演示了如何将密钥同步到存储帐户。
  
         Sync-AzureRmMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId
  
 3. 等待一小时左右。验证流式处理方案是否正常工作。
 4. 通过 PowerShell cmdlet 或 Azure 门户更改存储帐户辅助密钥。
 5. 使用适当的参数调用 Sync-AzureRmMediaServiceStorageKeys cmdlet，强制媒体帐户选取新的存储帐户密钥。
 6. 等待一小时左右。验证流式处理方案是否正常工作。
 
### PowerShell cmdlet 示例 

以下示例演示了如何获取存储帐户并通过 AMS 帐户将其同步。

	$regionName = "China East"
	$resourceGroupName = "SkyMedia-ChinaEast-App"
	$mediaAccountName = "sky"
	$storageAccountName = "skystorage"
	$storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

	Sync-AzureRmMediaServiceStorageKeys -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccountId $storageAccountId

 
## 将存储帐户添加到 AMS 帐户的步骤

以下主题介绍了如何将存储帐户添加到 AMS 帐户：[将多个存储帐户附加到媒体服务帐户](/documentation/articles/meda-services-managing-multiple-storage-accounts/)。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: whole content update to new powershell scripts-->