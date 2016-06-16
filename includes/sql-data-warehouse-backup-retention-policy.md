
<!--
includes/sql-data-warehouse-backup-retention-policies.md

Latest Freshness check:  2016-05-05 , barbkess.

As of circa 2016-04-22, the following topics might include this include:
articles/sql-data-warehouse/sql-data-warehouse-overview-expectations.md
articles/sql-data-warehouse/sql-data-warehouse-overview-backup-and-restore.md
-->
SQL 数据仓库使用 Azure 存储空间快照，每隔 8 小时备份所有实时数据至少一次。这些快照将保留 7 天。这样，便可以将数据还原到生成最新快照前 7 天内至少 21 个时间点中的一个。

SQL 数据仓库在删除数据库之前会创建数据库快照，并将其保留 7 天。此后，它不再保留实时数据库中的快照。这样，你便可以将删除的数据库还原到删除时间点。

在发生区域性的故障时，SQL 数据仓库以异步方式将快照复制到其他 Azure 地理区域，以提高可恢复性。如果你由于 Azure 区域故障而无法访问数据库，可以将数据库还原到某个异地冗余的快照。
<!---HONumber=Mooncake_0523_2016-->