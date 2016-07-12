<properties
	pageTitle="数据迁移的监视和故障排除（延伸数据库）| Azure"
	description="了解如何监视数据迁移状态。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="04/11/2016"/>

# 数据迁移的监视和故障排除（SQL Server Stretch Database）

若要在SQL Server Stretch Database监视器中监视数据迁移，请在 SQL Server Management Studio 中选择数据库对应的“任务 | 延伸 | 监视”。

## 在SQL Server Stretch Database监视器中检查数据迁移状态
在 SQL Server Management Studio 中选择数据库对应的“任务 | 延伸 | 监视”，以打开SQL Server Stretch Database监视器并监视数据迁移。

-   监视器的上半部分显示有关已启用延伸的 SQL Server 数据库和远程 Azure 数据库的一般信息。

-   监视器的下半部分显示数据库中每个已启用延伸的表的数据迁移状态。

![SQL Server Stretch Database监视器][StretchMonitorImage1]

## <a name="Migration"></a>在动态管理视图中检查数据迁移状态
打开动态管理视图 **sys.dm\_db\_rda\_migration\_status** 查看已迁移的数据批数与行数。有关详细信息，请参阅 [sys.dm\_db\_rda\_migration\_status (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn935017.aspx)。

## <a name="Firewall"></a>数据迁移的故障排除
**Azure 防火墙阻止正在阻止来自我的本地服务器的连接。**

你可能必须在 Azure 服务器的 Azure 防火墙设置中添加一条规则，以使 SQL Server 可与远程 Azure 服务器进行通信。

**我的已启用延伸的表中的行未迁移到 Azure。这是什么问题呢？**

有几个问题可能会影响迁移。请检查以下事项。

-   检查 SQL Server 计算机的网络连接。

-   检查 Azure 防火墙，确保其未阻止你的 SQL 服务器连接到远程终结点。

-   在动态管理视图 **sys.dm\_db\_rda\_migration\_status** 中检查最新一批的状态。如果发生错误，请检查该批的 error\_number、 error\_state 和 error\_severity 值。

    -   有关该视图的详细信息，请参阅 [sys.dm\_db\_rda\_migration\_status (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn935017.aspx)。

    -   有关 SQL Server 错误消息内容的详细信息，请参阅 [sys.messages (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms187382.aspx)。

## 另请参阅

[SQL Server Stretch Database的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

<!--Image references-->
[StretchMonitorImage1]: ./media/sql-server-stretch-database-monitor/StretchDBMonitor.png

<!---HONumber=Mooncake_0405_2016-->