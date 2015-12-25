<properties
   pageTitle="将 SQL Server 数据库迁移到 Azure SQL 数据库"
   description="Windows Azure SQL 数据库, 数据库部署, 数据库迁移, 导入数据库, 导出数据库, 迁移向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="10/12/2015"
   wacn.date="12/22/2015"/>

# 将 SQL Server 数据库迁移到 Azure SQL 数据库

将本地数据库迁移到 Azure SQL 数据库的复杂性根据数据库和应用程序设计，以及对于停机时间的容忍度而异。对于兼容的数据库来说，迁移到 Azure SQL 数据库是一个直截了当的架构和数据转移操作，它只需对架构进行少量的更改（如果有），并且只需对应用程序进行轻微的改造，甚至不需要任何改造。[Azure SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new) 几乎与 SQL Server 2014 及 SQL Server 2016 的引擎完全兼容。Microsoft Azure SQL 数据库完全支持大多数 SQL Server 2016 Transact-SQL 语句。这包括 SQL Server 数据类型、运算符、字符串、算术、逻辑、游标函数以及大多数应用程序依赖的其他 Transact-SQL 元素。部分支持或不支持的函数通常与 SQL 数据库管理数据库的差异相关（例如文件、高可用性和安全功能）或者与专用功能（例如服务代理）相关。由于 SQL 数据库会将许多功能与 master 数据库上的依赖性隔离，因此，许多服务器级活动都是不适当的且不受支持。在 SQL Server 中弃用的功能一般不受 SQL 数据库的支持。依赖[部分支持或不受支持功能](/documentation/articles/sql-database-transact-sql-information)的数据库和应用程序需要进行一些重新设计才能进行迁移。

将 SQL Server 数据库迁移到 Azure SQL 数据库的工作流包括：

 1. [确定数据库是否兼容](#determine-if-your-database-is-compatible)
 2. [如果不兼容，修复数据库兼容性问题](#fix-database-compatibility-issues)
 3. [迁移兼容的数据库](#options-to-migrate-a-compatible-database-to-azure-sql-database)

>[AZURE.NOTE]若要将其他类型的数据库（包括 Microsoft Access、Sybase、MySQL Oracle 和 DB2）迁移到 Azure SQL 数据库，请参阅 [SQL Server 迁移助手](http://blogs.msdn.com/b/ssma/)。

## 确定数据库是否兼容
有两种主要方法可用于确定源数据库是否兼容。
- 导出数据层应用程序：此方法使用 Management Studio 中的向导分析数据库，并在控制台上显示数据库兼容问题（如果有）。
- SQLPackage：此方法使用 SQLPackage.exe [sqlpackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用程序分析数据库并生成报告。SQLPackage 随 Visual Studio 和 SQL Server 一起提供。

> [AZURE.NOTE]第三种方法使用跟踪文件作为额外的源信息测试应用程序级别以及数据库级别的兼容性。这是 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com/)，Codeplex 上的一个免费工具。但是，此工具当前可以发现兼容性错误，这些错误对于 Azure SQL 数据库 V11 是问题，对于 Azure SQL 数据库 V12 不是问题。

如果检测到数据库不兼容，你必须修复这些不兼容，然后才能将数据库迁移到 Azure SQL 数据库。有关如何修复数据库兼容性问题的指导，请转到[修复数据库兼容性问题](#fix-database-compatibility-issues)。

> [AZURE.IMPORTANT]这些选项不会捕获不同级别（即级别 90、100 和 110）的 SQL Server 数据库之间的所有兼容性问题。如果你从较旧的数据库（级别 80、90、100 和 110）进行迁移，则应先完成升级过程（至少在开发环境中），打开 SQL Server 2014 或更高版本，然后迁移到 Azure SQL 数据库。

## 使用 sqlpackage.exe 确定数据库是否兼容

1. 打开命令提示符并更改包含 sqlpackage.exe 最新版本的目录。此实用程序随 Visual Studio 和 SQL Server 一起提供。你还可以[下载](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)最新版本的 SQL Server Data Tools 以获取此实用程序。
2. 结合环境的以下参数运行以下 sqlpackage.exe 命令：

	'sqlpackage.exe /Action:Export /ssn:< server_name > /sdn:< database_name > /tf:< target_file > /p:TableData=< schema_name.table_name > > < output_file > 2>&1'

	| 参数 | 说明 |
	|---|---|
	| < server_name > | 源服务器名称 |
	| < database_name > | 源数据库名称 |
	| < target_file > | BACPAC 文件的文件名和位置 |
	| < schema_name.table_name > | 其数据将输出到目标文件的表 |
	| < output_file > | 包含错误（如果有）的输出文件的文件名和位置 |

	使用 /P:TableName 参数的原因是：我们只想要测试导出到 Azure SQL 数据库 V12 的数据库兼容性，而不是要测试从所有表导出数据。遗憾的是，sqlpackage.exe 的导出参数不支持不提取任何表，因此将需要指定一个小型表。< output_file > 将包含任何错误的报告。“> 2>&1”字符串将执行命令后生成的标准输出和标准错误传送到指定的输出文件。

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01.png)

3. 打开输出文件并查看兼容性错误（如果有）。有关如何修复数据库兼容性问题的指导，请转到[修复数据库兼容性问题](#fix-database-compatibility-issues)。

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage02.png)

## 使用导出数据层应用程序确定数据库是否兼容

1. 确认你安装了版本 13.0.600.65 或更高版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

 	 >[AZURE.IMPORTANT]下载[最新](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)版本的 SQL Server Management Studio。建议你始终使用最新版本的 Management Studio。

2. 打开 Management Studio 并连接到你在对象资源管理器中的源数据库。
3. 右键单击对象资源管理器中的源数据库，指向“任务”，然后单击“导出数据层应用程序...”

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS01.png)

4. 在导出向导中，单击“下一步”，然后在“设置”选项卡上配置导出，以将 BACPAC 文件保存到本地磁盘位置或 Azure Blob。只有在没有数据库兼容性问题时，才会保存 BACPAC 文件。如果有兼容性问题，这些问题将显示在控制台上。

	![导出设置](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS02.png)

5. 单击“高级”选项卡并清除“全选”复选框以跳过导出数据。此时我们的目标仅是测试兼容性。

	![导出设置](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS03.png)

6. 单击“下一步”，然后单击“完成”。在向导验证架构后，将显示数据库兼容性问题（如果有）。

	![导出设置](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS04.png)

7. 如果未显示任何错误，则数据库是兼容的，你已可以进行迁移。如果有错误，你需要修复它们。若要查看错误，请单击“验证架构”所对应的错误。有关如何修复这些错误的信息，请转到[修复数据库兼容性问题](#fix-database-compatibility-issues)。

	![导出设置](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS05.png)

8.	如果 *.BACPAC 文件已成功生成，则数据库与 SQL 数据库兼容，并已准备好迁移。

## 用于将兼容数据库迁移到 Azure SQL 数据库的选项

确认具有兼容的数据库之后，需要选择迁移方法。首先，需要确定是否经得起在迁移期间将数据库移出生产环境以外。如果不能，请使用下面讨论的 SQL Server 事务复制。如果可以承受一定的停机时间，或者是执行生产数据库的测试迁移，稍后可能选择使用事务复制进行迁移，请考虑以下三种方法之一。

### 在造成停机的情况下迁移兼容的数据库   
以下列表介绍了当迁移发生时，以及在将用户和应用程序指向 Azure SQL 数据库中的迁移数据库之前，可以承受一定停机时间的情况下，将兼容数据库迁移到 Azure SQL 数据库的选项。使用这些方法可以在数据库存在的某个时间点迁移数据库。

> [AZURE.WARNING]在使用任何一种方法迁移数据库之前，请确保没有任何活动的事务，以确保迁移期间的事务一致性。有许多方法可以静止数据库，例如禁用客户端连接或者创建[数据库快照](https://msdn.microsoft.com/zh-cn/library/ms175876.aspx)。

- 对于小型到中型数据库，迁移[兼容的](#determine-if-your-database-is-compatible) SQL Server 2005 或更高版本数据库只需运行 SQL Server Management Studio 中的“将数据库部署到 Microsoft Azure 数据库”向导即可[](#use-the-deploy-database-to-microsoft-azure-database-wizard)。如果存在连接问题（无连接、低带宽或超时问题），你可以[使用 BACPAC 将](#use-a-bacpac-to-migrate-a-database-to-azure-sql-database) SQL Server 数据库迁移到 Azure SQL 数据库。
- 对于中型到大型数据库或存在连接问题时，可[使用 BACPAC 将](#use-a-bacpac-to-migrate-a-database-to-azure-sql-database) SQL Server 数据库迁移到 Azure SQL 数据库。使用此方法时，可使用 SQL Server Management Studio 将数据和架构导出到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件（存储在本地或 Azure blob 中），然后将 BACPAC 文件导入到 Azure SQL 实例中。如果将 BACPAC 存储在 Azure blob 中，还可以从 [Azure 门户](/documentation/articles/sql-database-import)中或[使用 PowerShell](/documentation/articles/sql-database-import-powershell) 导入 BACPAC 文件。
- 对于大型数据库，通过单独迁移架构和数据可以实现最佳性能。使用此方法，创建[没有数据的 BACPAC 文件](#use-a-bacpac-to-migrate-a-database-to-azure-sql-database)，并将此 BACPAC 导入 Azure SQL 数据库。将架构导入 Azure SQL 数据库后，然后使用 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 将数据提取到平面文件中，然后将这些文件导入到 Azure SQL 数据库。

	 ![SSMS 迁移示意图](./media/sql-database-cloud-migrate/01SSMSDiagram_new.png)

### 在不造成停机的情况下迁移兼容的数据库

如果在迁移发生时你无法承受从实际运行中删除 SQL Server 数据库，可以使用 SQL Server 事务复制作为迁移解决方案。使用事务复制，开始迁移和完成迁移之间发生的数据或架构的所有更改都出现在 Azure SQL 数据库中。迁移完成后，你只需要更改应用程序的连接字符串，将其指向 Azure SQL 数据库，而不是指向本地数据库。一旦事务复制清空保留在本地数据库中的任何更改，并且所有应用程序指向 Azure DB，你就可以安全地卸载复制，使 Azure SQL 数据库作为生产系统。

 ![SeedCloudTR 示意图](./media/sql-database-cloud-migrate/SeedCloudTR.png)


事务复制是一种内置技术，与 SQL Server 6.5 和更高版本的 SQL Server 集成。它是非常成熟且经过证实的技术，大多数 DBA 运用现有的经验就能操作。现在，可以使用 [SQL Server 2016 预览版](http://www.microsoft.com/server-cloud/products/sql-server-2016/)将 Azure SQL 数据库配置为本地发布的[事务复制订阅者](https://msdn.microsoft.com/zh-cn/library/mt589530.aspx)。从 Management Studio 设置的体验与在本地服务器上设置事务复制订阅者完全相同。以下 SQL Server 版本支持此方案：

 - SQL Server 2016 CTP3（预览版）和更高版本 
 - SQL Server 2014 SP1 CU3 和更高版本
 - SQL Server 2014 RTM CU10 和更高版本
 - SQL Server 2012 SP2 CU8 和更高版本
 - SQL Server 2013 SP3（发行时）

还可以使用事务复制来迁移本地数据库的子集。复制到 Azure SQL 数据库的发布可以限制为复制的数据库中表的子集。此外，对于复制的每一个表，可以将数据限制为行的子集和/或列的子集。

## 使用“将数据库部署到 Microsoft Azure 数据库”向导

SQL Server Management Studio 中的“将数据库部署到 Windows Azure 数据库”向导可将[兼容的](#determine-if-your-database-is-compatible) SQL Server 2005 或更高版本数据库直接迁移到 Azure SQL 逻辑服务器实例。

> [AZURE.NOTE]下面的步骤假定你[已预配](/documentation/articles/sql-database-get-started) Azure SQL 逻辑实例并且手头有连接信息。

1. 确认你安装了版本 13.0.600.65 或更高版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

	 >[AZURE.IMPORTANT]下载[最新](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)版本的 SQL Server Management Studio。建议你始终使用最新版本的 Management Studio。

2. 打开 Management Studio 并连接到你在对象资源管理器中的源数据库。
3. 右键单击对象资源管理器中的源数据库，指向“任务”，然后单击“将数据库部署到 Windows Azure SQL 数据库”

	![通过“任务”菜单部署到 Azure](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard01.png)

4.	在部署向导中，单击“下一步”，然后单击“连接”以配置与 Azure SQL 数据库服务器的连接。

	![通过“任务”菜单部署到 Azure](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard002.png)

5. 在“连接到服务器”对话框中，输入用于连接到 Azure SQL 数据库服务器的连接信息。

	![通过“任务”菜单部署到 Azure](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard00.png)

5.	为 Azure SQL 数据库中的数据库提供**新数据库名称**，设置 **Windows Azure SQL 数据库的版本**（服务层）、**最大数据库大小**、**服务目标**（性能级别）以及此向导将在迁移过程中创建的 BACPAC 文件的**临时文件名**。有关服务层和性能级别的详细信息，请参阅 [Azure SQL 数据库服务层](/documentation/articles/sql-database-service-tiers)。

	![导出设置](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard02.png)

6.	完成向导，以执行数据库迁移。根据数据库的大小和复杂性，部署可能需要花费几分钟到几小时。
7.	使用对象资源管理器连接到 Azure SQL 数据库服务器中的已迁移数据库。
8.	使用 Azure 门户查看数据库及其属性。

## 使用 BACPAC 将 SQL Server 数据库迁移到 Azure SQL 数据库

对于中型到大型数据库或当存在连接问题时，可以将迁移过程分为两个独立的步骤。你可以使用一种或两种方法将架构及其数据导出到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件中。

- [使用 SQL Server Management Studio 导出到 BACPAC 文件](/documentation/articles/#export-a-compatible-sql-server-database-to-a-bacpac-file-using-sql-server-management-studio)
- [使用 SqlPackage 导出到 BACPAC](#export-a-compatible-sql-server-database-to-a-bacpac-file-using-sqlpackage)

你可以将此 BACPAC 存储在本地或 Azure blob 中。然后可以使用几种方法之一将此 BACPAC 文件导入到 Azure SQL 数据库。

- [使用 SQL Server Management Studio 从 BACPAC 文件导入到 Azure SQL 数据库](#import-from-a-bacpac-file-into-azure-sql-database-using-sql-server-management-studio)
- [使用 SqlPackage 从 BACPAC 文件导入到 Azure SQL 数据库](#import-from-a-bacpac-file-into-azure-sql-database-using-sqlpackage)
- [使用 Azure 门户从 BACPAC 文件导入到 Azure SQL 数据库](/documentation/articles/sql-database-import)
- [使用 PowerShell 从 BACPAC 文件导入到 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell)

## 使用 SQL Server Management Studio 将兼容的 SQL Server 数据库导出到 BACPAC 文件

使用以下步骤通过 Management Studio 将[兼容的](#determine-if-your-database-is-compatible) SQL Server 数据库导出到 BACPAC 文件。

1. 确认你安装了版本 13.0.600.65 或更高版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

	 >[AZURE.IMPORTANT]下载[最新](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)版本的 SQL Server Management Studio。建议你始终使用最新版本的 Management Studio。

2. 打开 Management Studio 并连接到你在对象资源管理器中的源数据库。

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/MigrateUsingBACPAC01.png)

3. 右键单击对象资源管理器中的源数据库，指向“任务”，然后单击“导出数据层应用程序...”

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS01.png)

4. 在导出向导中，配置导出以将 BACPAC 文件保存到本地磁盘位置或 Azure blob。导出的 BACPAC 始终包括完整的数据库架构，默认情况下还包括所有表中的数据。如果你要排除部分或全部表中的数据，请使用“高级”选项卡。例如，你可以选择仅导出引用表的数据，而不是导出所有表中的数据。

	![导出设置](./media/sql-database-cloud-migrate/MigrateUsingBACPAC02.png)

## 使用 SqlPackage 将兼容的 SQL Server 数据库导出到 BACPAC 文件

使用下面的步骤通过 [SqlPackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用工具将[兼容的](#determine-if-your-database-is-compatible)数据库导出到 BACPAC 文件。

> [AZURE.NOTE]下面的步骤假定你已预配 Azure SQL 数据库服务器，手头有连接信息，并且已验证你的源数据库兼容。

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用工具的目录 - 此实用程序与 Visual Studio 和 SQL Server 一起提供。
2. 结合环境的以下参数运行以下 sqlpackage.exe 命令：

	'sqlpackage.exe /Action:Export /ssn:< server_name > /sdn:< database_name > /tf:< target_file >

	| 参数 | 说明 |
	|---|---|
	| < server_name > | 源服务器名称 |
	| < database_name > | 源数据库名称 |
	| < target_file > | BACPAC 文件的文件名和位置 |

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01b.png)

## 使用 SQL Server Management Studio 从 BACPAC 文件导入到 Azure SQL 数据库

使用以下步骤从 BACPAC 文件导入到 Azure SQL 数据库。

> [AZURE.NOTE]下面的步骤假定你已预配 Azure SQL 逻辑实例并且手头有连接信息。

1. 确认你安装了版本 13.0.600.65 或更高版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

	> [AZURE.IMPORTANT]下载[最新](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)版本的 SQL Server Management Studio。建议你始终使用最新版本的 Management Studio。

2. 打开 Management Studio 并连接到你在对象资源管理器中的源数据库。

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/MigrateUsingBACPAC01.png)

3. 创建 BACPAC 后，连接到 Azure SQL 数据库服务器，右键单击“数据库”文件夹，然后单击“导入数据层应用程序...”

    ![导入数据层应用程序菜单项](./media/sql-database-cloud-migrate/MigrateUsingBACPAC03.png)

4.	在导入向导中，选择你刚导出的 BACPAC 文件，以在 Azure SQL 数据库中创建新的数据库。

    ![导入设置](./media/sql-database-cloud-migrate/MigrateUsingBACPAC04.png)

5.	为 Azure SQL 数据库中的数据库提供**新数据库名称**，设置 **Windows Azure SQL 数据库的版本**（服务层）、**最大数据库大小**和**服务目标**（性能级别）。

    ![数据库设置](./media/sql-database-cloud-migrate/MigrateUsingBACPAC05.png)

6.	单击“下一步”，然后单击“完成”以将该 BACPAC 文件导入 Azure SQL 数据库服务器中的新数据库。

7. 使用对象资源管理器连接到 Azure SQL 数据库服务器中的已迁移数据库。

8.	使用 Azure 门户查看数据库及其属性。

## 使用 SqlPackage 从 BACPAC 文件导入到 Azure SQL 数据库

使用下面的步骤通过 [SqlPackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用工具从 BACPAC 文件导入兼容的 SQL Server 数据库（或 Azure SQL 数据库）。

> [AZURE.NOTE]下面的步骤假定你已预配 Azure SQL 数据库服务器并且手头有连接信息。

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用工具的目录 - 此实用程序与 Visual Studio 和 SQL Server 一起提供。
2. 结合环境的以下参数运行以下 sqlpackage.exe 命令：

	'sqlpackage.exe /Action:Import /tsn:< server_name > /tdn:< database_name > /tu:< user_name > /tp:< password > /sf:< source_file >

	| 参数 | 说明 |
	|---|---|
	| < server_name > | 目标服务器名称 |
	| < database_name > | 目标数据库名称 |
	| < user_name > | 目标服务器中的用户名 |
	| < password > | 用户的密码 |
	| < source_file > | 要导入的 BACPAC 文件的文件名和位置 |

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01c.png)


## 修复数据库兼容性问题

如果你确定源 SQL Server 数据库不兼容，则有几个选项可修复[先前确定的](#determine-if-your-database-is-compatible)数据库兼容性问题。

- 使用 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com/)。可以使用此 Codeplex 工具从不兼容的源数据库生成 T-SQL 脚本，向导随后将转换该脚本使其与 SQL 数据库兼容，然后连接到 Azure SQL 数据库以执行该脚本。此工具还将分析跟踪文件以确定兼容性问题。生成的脚本可以只包含架构，也可以包含 BCP 格式的数据。其他文档（包括分步指南）可在 Codeplex 上的 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com/)中找到。  

 ![SAMW 迁移示意图](./media/sql-database-cloud-migrate/02SAMWDiagram.png)

 >[AZURE.NOTE]请注意，向导的内置转换不一定能够处理它可检测的兼容架构。无法解决的非兼容脚本将报告为错误，将在生成的脚本中注入注释。如果检测到许多错误，请使用 Visual Studio 或 SQL Server Management Studio 来单步执行并修复无法使用 SQL Server 迁移向导修复的每个错误。

- 使用 Visual Studio。你可以使用 Visual Studio 将数据库架构导入到 Visual Studio 数据库项目中以进行分析。若要分析，请将该项目的目标平台指定为 SQL 数据库 V12，然后生成该项目。如果生成成功，则数据库是兼容的。如果生成失败，你可以在 Visual Studio 的 SQL Server Data Tools（“SSDT”）中解决错误。成功生成项目后，你便可以将其发布为源数据库的副本，然后使用 SSDT 中的数据比较功能将数据从源数据库复制到这个与 Azure SQL V12 兼容的数据库。然后，使用[前面所述](#options-to-migrate-a-compatible-database-to-azure-sql-database)的选项将这个已更新的数据库部署到 Azure SQL 数据库。

 ![VSSSDT 迁移示意图](./media/sql-database-cloud-migrate/03VSSSDTDiagram.png)

 >[AZURE.NOTE]如果需要进行仅有架构的迁移，则可以将架构直接从 Visual Studio 发布到 Azure SQL 数据库。当数据库架构所需的更改量超过了 SAMW 单独可处理的数量时使用。

- SQL Server Management Studio。你可以在 Management Studio 中使用各种 Transact-SQL 命令（如 **ALTER DATABASE**）修复问题。

<!---HONumber=Mooncake_1207_2015-->