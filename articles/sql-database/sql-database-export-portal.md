<properties
    pageTitle="Azure 门户：将 Azure SQL 数据库导出到 BACPAC 文件 | Azure"
    description="使用 Azure 门户将 Azure SQL 数据库导出到 BACPAC 文件"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA" />  


# 使用 Azure 门户将 Azure SQL 数据库导出到 BACPAC 文件

本文说明了如何使用 [Azure 门户](https://portal.azure.cn)将 Azure SQL 数据库导出到 BACPAC 文件（存储在 Azure Blob 存储中）。如需大致了解如何导出到 BACPAC 文件，请参阅[导出到 BACPAC](/documentation/articles/sql-database-export/)。

> [AZURE.NOTE]
也可使用 [SQL Server Management Studio](/documentation/articles/sql-database-export-ssms/)、[PowerShell](/documentation/articles/sql-database-export-powershell/) 或 [SQLPackage](/documentation/articles/sql-database-export-sqlpackage/) 将 Azure SQL 数据库文件导出到 BACPAC 文件。
>

## 先决条件

若要完成本文，需要以下各项：

* Azure 订阅。
* Azure SQL 数据库。
* [Azure 标准存储帐户](/documentation/articles/storage-create-storage-account/)，具有可在标准存储中存储 BACPAC 的 blob 容器。

## 导出数据库

1. 转到 [Azure 门户](https://portal.azure.cn)。
2. 打开要导出的数据库对应的 SQL 数据库边栏选项卡。
3. 确保在导出过程中没有事务处理。
    
    > [AZURE.IMPORTANT]
    >若要确保获得事务处理一致性 BACPAC 文件，可以[创建数据库的副本](/documentation/articles/sql-database-copy/)，然后从该数据库副本进行导出。
    > 

4. 单击“SQL 数据库”。
5. 单击要存档的数据库。
6. 在“SQL 数据库”边栏选项卡中，单击“导出”以打开“导出数据库”边栏选项卡：
   
    ![导出按钮][1]  

7. 单击“存储”并选择用于存储 BACPAC 的存储帐户和 Blob 容器：
   
    ![导出数据库][2]  

8. 选择身份验证类型。
9. 为包含要导出的数据库的 Azure SQL Server 输入相应的身份验证凭据。
10. 单击“确定”导出数据库。单击“确定”创建一个导出数据库请求并将其提交给服务。导出所需的时间取决于数据库的大小和复杂性，以及使用的服务级别。
11. 请查看收到的通知。
   
    ![导出通知][3]  


## 监视导出操作的进度
1. 单击“SQL Server”。
2. 单击包含存档的原始（源）数据库的服务器。
3. 向下滚动到操作。
4. 在“SQL Server”边栏选项卡中，单击“导入/导出历史记录”：
   
    ![导入导出历史记录][4]

## 确认 BACPAC 位于你的存储容器中
1. 单击“存储帐户”。
2. 单击你在其中存储 BACPAC 存档的存储帐户。
3. 单击“容器”，然后选择数据库所导出到的容器以了解详细信息（从这里可以下载和保存 BACPAC）。
   
    ![.bacpac 文件详细信息][5]

## 后续步骤


* 若要了解如何将 BACPAC 导入到 SQL Server 数据库，请参阅[将 BACPCAC 导入 SQL Server 数据库](https://msdn.microsoft.com/zh-cn/library/hh710052.aspx)

<!--Image references-->

[1]: ./media/sql-database-export/export.png
[2]: ./media/sql-database-export/export-blade.png
[3]: ./media/sql-database-export/export-notification.png
[4]: ./media/sql-database-export/export-history.png
[5]: ./media/sql-database-export/bacpac-archive.png

<!---HONumber=Mooncake_0320_2017-->