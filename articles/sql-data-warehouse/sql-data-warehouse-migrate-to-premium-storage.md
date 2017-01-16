<properties
    pageTitle="将现有 Azure 数据仓库迁移到高级存储 | Azure"
    description="将现有数据仓库迁移到高级存储的说明"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="happynicolle"
    manager="barbkess"
    editor="" />
<tags
    ms.assetid="04b05dea-c066-44a0-9751-0774eb84c689"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="11/29/2016"
    wacn.date="01/13/2017"
    ms.author="rortloff;barbkess" />  


# 将数据仓库迁移到高级存储
Azure SQL 数据仓库最近推出了[具有更好的性能可预测性的高级存储][premium storage for greater performance predictability]。现在可以将标准存储上现有的数据仓库迁移到高级存储。可以利用自动迁移，如果想要控制迁移时间（这涉及某种停机时间），也可自己执行迁移操作。

如果有多个数据仓库，请使用[自动迁移计划][]来确定何时进行迁移。

## 确定存储类型
如果是在以下日期之前创建的数据仓库，则目前使用的是标准存储。

| **区域** | **在此日期之前创建的数据仓库** |
|:--- |:--- |
| 中国东部 |2016 年 11 月 1 日 |
| 中国北部 |2016 年 11 月 1 日 |


## 自动迁移的详细信息
默认情况下，我们将在[自动迁移计划][automatic migration schedule]期间，在用户所在地区当地时间的下午 6:00 至早上 6:00 期间，对用户的数据库进行迁移。迁移期间，用户的现有数据仓库将不可用。此迁移对于每个数据仓库中的每 TB 的存储将需要大约 1 小时。自动迁移过程中的任何部分都不会向用户收取费用。

> [AZURE.NOTE] 迁移完成后，用户的数据仓库将恢复联机状态并可用。

我们将执行以下步骤来完成迁移（这些步骤不需要用户参与）。在本示例中，假设用户在标准存储上的现有数据仓库目前名为“MyDW”。

1. 我们将“MyDW”重命名为“MyDW\_DO\_NOT\_USE\_[Timestamp]”。
2. 我们将暂停“MyDW\_DO\_NOT\_USE\_[Timestamp]”。 在此期间，进行了备份。如果在此过程中遇到任何问题，用户将看到多次暂停和恢复。
3. 我们将在步骤 2 进行的备份中，在高级存储上创建一个名为“MyDW”的新数据仓库。还原完成之前，“MyDW”不会出现。
4. 还原完成后，“MyDW”将返回到迁移之前的相同数据仓库单位和状态（“暂停”或“活动”）。
5. 我们将删除“MyDW\_DO\_NOT\_USE\_[Timestamp]”。
	
> [AZURE.NOTE] 这些设置不会作为迁移的一部分执行：
> 
>	-  需要重新启用数据库级别的审核
>	-  需要重新添加**数据库**级别的防火墙规则。**服务器**级别的防火墙规则将不受影响

###<a name="automatic-migration-schedule"></a> 自动迁移计划
在以下服务中断计划期间，将在下午 6:00 到上午 6:00（每个区域的本地时间）之间执行自动迁移。

| **区域** | **预计开始日期** | **预计结束日期** |
|:--- |:--- |:--- |
| 中国东部 |2017 年 1 月 9 日 |2017 年 1 月 13 日 |
| 中国北部 |2017 年 1 月 9 日 |2017 年 1 月 13 日 |

##<a name="self-migration-to-premium-storage"></a> 自行迁移到高级存储
如果要控制何时发生停机时间，则可以使用以下步骤将标准存储上的现有数据仓库迁移到高级存储。如果选择此选项，则必须在该区域的自动迁移开始之前，完成自行迁移。这可确保避免导致冲突的自动迁移的任何风险（请参阅[自动迁移计划][automatic migration schedule]）。

### 自行迁移说明
若要自行迁移数据仓库，请使用备份和还原功能。预计迁移的还原部分对于每个数据仓库中的每 TB 的存储将花费大约 1 小时。如果想在迁移完成后保留相同的名称，请按照[迁移期间重命名的步骤][steps to rename during migration]进行操作。

1. [暂停][Pause]数据仓库。这将执行自动备份。
2. 从最新的快照进行[还原][Restore]。
3. 删除标准存储上的现有数据仓库。**如果此步骤操作失败，用户将需要为两个数据仓库支付费用。**

> [AZURE.NOTE] 这些设置不会作为迁移的一部分执行：
> 
> * 需要重新启用数据库级别的审核。
> * 需要重新添加数据库级别的防火墙规则。服务器级别的防火墙规则将不受影响。

####<a name="optional-steps-to-rename-during-migration"></a> 在迁移期间重命名数据仓库（可选）
同一逻辑服务器上的两个数据库不能具有相同的名称。SQL 数据仓库现在支持重命名数据仓库。

在本示例中，假设用户在标准存储上的现有数据仓库目前名为“MyDW”。

1. 使用以下 ALTER DATABASE 命令重命名“MyDW”。（在此示例中，我们将其重命名为“MyDW\_BeforeMigration”。）此命令将停止所有现有的事务，并且必须在 master 数据库中执行才能成功。
```
ALTER DATABASE CurrentDatabasename MODIFY NAME = NewDatabaseName;
```
2. [暂停][Pause]“MyDW\_BeforeMigration”。 这将执行自动备份。
3. 从最新的快照进行[还原][Restore]，并使用它以前的名称（例如，“MyDW”）创建一个新数据库。
4. 删除“MyDW\_BeforeMigration”。 **如果此步骤操作失败，用户将需要为两个数据仓库支付费用。**


## 后续步骤
通过对高级存储的更改，我们还增加了数据仓库基础结构中的数据库 blob 文件的数量。若要充分利用此更改的性能优势，请使用以下脚本重新生成聚集列存储索引。该脚本的工作原理是强制将一些现有数据分发到其他 blob。如果不采取任何操作，当用户将更多数据加载到表时，这些数据将随着时间的推移自然地重新分发。

**先决条件：**

- 数据仓库应以 1,000 或更高的数据仓库单位运行（请参阅[缩放计算能力][scale compute power]）。
- 执行脚本的用户应为 [mediumrc 角色][mediumrc role]或更高级角色。若要将用户添加到此角色中，请执行下列语句：
		a. `EXEC sp_addrolemember 'xlargerc', 'MyUser'`



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
            -- 步骤 2：执行索引重新生成。如果脚本失败，可以重新运行以下语句以从上次中断的地方重新启动。
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


如果有任何关于数据仓库的问题，请[在线提交工单][在线提交工单]，并参阅“迁移到高级存储”寻找可能的原因。

<!--Image references-->


<!--Article references-->
[automatic migration schedule]: #automatic-migration-schedule
[自动迁移计划]: #automatic-migration-schedule
[self-migration to Premium Storage]: #self-migration-to-premium-storage
[在线提交工单]: /support/support-ticket-form/?l=zh-cn
[Azure paired region]: /documentation/articles/best-practices-availability-paired-regions
[main documentation site]: /documentation/articles/services/sql-data-warehouse
[Pause]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#pause-compute
[Restore]: /documentation/articles/sql-data-warehouse-restore-database-portal
[steps to rename during migration]: #optional-steps-to-rename-during-migration
[scale compute power]: /documentation/articles/sql-data-warehouse-manage-compute-portal/#scale-compute-power
[mediumrc role]: /documentation/articles/sql-data-warehouse-develop-concurrency/

<!--MSDN references-->


<!--Other Web references-->
[Premium Storage for greater performance predictability]: https://azure.microsoft.com/zh-CN/blog/azure-sql-data-warehouse-introduces-premium-storage-for-greater-performance/
[Azure 门户]: https://portal.azure.cn

<!---HONumber=Mooncake_1212_2016-->
<!--Update_Descrtipion: wording update; update date format in tables-->