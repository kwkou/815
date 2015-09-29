<properties 
	pageTitle="多分片查询" 
	description="使用弹性数据库客户端库运行跨分片查询。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="sidneyh" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="07/24/2015" 
	wacn.date="09/15/2015"/>

# 多分片查询

## 概述

**多分片查询**用于诸如数据收集/报告等需要跨多个分片运行查询的任务。（相比之下，[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)会在单个分片上执行所有操作。） 若要使用 SQL Server Management Studio，请参阅[弹性数据库查询入门](/documentation/articles/sql-database-elastic-query-getting-started)。

弹性数据库客户端库引入了名为 **Microsoft.Azure.SqlDatabase.ElasticScale.Query** 的新命名空间，以提供使用单个查询及结果查询多个分片的功能。它提供对分片集合进行查询抽象的功能。它还提供了备用执行策略，尤其是部分结果，以处理在对多个分片进行查询时所出现的故障。

多分片查询的主要入口点是 **MultiShardConnection** 类。与使用数据相关路由一样，该 API 遵循 **[System.Data.SqlClient](http://msdn.microsoft.com/zh-cn/library/System.Data.SqlClient(v=vs.110).aspx)** 类和方法的熟悉体验。在 **SqlClient** 库中，第一步是创建 **SqlConnection**，然后创建 **SqlCommand** 以用于连接，再通过某个 **Execute** 方法执行命令。最后，**SqlDataReader** 将循环访问从命令执行返回的结果集。多分片查询 API 方面的体验遵循以下步骤：

1. 创建 **MultiShardConnection**。
2. 为 **MultiShardConnection** 创建 **MultiShardCommand**。
3. 执行命令。
4. 通过 **MultiShardDataReader** 使用结果。 

主要区别在于多分片连接的构建。其中 **SqlConnection** 在单一数据库上进行操作，而 **MultiShardConnection** 将***分片集合***用作其输入。用户可以填充分片映射中的分片集合。然后，使用 **UNION ALL** 语义组成一个总体结果在分片集合上执行查询。或者，也可以在命令上使用 **ExecutionOptions** 属性，以将行所源自的分片的名称添加到输出。以下代码使用给定的 **ShardMap**（名为*myShardMap*）来演示多分片查询的用法。

    using (MultiShardConnection conn = new MultiShardConnection( 
                                        myShardMap.GetShards(), 
                                        myShardConnectionString) 
          ) 
    { 
    using (MultiShardCommand cmd = conn.CreateCommand())
           { 
            cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable"; 
            cmd.CommandType = CommandType.Text; 
            cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn; 
            cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults; 

            using (MultiShardDataReader sdr = cmd.ExecuteReader()) 
            	{ 
                	while (sdr.Read())
                    	{ 
                        	var c1Field = sdr.GetString(0); 
                        	var c2Field = sdr.GetFieldValue<int>(1); 
                        	var c3Field = sdr.GetFieldValue<Int64>(2);
                    	} 
             	} 
           } 
    } 
 

请注意对 **myShardMap.GetShards()** 的调用。通过此方法可从分片映射中检索所有分片，还可轻松在该分片映射中的所有分片之间运行查询。对通过调用 **myShardMap.GetShards()** 返回的集合执行 LINQ 查询，以进一步优化用于多分片查询的分片集合。多分片查询中的当前功能已随部分结果策略一起被设计为供数十至数百种分片使用。多分片查询目前存在的一个限制是，缺少对需要查询的分片和 shardlet 进行验证。尽管数据相关路由会在查询时验证给定的分片是否为分片映射的一部分，但多分片查询不会执行此检查。这可能会导致多分片查询在已从分片映射中删除的分片上运行。

## 多分片查询和拆分/合并操作

多分片查询不会验证查询的分片上的 shardlet 是否参与了正在进行的拆分/合并操作。这可能会导致不一致问题，即为同一多分片查询中的多个分片显示同一 shardlet 中的行。请注意这些限制并在执行多分片查询时，考虑关闭正在进行的合拆分操作以及对分片映射的更改。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!---HONumber=69-->