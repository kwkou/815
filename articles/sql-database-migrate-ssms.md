<properties
   pageTitle="使用 SSMS 迁移到 SQL 数据库"
   description="Microsoft Azure SQL 数据库, 迁移 SQL 数据库, 使用 ssms 迁移"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="08/24/2015"
   wacn.date="11/02/2015"/>

#使用迁移 SSMS 兼容的数据库

![SSMS 迁移示意图](./media/sql-database-migrate-ssms/01SSMSDiagram.png)

如果数据库架构与 Azure SQL 数据库兼容，则迁移只需将数据库导入到 Azure。你可以使用 SSMS，通过将数据库部署到 Azure SQL 数据库，以一个步骤来实现此目的；或者，可以分两步来完成：先导出数据库的 BACPAC，然后将该 BACPAC 导入到 Azure SQL Server 作为新数据库。

导出 BACPAC 时，可以将其导出到本地文件或直接导出到 Azure blob。如果本地导出它，可以将导出的 BACPAC 上载到 Azure blob 中。将 BACPAC 文件存储在 Azure blob 中后，可以使用 Azure 门户导入 BACPAC 作为数据库。在 Azure 门户中运行导入可以降低导入步骤出现的延迟，从而提高迁移大型数据库时的性能和可靠性。

直接从 SSMS 部署将始终部署整个架构和所有数据，而使用 BACPAC 导出则允许你选择要导出到 BACPAC 的对象子集。导出的 BACPAC 始终包括完整的数据库架构，默认情况下还包括所有表中的数据。无论是从 SSMS 部署，还是从 SSMS（或 Azure 门户）导出再导入，实质上使用的是相同的 DAC 技术，而结果也是相同的。

此选项也用作选项 #2 中的最后一个步骤，用于在更新数据库后迁移数据库，使之与 Azure SQL 数据库兼容。

## 获取最新版本的 SQL Server Management Studio

使用最新版本的 Microsoft SQL Server Management Studio for SQL Server 可确保你使用 SSMS 中的最新工具更新来与 Azure 门户进行交互。若要获取 Microsoft SQL Server Management Studio for SQL Server 的最新版本，请[下载它](https://msdn.microsoft.com/library/mt238290.aspx)并将其安装在与计划迁移的数据库和 Internet 建立了连接的客户端计算机上。

##使用 SSMS 部署到 Azure SQL 数据库
1.	使用 Azure 管理门户设置逻辑服务器。
2.	在 SSMS 对象资源管理器中找到源数据库，然后执行**导出数据层应用程序…** 任务

	![通过“任务”菜单部署到 Azure](./media/sql-database-migrate-ssms/02MigrateusingSSMS.png)

3.	在部署向导中，配置与你在步骤 1 中预配的目标 Azure SQL 数据库逻辑服务器的连接。
4.	提供数据库的**名称**，并设置“版本”（服务层）和“服务目标”（性能级别）。<!--有关配置这些设置的详细信息，请参阅 [Azure SQL 数据库服务层](sql-database-service-tiers.md)。-->

	![导出设置](./media/sql-database-migrate-ssms/03MigrateusingSSMS.png)

5.	完成向导，以执行数据库迁移。根据数据库的大小和复杂性，部署可能需要花费几分钟到几小时。如果存在不兼容项，架构验证例程将在任何到 Azure 的部署实际进行前快速检测错误。如果错误指示数据库架构与 SQL 数据库不兼容，请使用迁移选项 #2。请参阅[就地更新数据库，然后部署到 Azure SQL 数据库](/documentation/articles/sql-database-migrate-visualstudio-ssdt)。

##使用 SSMS 导出 BACPAC，然后将其导入 SQL 数据库
部署过程可分为两个步骤：导出和导入。在第一个步骤中，将创建一个 BACPAC 文件，该文件稍后将用作第二个步骤的输入。

1.	使用 Azure 管理门户设置逻辑服务器。
2.	在 SSMS 对象资源管理器中找到源数据库，然后选择**将数据库部署到 Azure SQL 数据库...** 任务

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-migrate-ssms/04MigrateusingSSMS.png)

3. 在导出向导中，配置导出以将 BACPAC 文件保存到本地磁盘位置或 Azure blob。导出的 BACPAC 始终包括完整的数据库架构，默认情况下还包括所有表中的数据。如果你要排除部分或全部表中的数据，请使用“高级”选项卡。例如，你可以选择仅导出引用表的数据。

	![导出设置](./media/sql-database-migrate-ssms/05MigrateusingSSMS.png)

4.	在创建 BACPAC 后，请连接到步骤 1 中创建的服务器，右键单击“数据库”文件夹，然后选择“导入数据层应用程序...”
	
	>[AZURE.NOTE] 注意：你还可以从 Azure 管理门户中导入存储在 Azure blob 中的 BACPAC 文件。

	![导入数据层应用程序菜单项](./media/sql-database-migrate-ssms/06MigrateusingSSMS.png)

5.	在导入向导中，选择你刚导出的 BACPAC 文件，以在 Azure SQL 数据库中创建新的数据库。

	![导入设置](./media/sql-database-migrate-ssms/07MigrateusingSSMS.png)

6.	提供数据库的名称，并设置“版本”（服务层）和“服务目标”（性能级别）。

7.	完成向导以导入 BACPAC 文件，并在 Azure SQL 数据库中创建数据库。

	![数据库设置](./media/sql-database-migrate-ssms/08MigrateusingSSMS.png)

##备选方法
你也可以使用命令行实用工具 sqlpackage.exe 来部署数据库，或者导出并导入 BACPAC。Sqlpackage.exe 使用的 DAC 技术与 SSMS 相同，因此结果是相同的。有关详细信息，请参阅 [MSDN 上的 SqlPackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx)。

<!---HONumber=76-->