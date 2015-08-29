<properties
   pageTitle="使用 SSMS 迁移到 SQL Database"
   description="Microsoft Azure SQL Database, 迁移 sql Database, 使用 ssms 迁移"
   services="sql-database"
   documentationCenter=""
   authors="kaivalyah2015"
   manager="jeffreyg"
   editor="monicar"/>

<tags
 wacn.date="05/20/2015" 
   ms.service="sql-database"
   ms.date="04/14/2015"/>

# 使用迁移 SSMS 兼容的数据库 

![SSMS 迁移示意图](./media/sql-database-migrate-ssms/01SSMSDiagram.png)

如果数据库架构已与 Azure SQL Database 兼容，则迁移将十分直截了当。如果不需要进行架构修改，则迁移过程只需将数据库导入 Azure。你可以使用 SSMS，通过将数据库“部署”到 Azure SQL Database，以一个步骤来实现此目的；或者，可以分两步来完成：先导出当前数据库的 BACPAC，然后将该 BACPAC 导入到 Azure 以创建新的 Azure SQL 数据库。

可以将导出的 BACPAC 上载到 Azure 存储空间，然后使用门户导入 BACPAC。在云中运行导入可以降低导入步骤出现的延迟，从而提高迁移大型数据库时的性能和可靠性。

直接从 SSMS 部署的方法始终会部署架构和数据，而导出再导入的方法始终会部署架构，并提供一个选项用于部署所有或一部分表中的数据。不管是从 SSMS 部署，还是从 SSMS（然后通过门户）导出再导入，实质上使用的是相同的 DAC 技术，而结果也是相同的。

此选项也相当于选项 #3 中的最后一个步骤，用于在更新数据库后迁移数据库，使之与 Azure SQL Database 兼容。

## 使用 SSMS 部署到 Azure SQL Database
1.	使用 Azure 管理门户设置逻辑服务器。
2. 在 SSMS 对象资源管理器中找到源数据库，然后执行**将数据库部署到 Azure SQL Database...** 任务

	![通过“任务”菜单部署到 Azure](./media/sql-database-migrate-ssms/02MigrateusingSSMS.png)

3.	在部署向导中，配置与设置的目标 Azure SQL Database 服务器的连接。
4.	提供数据库的名称，并设置“版本”（服务层）和“服务目标”（性能级别）。有关配置这些设置的详细信息，请参阅“选择数据库性能级别/定价层以进行迁移”。 

	![导出设置](./media/sql-database-migrate-ssms/03MigrateusingSSMS.png)

5.	完成向导，以执行数据库迁移。根据数据库的大小和复杂性，部署可能需要花费几分钟到几小时。如果发生了指出数据库架构与 SQL Database 不兼容的错误，你必须使用其他选项。
## 使用 SSMS 导出 BACPAC，然后将其导入 SQL Database
部署过程可分为两个步骤：导出和导入。在第一个步骤中，将创建一个 BACPAC 文件，该文件稍后将用作第二个步骤的输入。

1.	使用 Azure 管理门户设置逻辑服务器。
2.	在 SSMS 对象资源管理器中找到源数据库，然后选择**将数据库部署到 Azure SQL Database...** 任务

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-migrate-ssms/04MigrateusingSSMS.png)

3. 在导出向导中配置导出，以在本地保存 BACPAC 文件。导出的 BACPAC 始终包括完整的数据库架构，默认情况下还包括所有表中的数据。如果你要排除部分或全部表中的数据，请使用“高级”选项卡。例如，你可以选择仅导出引用表的数据。
	>[AZURE.NOTE]注意：在 Azure 管理门户支持在 Azure 中运行导入后，你可以选择将导出的 BACPAC 文件保存到 Azure 存储空间，并在云中运行导入。

	![导出设置](./media/sql-database-migrate-ssms/05MigrateusingSSMS.png)

4.	在创建 BACPAC 后，请连接到步骤 1 中创建的服务器，右键单击“数据库”文件夹，然后选择“导入数据层应用程序...”

	![导入数据层应用程序菜单项](./media/sql-database-migrate-ssms/06MigrateusingSSMS.png)

5.	在导入向导中，选择你刚导出的 BACPAC 文件，以在 Azure SQL Database 中创建新的数据库。

	![导入设置](./media/sql-database-migrate-ssms/07MigrateusingSSMS.png)

6.	提供数据库的名称，并设置“版本”（服务层）和“服务目标”（性能级别）。
	 
7.	完成向导以导入 BACPAC 文件，并在 Azure SQL Database 中创建数据库。

	![数据库设置](./media/sql-database-migrate-ssms/08MigrateusingSSMS.png)
 
## 备选方法
你也可以使用命令行实用工具 sqlpackage.exe 来部署数据库，或者导出并导入 BACPAC。Sqlpackage.exe 使用的 DAC 技术与 SSMS 相同，因此结果是相同的。有关详细信息，请转到[此处](https://msdn.microsoft.com/zh-CN/library/hh550080.aspx)。

<!---HONumber=66-->