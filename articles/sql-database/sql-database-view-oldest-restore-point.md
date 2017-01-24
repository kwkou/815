<properties
    pageTitle="查看服务生成的 Azure SQL 数据库备份的最早还原点 | Azure"
    description="有关如何查看服务生成的数据库备份的最早还原点的快速参考"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.service="sql-database"
    ms.custom="business continuity"
    ms.devlang="NA"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.date="12/07/2016"
    wacn.date="01/20/2017"
    ms.author="carlrab" />  


# 查看服务生成的 Azure SQL 数据库备份的最早还原点

本操作方法主题介绍如何查看服务生成的 Azure SQL 数据库备份的最早还原点。

## 使用 Azure 门户预览查看最早还原点

1. 打开数据库的“SQL 数据库”边栏选项卡。

    ![“新建示例数据库”边栏选项卡](./media/sql-database-get-started/new-sample-db-blade.png)  


2. 在工具栏上，单击“还原”。

    ![还原工具栏](./media/sql-database-get-started-backup-recovery/restore-toolbar.png)  


3. 在“还原”边栏选项卡上，查看最早还原点。

    ![最早还原点](./media/sql-database-get-started-backup-recovery/oldest-restore-point.png)  


> [AZURE.TIP]
有关教程，请参阅[开始使用备份和还原进行数据保护和恢复](/documentation/articles/sql-database-get-started-backup-recovery/)
>

## 后续步骤

- 若要了解服务生成的自动备份，请参阅[自动备份](/documentation/articles/sql-database-automated-backups/)
- 若要了解如何从备份还原，请参阅[从备份还原](/documentation/articles/sql-database-recovery-using-backups/)

<!---HONumber=Mooncake_0116_2017-->