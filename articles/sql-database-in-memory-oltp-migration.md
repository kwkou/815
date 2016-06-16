<properties
	pageTitle="In-Memory OLTP 改善 SQL 事务性能 | Azure"
	description="利用 In-Memory OLTP 改善现有 SQL 数据库中的事务性能。"
	services="sql-database"
	documentationCenter=""
	authors="jodebrui"
	manager="jhubbard"
	editor="MightyPen"/>


<tags
	ms.service="sql-database"
	ms.date="02/11/2016"
	wacn.date="06/14/2016"/>


# 使用 In-Memory（预览版）改善 SQL 数据库中的应用程序性能

请遵循以下步骤，使用 [In-Memory](/documentation/articles/sql-database-in-memory) 功能优化现有[高级](/documentation/articles/sql-database-service-tiers) Azure SQL 数据库的事务性能。


## 步骤 1：确保你的高级数据库支持 In-Memory

在 2015 年 11 月或之后创建的高级数据库支持 In-Memory 功能。可以通过运行以下 Transact-SQL 语句，来判断你的高级数据库是否支持 In-Memory 功能。如果返回的结果为 1（不是 0），则支持 In-Memory：

```
SELECT DatabasePropertyEx(Db_Name(), 'IsXTPSupported');
```

*XTP* 代表极端事务处理 (Extreme Transaction Processing)

如果现有的数据库必须迁移到新的 V12 高级数据库，你可以使用以下方法导出再导入你的数据。

#### 导出步骤

使用以下方式之一将生产数据库导出到 bacpac：

- [门户](https://manage.windowsazure.cn)中的“导出”功能。[](/documentation/articles/sql-database-export)

- [最新SSMS.exe](http://msdn.microsoft.com/zh-cn/library/mt238290.aspx) (SQL Server Management Studio) 中的“导出数据层应用程序”功能。
 1. 在“对象资源管理器”中，展开“数据库”节点。
 2. 右键单击你的数据库节点。
 3. 单击“任务”>“导出数据层应用程序”。
 4. 操作显示的向导窗口。


#### 导入步骤

将 bacpac 导入新的高级数据库。

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，
 - 导航到服务器。
 - 选择“导入数据库”选项。[](/documentation/articles/sql-database-import)
 - 选择一个高级定价层。

2. 使用 SSMS 导入 bacpac：
 - 在“对象资源管理器”中，右键单击“数据库”节点。
 - 单击“导入数据层应用程序”。
 - 操作显示的向导窗口。


## 步骤 2：标识要迁移到 In-Memory OLTP 的对象

SSMS 包含可以针对具有活动工作负荷的数据库运行的“事务性能分析概述”报告。该报告识别要迁移到 In-Memory OLTP 的候选表和存储过程。

若要在 SSMS 中生成报告，请执行以下操作：
- 在“对象资源管理器”中，右键单击你的数据库节点。
- 单击“报告”>“标准报告”>“事务能分析概述”。

有关详细信息，请参阅[确定是否应将某个表或存储过程移植到 In-Memory OLTP](http://msdn.microsoft.com/zh-cn/library/dn205133.aspx)。


## 步骤 3：创建可比较的测试数据库

假设报告指出数据库的某个表在转换成内存优化的表后会带来好处。我们建议先进行测试，以确认这项指示。

你需要创建生产数据库的测试副本。测试数据库必须位于与生产数据库相同的服务层级别。

为了简化测试，请按以下方式调整测试数据库：

1. 使用 SSMS 连接到测试数据库。

2. 若要避免在查询中用到 WITH (SNAPSHOT) 选项，请按照以下 T-SQL 语句中所示设置数据库选项：
```
ALTER DATABASE CURRENT
	SET
		MEMORY_OPTIMIZED_ELEVATE_TO_SNAPSHOT = ON;
```


## 步骤 4：迁移表

必须创建并填充想要测试的表的内存优化副本。你可以使用以下方式之一来创建该副本：

- SSMS 中提供便利的内存优化向导。
- 手动 T-SQL。


#### SSMS 中提供的内存优化向导

若要使用此迁移选项，请执行以下操作：

1. 使用 SSMS 连接到测试数据库。

2. 在“对象资源管理器”中，右键单击该表，然后单击“内存优化顾问”。
 - 此时将显示“表内存优化顾问”向导。

3. 在向导中单击“迁移验证”（或“下一步”按钮），以查看该表是否有任何在内存优化表中不受支持的功能。有关详细信息，请参阅：
 - [内存优化顾问](http://msdn.microsoft.com/zh-cn/library/dn284308.aspx)中的*内存优化清单*。
 - [In-Memory OLTP 不支持的 Transact-SQL 构造](http://msdn.microsoft.com/zh-cn/library/dn246937.aspx)
 - [迁移到 In-Memory OLTP](http://msdn.microsoft.com/zh-cn/library/dn247639.aspx)。

4. 如果该表没有不受支持的功能，顾问可为你执行实际的架构和数据迁移。


#### 手动 T-SQL

若要使用此迁移选项，请执行以下操作：

1. 使用 SSMS（或类似的实用程序）连接到测试数据库。

2. 获取表及其索引的完整 T-SQL 脚本。
 - 在 SSMS 中，右键单击你的表节点。
 - 单击“编写表脚本为”>“创建到”>“新建查询窗口”。

3. 在脚本窗口中，将 WITH (MEMORY\_OPTIMIZED = ON) 添加到 CREATE TABLE 语句。

4. 如果存在 CLUSTERED 索引，请将其更改为 NONCLUSTERED。

5. 使用 SP\_RENAME 重命名现有表。

6. 通过运行已编辑的 CREATE TABLE 脚本，创建新的内存优化表副本。

7. 使用 INSERT...SELECT * INTO 将数据复制到内存优化表：
	
```
INSERT INTO <new_memory_optimized_table>
		SELECT * FROM <old_disk_based_table>;
```


## 步骤 5（可选）：迁移存储过程

In-Memory 功能还可以修改存储过程，以改善性能。


### 本机编译存储过程的注意事项

本机编译存储过程的 T-SQL WITH 子句必须包含以下选项：

- NATIVE\_COMPILATION

- SCHEMABINDING：表示除非丢弃存储过程，否则无法由存储过程以任何影响到存储过程的方式更改其列定义的表。


本机模块必须使用一个大型 [ATOMIC 块](http://msdn.microsoft.com/zh-cn/library/dn452281.aspx)进行事务管理。显式 BEGIN TRANSACTION 或 ROLLBACK TRANSACTION 没有角色。如果你的代码检测到违反业务规则，它可以使用 [THROW](http://msdn.microsoft.com/zh-cn/library/ee677615.aspx) 语句终止原子块。


### 本机编译存储过程的典型 CREATE PROCEDURE

创建本机编译存储过程的 T-SQL 通常类似于以下模板：

```
CREATE PROCEDURE schemaname.procedurename
	@param1 type1, …
	WITH NATIVE_COMPILATION, SCHEMABINDING
	AS
		BEGIN ATOMIC WITH
			(TRANSACTION ISOLATION LEVEL = SNAPSHOT,
			LANGUAGE = N'your_language__see_sys.languages'
			)
		…
		END;
```

- 就 TRANSACTION\_ISOLATION\_LEVEL 而言，SNAPSHOT 是本机编译存储过程最常用的值。但是，也支持其他值的子集：
 - REPEATABLE READ
 - SERIALIZABLE


- sys.languages 视图中必须存在 LANGUAGE 值。


### 如何迁移存储过程

迁移步骤如下：


1. 获取常规解释的存储过程的 CREATE PROCEDURE 脚本。

2. 重写其标头以符合前面的模板。

3. 确认存储过程 T-SQL 代码是否使用了任何不支持本机编译存储过程的功能。根据需要实施应对措施。
 - 有关详细信息，请参阅[本机编译存储过程的迁移问题](http://msdn.microsoft.com/zh-cn/library/dn296678.aspx)。

4. 使用 SP\_RENAME 重命名旧存储过程。或直接将它删除。

5. 运行已编辑的 CREATE PROCEDURE T-SQL 脚本。


## 步骤 6：在测试环境中运行工作负荷

在测试数据库中，运行与在生产数据库中运行的工作负荷类似的工作负荷。这样，将 In-Memory 功能用于表和存储过程所达到的性能改善应可体现出来。

工作负荷的主要属性包括：

- 并发连接数。

- 读/写比率。


若要修改并运行测试工作负荷，请考虑使用[此处](/documentation/articles/sql-database-in-memory)所示的 ostress.exe 便利工具。


为了尽可能减少网络延迟，请在数据库所在的同一 Azure 地理区域运行测试。


## 步骤 7：实施后的监视

建议你监视在生产环境中实施 In-Memory 后的性能影响：

- [监视内存中存储](/documentation/articles/sql-database-in-memory-oltp-monitoring)。

- [使用动态管理视图监视 Azure SQL 数据库](/documentation/articles/sql-database-monitoring-with-dmvs)


## 相关链接

- [In-Memory OLTP（内存中优化）](http://msdn.microsoft.com/zh-cn/library/dn133186.aspx)

- [本机编译的存储过程简介](http://msdn.microsoft.com/zh-cn/library/dn133184.aspx)

- [内存优化顾问](http://msdn.microsoft.com/zh-cn/library/dn284308.aspx)


<!---HONumber=Mooncake_0606_2016-->
