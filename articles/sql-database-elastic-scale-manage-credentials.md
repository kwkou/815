<properties 
	pageTitle="管理弹性数据库客户端库中的凭据" 
	description="如何为弹性数据库应用设置正确的凭据级别（从管理员到只读权限）。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="sidneyh" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="08/04/2015" 
	wacn.date="09/15/2015"/>

# 管理弹性数据库客户端库中的凭据

[弹性数据库客户端库](http://go.microsoft.com/?linkid=9862605) 将凭据用于不同类型的操作 - 在创建或操作[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management)、引用现有的分片映射管理器获取有关分片的信息，以及连接到分片时尤其如此。下面讨论了用于这些类型的操作的凭据。 


* **用于分片映射访问的管理凭据**：在针对操作分片映射的应用程序实例化 **ShardMapManager** 对象时，使用管理凭据。弹性缩放客户端库的用户必须创建必需的 SQL 用户和 SQL 登录，并确保在全局分片映射数据库以及所有分片数据库上授予他们读/写权限。对分片映射执行更改时，这些凭据将用于维护全局分片映射和局部分片映射。例如，使用管理凭据实例化分片映射管理器对象，如以下代码所示： 

        // Obtain a shard map manager. 
        ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager( 
                smmAdminConnectionString, 
                ShardMapManagerLoadPolicy.Lazy 
        ); 


     变量 **smmAdminConnectionString** 是包含管理凭据的连接字符串。用户 ID 和密码提供对分片映射数据库和单独的分片的读/写访问权限。管理连接字符串还包括服务器名称和数据库名称，以标识全局分片映射数据库。下面是用于实现此目的的典型连接字符串：

        "Server=<yourserver>.database.chinacloudapi.cn;Database=<yourdatabase>;User ID=<yourmgmtusername>;Password=<yourmgmtpassword>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;” 

     请不要使用“用户名@服务器”格式的用户 ID 值 - 只需使用“用户名”。这是因为凭据必须同时适用于分片映射管理器数据库和各个分片，它们可能位于不同的服务器上。
     
* **用于分片映射管理器访问的用户凭据**：在不用于管理分片映射的应用程序中实例化分片映射管理器时，使用在全局分片映射上具有只读权限的凭据。在这些凭据下从全局分片映射检索的信息可用于[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)，并且用于在客户端上填充分片映射缓存。通过与如上所示的 **GetSqlShardMapManager** 相同的调用模式提供凭据：
 
        // Obtain shard map manager. 
        ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager( 
                smmReadOnlyConnectionString, 
                ShardMapManagerLoadPolicy.Lazy
        );  

     记下 **smmReadOnlyConnectionString** 的使用，以代表**非管理员**用户反映用于此访问的其他凭据的使用 - 这些凭据不应在全局分片映射上提供写入权限。 

* **用于用户数据访问的用户凭据**：当使用 **OpenConnectionForKey** 方法访问与某个键相关联的分片时，还需使用其他凭据。这些凭据需要提供对驻留在该分片上的局部分片映射表的只读访问权限。若要对分片上的数据依赖路由执行连接验证，需要执行此操作。以下代码段演示了在数据依赖路由的上下文中，如何将用户凭据用于用户数据访问：
 
        using (SqlConnection conn = rangeMap.OpenConnectionForKey<int>( 
        targetWarehouse, smmUserConnectionString, ConnectionOptions.Validate)) 

    在本示例中，**smmUserConnectionString** 包含用户凭据的连接字符串。对于 Azure SQL DB，下面是用户凭据的典型连接字符串：

        "User ID=<yourusername>; Password=<youruserpassword>; Trusted_Connection=False; Encrypt=True; Connection Timeout=30;”  

    与管理员凭据一样，请不要使用“用户名@服务器”格式的用户 ID 值 - 只需使用“用户名”。另请注意，连接字符串不包含服务器名称和数据库名称。这是因为，**OpenConnectionForKey** 调用基于键自动将连接指向正确的分片。因此，请勿提供数据库名称和服务器名称。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!---HONumber=69-->