
<!--
includes/sql-data-warehouse-include-pause-description.md

Latest Freshness check:  2016-04-22 , barbkess.

As of circa 2016-04-22, the following topics might include this include:
articles/sql-data-warehouse/sql-data-warehouse-manage-scale-out-tasks.md
articles/sql-data-warehouse/sql-data-warehouse-manage-scale-out-tasks-powershell.md
articles/sql-data-warehouse/sql-data-warehouse-manage-scale-out-tasks-rest-api.md

-->
为了节省成本，可以按需暂停和恢复计算资源。例如，如果你晚上和周末不使用数据库，那么可以在这些时间暂停数据库的使用，然后在白天时恢复使用。当数据库暂停时不对 DWU 进行收费。

当你暂停数据库时：

- 计算和内存资源返回到数据中心的可用资源池中
- 暂停期间 DWU 成本为零。
- 不影响数据存储，你的数据保持不变。 
- SQL 数据仓库将取消所有正在运行或已排队的操作。
<!---HONumber=Mooncake_0530_2016-->