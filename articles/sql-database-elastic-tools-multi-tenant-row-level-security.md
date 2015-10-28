<properties 
	title="Multi-tenant applications with elastic database tools and row-level security" 
	pageTitle="具有弹性数据库工具和行级安全性的多租户应用程序" 
	description="了解如何将弹性数据库工具和行级安全性一起使用，在 Azure SQL Database 上构建具有高度可伸缩性数据层、支持多租户分片的应用程序。" 
	metaKeywords="azure sql database elastic tools multi tenant row level security rls" 
	services="sql-database" documentationCenter=""  
	manager="jeffreyg" 
	authors="tmullaney"/>

<tags 
	ms.service="sql-database" 
	ms.date="08/19/2015" 
	wacn.date="09/15/2015" />

# 具有弹性数据库工具和行级安全性的多租户应用程序 

[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started)和[行级安全性 (RLS)](https://msdn.microsoft.com/zh-cn/library/dn765131) 提供了一组强大功能，让你灵活高效地缩放装有 Azure SQL Database 的多租户应用程序与的数据层。本文将演示如何使用 **ADO.NET SqlClient** 和/或**实体框架**，同时运用这些技术来构建具有高度可伸缩性数据层、支持多租户分片的应用程序。

* **弹性数据库工具**可让开发人员使用一组 .NET 库和 Azure 服务模板通过行业标准分片实践扩大应用程序的数据层。使用弹性数据库客户端库管理分片有助于自动化和简化通常与分片关联的许多基础结构任务。 

* **行级安全性**可让开发人员使用安全策略来筛选掉不属于执行查询的租户的行，从而将多个租户的数据存储在同一个数据库中。集中化数据库而不是应用程序中的 RLS 访问逻辑可以简化维护，降低由于应用程序代码库不断增长而带来的出错风险。RLS 需要最新的 [Azure SQL Database 更新版 (V12)](/documentation/articles/sql-database-preview-whats-new)。

将这些功能一起使用，应用程序可以在同一个分片数据库中存储多个租户的数据，从而带来成本节约和效率提高的好处。同时，应用程序仍然能够为需要更严格性能保证的“高级”租户提供隔离的单租户分片，因为多租户分片不保证在租户之间平衡分配资源。

简单而言，弹性数据库客户端库的[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing) API 会自动将租户连接到包含其分片键（通常为“TenantId”）的正确分片数据库。连接后，数据库中的 RLS 安全策略可确保租户只能访问包含其 TenantId 的行。假设条件是所有表都包含一个 TenantId 列用于指示哪些行属于每个租户。

![应用程序体系结构博客][1]

## 下载示例项目

### 先决条件
* 使用 Visual Studio（2012 或更高版本） 
* 创建三个 Azure SQL 数据库 
* 下载示例项目：[Azure SQL 的弹性数据库工具 - 多租户分片](http://go.microsoft.com/?linkid=9888163)
  * 在 **Program.cs** 的开头填入数据库的信息 

此项目通过添加对多租户分片数据库的支持，扩展了 [Azure SQL 的弹性数据库工具 - 实体框架集成](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio)中所述的项目。它将构建一个用于创建博客和文章的简单控制台应用程序，其中包含四个租户和两个多租户分片数据库，如上图中所示。

构建并运行应用程序。这会引导弹性数据库工具的分片映射管理器，并运行以下测试：

1. 使用实体框架和 LINQ 创建新博客，然后显示每个租户的所有博客文章
2. 使用 ADO.NET SqlClient 显示租户的所有博客文章
3. 尝试插入错误租户的博客，以验证是否会引发错误  

请注意，由于 RLS 尚未在分片数据库中启用，其中的每个测试都会揭露一个问题：租户能够看到不属于他们的博客，并且系统不会阻止应用程序插入错误租户的博客。本文的余下部分将介绍如何通过使用 RLS 强制隔离租户来解决这些问题。执行以下两个步骤：

1. **应用程序层**：打开连接后，修改应用程序代码以始终将 CONTEXT\_INFO 设置为当前 TenantId。示例项目已执行此操作。 
2. **数据层**：在每个分片数据库中创建一个 RLS 安全策略，以根据 CONTEXT\_INFO 的值筛选行。你需要针对每个分片数据库执行此操作，否则不会筛选多租户分片中的行。 


## 步骤 1) 应用程序层：将 CONTEXT\_INFO 设置为 TenantId

使用弹性数据库客户端库的数据相关路由 API 连接到分片数据库后，应用程序仍然需要让数据库知道哪个 TenantId 正在使用该连接，以便 RLS 安全策略可以筛选出属于其他租户的行。传递此信息的建议方法是将 [CONTEXT\_INFO](https://msdn.microsoft.com/zh-cn/library/ms180125) 设置为该连接的当前 TenantId。请注意，在 Azure SQL 数据库上，CONTEXT\_INFO 中预先填充了特定于会话的 GUID，因此你*必须*先将 CONTEXT\_INFO 设置为正确的 TenantId，然后才能对新连接执行任何查询，以确保不会无意中泄漏任何行。

### 实体框架

对于使用实体框架的应用程序，最简单的方法是根据[使用 EF DbContext 进行数据相关的路由](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio#data-dependent-routing-using-ef-dbcontext)中所述，在 ElasticScaleContext 重写中设置 CONTEXT\_INFO。在返回通过数据相关路由中转的连接之前，只需创建并执行一个 SqlCommand，用于将 CONTEXT\_INFO 设置为针对该连接指定的 shardingKey (TenantId)。这样，只需编写代码一次就能设置 CONTEXT\_INFO。

```
// ElasticScaleContext.cs 
// ... 
// C'tor for data dependent routing. This call will open a validated connection routed to the proper 
// shard by the shard map manager. Note that the base class c'tor call will fail for an open connection 
// if migrations need to be done and SQL credentials are used. This is the reason for the  
// separation of c'tors into the DDR case (this c'tor) and the internal c'tor for new shards. 
public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
    : base(OpenDDRConnection(shardMap, shardingKey, connectionStr), true /* contextOwnsConnection */)
{
}

public static SqlConnection OpenDDRConnection(ShardMap shardMap, T shardingKey, string connectionStr)
{
    // No initialization
    Database.SetInitializer<ElasticScaleContext<T>>(null);

    // Ask shard map to broker a validated connection for the given key
    SqlConnection conn = null;
    try
    {
        conn = shardMap.OpenConnectionForKey(shardingKey, connectionStr, ConnectionOptions.Validate);

        // Set CONTEXT_INFO to shardingKey to enable Row-Level Security filtering
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText = @"SET CONTEXT_INFO @shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", shardingKey);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }

        throw;
    }
} 
// ... 
```

现在，每当调用 ElasticScaleContext 时，会自动将 CONTEXT\_INFO 设置为指定的 TenantId：

```
// Program.cs 
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() => 
{   
	using (var db = new ElasticScaleContext<int>(sharding.ShardMap, tenantId, connStrBldr.ConnectionString))   
	{     
		var query = from b in db.Blogs
	                orderby b.Name
	                select b;
		
		Console.WriteLine("All blogs for TenantId {0}:", tenantId);     
		foreach (var item in query)     
		{       
			Console.WriteLine(item.Name);     
		}   
	} 
}); 
```

### ADO.NET SqlClient 

对于使用 ADO.NET SqlClient 的应用程序，建议的方法是围绕 ShardMap.OpenConnectionForKey() 创建一个包装函数，用于在返回连接之前将 CONTEXT\_INFO 自动设置为正确的 TenantId。为确保始终正确设置 CONTEXT\_INFO，应只使用此包装函数打开连接。

```
// Program.cs
// ...

// Wrapper function for ShardMap.OpenConnectionForKey() that automatically sets CONTEXT_INFO to the correct
// tenantId before returning a connection. As a best practice, you should only open connections using this 
// method to ensure that CONTEXT_INFO is always set before executing a query.
public static SqlConnection OpenConnectionForTenant(ShardMap shardMap, int tenantId, string connectionStr)
{
    SqlConnection conn = null;
    try
    {
        // Ask shard map to broker a validated connection for the given key
        conn = shardMap.OpenConnectionForKey(tenantId, connectionStr, ConnectionOptions.Validate);

        // Set CONTEXT_INFO to shardingKey to enable Row-Level Security filtering
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText = @"SET CONTEXT_INFO @shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", tenantId);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }

        throw;
    }
}

// ...

// Example query via ADO.NET SqlClient
// If row-level security is enabled, only Tenant 4's blogs will be listed
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (SqlConnection conn = OpenConnectionForTenant(sharding.ShardMap, tenantId4, connStrBldr.ConnectionString))
    {
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText = @"SELECT * FROM Blogs";

        Console.WriteLine("--\nAll blogs for TenantId {0} (using ADO.NET SqlClient):", tenantId4);
        SqlDataReader reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine("{0}", reader["Name"]);
        }
    }
});

```

## 步骤 2) 数据层：创建行级安全策略和约束 

### 创建安全策略来筛选 SELECT、UPDATE 和 DELETE 查询 

既然应用程序在查询之前会将 CONTEXT\_INFO 设置为当前 TenantId，RLS 安全策略就可以筛选查询并排除具有不同 TenantId 的行。

RLS 在 T-SQL 中实现：用户定义的谓词函数定义筛选逻辑，安全策略将此函数绑定到任意数量的表。对于此项目，谓词函数只会验证应用程序（而不是其他某个 SQL 用户）是否已连接到数据库，并且 CONTEXT\_INFO 的值是否与给定行的 TenantId 相匹配。允许满足这些条件的行通过筛选器以执行 SELECT、UPDATE 和 DELETE 查询。如果尚未设置 CONTEXT\_INFO，将不返回任何行。

若要启用 RLS，请使用 Visual Studio (SSDT)、SSMS 或项目中包含的 PowerShell 脚本对所有分片执行以下 T-SQL（或者，如果你正在使用[弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-overview)，可以使用它来对所有分片自动执行此 T-SQL）：

```
CREATE SCHEMA rls -- separate schema to organize RLS objects 
GO

CREATE FUNCTION rls.fn_tenantAccessPredicate(@TenantId int)     
	RETURNS TABLE     
	WITH SCHEMABINDING
AS
	RETURN SELECT 1 AS fn_accessResult          
		WHERE DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo') -- the user in your application’s connection string (dbo is only for demo purposes!)         
		AND CONVERT(int, CONVERT(varbinary(4), CONTEXT_INFO())) = @TenantId -- @TenantId (int) is 4 bytes 
GO

CREATE SECURITY POLICY rls.tenantAccessPolicy
	ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
	ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts
GO 
```

> [AZURE.TIP]对于需要在数百个表中添加谓词函数的更复杂项目，你可以使用一个帮助器存储过程，通过在架构中的所有表内添加谓词，来自动生成安全策略。请参阅[向所有表应用行级安全性 – 帮助器脚本（博客）](http://blogs.msdn.com/b/sqlsecurity/archive/2015/03/31/apply-row-level-security-to-all-tables-helper-script)。

如果以后添加了新表，只需更改安全策略，并在新表中添加筛选器谓词：

```
ALTER SECURITY POLICY rls.tenantAccessPolicy     
	ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable 
GO 
```

现在如果你再次运行示例应用程序，租户将无法看到不属于他们的行。

### 添加 check 约束以阻止插入和更新错误的租户

目前，RLS 安全策略不会阻止应用程序意外插入错误 TenantId 的行，或者将可见行的 TenantId 更新为新值。对于某些应用程序（例如，只读的报告应用程序），这不是一个问题。但是，由于此应用程序允许租户插入新博客，因此有必要创建额外的保护机制，以便在应用程序代码错误地尝试插入或更新违反筛选器谓词的行时引发错误。根据[行级安全性：阻止未授权的插入（博客）](http://blogs.msdn.com/b/sqlsecurity/archive/2015/03/23/row-level-security-blocking-unauthorized-inserts)中所述，建议的解决方法是在每个表中创建一个 check 约束，以强制对插入和更新操作执行相同的 RLS 筛选器谓词。

若要添加 check 约束，请使用 SSMS、SSDT 或上述包含的 PowerShell 脚本（或弹性数据库作业）对分片执行以下 T-SQL：

```
-- Create a scalar version of the predicate function for use in check constraints 
CREATE FUNCTION rls.fn_tenantAccessPredicateScalar(@TenantId int)     
	RETURNS bit 
AS     
	BEGIN     
		IF EXISTS( SELECT 1 FROM rls.fn_tenantAccessPredicate(@TenantId) )         
			RETURN 1     
		RETURN 0 
	END 
GO 

-- Add the function as a check constraint on all sharded tables 
ALTER TABLE Blogs     
	WITH NOCHECK -- don't check data already in table     
	ADD CONSTRAINT chk_blocking_Blogs -- needs a unique name     
	CHECK( rls.fn_tenantAccessPredicateScalar(TenantId) = 1 ) 
GO

ALTER TABLE Posts     
	WITH NOCHECK     
	ADD CONSTRAINT chk_blocking_Posts     
	CHECK( rls.fn_tenantAccessPredicateScalar(TenantId) = 1 ) 
GO 
```

现在，应用程序只能插入当前已连接到分片数据库的租户的行，而不能插入属于其他租户的行。同样，应用程序无法将可见行更新为包含不同的 TenantId。如果应用程序尝试执行上述任一操作，将会引发 DbUpdateException。


### 添加默认约束以自动填充 INSERT 的 TenantId 

除了使用 check 约束来阻止插入错误的租户以外，你还可以在每个表中添加一个默认约束，以便在插入行时，自动使用 CONTEXT\_INFO 的当前值填充 TenantId。例如：

```
-- Create default constraints to auto-populate TenantId with the value of CONTEXT_INFO for inserts 
ALTER TABLE Blogs     
	ADD CONSTRAINT df_TenantId_Blogs      
	DEFAULT CONVERT(int, CONVERT(varbinary(4), CONTEXT_INFO())) FOR TenantId 
GO

ALTER TABLE Posts     
	ADD CONSTRAINT df_TenantId_Posts      
	DEFAULT CONVERT(int, CONVERT(varbinary(4), CONTEXT_INFO())) FOR TenantId 
GO 
```

现在，应用程序在插入行时不需要指定 TenantId：

```
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() => 
{   
	using (var db = new ElasticScaleContext<int>(sharding.ShardMap, tenantId, connStrBldr.ConnectionString))
	{
		var blog = new Blog { Name = name }; // default constraint sets TenantId automatically     
		db.Blogs.Add(blog);     
		db.SaveChanges();   
	} 
}); 
```

> [AZURE.NOTE]如果你对实体框架项目使用默认约束，则不建议在 EF 数据模型中包括 TenantId 列。这是因为实体框架查询会自动提供默认值，而这些值会重写 T-SQL 中创建的、使用 CONTEXT\_INFO 的默认约束。举例来说，若要在示例项目中使用默认约束，你应该从 DataClasses.cs 中删除 TenantId（并在 Package Manager Console 中运行 Add-Migration），然后使用 T-SQL 来确保该字段仅存在于数据库表中。这样，在插入数据时，EF 不会自动提供错误的默认值。

### （可选）启用“超级用户”来访问所有行
有些应用程序可能需要创建一个能够访问所有行“超级用户”，例如，为了跨所有分片上的所有租户来生成报告，或在涉及到数据库之间移动租户行的分片上执行拆分/合并操作。为此，你应该在每个分片数据库中创建新的 SQL 用户（在本例中为 “superuser”）。然后使用新的谓词函数更改安全策略，以允许此用户访问所有行：

```
-- New predicate function that adds superuser logic
CREATE FUNCTION rls.fn_tenantAccessPredicateWithSuperUser(@TenantId int)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult 
        WHERE 
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo') -- note, should not be dbo!
            AND CONVERT(int, CONVERT(varbinary(4), CONTEXT_INFO())) = @TenantId
        ) 
        OR
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('superuser')
        )
GO

-- Atomically swap in the new predicate function on each table
ALTER SECURITY POLICY rls.tenantAccessPolicy
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts
GO
```


### 维护 

* **添加新分片**：必须对所有新分片执行 T-SQL 脚本以启用 RLS（并添加 check 约束），否则，将不会筛选针对这些分片的查询。

* **添加新表**：每当创建新表时，你必须将筛选器谓词添加到所有分片上的安全策略，否则不会筛选对新表的查询。根据[自动向新建的表应用行级安全性（博客）](http://blogs.msdn.com/b/sqlsecurity/archive/2015/05/22/apply-row-level-security-automatically-to-newly-created-tables.aspx)中所述，此操作可以使用 DDL 触发器来自动完成。


## 摘要 

可将弹性数据库工具和行级安全性一起使用，以扩大支持多租户和单租户分片的应用程序的数据层。使用多租户分片可以更有效地存储数据（尤其是对于大量租户只有几行数据的情况），而使用单租户分片可以支持性能和隔离要求更严格的高级租户。有关详细信息，请参阅 MSDN 上的[弹性数据库工具文档结构图](/documentation/articles/sql-database-elastic-scale-documentation-map)或[行级安全性参考](https://msdn.microsoft.com/zh-cn/library/dn765131)。


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-tools-multi-tenant-row-level-security/blogging-app.png
<!--anchors-->

<!---HONumber=69-->