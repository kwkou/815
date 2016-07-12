<properties
	pageTitle="使用 Azure PowerShell 管理 Azure Redis 缓存 | Azure"
	description="了解如何使用 Azure PowerShell 对 Azure Redis 缓存执行管理任务。"
	services="redis-cache"
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="04/27/2016"
	wacn.date="06/29/2016"/>

# 使用 Azure PowerShell 管理 Azure Redis 缓存

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/cache-howto-manage-redis-cache-powershell/)
- [Azure CLI](/documentation/articles/cache-manage-cli/)

本主题说明如何执行创建、更新和缩放 Azure Redis 缓存实例等常见任务、如何重新生成访问密钥，以及如何查看有关缓存的信息。有关 Azure Redis 缓存 PowerShell cmdlet 的完整列表，请参阅 [Azure Redis 缓存 cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt634513.aspx)。

## 先决条件

如果已安装 Azure PowerShell，则必须确保安装的是 Azure PowerShell 版本 1.0.0 或更高版本。可以使用此命令在 Azure PowerShell 命令提示符下查看已安装的 Azure PowerShell 版本。

	Get-Module azure | format-table version

[AZURE.INCLUDE [powershell 预览](../includes/powershell-preview-inline-include.md)]

首先，必须使用以下命令登录到 Azure。

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]

在 Azure 登录对话框中指定 Azure 帐户的电子邮件地址及其密码。

接下来，如果你有多个 Azure 订阅，则需要设置你的 Azure 订阅。若要查看当前订阅的列表，请运行以下命令。

	Get-AzureRmSubscription | sort SubscriptionName | Select SubscriptionName

若要指定订阅，请运行以下命令。在以下示例中，订阅名为 `ContosoSubscription`。

	Select-AzureRmSubscription -SubscriptionName ContosoSubscription

在可以将 Windows PowerShell 与 Azure 资源管理器一起使用之前，你需要具备以下项：

- Windows PowerShell 3.0 版或 4.0 版。若要查找 Windows PowerShell 的版本，请键入：`$PSVersionTable` 并验证 `PSVersion` 的值是否为 3.0 或 4.0。若要安装兼容版本，请参阅 [Windows Management Framework 3.0 ](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855)。

若要获取你在本教程中看到的任何 cmdlet 的详细帮助，请使用 Get-Help cmdlet。

	Get-Help <cmdlet-name> -Detailed

例如，若要获取有关 `New-AzureRmRedisCache` cmdlet 的帮助，请键入：

	Get-Help New-AzureRmRedisCache -Detailed

### 连接到 Azure 中国云

若要连接到 Azure 中国云，请使用以下命令之一。

	Add-AzureRMAccount -EnvironmentName AzureChinaCloud

或

	Add-AzureRmAccount -Environment (Get-AzureRmEnvironment -Name AzureChinaCloud)

若要在 Azure 中国云中创建缓存，请使用以下位置之一。

-	中国东部
-	中国北部

有关 Azure 中国云的详细信息，请参阅[中国 21Vianet 运营的 AzureChinaCloud for Azure](http://www.azure.cn/)。

## Azure Redis 缓存 PowerShell 使用的属性

下表包含使用 Azure PowerShell 创建和管理 Azure Redis 缓存实例时常用的参数的属性和说明。

| 参数 | 说明 | 默认 |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| Name | 缓存的名称 | |
| 位置 | 缓存的位置 | |
| ResourceGroupName | 将在其中创建缓存的资源组名称 | |
| 大小 | 缓存的大小。有效值为：P1、P2、P3、P4、C0、C1、C2、C3、C4、C5、C6、250MB、1GB、2.5GB、6GB、13GB、26GB、53GB | 1GB |
| ShardCount | 在启用群集的情况下创建高级缓存时要创建的分片数目。有效值为：1、2、3、4、5、6、7、8、9、10 | |
| SKU | 指定缓存的 SKU。有效值为：Basic、Standard、Premium | 标准 |
| RedisConfiguration | 指定 maxmemory-delta、maxmemory-policy 和 notify-keyspace-events 的 Redis 配置设置。请注意，maxmemory-delta 和 notify-keyspace-events 只能用于标准和高级缓存。 | |
| EnableNonSslPort | 指出是否启用非 SSL 端口。 | False |
| MaxMemoryPolicy | 此参数已弃用 - 请改用 RedisConfiguration。 | |
| StaticIP | 在 VNET 中托管缓存时，指定缓存在子网中的唯一 IP 地址。如果未提供此值，系统将从子网中为你选择一个。 | |
| 子网 | 在 VNET 中托管缓存时，指定要在其中部署缓存的子网。 | |
| VirtualNetwork | 在 VNET 中托管缓存时，指定要在其中部署缓存的 VNET 的资源 ID。 | |
| KeyType | 指定续订访问密钥时要重新生成哪个访问密钥。有效值为：Primary、Secondary | | | |


## 创建 Redis 缓存

使用 [New-AzureRmRedisCache](https://msdn.microsoft.com/zh-cn/library/azure/mt634517.aspx) cmdlet 创建新的 Azure Redis 缓存实例。

若要查看 `New-AzureRmRedisCache` 的可用参数列表及其说明，请运行以下命令。

	PS C:\> Get-Help New-AzureRmRedisCache -detailed
	
	NAME
	    New-AzureRmRedisCache
	
	SYNOPSIS
	    Creates a new redis cache.
	
	SYNTAX
	    New-AzureRmRedisCache -Name <String> -ResourceGroupName <String> -Location <String> [-RedisVersion <String>]
	    [-Size <String>] [-Sku <String>] [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort
	    <Boolean>] [-ShardCount <Integer>] [-VirtualNetwork <String>] [-Subnet <String>] [-StaticIP <String>]
	    [<CommonParameters>]
	
	DESCRIPTION
	    The New-AzureRmRedisCache cmdlet creates a new redis cache.
	
	PARAMETERS
	    -Name <String>
	        Name of the redis cache to create.
	
	    -ResourceGroupName <String>
	        Name of resource group in which to create the redis cache.
	
	    -Location <String>
	        Location in which to create the redis cache.
	
	    -RedisVersion <String>
	        RedisVersion is deprecated and will be removed in future release.
	
	    -Size <String>
	        Size of the redis cache. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
	        C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.
	
	    -Sku <String>
	        Sku of redis cache. The default value is Standard. Possible values are Basic, Standard and Premium.
	
	    -MaxMemoryPolicy <String>
	        The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
	        MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}
	
	    -RedisConfiguration <Hashtable>
	        All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
	        rdb-backup-frequency, maxmemory-delta, maxmemory-policy, notify-keyspace-events, maxmemory-samples,
	        slowlog-log-slower-than, slowlog-max-len, list-max-ziplist-entries, list-max-ziplist-value,
	        hash-max-ziplist-entries, hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries,
	        zset-max-ziplist-value etc.
	
	    -EnableNonSslPort <Boolean>
	        EnableNonSslPort is used by Azure Redis Cache. If no value is provided, the default value is false and the
	        non-SSL port will be disabled. Possible values are true and false.

	    -ShardCount <Integer>
	        The number of shards to create on a Premium Cluster Cache.
	
	    -VirtualNetwork <String>
	        The exact ARM resource ID of the virtual network to deploy the redis cache in. Example format:
	        /subscriptions/{subid}/resourceGroups/{resourceGroupName}/providers/Microsoft.ClassicNetwork/VirtualNetworks/{vnetName}
	
	    -Subnet <String>
	        Required when deploying a redis cache inside an existing Azure Virtual Network.
	
	    -StaticIP <String>
	        Required when deploying a redis cache inside an existing Azure Virtual Network.
	
	    <CommonParameters>
	        This cmdlet supports the common parameters: Verbose, Debug,
	        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
	        OutBuffer, PipelineVariable, and OutVariable. For more information, see
	        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要使用默认参数创建缓存，请运行以下命令。

	New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "China North"

`ResourceGroupName`、`Name` 和 `Location` 是必需的参数，其余则是可选参数并且有默认值。运行前面的命令会使用指定的名称、位置和资源组创建标准 SKU Azure Redis 缓存实例，其大小为 1 GB，且禁用了非 SSL 端口。

若要创建高级缓存，请指定大小 P1 (6 GB - 60 GB)、P2 (13 GB - 130 GB)、P3 (26 GB - 260 GB) 或 P4 (53 GB - 530 GB)。若要启用群集，使用 `ShardCount` 参数指定分片计数。以下示例将创建包含 3 个分片的 P1 高级缓存。P1 高级缓存的大小为 6 GB。由于我们指定了 3 个分片，因此总大小为 18 GB (3 x 6 GB)。

	New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "China North" -Sku Premium -Size P1 -ShardCount 3

若要指定 `RedisConfiuration` 参数的值，请以键/值对的方式将值括在 `{}` 内，例如 `@{"maxmemory-policy" = "allkeys-random"; "notify-keyspace-events" = "KEA"}`。以下示例将创建标准 1 GB 缓存，其包含 `allkeys-random` maxmemory 策略，以及使用 `KEA` 配置的 keyspace 通知。

	New-AzureRmRedisCache -ResourceGroupName myGroup -Name mycache -Location "China North" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random"; "notify-keyspace-events" = "KEA"}

## 更新 Redis 缓存

使用 [Set-AzureRmRedisCache](https://msdn.microsoft.com/zh-cn/library/azure/mt634518.aspx) cmdlet 更新 Azure Redis 缓存实例。

若要查看 `Set-AzureRmRedisCache` 的可用参数列表及其说明，请运行以下命令。

	PS C:\> Get-Help Set-AzureRmRedisCache -detailed
	
	NAME
	    Set-AzureRmRedisCache
	
	SYNOPSIS
	    Set redis cache updatable parameters.
	
	SYNTAX
	    Set-AzureRmRedisCache -Name <String> -ResourceGroupName <String> [-Size <String>] [-Sku <String>]
	    [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort <Boolean>] [-ShardCount
	    <Integer>] [<CommonParameters>]
	
	DESCRIPTION
	    The Set-AzureRmRedisCache cmdlet sets redis cache parameters.
	
	PARAMETERS
	    -Name <String>
	        Name of the redis cache to update.
	
	    -ResourceGroupName <String>
	        Name of the resource group for the cache.
	
	    -Size <String>
	        Size of the redis cache. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
	        C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.
	
	    -Sku <String>
	        Sku of redis cache. The default value is Standard. Possible values are Basic, Standard and Premium.
	
	    -MaxMemoryPolicy <String>
	        The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
	        MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}
	
	    -RedisConfiguration <Hashtable>
	        All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
	        rdb-backup-frequency, maxmemory-delta, maxmemory-policy, notify-keyspace-events, maxmemory-samples,
	        slowlog-log-slower-than, slowlog-max-len, list-max-ziplist-entries, list-max-ziplist-value,
	        hash-max-ziplist-entries, hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries,
	        zset-max-ziplist-value etc.
	
	    -EnableNonSslPort <Boolean>
	        EnableNonSslPort is used by Azure Redis Cache. The default value is null and no change will be made to the
	        currently configured value. Possible values are true and false.
	
	    -ShardCount <Integer>
	        The number of shards to create on a Premium Cluster Cache.
	
	    <CommonParameters>
	        This cmdlet supports the common parameters: Verbose, Debug,
	        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
	        OutBuffer, PipelineVariable, and OutVariable. For more information, see
	        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

`Set-AzureRmRedisCache` 可用于更新属性，例如 `Size`、`Sku`、`EnableNonSslPort` 和 `RedisConfiguration` 值。

以下命令将更新名为 myCache 的 Redis 缓存的 maxmemory-policy。

	Set-AzureRmRedisCache -ResourceGroupName "myGroup" -Name "myCache" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random"}

<a name="scale"></a>
## 缩放 Redis 缓存

修改 `Size`、`Sku` 或 `ShardCount` 属性时，可以使用 `Set-AzureRmRedisCache` 来缩放 Azure Redis 缓存实例。

>[AZURE.NOTE] 你可以扩展到不同定价层，但有以下限制。
>
>-	不能向上缩放到**高级**缓存，或者从此层向下缩放。
>-	不能从**标准**缓存缩放到**基本**缓存。
>-	可以从**基本**缓存缩放为**标准**缓存，但不能同时更改大小。如果你需要不同大小，则可以执行后续缩放操作以缩放为所需大小。
>-	不能从较大的大小减小为 **C0 (250 MB)** 大小。
>
>有关详细信息，请参阅[如何缩放 Azure Redis 缓存](/documentation/articles/cache-how-to-scale/)。

以下示例演示了如何将名为 `myCache` 的缓存缩放为 2.5 GB 缓存。请注意，此命令适用于基本或标准缓存。

	Set-AzureRmRedisCache -ResourceGroupName myGroup -Name myCache -Size 2.5GB

发出此命令之后，将返回缓存的状态（类似于调用 `Get-AzureRmRedisCache`）。请注意，`ProvisioningState` 为 `Scaling`。

	PS C:\> Set-AzureRmRedisCache -Name myCache -ResourceGroupName myGroup -Size 2.5GB
	
	
	Name               : mycache
	Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/mygroup/providers/Mi
	                     crosoft.Cache/Redis/mycache
	Location           : China East
	Type               : Microsoft.Cache/Redis
	HostName           : mycache.redis.cache.chinacloudapi.cn
	Port               : 6379
	ProvisioningState  : Scaling
	SslPort            : 6380
	RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 150], [notify-keyspace-events, KEA],
	                     [maxmemory-delta, 150]...}
	EnableNonSslPort   : False
	RedisVersion       : 3.0
	Size               : 1GB
	Sku                : Standard
	ResourceGroupName  : mygroup
	PrimaryKey         : ....
	SecondaryKey       : ....
	VirtualNetwork     :
	Subnet             :
	StaticIP           :
	TenantSettings     : {}
	ShardCount         :

缩放操作完成后，`ProvisioningState` 将更改为 `Succeeded`。如果你需要进行后续的缩放操作，例如先从基本缓存更改为标准缓存，然后再更改大小，则必须等到前面操作完成，否则会收到类似于下面的错误。

	Set-AzureRmRedisCache : Conflict: The resource '...' is not in a stable state, and is currently unable to accept the update request.

## 获取有关 Redis 缓存的信息

可以使用[ Get-AzureRmRedisCache](https://msdn.microsoft.com/zh-cn/library/azure/mt634514.aspx) cmdlet 检索有关缓存的信息。

若要查看 `Get-AzureRmRedisCache` 的可用参数列表及其说明，请运行以下命令。

	PS C:\> Get-Help Get-AzureRmRedisCache -detailed
	
	NAME
	    Get-AzureRmRedisCache
	
	SYNOPSIS
	    Gets details about a single cache or all caches in the specified resource group or all caches in the current
	    subscription.
	
	SYNTAX
	    Get-AzureRmRedisCache [-Name <String>] [-ResourceGroupName <String>] [<CommonParameters>]
	
	DESCRIPTION
	    The Get-AzureRmRedisCache cmdlet gets the details about a cache or caches depending on input parameters. If both
	    ResourceGroupName and Name parameters are provided then Get-AzureRmRedisCache will return details about the
	    specific cache name provided.
	
	    If only ResourceGroupName is provided than it will return details about all caches in the specified resource group.
	
	    If no parameters are given than it will return details about all caches the current subscription.
	
	PARAMETERS
	    -Name <String>
	        The name of the cache. When this parameter is provided along with ResourceGroupName, Get-AzureRmRedisCache
	        returns the details for the cache.
	
	    -ResourceGroupName <String>
	        The name of the resource group that contains the cache or caches. If ResourceGroupName is provided with Name
	        then Get-AzureRmRedisCache returns the details of the cache specified by Name. If only the ResourceGroup
	        parameter is provided, then details for all caches in the resource group are returned.
	
	    <CommonParameters>
	        This cmdlet supports the common parameters: Verbose, Debug,
	        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
	        OutBuffer, PipelineVariable, and OutVariable. For more information, see
	        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要返回当前订阅中所有缓存的相关信息，请运行不带任何参数的 `Get-AzureRmRedisCache`。

	Get-AzureRmRedisCache

若要返回特定资源组中所有缓存的相关信息，请结合 `ResourceGroupName` 参数运行 `Get-AzureRmRedisCache`。

	Get-AzureRmRedisCache -ResourceGroupName myGroup

若要返回特定缓存的相关信息，请运行 `Get-AzureRmRedisCache`，并使用包含缓存名称的 `Name` 参数，和包含该缓存所在资源组的 `ResourceGroupName` 参数。

	PS C:\> Get-AzureRmRedisCache -Name myCache -ResourceGroupName myGroup
	
	Name               : mycache
	Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/myGroup/providers/Mi
	                     crosoft.Cache/Redis/mycache
	Location           : China East
	Type               : Microsoft.Cache/Redis
	HostName           : mycache.redis.cache.chinacloudapi.cn
	Port               : 6379
	ProvisioningState  : Succeeded
	SslPort            : 6380
	RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 62], [notify-keyspace-events, KEA],
	                     [maxclients, 1000]...}
	EnableNonSslPort   : False
	RedisVersion       : 3.0
	Size               : 1GB
	Sku                : Standard
	ResourceGroupName  : myGroup
	VirtualNetwork     :
	Subnet             :
	StaticIP           :
	TenantSettings     : {}
	ShardCount         :

## 检索 Redis 缓存的访问密钥

若要检索缓存的访问密钥，可以使用 [Get AzureRmRedisCacheKey](https://msdn.microsoft.com/zh-cn/library/azure/mt634516.aspx) cmdlet。

若要查看 `Get-AzureRmRedisCacheKey` 的可用参数列表及其说明，请运行以下命令。

	PS C:\> Get-Help Get-AzureRmRedisCacheKey -detailed
	
	NAME
	    Get-AzureRmRedisCacheKey
	
	SYNOPSIS
	    Gets the accesskeys for the specified redis cache.
	
	
	SYNTAX
	    Get-AzureRmRedisCacheKey -Name <String> -ResourceGroupName <String> [<CommonParameters>]
	
	DESCRIPTION
	    The Get-AzureRmRedisCacheKey cmdlet gets the access keys for the specified cache.
	
	PARAMETERS
	    -Name <String>
	        Name of the redis cache.
	
	    -ResourceGroupName <String>
	        Name of the resource group for the cache.
	
	    <CommonParameters>
	        This cmdlet supports the common parameters: Verbose, Debug,
	        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
	        OutBuffer, PipelineVariable, and OutVariable. For more information, see
	        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

若要检索缓存的密钥，请调用 `Get-AzureRmRedisCacheKey` cmdlet，并传递缓存的名称以及包含该缓存的资源组的名称。

	PS C:\> Get-AzureRmRedisCacheKey -Name myCache -ResourceGroupName myGroup
	
	PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
	SecondaryKey : ABhfB757JgjIgt785JgKH9865eifmekfnn649303JKL=

## 重新生成 Redis 缓存的访问密钥

若要重新生成缓存的访问密钥，可以使用 [New-AzureRmRedisCacheKey](https://msdn.microsoft.com/zh-cn/library/azure/mt634512.aspx) cmdlet。

若要查看 `New-AzureRmRedisCacheKey` 的可用参数列表及其说明，请运行以下命令。

	PS C:\> Get-Help New-AzureRmRedisCacheKey -detailed
	
	NAME
	    New-AzureRmRedisCacheKey
	
	SYNOPSIS
	    Regenerates the access key of a redis cache.
	
	SYNTAX
	    New-AzureRmRedisCacheKey -Name <String> -ResourceGroupName <String> -KeyType <String> [-Force] [<CommonParameters>]
	
	DESCRIPTION
	    The New-AzureRmRedisCacheKey cmdlet regenerate the access key of a redis cache.
	
	PARAMETERS
	    -Name <String>
	        Name of the redis cache.
	
	    -ResourceGroupName <String>
	        Name of the resource group for the cache.
	
	    -KeyType <String>
	        Specifies whether to regenerate the primary or secondary access key. Possible values are Primary or Secondary.
	
	    -Force
	        When the Force parameter is provided, the specified access key is regenerated without any confirmation prompts.
	
	    <CommonParameters>
	        This cmdlet supports the common parameters: Verbose, Debug,
	        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
	        OutBuffer, PipelineVariable, and OutVariable. For more information, see
	        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
	
若要重新生成缓存的主要或辅助密钥，请调用 `New-AzureRmRedisCacheKey` cmdlet，传入名称和资源组，并为 `KeyType` 参数指定 `Primary` 或 `Secondary`。以下示例将重新生成缓存的辅助访问密钥。

	PS C:\> New-AzureRmRedisCacheKey -Name myCache -ResourceGroupName myGroup -KeyType Secondary
	
	Confirm
	Are you sure you want to regenerate Secondary key for redis cache 'myCache'?
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
	
	
	PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
	SecondaryKey : c53hj3kh4jhHjPJk8l0jji785JgKH9865eifmekfnn6=

## 删除 Redis 缓存

若要删除 Redis 缓存，请使用 [Remove-AzureRmRedisCache](https://msdn.microsoft.com/zh-cn/library/azure/mt634515.aspx) cmdlet。

若要查看 `Remove-AzureRmRedisCache` 的可用参数列表及其说明，请运行以下命令。

	PS C:\> Get-Help Remove-AzureRmRedisCache -detailed
	
	NAME
	    Remove-AzureRmRedisCache
	
	SYNOPSIS
	    Remove redis cache if exists.
	
	SYNTAX
	    Remove-AzureRmRedisCache -Name <String> -ResourceGroupName <String> [-Force] [-PassThru] [<CommonParameters>
	
	DESCRIPTION
	    The Remove-AzureRmRedisCache cmdlet removes a redis cache if it exists.
	
	PARAMETERS
	    -Name <String>
	        Name of the redis cache to remove.
	
	    -ResourceGroupName <String>
	        Name of the resource group of the cache to remove.
	
	    -Force
	        When the Force parameter is provided, the cache is removed without any confirmation prompts.
	
	    -PassThru
	        By default Remove-AzureRmRedisCache removes the cache and does not return any value. If the PassThru par
	        is provided then Remove-AzureRmRedisCache returns a boolean value indicating the success of the operatio
	
	    <CommonParameters>
	        This cmdlet supports the common parameters: Verbose, Debug,
	        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
	        OutBuffer, PipelineVariable, and OutVariable. For more information, see
	        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

以下示例将删除名为 `myCache` 的缓存。

	PS C:\> Remove-AzureRmRedisCache -Name myCache -ResourceGroupName myGroup
	
	Confirm
	Are you sure you want to remove redis cache 'myCache'?
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y

<a name="classic"></a>
## 使用 PowerShell 经典部署模型管理 Azure Redis 缓存实例

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]

以下脚本演示了如何使用经典部署模型创建、更新和删除 Azure Redis 缓存。
		
		$VerbosePreference = "Continue"

    	# Create a new cache with date string to make name unique.
		$cacheName = "MovieCache" + $(Get-Date -Format ('ddhhmm'))
		$location = "China North"
		$resourceGroupName = "Default-Web-ChinaNorth"
		
		$movieCache = New-AzureRedisCache -Location $location -Name $cacheName  -ResourceGroupName $resourceGroupName -Size 250MB -Sku Basic
		
		# Wait until the Cache service is provisioned.
		
		for ($i = 0; $i -le 60; $i++)
		{
		    Start-Sleep -s 30
		    $cacheGet = Get-AzureRedisCache -ResourceGroupName $resourceGroupName -Name $cacheName
		    if ([string]::Compare("succeeded", $cacheGet[0].ProvisioningState, $True) -eq 0)
		    {
		        break
		    }
		    If($i -eq 60)
		    {
		        exit
		    }
		}
		
		# Update the access keys.
		
		Write-Verbose "PrimaryKey: $($movieCache.PrimaryKey)"
		New-AzureRedisCacheKey -KeyType "Primary" -Name $cacheName  -ResourceGroupName $resourceGroupName -Force
		$cacheKeys = Get-AzureRedisCacheKey -ResourceGroupName $resourceGroupName  -Name $cacheName
		Write-Verbose "PrimaryKey: $($cacheKeys.PrimaryKey)"
		
		# Use Set-AzureRedisCache to set Redis cache updatable parameters.
		# Set the memory policy to Least Recently Used.
		
		Set-AzureRedisCache -Name $cacheName -ResourceGroupName $resourceGroupName -RedisConfiguration @{"maxmemory-policy" = "AllKeys-LRU"}
		
		# Delete the cache.
		
		Remove-AzureRedisCache -Name $movieCache.Name -ResourceGroupName $movieCache.ResourceGroupName  -Force

## 后续步骤

若要了解有关将 Windows PowerShell 与 Azure 配合使用的详细信息，请参阅以下资源：

- [MSDN 上的 Azure Redis 缓存 cmdlet 文档](https://msdn.microsoft.com/zh-cn/library/azure/mt634513.aspx)
- [Azure 资源管理器 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt125356.aspx)：了解如何在 AzureResourceManager 模块中使用这些 cmdlet。
- [Azure 博客](/blog/)：了解 Azure 中的新功能。
- [Windows PowerShell 博客](http://blogs.msdn.com/powershell)：了解 Windows PowerShell 中的新功能。
- [“你好，脚本编写专家！” 博客](http://blogs.technet.com/b/heyscriptingguy/)：从 Windows PowerShell 社区获取实用提示和技巧。

<!---HONumber=Mooncake_0321_2016-->