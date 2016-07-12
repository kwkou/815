<properties
	pageTitle="备份和还原已启用延伸的数据库 | Azure"
	description="了解如何备份和还原已启用延伸的数据库。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>


# 备份和还原已启用延伸的数据库 

若要备份和还原已启用延伸的数据库，可以继续使用当前所用的方法。有关 SQL Server 备份和还原的详细信息，请参阅[备份和还原 SQL Server 数据库](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx)。

对已启用延伸的数据库进行的备份是一种浅表备份，这种备份不包括将数据迁移到远程服务器。

SQL Server Stretch Database完全支持时间点还原。将 SQL Server 数据库还原到某个时间点并重新授权与 Azure 的连接之后，SQL Server Stretch Database会将远程数据协调到同一个时间点。有关 SQL Server 中的时间点还原的详细信息，请参阅[将 SQL Server 数据库还原到某个时间点（完全恢复模型）](https://msdn.microsoft.com/zh-cn/library/ms179451.aspx)。有关在还原之后重新授权与 Azure 的连接时所要运行的存储过程的信息，请参阅 [sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx)。

## <a name="Reconnect"></a>从备份中还原已启用延伸的数据库

1.  从备份中还原数据库。

2.  为已启用延伸的数据库创建数据库主密钥。有关详细信息，请参阅 [CREATE MASTER KEY (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx)

3.  为已启用延伸的数据库创建数据库范围的凭据。有关详细信息，请参阅 [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)。

4.  运行存储过程 [sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx)，以将已启用延伸的本地数据库重新连接到 Azure。提供在前一步骤中创建的数据库范围凭据作为 sysname 或 varchar(128) 值。（不要使用 varchar(max)。）

    ```tsql
    Declare @credentialName nvarchar(128);
    SET @credentialName = N’<database_scoped_credential_name_created_previously>’;
    EXEC sp_rda_reauthorize_db @credentialName;
    ```

## <a name="MoreInfo"></a>有关备份和还原的详细信息
对已启用SQL Server Stretch Database的数据库创建的备份只包含运行备份时所处时间点的本地数据和符合条件的数据。这些备份还包含有关数据库远程数据所在的远程终结点的信息。这称为“浅表备份”。包含数据库中所有本地和远程数据的深度备份不受支持。

![SQL Server Stretch Database备份示意图][StretchBackupImage1]

当你还原已启用SQL Server Stretch Database的数据库备份时，此操作会按预期将本地数据和符合条件的数据还原到数据库。（符合条件的数据是指尚未移动，但会根据表的SQL Server Stretch Database配置移到 Azure 的数据。） 运行还原操作后，数据库将包含运行备份时所处时间点的本地数据和符合条件的数据，但不包含连接到远程终结点所需的凭据和项目。

![备份后的SQL Server Stretch Database][StretchBackupImage2]

必须运行存储过程 **sys.sp\_rda\_reauthorize\_db** 才能在本地数据库及其远程终结点之间重新建立连接。只有 db\_owner 才能执行此操作。此存储过程还需要远程终结点的管理员用户名和密码。这意味着，你必须提供目标 Azure 服务器的管理员登录名和密码。

![备份后的SQL Server Stretch Database][StretchBackupImage3]

重新建立连接后，SQL Server Stretch Database会尝试通过在远程终结点上创建远程数据的副本并将其链接到本地数据库，将本地数据库中符合条件的数据与远程数据相协调。此过程是自动进行的，无需用户干预。运行这种协调后，本地数据库和远程终结点将处于一致状态。

![备份后的SQL Server Stretch Database][StretchBackupImage4]

## 另请参阅
[SQL Server Stretch Database的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)
[sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx)
[备份和还原 SQL Server 数据库](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx)

<!--Image references-->
[StretchBackupImage1]: ./media/sql-server-stretch-database-backup/StretchDBBackup1.png
[StretchBackupImage2]: ./media/sql-server-stretch-database-backup/StretchDBBackup2.png
[StretchBackupImage3]: ./media/sql-server-stretch-database-backup/StretchDBBackup3.png
[StretchBackupImage4]: ./media/sql-server-stretch-database-backup/StretchDBBackup4.png

<!---HONumber=Mooncake_0307_2016-->