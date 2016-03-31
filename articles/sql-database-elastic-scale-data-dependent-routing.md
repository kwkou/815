<properties 
	pageTitle="数据相关的路由 | Azure" 
	description="如何将 ShardMapManager 用于数据相关的路由（Azure SQL 数据库弹性数据库的一项功能）" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="torsteng" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.date="11/04/2015" 
	wacn.date="01/05/2016"/>

#依赖于数据的路由

**ShardMapManager** 类使 ADO.NET 应用程序能够轻松地将数据库查询和命令指向分片环境中的相应物理数据库。这称为**数据相关的路由**，在使用分片数据库时，它是一种基础模式。将应用程序中使用数据依赖路由的每个特定查询和事务限制为针对每个请求访问单个数据库。

通过使用数据依赖路由，应用程序无需在分片环境中跟踪与不同的数据片相关联的各种连接字符串或数据库位置。但是，[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management)有责任在需要时基于分片映射中的数据以及作为应用程序请求目标的分片键值，将开放连接分发给正确的数据库。（该键通常为 *customer\_id*、*tenant\_id*、*date\_key* 或一些作为数据库请求的基础参数的其他特定标识符）。

## 下载客户端库

若要安装该库，请转到[弹性数据库客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。

## 在数据相关的路由应用程序中使用 ShardMapManager 

对于使用数据相关的路由的应用程序，应在初始化期间使用工厂调用 **GetSQLShardMapManager** 针对每个应用域实例化 **ShardMapManager** 一次。

    ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnnectionString, 
                      ShardMapManagerLoadPolicy.Lazy);
    RangeShardMap<int> customerShardMap = smm.GetRangeShardMap<int>("customerMap"); 

在本示例中，将同时初始化 **ShardMapManager** 以及它所包含的特定 **ShardMap**。

对于本身不操作分片映射的应用程序，在工厂方法中用于获取 **ShardMapManager**（在上面的示例中为 *smmConnectionString*）的凭据在由连接字符串引用的**全局分片映射**数据库上，应仅具有只读权限。这些凭据通常与用于到分片映射管理器的开放连接的凭据不同。另请参阅[使用弹性数据库客户端库中的凭据](/documentation/articles/sql-database-elastic-scale-manage-credentials)。

## 调用数据相关的路由 

方法 **ShardMap.OpenConnectionForKey(key, connectionString, connectionOptions)** 将返回 ADO.Net 连接，该连接可随时基于 **key** 参数的值将命令分发到相应的数据库中。**ShardMapManager** 将分片信息缓存在应用程序中，因此这些请求通常不会针对**全局分片映射**数据库调用数据库查找。

* **key** 参数在分片映射中用作查找键，以确定该请求的相应数据库。 

* **connectionString** 用于仅传递所需连接的用户凭据。此 *connectionString* 中不包含数据库名称或服务器名称，因为该方法将使用 **ShardMap** 确定数据库和服务器。

* **connectionOptions** 枚举用于指示在传送开放连接时是否发生验证。建议使用 **ConnectionOptions.Validate**。在以下环境中，验证可确保基于键值的数据库的缓存查找仍然正确：分片映射可能发生更改，以及可能将行作为拆分或合并操作的结果移动到其他数据库。在将连接传送到应用程序之前，验证将涉及在目标数据库上简要查询局部分片映射（而不是全局分片映射）。

如果针对局部分片映射进行的验证失败（指示缓存不正确），分片映射管理器将查询全局分片映射来获取新的正确值以供查找、更新缓存，以及获取和返回相应的数据库连接。

仅在应用程序处于联机状态时没有按预期进行分片映射更改的情况下，接受 **ConnectionOptions.None**（不验证）。在该情况下，可以假设缓存的值始终正确，并且可以安全地跳过对目标数据库的额外双向验证调用。这可能会减少事务延迟和数据库流量。还可以通过配置文件中的某个值设置 **connectionOptions**，以指示在此期间是否按预期进行了分片更改。

这是一个通过名为 **customerShardMap** 的 **ShardMap** 对象，基于整数键的值 **CustomerID** 使用分片映射管理器执行数据相关的路由的代码示例。

## 示例：数据相关的路由 

    int customerId = 12345; 
    int newPersonId = 4321; 

    // Connection to the shard for that customer ID
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
    }  

请注意，将使用 **OpenConnectionForKey** 方法，它提供了一个新的到正确数据库的已开放连接，而不是使用 **SqlConnection** 构造函数，后跟对连接对象的 **Open()** 调用。此方法中所采用的连接仍然可以充分利用 ADO.Net 连接池。如果一个分片一次可满足事务和请求，则只需在已使用 ADO.Net 的应用程序中进行此唯一修改。

如果应用程序配合 ADO.Net 使用异步编程，则也可以使用方法 **OpenConnectionForKeyAsync**。应用程序的行为等效于 ADO.Net 方法 **Connection.OpenAsync** 的数据相关路由。

## 与暂时性故障处理集成 

在云中开发数据访问应用程序的最佳做法是，确保连接到数据库或查询数据库时的暂时性故障是由应用引起的，并且确保在引发错误之前重试几次这些操作。[暂时性故障处理](http://msdn.microsoft.com/zh-cn/library/dn440719(v=pandp.60).aspx)中讨论了云应用程序的暂时性故障处理。
 
暂时性故障处理在本质上可以与数据依赖路由模式共存。关键需求是重试整个数据访问请求，包括已获取数据相关的路由连接的 **using** 块。可按照如下方式重写上面的示例（请注意突出显示的更改）。

### 示例 - 数据相关的路由与暂时性故障处理 

<pre><code>int customerId = 12345; 
int newPersonId = 4321; 

<span style="background-color:  #FFFF00">Configuration.SqlRetryPolicy.ExecuteAction(() => </span> 
<span style="background-color:  #FFFF00">    { </span>
        // Connection to the shard for that customer ID 
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


当你生成弹性数据库示例应用程序时，将自动下载需要实现暂时性故障处理的程序包。[Enterprise Library - 暂时性故障处理应用程序块](http://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling/)也将单独提供程序包。使用版本 6.0 或更高版本。

## 事务一致性 

确保分片的所有局部操作的事务属性。例如，通过数据依赖路由提交的事务将在目标分片范围内执行以供连接。此时，没有提供用于将多个连接包含在一个事务中的功能，因此对于在分片上执行的操作，没有事务保证。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
 

<!---HONumber=Mooncake_1221_2015-->
