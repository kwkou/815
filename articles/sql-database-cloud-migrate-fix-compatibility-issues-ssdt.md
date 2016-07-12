<properties
   pageTitle="在迁移到 SQL 数据库之前，修复 SQL Server 数据库兼容性问题"
   description="Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导, SSDT"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="03/22/2016"
   wacn.date="05/16/2016"/> 

# 使用 SQL Server Data Tools for Visual Studio 将 SQL Server 数据库迁移到 Azure SQL 数据库

可以使用 SQL Server Data Tools for Visual Studio ("SSDT") 将数据库架构导入到 Visual Studio 数据库项目中以进行分析。若要分析，请将该项目的目标平台指定为 SQL 数据库 V12，然后生成该项目。如果生成成功，则数据库是兼容的。如果生成失败，则可以解决 SSDT（或本主题中讨论的其他工具之一）中的错误。成功生成项目后，你便可以将其发布为源数据库的副本，然后使用 SSDT 中的数据比较功能将数据从源数据库复制到这个与 Azure SQL V12 兼容的数据库。然后，可以迁移这个已更新的数据库。若要使用此选项，请下载[最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)。

  ![VSSSDT 迁移示意图](./media/sql-database-cloud-migrate/03VSSSDTDiagram.png)

  > [AZURE.NOTE] 如果需要进行仅有架构的迁移，则可以将架构直接从 Visual Studio 发布到 Azure SQL 数据库。当数据库架构所需的更改量超过了 SAMW 单独可处理的数量时使用。

## 使用 SQL Server Data Tools for Visual Studio 检测兼容性问题
   
1.	在 Visual Studio 中打开“SQL Server 对象资源管理器”。使用“添加 SQL Server ”连接到包含被迁移数据库的 SQL Server 实例。在资源管理器中找到数据库，右键单击它并选择“创建新项目...”     
    
	![新建项目](./media/sql-database-migrate-visualstudio-ssdt/02MigrateSSDT.png)
   
2.	将导入设置配置为“仅导入应用程序范围对象”。取消选中导入以下项的选项：引用的登录名、权限和数据库设置。

    ![替换文字](./media/sql-database-migrate-visualstudio-ssdt/03MigrateSSDT.png)

3.	单击“启动”以导入数据库并创建项目，项目将包含数据库中每个对象的 T-SQL 脚本文件。这些脚本文件嵌入到项目中的文件夹内。

    ![替换文字](./media/sql-database-migrate-visualstudio-ssdt/04MigrateSSDT.png)

4.	在 Visual Studio 解决方案资源管理器中，右键单击数据库项目并选择“属性”。此操作将打开“项目设置”页，在该页上应将“目标平台”配置为“Azure SQL 数据库 V12”。
    
    ![替换文字](./media/sql-database-migrate-visualstudio-ssdt/05MigrateSSDT.png)
    
5.	右键单击项目并选择“生成”以生成项目。
    
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/06MigrateSSDT.png)
    
6.	**错误列表**将显示每个不兼容项。在此示例中，用户名 NT AUTHORITY\\NETWORK SERVICE 是不兼容的。由于它不兼容，你可以将它注释掉或删除（并解决从数据库解决方案中删除此登录名和角色的影响）。
    
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/07MigrateSSDT.png)
    
## 使用 SQL Server Data Tools for Visual Studio 解决兼容性问题      

1.	双击第一个脚本以在查询窗口中打开该脚本，注释掉该脚本，然后执行该脚本。        

  	 ![替换文字](./media/sql-database-migrate-visualstudio-ssdt/08MigrateSSDT.png)    

2.	对包含不兼容项的每个脚本重复执行此过程，直到不再存在错误。    

	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/09MigrateSSDT.png)

3.	当数据库无错误后，右键单击项目并选择“发布”以生成数据库并将其发布到源数据库副本（强烈建议至少在开始时使用副本）。     
 - 在发布前，根据源 SQL Server 版本（早于 SQL Server 2014），可能需要重置项目的目标平台才能进行部署。     
 - 如果迁移的是较早的 SQL Server 数据库，切勿向项目中引入任何不受源 SQL Server 支持的功能，除非先将数据库迁移到较新版本的 SQL Server。     

    	![alt text](./media/sql-database-migrate-visualstudio-ssdt/10MigrateSSDT.png)    
    
    	![alt text](./media/sql-database-migrate-visualstudio-ssdt/11MigrateSSDT.png)    
    
4.	在 SQL Server 对象资源管理器中，右键单击你的源数据库，然后单击“数据比较”，将项目与原始数据库进行比较以了解向导进行了哪些更改。选择 Azure SQL V12 版本的数据库，然后单击“完成”。    
    
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/12MigrateSSDT.png)
    
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/13MigrateSSDT.png)
    
5.	查看检测到的差异，然后单击“更新目标”，以将源数据库中的数据迁移到 Azure SQL V12 数据库。
    
	![替换文字](./media/sql-database-migrate-visualstudio-ssdt/14MigrateSSDT.png)
    
6.	选择部署方法。请参阅 [Migrate a compatible SQL Server database to SQL Database（将兼容的 SQL Server 数据库迁移到 SQL 数据库）](/documentation/articles/sql-database-cloud-migrate/)。  

<!---HONumber=Mooncake_0503_2016-->
