<properties 
	pageTitle="管理弹性数据库客户端库中的凭据 | Azure" 
	description="如何为弹性数据库应用设置正确的凭据级别（从管理员到只读权限）" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="ddove" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="05/03/2016" 
	wacn.date="05/23/2016"/>

# 用于访问弹性数据库客户端库的凭据

[弹性数据库客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)使用三种不同的凭据来访问[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。使用凭据时，应根据需要尽可能采用最低访问级别。

* **管理凭据**：用于创建或操作分片映射管理器。（请参阅[词汇表](/documentation/articles/sql-database-elastic-scale-glossary/)。） 
* **访问凭据**：用于访问现有分片映射管理器以获取有关分片的信息。
* **连接凭据**：用于连接到分片。 

另请参阅[管理 Azure SQL 数据库的数据库和登录名](/documentation/articles/sql-database-manage-logins/)。
 
## 关于管理凭据

使用管理凭据可以针对操作分片映射的应用程序创建 [**ShardMapManager**](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx) 对象。有关示例，请参阅[使用弹性数据库工具添加分片](/documentation/articles/sql-database-elastic-scale-add-a-shard/)和[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)）。弹性缩放客户端库的用户创建 SQL 用户和 SQL 登录，并确保在全局分片映射数据库以及所有分片数据库上授予每个用户读/写权限。对分片映射执行更改时，这些凭据将用于维护全局分片映射和局部分片映射。例如，使用管理凭据创建分片映射管理器对象（使用 [**GetSqlShardMapManager**](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx)）：

	// Obtain a shard map manager. 
	ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager( 
	        smmAdminConnectionString, 
	        ShardMapManagerLoadPolicy.Lazy 
	); 

变量 **smmAdminConnectionString** 是包含管理凭据的连接字符串。用户 ID 和密码提供对分片映射数据库和单独的分片的读/写访问权限。管理连接字符串还包括服务器名称和数据库名称，以标识全局分片映射数据库。下面是用于实现此目的的典型连接字符串：

	 "Server=<yourserver>.database.chinacloudapi.cn;Database=<yourdatabase>;User ID=<yourmgmtusername>;Password=<yourmgmtpassword>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;” 

请不要使用“用户名@服务器”格式的值 - 只需使用“用户名”格式的值。这是因为凭据必须同时适用于分片映射管理器数据库和各个分片，它们可能位于不同的服务器上。

## 访问凭据
  
在不用于管理分片映射的应用程序中创建分片映射管理器时，使用在全局分片映射上具有只读权限的凭据。在这些凭据下从全局分片映射检索的信息可用于[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)，并且用于在客户端上填充分片映射缓存。通过与如上所示的 **GetSqlShardMapManager** 相同的调用模式提供凭据：

    // Obtain shard map manager. 
    ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager( 
            smmReadOnlyConnectionString, 
            ShardMapManagerLoadPolicy.Lazy
    );  

记下 **smmReadOnlyConnectionString** 的使用，以代表 **non-admin** 用户反映用于此访问的其他凭据的使用：这些凭据不应在全局分片映射上提供写入权限。

## 连接凭据 

当使用 [**OpenConnectionForKey**](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx) 方法访问与某个分片键相关联的分片时，还需使用其他凭据。这些凭据需要提供对驻留在该分片上的局部分片映射表的只读访问权限。若要对分片上的数据依赖路由执行连接验证，需要执行此操作。此代码段允许在使用数据相关路由的情况下进行数据访问：
 
	using (SqlConnection conn = rangeMap.OpenConnectionForKey<int>( 
	targetWarehouse, smmUserConnectionString, ConnectionOptions.Validate)) 

在本示例中，**smmUserConnectionString** 包含用户凭据的连接字符串。对于 Azure SQL DB，下面是用户凭据的典型连接字符串：

	"User ID=<yourusername>; Password=<youruserpassword>; Trusted_Connection=False; Encrypt=True; Connection Timeout=30;”  

与管理员凭据一样，请不要使用“用户名@服务器”格式的值，而应使用“用户名”格式的值。另请注意，连接字符串不包含服务器名称和数据库名称。这是因为，**OpenConnectionForKey** 调用基于键自动将连接指向正确的分片。因此，不需提供数据库名称和服务器名称。

## 另请参阅
[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)

[保护你的 SQL 数据库](/documentation/articles/sql-database-security/)

[弹性数据库作业入门](/documentation/articles/sql-database-elastic-jobs-getting-started/)

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
 

<!---HONumber=Mooncake_0314_2016-->
