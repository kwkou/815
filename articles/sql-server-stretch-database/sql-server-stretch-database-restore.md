<properties
	pageTitle="还原已启用延伸的数据库 | Azure"
	description="了解如何还原已启用延伸的数据库。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="08/01/2016"
	wacn.date=""/>

# 还原已启用延伸的数据库

必要时还原备份的数据库可从许多类型的故障、错误和灾难中恢复。

有关备份的详细信息，请参阅[备份已启用延伸的数据库](/documentation/articles/sql-server-stretch-database-backup/)。

>   [AZURE.NOTE] 备份只是完整的高可用性和业务连续性解决方案的一个部分。有关高可用性的详细信息，请参阅[高可用性解决方案](https://msdn.microsoft.com/zh-cn/library/ms190202.aspx)。

## 还原 SQL Server 数据
若要从硬件故障或损坏中恢复，请从备份还原已启用延伸的 SQL Server 数据库。你可以继续使用当前使用的 SQL Server 还原方法。有关详细信息，请参阅[还原和恢复概述](https://msdn.microsoft.com/zh-cn/library/ms191253.aspx)。

还原 SQL Server 数据库之后，必须运行存储过程 **sys.sp\_rda\_reauthorize\_db** 才能在已启用延伸的 SQL Server 数据库和远程 Azure 数据库之间重新建立连接。

## 还原远程 Azure 数据

### 恢复实时 Azure 数据库
Azure 上的 SQL Server Stretch Database 服务至少每隔 8 小时使用 Azure 存储快照为所有实时数据创建快照一次。这些快照将保留 7 天。这样，便可以将数据还原到生成最新快照前 7 天内至少 21 个时间点中的一个。

若要使用 Azure 经典管理门户将实时 Azure 数据库还原到较早的时间点，请执行以下操作。

1. 登录到 Azure 经典管理门户。
2. 在屏幕左侧选择“浏览”，然后选择“SQL 数据库”。
3. 导航到你的数据库并选择它。
4. 在数据库边栏选项卡顶部，单击“还原”。
5. 指定新的**数据库名称**，选择一个**还原点**，然后单击“创建”。
6. 数据库还原过程随即将会开始，你可以使用“通知”监视还原进度。

### 恢复已删除的 Azure 数据库
Azure 上的 SQL Server Stretch Database 服务在删除数据库之前会创建数据库快照，并将其保留 7 天。此后，它不再保留实时数据库中的快照。这样，你便可以将删除的数据库还原到删除时间点。

若要使用 Azure 经典管理门户将删除的 Azure 数据库还原到删除时间点，请执行以下操作。

1. 登录到 Azure 经典管理门户。
2. 在屏幕左侧选择“浏览”，然后选择“SQL Sever”。
3. 导航到你的服务器并选择它。
4. 在服务器的边栏选项卡上向下滚动到“操作”，然后单击“删除的数据库”磁贴。
5. 选择要还原的已删除数据库。
5. 指定新的**数据库名称**，然后单击“创建”。
6. 数据库还原过程随即将会开始，你可以使用“通知”监视还原进度。

## 还原 SQL Server 数据库和远程 Azure 数据库之间的连接

1.  如果你要使用不同的名称或在不同区域中连接到已还原的 Azure 数据库，请运行存储过程 [sys.sp\_rda\_deauthorize\_db](https://msdn.microsoft.com/zh-cn/library/mt703716.aspx) 以与以前的 Azure 数据库断开连接。  

2.  运行存储过程 [sys.sp\_rda\_reauthorize\_db](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx)，以将已启用延伸的本地数据库重新连接到 Azure 数据库。
    -   提供现有的数据库范围凭据作为 sysname 或 varchar(128) 值。（不要使用 varchar(max)。） 你可以在视图 **sys.database\_scoped\_credentials** 中查找凭据名称。
	-   指定是否要制作远程数据的副本并连接到该副本（推荐）。


	    	USE <Stretch-enabled database name>;
    		GO
    		EXEC sp_rda_reauthorize_db
    		    @credential = N'<existing_database_scoped_credential_name>',
    			@with_copy = 1 ;  
    		GO


## 另请参阅

[延伸数据库的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

[sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx)

[备份和还原 SQL Server 数据库](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx)

<!---HONumber=Mooncake_0829_2016-->