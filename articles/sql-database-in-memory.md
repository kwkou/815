<properties
	pageTitle="SQL In-Memory 入门 | Windows Azure"
	description="SQL In-Memory 技术大幅提升了事务和分析工作负荷的性能。了解如何利用这些技术。"
	services="sql-database"
	documentationCenter=""
	authors="jodebrui"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="11/16/2015"
	wacn.date="12/22/2015"/>


# SQL 数据库中的 In-Memory（预览版）入门

普通的 SQL 表只会存储在硬盘驱动器上。通过使用 In-Memory 功能，你可以增强表，以便在服务器运行时，使该表能够驻留在活动的内存中。


In-Memory 技术大幅提升了事务和分析工作负荷的性能：

- 通过 In-Memory OLTP（联机事务处理），根据工作负荷的具体情况，最高可以将事务吞吐量提高 30 倍。
 - 内存优化表。
 - 本机编译的存储过程。

- 使用 In-Memory Analytics，最高可以将查询性能提高 100 倍。
 - 列存储索引。


对于[实时分析](http://msdn.microsoft.com/zh-cn/library/dn817827.aspx)，可以结合这些技术来获得：

- 基于操作数据的实时业务见解。
- 飞快的 OLTP。


#### 可用性


- *GA：*In-Memory Analytics 已推出正式版 (GA)。
- *预览版：*In-Memory OLTP 和 Real-Time Operational Analytics 都以预览版提供。
 - 这两个产品仅适用于[高级](/documentation/articles/sql-database-service-tiers) Azure SQL 数据库。


#### OLTP 和 Analytics

本主题中的两个主要部分为：

- A.In-Memory [OLTP](#install_oltp_manuallink)，涉及到读取和写入。
- B.In-Memory [Analytics](#install_analytics_manuallink)，涉及到读取。



<a id="install_oltp_manuallink" name="install_oltp_manuallink">

&nbsp;

## A.安装 In-Memory OLTP 示例

在 [Azure 门户](https://manage.windowsazure.cn)中按几下鼠标，即可创建 AdventureWorksLT [V12] 示例数据库。本部分中的步骤说明如何使用以下项目扩充 AdventureWorksLT 数据库：

- In-Memory 表。
- 本机编译的存储过程。


#### 安装步骤

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，于 V12 服务器上创建一个高级数据库。将“源”设置为 AdventureWorksLT [V12] 示例数据库。
 - 有关详细指示，请参阅[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)。

2. 使用 SQL Server Management Studio [(SSMS.exe)](http://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 连接到该数据库。

3. 将 [In-Memory OLTP Transact-SQL 脚本](http://raw.githubusercontent.com/Azure/azure-sql-database-samples/master/T-SQL/In-Memory/sql_in-memory_oltp_sample.sql)复制到剪贴板。
 - T-SQL 脚本在步骤 1 创建的 AdventureWorksLT 示例数据库中创建所需的 In-Memory 对象。

4. 将 T-SQL 脚本粘贴到 SSMS，然后执行该脚本。
 - 重要的是 `MEMORY_OPTIMIZED = ON` 子句 CREATE TABLE 语句，如下所示：


```
CREATE TABLE [SalesLT].[SalesOrderHeader_inmem](
	[SalesOrderID] int IDENTITY NOT NULL PRIMARY KEY NONCLUSTERED ...,
	...
) WITH (MEMORY_OPTIMIZED = ON);
```


#### 错误 40536


如果你在运行 T-SQL 脚本时收到错误 40536，请运行以下 T-SQL 脚本来验证数据库是否支持 In-Memory：


```
SELECT DatabasePropertyEx(DB_Name(), 'IsXTPSupported');
```


结果为 **0** 表示不支持 In-Memory，而 1 表示支持。


#### 关于创建的内存优化项

**表**：此示例包含以下内存优化表：

- SalesLT.Product\_inmem
- SalesLT.SalesOrderHeader\_inmem
- SalesLT.SalesOrderDetail\_inmem
- Demo.DemoSalesOrderHeaderSeed
- Demo.DemoSalesOrderDetailSeed


可以通过 SSMS 中的**对象资源管理器**检查内存优化表，方法是：

- 右键单击“表”>“筛选器”>“筛选器设置”>“内存优化”等于 1。


或者可以查询目录视图，例如：


```
SELECT is_memory_optimized, name, type_desc, durability_desc
	FROM sys.tables
	WHERE is_memory_optimized = 1;
```


**本机编译的存储过程**：可以通过目录视图查询来检查 SalesLT.usp\_InsertSalesOrder\_inmem：


```
SELECT uses_native_compilation, OBJECT_NAME(object_id), definition
	FROM sys.sql_modules
	WHERE uses_native_compilation = 1;
```


&nbsp;

## 运行示例 OLTP 工作负荷

以下两个*存储过程*的唯一差别在于，第一个过程使用内存优化表版本，而第二个过程使用普通磁盘表：

- SalesLT**.**usp\_InsertSalesOrder**\_inmem**
- SalesLT**.**usp\_InsertSalesOrder**\_ondisk**


在本部分中，你将了解如何使用便利的 **ostress.exe** 实用程序在压力级别执行两个存储过程。可以比较完成两个压力回合所需的时间。


运行 ostress.exe 时，建议你将参数值传递到这两个存储过程：

- 运行大量的并发连接（例如使用 -n100）。
- 让每个连接循环数百次（例如使用 -r500）。


但是，建议你从较小的值（-n10 和 -r50）开始，以确保一切运行正常。


### Ostress.exe 的脚本


本部分显示 ostress.exe 命令行中内嵌的 T-SQL 脚本。此脚本使用前面安装的 T-SQL 脚本所创建的项。


以下脚本将在以下内存优化*表*中插入包含 5 个细目的示例销售订单：

- SalesLT.SalesOrderHeader\_inmem
- SalesLT.SalesOrderDetail\_inmem


```
DECLARE
	@i int = 0,
	@od SalesLT.SalesOrderDetailType_inmem,
	@SalesOrderID int,
	@DueDate datetime2 = sysdatetime(),
	@CustomerID int = rand() * 8000,
	@BillToAddressID int = rand() * 10000,
	@ShipToAddressID int = rand() * 10000;
	
INSERT INTO @od
	SELECT OrderQty, ProductID
	FROM Demo.DemoSalesOrderDetailSeed
	WHERE OrderID= cast((rand()*60) as int);
	
EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT,
	@DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od;
```


若要创建上述适用于 ostress.exe 的 T-SQL 的 \_ondisk 版本，只需将两处出现的 *\_inmem* 子字符串替换为 *\_ondisk*。这种替换将影响表和存储过程的名称。


### 安装 RML 实用程序和 ostress


最好规划在 Azure VM 上运行 ostress.exe。在 AdventureWorksLT 数据库所在的同一 Azure 地理区域中创建 [Azure 虚拟机](/documentation/services/virtual-machines/)。但你可以改为在便携式计算机上运行 ostress.exe。


在 VM 或任何选择的主机上，安装包含 ostress.exe 的重放标记语言 (RML) 实用程序。

- 请参阅[用于演示 In-Memory OLTP 的 AdventureWorks 扩展](http://msdn.microsoft.com/zh-cn/library/dn511655&#x28;v=sql.120&#x29;.aspx)中的 ostress.exe 介绍。
 - 或参阅 [In-Memory OLTP 的示例数据库](http://msdn.microsoft.com/zh-cn/library/mt465764.aspx)。
 - 或参阅[有关安装 ostress.exe 的博客](http://blogs.msdn.com/b/psssql/archive/2013/10/29/cumulative-update-2-to-the-rml-utilities-for-microsoft-sql-server-released.aspx)



<!--
dn511655.aspx is for SQL 2014, whereas mt465764.aspx is for SQL 2016.
-->



### 首先运行 \_inmem 压力工作负荷


可以使用 *RML 命令提示符*窗口来运行 ostress.exe 命令行。

从 RML 命令提示符运行时，以下 ostress.exe 命令将会：

- 在内存优化表中插入 100 万个销售订单，每个订单包含 5 个细目。
- 使用 100 个并发连接 (-n100)。


```
ostress.exe -n100 -r500 -S<servername>.database.chinacloudapi.cn
	 -U<login> -P<password> -d<database>
	 -q -Q"DECLARE @i int = 0, @od SalesLT.SalesOrderDetailType_inmem,
	 @SalesOrderID int, @DueDate datetime2 = sysdatetime(), @CustomerID int = rand() *
	 8000, @BillToAddressID int = rand() * 10000, @ShipToAddressID int = rand()*
	 10000;
	 INSERT INTO @od SELECT OrderQty, ProductID FROM
	 Demo.DemoSalesOrderDetailSeed WHERE OrderID= cast((rand()*60) as int);
	 WHILE (@i < 20) begin;
	 EXECUTE SalesLT.usp_InsertSalesOrder_inmem
	 @SalesOrderID OUTPUT, @DueDate, @CustomerID, @BillToAddressID,
	 @ShipToAddressID, @od; set @i += 1; end"
```


若要运行上述 ostress.exe 命令行：


1. 在 SSMS 中运行以下命令重置数据库，以删除前面运行的命令所插入的所有数据：
```
EXECUTE Demo.usp_DemoReset;
```

2. 将文本复制到剪贴板。

3. 所有行尾字符 (\\r\\n) 和所有制表符 (\\t) 替换为空格。

4. 将参数 -S -U -P -d 的 <placeholders> 替换为正确的实数值。


#### 结果是一个持续时间


当 ostress.exe 完成时，它将在 RML 命令窗口中写入运行持续时间作为输出的最后一行。例如，一个较短的测试回合持续大约 1.5 分钟：

`11/12/15 00:35:00.873 [0x000030A8] OSTRESS exiting normally, elapsed time: 00:01:31.867`


#### 重置，编辑 \_ondisk，然后重新运行


在获得 \_inmem 运行结果之后，请针对 \_ondisk 运行执行以下步骤：


1. 在 SSMS 中运行以下命令重置数据库，以删除前面运行的命令所插入的所有数据：
```
EXECUTE Demo.usp_DemoReset;
```

2. 编辑 ostress.exe 命令行，将所有的 *\_inmem* 替换为 *\_ondisk*。

3. 重新运行 ostress.exe，并捕获持续时间结果。

4. 再次重置数据库，以确保清理数据。
 - 插入 100 万个销售订单的单次测试运行将在表中生成 500 MB 或更多的数据。


#### 预期比较结果

在与数据库相同的 Azure 区域的 Azure VM 上运行 ostress，In-Memory 测试已显示此简化工作负荷大约有 **9 倍**的性能改善。

添加转换成本机编译的存储过程时，性能可以获得更大改善。


## B.安装 In-Memory Analytics 示例


在本部分中，你将比较使用列存储索引与常规索引时的 IO 和统计信息结果。


列存储索引与常规索引在逻辑上相同，但物理上却不同。列存储索引以非本机方式组织数据，以便大幅压缩数据。这可以明显提高性能。


通常，在对 OLTP 工作负荷进行实时分析时，最好是使用非群集列存储索引。有关详细信息，请参阅[列存储索引介绍](http://msdn.microsoft.com/zh-cn/library/gg492088.aspx)。



### 准备列存储分析测试


1. 使用 Azure 门户基于示例创建全新的 AdventureWorksLT 数据库。
 - 使用相同的名称。
 - 选择任一高级服务层。

2. 将 [sql\_in-memory\_analytics\_sample](http://raw.githubusercontent.com/Azure/azure-sql-database-samples/master/T-SQL/In-Memory/sql_in-memory_analytics_sample.sql) 复制到剪贴板。
 - T-SQL 脚本在步骤 1 创建的 AdventureWorksLT 示例数据库中创建所需的 In-Memory 对象。
 - 该脚本将创建维度表和两个事实表。每个事实表中填充了 350 万行。
 - 该脚本可能需要 15 分钟才能完成。

3. 将 T-SQL 脚本粘贴到 SSMS，然后执行该脚本。
 - 重要的是 **CREATE INDEX** 语句中的 **COLUMNSTORE** 关键词，如下所示：<br/>`CREATE NONCLUSTERED COLUMNSTORE INDEX ...;`

4. 将 AdventureWorksLT 设置为兼容级别 130：<br/>`ALTER DATABASE AdventureworksLT SET compatibility_level = 130;`


#### 关键表和列存储索引


- dbo.FactResellerSalesXL\_CCI 是具有群集**列存储索引**的表，已在*数据*级别进一步压缩。

- dbo.FactResellerSalesXL\_PageCompressed 是具有等效常规群集索引的表，只在*页面*级别压缩。


#### 用于比较列存储索引的关键查询


[此处](http://raw.githubusercontent.com/Azure/azure-sql-database-samples/master/T-SQL/In-Memory/clustered_columnstore_sample_queries.sql)提供了你可以运行的用于查看性能改进情况的几种 T-SQL 查询类型。在 T-SQL 脚本的步骤 2 中，有一对直接相关的查询。这两个查询只是在一行上有所不同：


- `FROM FactResellerSalesXL_PageCompressed a`
- `FROM FactResellerSalesXL_CCI a`


群集列存储索引位于 FactResellerSalesXL**\_CCI** 表上。

以下 T-SQL 脚本摘录列出了每个表查询的 IO 和 TIME 统计信息。


```
/*********************************************************************
Step 2 -- Overview
-- Page Compressed BTree table v/s Columnstore table performance differences
-- Enable actual Query Plan in order to see Plan differences when Executing
*/
-- Ensure Database is in 130 compatibility mode
ALTER DATABASE AdventureworksLT SET compatibility_level = 130
GO

-- Execute a typical query that joins the Fact Table with dimension tables
-- Note this query will run on the Page Compressed table, Note down the time
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO

SELECT c.Year
	,e.ProductCategoryKey
	,FirstName + ' ' + LastName AS FullName
	,count(SalesOrderNumber) AS NumSales
	,sum(SalesAmount) AS TotalSalesAmt
	,Avg(SalesAmount) AS AvgSalesAmt
	,count(DISTINCT SalesOrderNumber) AS NumOrders
	,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_PageCompressed a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
WHERE e.ProductCategoryKey =2
	AND c.FullDateAlternateKey BETWEEN '1/1/2014' AND '1/1/2015'
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO
SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO


-- This is the same Prior query on a table with a Clustered Columnstore index CCI 
-- The comparison numbers are even more dramatic the larger the table is, this is a 11 million row table only.
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO
SELECT c.Year
	,e.ProductCategoryKey
	,FirstName + ' ' + LastName AS FullName
	,count(SalesOrderNumber) AS NumSales
	,sum(SalesAmount) AS TotalSalesAmt
	,Avg(SalesAmount) AS AvgSalesAmt
	,count(DISTINCT SalesOrderNumber) AS NumOrders
	,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_CCI a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
WHERE e.ProductCategoryKey =2
	AND c.FullDateAlternateKey BETWEEN '1/1/2014' AND '1/1/2015'
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO

SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO
```


## In-Memory 预览版注意事项


Azure SQL 数据库中的 In-Memory 功能将在 [2015 年 10 月 28 日推出预览版](http://azure.microsoft.com/updates/public-preview-in-memory-oltp-and-real-time-operational-analytics-for-azure-sql-database/)。


在正式版 (GA) 发布前的预览期间，只有下述数据库支持 In-Memory OLTP：

- 使用*高级*服务层的数据库。

- 在 In-Memory 功能生效后创建的数据库。
 - 从 In-Memory 激活日期前创建的备份还原的新数据库无法支持 In-memory。
 - 假如你备份了支持 In-Memory 的数据库，然后在旧的高级数据库中还原该备份。那么，该旧数据库现在将支持 In-Memory。


如有疑问，始终可以通过运行以下 T-SQL SELECT 来确定数据库是否支持 In-memory OLTP。结果为 **1** 表示数据库确实支持 In-Memory：

```
SELECT DatabasePropertyEx(DB_NAME(), 'IsXTPSupported');
```


如果查询返回 **1**，则表示此数据库支持 In-Memory OLTP，以及基于此数据库创建的任何数据库副本和数据库还原版本。


#### 只允许高级层对象


如果数据库包含以下任何种类的 In-Memory OLTP 对象或类型，则不支持将数据库的服务层从高级降级为基本或标准。若要降级数据库，必须先删除以下对象：

- 内存优化表
- 内存优化表类型
- 本机编译的模块


#### 其他关系


- 在预览期不支持将 In-Memory OLTP 功能用于弹性池中的数据库，但将来可能支持这种做法：

- 不支持将 In-Memory OLTP 与 SQL 数据仓库配合使用。
 - SQL 数据仓库支持 In-Memory Analytics 的列存储索引功能。

- 在预览期，“查询存储”不会捕获本机编译模块内的查询，但将来可能会这样做。

- In-Memory OLTP 不支持某些 TRANSACT-SQL 功能。此限制适用于 Microsoft SQL Server 和 Azure SQL 数据库。有关详细信息，请参阅：
 - [内存中 OLTP 的 Transact-SQL 支持](http://msdn.microsoft.com/zh-cn/library/dn133180.aspx)
 - [In-Memory OLTP 不支持的 Transact-SQL 构造](http://msdn.microsoft.com/zh-cn/library/dn246937.aspx)


## 后续步骤


- 尝试[在现有的 Azure SQL 应用程序中使用 In-Memory OLTP](/documentation/articles/sql-database-in-memory-oltp-migration)。


## 其他资源

#### 深入信息

- [了解适用于 Microsoft SQL Server 和 Azure SQL 数据库的 In-Memory OLTP](http://msdn.microsoft.com/zh-cn/library/dn133186.aspx)

- [在 MSDN 上了解 Real-Time Operational Analytics](http://msdn.microsoft.com/zh-cn/library/dn817827.aspx)

- [有关常用工作负荷模式和迁移注意事项](http://msdn.microsoft.com/zh-cn/library/dn673538.aspx)的白皮书，其中描述了 In-Memory OLTP 往往能够在其中提供显著性能改善的工作负荷模式。

#### 应用程序设计

- [In-Memory OLTP（内存中优化）](http://msdn.microsoft.com/zh-cn/library/dn133186.aspx)

- [在现有的 Azure SQL 应用程序中使用 In-Memory OLTP。](/documentation/articles/sql-database-in-memory-oltp-migration)

#### 工具

- [SQL Server Data Tools 预览版 (SSDT)](http://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 最新每月版本。

- [适用于 SQL Server 的重放标记语言 (RML) 实用程序介绍](http://support.microsoft.com/zh-cn/kb/944837)

- [监视内存中存储](/documentation/articles/sql-database-in-memory-oltp-monitoring)（适用于 In-Memory OLTP）。

<!---HONumber=Mooncake_1207_2015-->