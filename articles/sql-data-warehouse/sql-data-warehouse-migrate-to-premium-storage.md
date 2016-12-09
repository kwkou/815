<properties
   pageTitle="将现有 Azure SQL 数据仓库迁移到高级存储 | Azure"
   description="将 SQL 数据仓库迁移到高级存储的说明"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="happynicolle"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="11/29/2016"
   wacn.date="12/09/2016"
   ms.author="nicw;barbkess;sonyama"/>

# 迁移到高级存储的详细信息
SQL 数据仓库最近推出了[具有更好的性能可预测性的高级存储][具有更好的性能可预测性的高级存储]。现在我们已经准备好将目前在标准存储上的现有数据仓库迁移到高级存储。继续阅读，了解有关如何以及何时进行自动迁移的详细信息。如果您想控制何时发生停机时间，也请继续阅读以了解如何进行自行迁移。

如果您有多个数据仓库，请使用下面的[自动迁移计划][自动迁移计划]来确定何时进行迁移。

## 确定存储类型
如果是在以下日期之前创建的数据仓库，则目前使用的是标准存储。

| **区域** | **在此日期之前创建的数据仓库** |
| :------------------ | :-------------------------------- |
| 中国东部 | 2016-11-01 |
| 中国北部 | 2016-11-01 |

## 自动迁移的详细信息
在 2017 年 1 月 9 日至 1 月 13 日，您的 SQL 数据仓库将在当地时间晚上六点到早上六点之间进行有计划地维护。我们无法保证在某一天或者某个时间段进行迁移，但计划会按照约 1TB/ 小时的速度在上述 12 小时内完成。在此维护期间，您的现有数据仓库将会被重命名，且在数据迁移到高级存储时呈短暂不可用状态。您也可选择提前自助迁移。

> [AZURE.NOTE] 迁移期间你将不能使用你的现有数据仓库。迁移完成后，你的数据仓库将回到在线状态。

以下详细信息是 我们 代表你完成迁移进行的步骤，不需要你的参与。在本示例中，假设您在标准存储上的现有数据仓库目前名为“MyDW”。

1.	我们将“MyDW”重命名为“MyDW\_DO\_NOT\_USE\_[Timestamp]”
2.	我们将暂停“MyDW\_DO\_NOT\_USE\_[Timestamp]”。 在此期间，进行了备份。如果在次过程中遇到任何问题，你将看到多次暂停/恢复。
3.	我们将在步骤 2 进行的备份中，在高级存储上创建一个名为“MyDW”的新数据仓库。还原完成之前，“MyDW”不会出现。
4.	还原完成后，“MyDW”将返回到相同的 DWU，并与迁移之前的已暂停或活动状态相同。
5.	我们将删除“MyDW\_DO\_NOT\_USE\_[Timestamp]”
	
> [AZURE.NOTE] 这些设置不会作为迁移的一部分执行：
> 
>	-  需要在数据库级别重新启用审核
>	-  需要在**数据库**级别重新添加防火墙规则。  **服务器**级别的防火墙规则不受影响。

###<a name="automatic-migration-schedule"></a> 自动迁移计划
自动迁移会在下午 6 点 – 上午 6点（每个地区的本地时间）在以下中断计划期间发生。

| **区域** | **预计开始日期** | **预计结束日期** |
| :------------------ | :--------------------------- | :--------------------------- |
| 中国东部 | 2017-01-09 | 2017-01-13 |
| 中国北部 | 2017-01-09 | 2017-01-13 |

##<a name="self-migration-to-premium-storage"></a> 自行迁移到高级存储
如果您想控制何时发生停机时间，则可以使用以下步骤将标准存储上的现有数据仓库迁移到高级存储。如果选择自行迁移，则必须在该地区的自动迁移开始之前完成自行迁移，以避免自动迁移导致冲突的任何风险（请参阅[自动迁移计划][自动迁移计划]）。

### 自行迁移说明
如果你想控制自己的停机时间，则可以使用备份/还原自行迁移你的数据仓库。预计迁移的还原部分每个数据仓库中的每 TB 的存储将花费 1 小时。如果想在迁移完成后保持相同的名称，请按照[迁移期间重命名的步骤][迁移期间重命名的步骤]进行操作。

1.	[暂停][暂停]将进行自动备份的数据仓库
2.	从最新的快照进行[还原][还原]
3.	删除标准存储上的现有数据仓库。**如果此步骤操作失败，你将需要为两个数据仓库支付费用**

> [AZURE.NOTE] 这些设置不会作为迁移的一部分执行：
> 
>	-  需要在数据库级别重新启用审核
>	-  需要在**数据库**级别重新添加防火墙规则。  **服务器**级别的防火墙规则不受影响。

####<a name="optional-steps-to-rename-during-migration"></a> 可选：迁移过程中重命名的步骤 
同一逻辑服务器上的两个数据库不能具有相同的名称。SQL 数据仓库现在支持重命名数据仓库。

在本示例中，假设您在标准存储上的现有数据仓库目前名为“MyDW”。

1.	使用类似“MyDW\_BeforeMigration”的内容后跟的 ALTER DATABASE 命令重命名“MyDW”。 此命令终止所有现有事务，而且必须在 master 数据库执行才能成功。
```
ALTER DATABASE CurrentDatabasename MODIFY NAME = NewDatabaseName;
```
2.	[暂停][暂停]将自动备份的“MyDW\_BeforeMigration”
3.	从最新的快照进行[还原][还原]，并使用曾经拥有的名称（即“MyDW”）创建一个新数据库。
4.	删除“MyDW\_BeforeMigration”。**如果此步骤操作失败，你将需要为两个数据仓库支付费用**

> [AZURE.NOTE] 这些设置不会作为迁移的一部分执行：
> 
>	-  需要在数据库级别重新启用审核
>	-  需要在**数据库**级别重新添加防火墙规则。  **服务器**级别的防火墙规则不受影响。

## 后续步骤
通过对高级存储的更改，我们还增加了数据仓库基础结构中的数据库 blob 文件的数量。若要充分利用此更改的性能优势，建议使用以下脚本重新生成聚集列存储索引。下面脚本的工作原理是强制将一些现有数据分发到其他 blob。如果不采取任何操作，随着您将更多数据加载到数据仓库表中，数据将自然地重新分发。

**先决条件：**

1.	数据仓库应以 1,000 DWU 或更高 DWU 运行（请参阅[缩放计算能力][缩放计算能力]）
2.	执行脚本的用户应为 [mediumrc 角色][mediumrc 角色]或更高版本
	1.	若要将用户添加到此角色中，请执行下列语句：
		1.	````EXEC sp_addrolemember 'xlargerc', 'MyUser'````


````sql
-------------------------------------------------------------------------------
-- 步骤 1：创建表来控制索引重新生成
-- 在 mediumrc 或更高版本中以用户身份运行
--------------------------------------------------------------------------------
create table sql_statements
WITH (distribution = round_robin)
as select 
    'alter index all on ' + s.name + '.' + t.NAME + ' rebuild;' as statement,
    row_number() over (order by s.name, t.name) as sequence
from 
    sys.schemas s
    inner join sys.tables t
        on s.schema_id = t.schema_id
where
    is_external = 0
;
go
 
--------------------------------------------------------------------------------
-- 步骤 2：执行索引重新生成。如果脚本失败，可以重新运行以下语句以从最后中断的地方重新启动
-- 在 mediumrc 或更高版本中以用户身份运行
--------------------------------------------------------------------------------

declare @nbr_statements int = (select count(*) from sql_statements)
declare @i int = 1
while(@i <= @nbr_statements)
begin
      declare @statement nvarchar(1000)= (select statement from sql_statements where sequence = @i)
      print cast(getdate() as nvarchar(1000)) + ' Executing... ' + @statement
      exec (@statement)
      delete from sql_statements where sequence = @i
      set @i += 1
end;
go
-------------------------------------------------------------------------------
-- 步骤 3：清理步骤 1 中创建的表
--------------------------------------------------------------------------------
drop table sql_statements;
go
````

如果有任何关于数据仓库的问题，请[在线提交工单][在线提交工单]，并参阅“迁移到高级存储”寻找可能的原因。

<!--Image references-->

<!--Article references-->
[自动迁移计划]: #automatic-migration-schedule
[self-migration to Premium Storage]: #self-migration-to-premium-storage
[在线提交工单]: //support/support-ticket-form/?l=zh-cn
[Azure paired region]: /documentation/articles/best-practices-availability-paired-regions
[main documentation site]: /documentation/articles/services/sql-data-warehouse
[暂停]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#pause-compute
[还原]: /documentation/articles/sql-data-warehouse-manage-database-restore-portal
[迁移期间重命名的步骤]: #optional-steps-to-rename-during-migration
[缩放计算能力]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#scale-compute-power
[mediumrc 角色]: /documentation/articles/sql-data-warehouse-develop-concurrency/#workload-management

<!--MSDN references-->


<!--Other Web references-->
[具有更好的性能可预测性的高级存储]: https://azure.microsoft.com/zh-CN/blog/azure-sql-data-warehouse-introduces-premium-storage-for-greater-performance/
[Azure 门户]: https://portal.azure.cn

<!---HONumber=Mooncake_1010_2016-->