<properties
   pageTitle="将数据库迁移到 Azure SQL 数据库"
   description="Microsoft Azure SQL 数据库, 数据库部署, 数据库迁移, 导入数据库, 导出数据库, 迁移向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="09/02/2015"
   wacn.date="10/17/2015"/>

# 将数据库迁移到 Azure SQL 数据库

Azure SQL 数据库 V12 几乎与 SQL Server 2014 及更高版本的引擎完全兼容。因此，将大多数数据库从 SQL Server 2005 或更高版本的本地实例迁移到 Azure SQL 数据库的任务更简单。许多数据库的迁移是一个直截了当的架构和数据转移操作，它只需对架构做出少量的更改（如果有），并且只需对应用程序做出轻微的改造，甚至不需要任何改造。如果需要对数据库做出更改，你可以更好地限制这种更改的范围。

按照设计，SQL Server 的服务器设限功能不受 Azure SQL 数据库 V12 支持。依赖于这些功能的数据库和应用程序需要进行一些重新设计才能进行迁移。尽管 Azure SQL 数据库 V12 改进了与本地 SQL Server 数据库的兼容性，但迁移仍需要仔细地规划和执行，对于大型且复杂的数据库尤其如此。

## 确定兼容性
若要确定本地 SQL Server 数据库是否与 Azure SQL 数据库 V12 兼容，可以首先使用下面选项 #1 中讨论的两种方法之一迁移，并查看架构验证例程是否检测到不兼容，也可以按照下面选项 #2 中的讨论内容使用 Visual Studio 中的 SQL Server Data Tools 来验证兼容性。如果本地 SQL Server 数据库存在兼容性问题，你可以使用 Visual Studio 或 SQL Server Management Studio 中的 SQL Server Data Tools 来解决兼容性问题。

## 迁移方法
将兼容的本地 SQL Server 数据库迁移到 Azure SQL 数据库 V12 的方法有很多。

- 对于小型到中型数据库，迁移兼容的 SQL Server 2005 或更高版本数据库只需运行 SQL Server Management Studio 中的“将数据库部署到 Microsoft Azure 数据库向导”即可，但前提是你没有连接问题（没有连接、低带宽或超时问题）。
- 对于中型到大型数据库或存在连接问题时，你可以使用 SQL Server Management Studio 将数据和架构导出到 BACPAC 文件 （存储在本地或 Azure blob 中），然后将 BACPAC 文件导入到 Azure SQL 实例中。如果将 BACPAC 存储在 Azure blob 中，还可以从 Azure 门户中导入 BACPAC 文件。有关 BACPAC 文件的详细信息，请参阅[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)。
- 对于大型数据库，通过单独迁移架构和数据可以实现最佳性能。你可以使用 SQL Server Management Studio 或 Visual Studio 将架构提取到数据库项目，然后部署架构创建 Azure SQL 数据库。随后，你可以使用 BCP 提取数据，然后使用 BCP 通过并行流将数据导入 Azure SQL 数据库。不论你选择哪种方法，迁移大型、复杂的数据库需要花费数小时。

### 选项 #1
***使用 SQL Server Management Studio (SSMS) 迁移兼容的数据库***

SQL Server Management Studio 提供两种方法，用于将兼容的本地 SQL Server 数据库迁移到 Azure SQL 数据库。你可以使用“将数据库部署到 Windows Azure SQL 数据库”向导，也可以将数据库导出到 BACPAC 文件，然后可以导入该文件以创建一个新的 Azure SQL 数据库。向导验证 Azure SQL 数据库 V12 兼容性，将架构和数据提取到 BACPAC 文件中，然后将其导入到指定的 Azure SQL 数据库实例。若要使用此选项，请参阅[使用 SSMS](/documentation/articles/sql-database-migrate-ssms)。

### 方法 #2
***使用 Visual Studio 脱机更新数据库架构，然后使用 SQL Server Management Studio 进行部署***

如果本地 SQL Server 数据库不兼容或者要确定其是否兼容，你可以将数据库架构导入 Visual Studio 数据库项目进行分析。若要分析，请将该项目的目标平台指定为 SQL 数据库 V12，然后生成该项目。如果生成成功，则数据库是兼容的。如果生成失败，你可以在 Visual Studio 的 SQL Server Data Tools（“SSDT”）中解决错误。成功生成项目后，你便可以将其发布为源数据库的副本，然后使用 SSDT 中的数据比较功能将数据从源数据库复制到这个与 Azure SQL V12 兼容的数据库。然后，使用选项 #1 将这个更新的数据库部署到 Azure SQL 数据库。如果需要进行仅有架构的迁移，则可以将架构直接从 Visual Studio 发布到 Azure SQL 数据库。当数据库架构所需的更改量超过了 SAMW 单独可处理的数量时使用。若要使用此选项，请参阅[使用 Visual Studio](/documentation/articles/sql-database-migrate-visualstudio-ssdt)。

## 确定要用的选项
- 如果你预料到无需更改就可以迁移某个数据库，则应使用选项 #1，因为该选项可以快速方便地完成。如果你不太确定，请根据选项 #1 中所述，先从数据库中导出仅有架构的 BACPAC。如果导出成功且未出错，则你可以使用选项 #1 迁移数据库及其数据。  
- 如果在使用选项 #1 导出期间遇到错误，请根据选项 #2 中所述，使用 SQL Azure 迁移向导以仅有架构的模式处理数据库。如果迁移向导未报告任何错误，请使用选项 #2。 
- 如果在导出选项 #1 过程中遇到错误，使用选项 #2 并结合使用迁移向导和手动应用的架构更改在 Visual Studio 中脱机更正数据库架构。然后，就地更新源数据库的副本，并使用选项 #1 将它迁移到 Azure。

## 迁移工具
使用的工具包括 SQL Server Management Studio (SSMS)、Visual Studio 中的 SQL Server 工具（VS、SSDT）以及 Azure 门户。

> [AZURE.IMPORTANT]请务必安装最新版本的客户端工具，因为早期版本与 Azure SQL 数据库 V12 不兼容。

### SQL Server Management Studio (SSMS)
可以使用 SSMS 将兼容的数据库直接部署到 Azure SQL 数据库，或者将数据库的逻辑备份导出为 BACPAC，然后，仍然使用 SSMS 导入该 BACPAC 来创建新的 Azure SQL 数据库。

[下载最新版本的 SSMS](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)

### Visual Studio 中的 SQL Server 工具（VS、SSDT）
Visual Studio 中的 SQL Server 工具可用于创建和管理数据库项目，该项目包括架构中每个对象的 Transact-SQL 文件集。可以从数据库或者脚本文件导入该项目。创建后，可将该项目部署到 Azure SQL 数据库 v12；生成项目，然后验证架构兼容性。单击某个错误会打开相应的 Transact-SQL 文件，然后便可以编辑该文件并更正错误。修复所有错误后，可将项目直接发布到 SQL 数据库以创建空数据库，或者发布回到原始 SQL Server 数据库的（副本）以更新其架构，然后，便可以使用上述 SSMS 来部署该数据库及其数据。

请使用包含 Visual Studio 2013 Update 4 或更高版本的[最新 SQL Server Data Tools for Visual Studio](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)。

## 比较
| 选项 #1 | 方法 #2 |
| ------------ | ------------ | ------------ |
| 将兼容的数据库部署到 Azure SQL 数据库 | 就地更新数据库，然后部署到 Azure SQL 数据库 |
|![SSMS](./media/sql-database-cloud-migrate/01SSMSDiagram.png)| ![脱机编辑](./media/sql-database-cloud-migrate/03VSSSDTDiagram.png) |
| 使用 SSMS | 使用 VS 和 SSMS |
|简单的过程要求架构兼容。架构按原样迁移。 | 将架构导入 Visual Studio 中的数据库项目。使用 Visual Studio 的 SSDT 和所用的最终架构进行其他更新，以就地更新数据库。 |
| 始终部署或导出整个数据库。 | 全面控制迁移中包含的对象。 |
| 未规定出错时必须更改输出，源架构必须兼容。 | 可以使用 Visual Studio 的 SSDT 的完整功能。脱机更改架构。 | 在 Azure 中进行应用程序验证。应该很少，因为无需更改即可迁移架构。 | 将数据库部署到 Azure 之前，可以在 SQL Server 中执行应用程序验证。 |
| 轻松配置的单步或双步过程。 | 更复杂的多步过程（如果只部署架构，则更方便一些）。 |

<!---HONumber=74-->