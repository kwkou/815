<properties 
	pageTitle="使用 PowerShell 管理 Azure 媒体服务帐户" 
	description="了解如何使用 PowerShell cmdlet 管理 Azure 媒体服务帐户。" 
	authors="Juliako" 
	manager="erikre" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>  


<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/03/2016"
	wacn.date="11/21/2016"
	ms.author="juliako"/>  



#使用 PowerShell 管理 Azure 媒体服务帐户

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-create-account/)
- [PowerShell](/documentation/articles/media-services-manage-with-powershell/)
- [REST](http://msdn.microsoft.com/zh-cn/library/azure/dn194267.aspx)

> [AZURE.NOTE] 若要创建 Azure 媒体服务帐户，你必须有一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/?WT.mc_id=A8A8397B5" target="_blank">Azure 试用</a>。

##概述 

本文列出了 Azure Resource Manager 框架中适用于 Azure 媒体服务 (AMS) 的 Azure PowerShell cmdlet。这些 cmdlet 存在于 **Microsoft.Azure.Commands.Media** 命名空间。

## 版本

**ApiVersion**：“2015-10-01”
               

## New-AzureRmMediaService

创建媒体服务。

### 语法

参数集：StorageAccountIdParamSet

	New-AzureRmMediaService [-ResourceGroupName] <string> [-AccountName] <string> [-Location] <string> [-StorageAccountId] <string> [-Tags <hashtable>]  [<CommonParameters>]

参数集：StorageAccountsParamSet

	New-AzureRmMediaService [-ResourceGroupName] <string> [-AccountName] <string> [-Location] <string> [-StorageAccounts] <PSStorageAccount[]> [-Tags <hashtable>]  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 | 无
---|---
必需？ | 是
位置？ | 0
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |名称
---|---
必需？ |是
位置？ |1
默认值 |无
接受管道输入？ |false
接受通配符？ |false

**-Location &lt;字符串&gt;**

指定媒体服务的资源位置。

别名 |无
---|---
必需？ |是
位置？ |2
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-StorageAccountId &lt;字符串&gt;**

指定与媒体服务关联的主存储帐户。

- 仅限受支持的新存储帐户（使用资源管理器 API 创建）。

- 存储帐户必须存在，并具有与媒体服务相同的位置。

别名 |无
---|---
必需？ |是
位置？ |3
默认值 |无
接受管道输入？ |true(ByPropertyName)
参数集名称 |StorageAccountIdParamSet
接受通配符？|false

**-StorageAccounts &lt;PSStorageAccount[]&gt;**

指定与媒体服务关联的存储帐户。

- 仅限受支持的新存储帐户（使用资源管理器 API 创建）。

- 存储帐户必须存在，并具有与媒体服务相同的位置。

- 只能指定一个存储帐户作为主存储帐户。

别名 |无
---|---
必需？ |是
位置？ |3
默认值 |无
接受管道输入？ |true(ByPropertyName)
参数集名称 |StorageAccountsParamSet
接受通配符？ |false

**-Tags &lt;Hashtable&gt;**

指定与媒体服务关联的标记的哈希表。

- 示例：@{"tag1"="value1";"tag2"=:value2"}

别名 |无
---|---
必需？ |false
位置？ |名为
默认值 |无
接受管道输入？ |false
接受通配符？ |false

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。

## Set-AzureRmMediaService

更新媒体服务。

### 语法

	Set-AzureRmMediaService [-ResourceGroupName] <string> [-AccountName] <string> [-Tags <hashtable>] [-StorageAccounts <PSStorageAccount[]>]  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 |无
---|---
必需？ |是
位置？ |0
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |名称
---|---
必需？ |True
位置？ |1
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |False

**-StorageAccounts &lt;PSStorageAccount[]&gt;**

指定与媒体服务关联的存储帐户。

- 仅限受支持的新存储帐户（使用资源管理器 API 创建）。

- 存储帐户必须存在，并具有与媒体服务相同的位置。

- 只能指定一个存储帐户作为主存储帐户。

别名 |无
---|---
必需？ |false
位置？ |名为
默认值 |无
接受管道输入？ |true(ByPropertyName)
参数集名称 |StorageAccountsParamSet
接受通配符？ |false

**-Tags &lt;Hashtable&gt;**

指定与此媒体服务关联的标记的哈希表。

- 与媒体服务关联的标记替换为客户指定的值。

别名 |无
---|---
必需？ |False
位置？ |名为
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。

## Remove-AzureRmMediaService

删除媒体服务。

### 语法

	Remove-AzureRmMediaService [-ResourceGroupName] <string> [-AccountName] <string>  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 |无
---|---
必需？ |是
位置？ |0
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |无
---|---
必需？ |是
位置？ |2
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |False

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。

## Get-AzureRmMediaService

获取资源组中所有的媒体服务或具有给定名称的媒体服务。

### 语法

ParameterSet: ResourceGroupParameterSet

	Get-AzureRmMediaService [-ResourceGroupName] <string>  [<CommonParameters>]	

ParameterSet: AccountNameParameterSet

	Get-AzureRmMediaService [-ResourceGroupName] <string> [-AccountName] <string>  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 |无
---|---
必需？ |是
位置？ |0
默认值 |无
接受管道输入？ |true(ByPropertyName)
参数集名称 |ResourceGroupParameterSet、AccountNameParameterSet
接受通配符？false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |无
---|---
必需？ |是
位置？ |1
默认值 |无
接受管道输入？ |true(ByPropertyName)
参数集名称 |AccountNameParameterSet
接受通配符？ |false

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。

## Get-AzureRmMediaServiceKeys

获取媒体服务的密钥。

### 语法

	Get-AzureRmMediaServiceKeys [-ResourceGroupName] <string> [-AccountName] <string>  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 |无
---|---
必需？ |是
位置？ |0
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |无
---|---
必需？ |是
位置？ |1
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。

## Set-AzureRmMediaServiceKey

重新生成媒体服务的主密钥或辅助密钥。

### 语法

	Set-AzureRmMediaServiceKey [-ResourceGroupName] <string> [-AccountName] <string> [-KeyType] <KeyType> {Primary | Secondary}  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 |无
---|---
必需？ |是
位置？ |0
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |无
---|---
必需？ |是
位置？ |1
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-KeyType &lt;密钥类型&gt;**

指定媒体服务的密钥类型。

- 主密钥或辅助密钥

别名 |无
---|---
必需？ |是
位置？ |2
默认值 |无
接受管道输入？ |false
接受通配符？ |false

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。

## Sync-AzureRmMediaServiceStorageKeys

同步与媒体服务关联的存储帐户的存储帐户密钥。

### 语法

	Sync-AzureRmMediaServiceStorageKeys [-ResourceGroupName] <string> [-MediaServiceAccountName] <string>    [-StorageAccountId] <string>  [<CommonParameters>]

### 参数

**-ResourceGroupName &lt;字符串&gt;**

指定此媒体服务所属资源组的名称。

别名 |无
---|---
必需？ |是
位置？ |0
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-AccountName &lt;字符串&gt;**

指定媒体服务的名称。

别名 |无
---|---
必需？ |是
位置？ |1
默认值 |无
接受管道输入？ |true(ByPropertyName)
接受通配符？ |false

**-StorageAccountId &lt;字符串&gt;**

指定与媒体服务关联的存储帐户。

别名 |ID
---|---
必需？ |是
位置？ |2
默认值 |无
接受管道输入？ | true(ByPropertyName)
接受通配符？ |false

**&lt;CommandParameters&gt;**

此 cmdlet 支持以下常见参数：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。

### 输入

输入类型是可以发送到 cmdlet 的对象类型。

### 输出

输出类型是 cmdlet 发出的对象类型。





 

<!---HONumber=Mooncake_1114_2016-->