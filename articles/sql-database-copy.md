<properties
	pageTitle="复制 Azure SQL 数据库 | Azure"
	description="创建 Azure SQL 数据库的副本"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="01/20/2016"
	wacn.date="03/21/2016"/>


# 复制 Azure SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [Azure Portal](/documentation/articles/sql-database-copy)
- [PowerShell](/documentation/articles/sql-database-copy-powershell)
- [SQL](/documentation/articles/sql-database-copy-transact-sql)

以下步骤说明如何使用 [Azure 门户](https://manage.windowsazure.cn)复制 SQL 数据库。数据库复制操作将创建新的 SQL 数据库。副本是在相同或不同服务器上创建的数据库快照备份。

> [AZURE.NOTE] Azure SQL 数据库会自动为你可以还原的每个用户数据库创建和维护备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)。

复制过程完成后，新数据库将完全正常工作，并独立于源数据库。完成复制时，新数据库的事务处理方式将与源数据库保持一致。数据库副本的服务层和性能级别（定价层）都与源数据库相同。在完成该复制后，副本将成为能够完全行使功能的独立数据库。登录名、用户和权限可单独进行管理。


将某个数据库复制到同一逻辑服务器时，可以在这两个数据库上使用相同的登录名。用于复制该数据库的安全主体将成为新数据库上的数据库所有者 (DBO)。所有数据库用户、其权限及安全标识符 (SID) 都复制到数据库副本中。


若要复制 SQL 数据库，需要做好以下准备：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用版”，然后再回来完成本文的相关操作即可。
- 要复制的 SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)文章中的步骤创建一个。



## 复制 SQL 数据库

打开要复制的数据库对应的 SQL 数据库边栏选项卡：

1.	转到 [Azure 门户](https://manage.windowsazure.cn)。
2.	转到要复制的数据库：“浏览”>“SQL 数据库”
3.	在 SQL 数据库边栏选项卡中，单击“复制”以打开“复制”边栏选项卡：
1.  输入数据库副本的名称。系统会提供默认名称，但你可以根据需要更改。
2.  选择**目标服务器**。目标服务器是要在其中创建数据库副本的位置。你可以创建新的服务器，或者从列表中选择现有的服务器。
3.  单击“确定”开始复制过程。






## 监视复制操作的进度

2.	开始复制后，单击门户通知以了解详细信息。




 






## 验证数据库位于服务器上

- 单击“浏览”>“SQL 数据库”并检查新数据库是否处于“联机”状态。



## 后续步骤

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms)
- [将数据库导出到 BACPAC](/documentation/articles/sql-database-export)



## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases)


<!--Image references-->
[1]: ./media/sql-database-copy/copy.png
[2]: ./media/sql-database-copy/copy-ok.png
[3]: ./media/sql-database-copy/copy-notification.png
[4]: ./media/sql-database-copy/monitor-copy.png

<!---HONumber=Mooncake_0307_2016-->