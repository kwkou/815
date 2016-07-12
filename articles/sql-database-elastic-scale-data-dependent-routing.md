<properties 
	pageTitle="数据相关的路由 | Azure" 
	description="如何将 .NET 应用中的 ShardMapManager 类用于数据相关的路由（Azure SQL 数据库弹性数据库的一项功能）" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="torsteng" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="05/03/2016" 
	wacn.date="05/23/2016"/>

#依赖于数据的路由

数据相关的路由是使用查询中的数据将请求路由到相应数据库的功能。在使用分片数据库时，它是一种基础模式。请求上下文也可用于路由请求，尤其是当分片键不是查询的一部分时。将应用程序中使用数据依赖路由的每个特定查询和事务限制为针对每个请求访问单一数据库。对于 SQL Azure 数据库弹性工具，这种路由是通过 ADO.NET 应用程序中的 **[ShardMapManager 类](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx)** 实现的。

应用程序无需在分片环境中跟踪与不同的数据片相关联的各种连接字符串或数据库位置。但是，[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management/)在需要时基于分片映射中的数据以及作为应用程序请求目标的分片键值，将开放连接分发给正确的数据库。（该键通常为 *customer\_id*、*tenant\_id*、*date\_key* 或一些作为数据库请求的基础参数的其他特定标识符）。

有关详细信息，请参阅[使用数据相关的路由缩放 SQL Server](https://technet.microsoft.com/zh-cn/library/cc966448.aspx)。

## 下载客户端库

若要获取该类，请安装[弹性数据库客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)。

## 在数据相关的路由应用程序中使用 ShardMapManager 

应用程序应使用工厂调用 **[GetSQLShardMapManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx)**，在初始化期间实例化 ShardMapManager。在本示例中，将同时初始化 **ShardMapManager** 以及它所包含的特定 **ShardMap**。本例演示 GetSqlShardMapManager 和 [GetRangeShardMap](https://msdn.microsoft.com/zh-cn/library/azure/dn824173.aspx) 方法。

    ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnnectionString, 
                      ShardMapManagerLoadPolicy.Lazy);
    RangeShardMap<int> customerShardMap = smm.GetRangeShardMap<int>("customerMap"); 

### 尽可能使用最低特权凭据来获取分片映射

如果应用程序本身无法处理分片映射，在工厂方法中使用的凭据应该对全局分片映射数据库仅有只读权限。这些凭据通常与用于到分片映射管理器的开放连接的凭据不同。另请参阅[用于访问弹性数据库客户端库的凭据](/documentation/articles/sql-database-elastic-scale-manage-credentials/)。

## 调用 OpenConnectionForKey 方法

 **[ShardMap.OpenConnectionForKey 方法](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx))** 将返回 ADO.Net 连接，该连接可随时基于 **key** 参数的值将命令分发到相应的数据库中。**ShardMapManager** 将分片信息缓存在应用程序中，因此这些请求通常不会针对**全局分片映射**数据库调用数据库查找。

	// Syntax: 
	public SqlConnection OpenConnectionForKey<TKey>(
		TKey key,
		string connectionString,
		ConnectionOptions options
	)


* **key** 参数在分片映射中用作查找键，以确定该请求的相应数据库。 

* **connectionString** 用于仅传递所需连接的用户凭据。此 *connectionString* 中不包含数据库名称或服务器名称，因为该方法将使用 **ShardMap** 确定数据库和服务器。

* 在分片映射可能会发生更改并且行可能会由于拆分或合并操作而移到其他数据库的环境中， **[connectionOptions](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.connectionoptions.aspx)** 应设置为  **ConnectionOptions.Validate** 。在将连接传送到应用程序之前，这涉及在目标数据库上简要查询局部分片映射（而不是全局分片映射）。

如果针对局部分片映射进行的验证失败（指示缓存不正确），分片映射管理器将查询全局分片映射来获取新的正确值以供查找、更新缓存，以及获取和返回相应的数据库连接。

仅在应用程序处于联机状态时没有按预期进行分片映射更改的情况下，才使用 **ConnectionOptions.None** 。在该情况下，可以假设缓存的值始终正确，并且可以安全地跳过对目标数据库的额外双向验证调用。这可以减少数据库流量。还可以通过配置文件中的某个值设置 **connectionOptions**，以指示在此期间是否按预期进行了分片更改。

本示例使用整数键 **CustomerID** 的值，并使用名为 **customerShardMap** 的 **ShardMap** 对象。

    int customerId = 12345; 
    int newPersonId = 4321; 

    // Connect to the shard for that customer ID. No need to call a SqlConnection 
	// constructor followed by the Open method.
    using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, 
        Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate)) 
    { 
        // Execute a simple command. 
        SqlCommand cmd = conn.CreateCommand(); 
        cmd.CommandText = @"UPDATE Sales.Customer 
                            SET PersonID = @newPersonID 
                            WHERE CustomerID = @customerID"; 

        cmd.Parameters.AddWithValue("@customerID", customerId); 
        cmd.Parameters.AddWithValue("@newPersonID", newPersonId); 
        cmd.ExecuteNonQuery(); 
    }  

 **OpenConnectionForKey** 方法返回与正确数据库建立的、新的且已打开的连接。此方法中所采用的连接仍然可以充分利用 ADO.Net 连接池。如果一个分片一次可满足事务和请求，则只需在已使用 ADO.Net 的应用程序中进行此唯一修改。

如果应用程序配合 ADO.Net 使用异步编程，则也可以使用 **[OpenConnectionForKeyAsync 方法](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkeyasync.aspx)**。应用程序的行为等效于 ADO.Net 方法 **[Connection.OpenAsync](https://msdn.microsoft.com/zh-cn/library/hh223688(v=vs.110).aspx)** 方法)的数据相关路由。

## 与暂时性故障处理集成 

在云中开发数据访问应用程序的最佳做法是，确保暂时性故障是由应用引起的，并且确保在引发错误之前重试几次这些操作。[暂时性故障处理](https://msdn.microsoft.com/zh-cn/library/dn440719(v=pandp.60).aspx))中讨论了云应用程序的暂时性故障处理。
 
暂时性故障处理在本质上可以与数据依赖路由模式共存。关键需求是重试整个数据访问请求，包括已获取数据相关的路由连接的 **using** 块。可按照如下方式重写上面的示例（请注意突出显示的更改）。

### 示例 - 数据相关的路由与暂时性故障处理 

<pre><code>int customerId = 12345; 
int newPersonId = 4321; 

<span style="background-color:  #FFFF00">Configuration.SqlRetryPolicy.ExecuteAction(() => </span> 
<span style="background-color:  #FFFF00">    { </span>
        // Connect to the shard for a customer ID.
        using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, 
        Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate)) 
        { 
            // Execute a simple command 
            SqlCommand cmd = conn.CreateCommand(); 

            cmd.CommandText = @"UPDATE Sales.Customer 
                            SET PersonID = @newPersonID 
                            WHERE CustomerID = @customerID"; 

            cmd.Parameters.AddWithValue("@customerID", customerId); 
            cmd.Parameters.AddWithValue("@newPersonID", newPersonId); 
            cmd.ExecuteNonQuery(); 

            Console.WriteLine("Update completed"); 
        } 
<span style="background-color:  #FFFF00">    }); </span> 
</code></pre>


当你生成弹性数据库示例应用程序时，将自动下载需要实现暂时性故障处理的程序包。[Enterprise Library - 暂时性故障处理应用程序块](http://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling)也将单独提供程序包。使用版本 6.0 或更高版本。

## 事务一致性 

确保分片的所有局部操作的事务属性。例如，通过数据依赖路由提交的事务将在目标分片范围内执行以供连接。此时，没有提供用于将多个连接包含在一个事务中的功能，因此对于在分片上执行的操作，没有事务保证。

## 后续步骤
若要分离分片或重新附加分片，请参阅[使用 RecoveryManager 类解决分片映射问题](/documentation/articles/sql-database-elastic-database-recovery-manager/)

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
 

<!---HONumber=Mooncake_0314_2016-->
