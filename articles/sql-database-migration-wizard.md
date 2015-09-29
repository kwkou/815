<properties 
   pageTitle="使用 SQL Azure 迁移向导 | Windows Azure" 
   description="Windows Azure SQL 数据库, 数据库迁移, 导入数据库, 导出数据库, 迁移向导" 
   services="sql-database" 
   documentationCenter="" 
   authors="pehteh" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="07/01/2015"
   wacn.date="09/15/2015"/>


# 如何：使用 SQL Azure 迁移向导


![替换文字](./media/sql-database-migration-wizard/01SAMWDiagram.png)


此选项使用 SQL Azure 迁移向导从源数据库生成 T-SQL 脚本，向导随后将转换该脚本以使其与 SQL 数据库兼容，然后连接到 Azure SQL 数据库，以针对空的 Azure SQL 数据库执行该脚本。生成的脚本可以只包含架构，也可以包含 BCP 格式的数据。该向导还允许你在脚本中包括或排除数据库的特定对象。


请注意，向导的内置转换不一定能够处理它可检测的兼容架构。无法解决的非兼容脚本将报告为错误，将在生成的脚本中注入注释。如果发生这种情况，你必须保存脚本并手动对其进行编辑，然后才能将它提交到 Azure SQL 数据库。如果需要更改，你可以保存脚本，然后使用 SSMS 或 VS 中的 SQL Server 对它进行编辑，兼容后，针对 Azure SQL 数据库带外执行该脚本。


> **注意：**作为此选项的扩展，如果检测到多个错误并且无法直接更正这些错误，你可以将生成的脚本文件导入 Visual Studio 中的数据库项目。如果你将项目的目标设置为 SQL 数据库 V12，则可以生成项目，然后使用 VS 中的 SQL Server 工具逐个更正错误。排除架构中的错误后，你可以将架构发布到源数据库的副本，然后根据选项 #1 中所述，使用 SSMS 将数据库部署或导出/导入到 Azure SQL 数据库。


## 下载 SQLAzureMW.exe


可以从 CodePlex 下载 SQL Azure 迁移向导：


[下载 SQLAzureMW.exe](http://sqlazuremw.codeplex.com/)


## 迁移步骤


&nbsp;1.在 SQL 数据库中新的服务器上，或者在根据前面所述升级的现有服务器上，设置一个新数据库。在执行最后一个步骤时，你将要针对此数据库执行此选项中创建的迁移脚本。


&nbsp;2.打开迁移向导，选择“分析/迁移数据库”选项，将“目标服务器”设置为 Azure SQL 数据库 V12，然后单击“下一步”。

![替换文字](./media/sql-database-migration-wizard/02MigrationWizard.png)


&nbsp;3.在下一页中，单击“连接到服务器”，并提供源服务器的连接信息。在连接对话框中指定源数据库或连接到服务器，然后从数据库列表中选择源数据库。


![替换文字](./media/sql-database-migration-wizard/03MigrationWizard.png)


&nbsp;4.在下一页中，选择“对象”，你可以选择是要为所有数据库对象编写脚本（默认设置），还是要选择编写脚本的特定数据库对象。你可能会发现，一开始最好是为所有对象编写脚本，如果后面向导报告了脚本错误或转换错误，你可以根据需要返回此步骤并排除对象。该向导的工作原理是先为数据库中的对象编写脚本（使用 SMO），然后对生成的脚本进行后处理，以应用一系列基于正则表达式的验证和转换规则。


![替换文字](./media/sql-database-migration-wizard/04MigrationWizard.png)


&nbsp;5.选择“高级”，然后检查向导使用的配置选项。最初，尤其是对于大型数据库，你应该将“脚本表/数据”设置为“仅限表架构”，以确保将“目标服务器”设置为“Microsoft Azure SQL 数据库 V12”，将“兼容性检查”设置为“重写: 执行所有兼容性检查”。


![替换文字](./media/sql-database-migration-wizard/05MigrationWizard.png)


&nbsp;6.单击“下一步”复查选项，然后再次单击“下一步”生成并转换脚本。可以在“SQL 脚本”选项卡上检查脚本。


&nbsp;7.如果架构与 SQL 数据库不兼容并且无法由向导转换，脚本生成将报告错误。在以下屏幕截图中，转换过程发现一个存储过程存在问题。在这种情况下，编写了一个过程来使用全文搜索，不过 V12 尚不支持此功能（但在将来的版本中会受支持）。


![替换文字](./media/sql-database-migration-wizard/06MigrationWizard.png)


- 根据评估的错误，你可以选择返回向导并排除导致问题的对象，然后重新生成脚本。在考虑错误的解决计划时，请考虑该计划对数据库中其他对象以及使用该数据库的应用程序的影响。
- 如果脚本中的错误需要更正，你可以从“SQL 脚本”选项卡保存仅有架构的脚本文件，在偏好的编辑器中打开并编辑该脚本以更正错误，然后，在 SAMW 外部针对步骤 1 中创建的新数据库执行该脚本。如果脚本很大或者包含大量的错误，则你应使用选项 3。请注意，尽管你可以将生成的脚本文件导入 Visual Studio，但是，从 SQL 文件导入所需的时间，可能比根据选项 3 中所述从数据库导入花费的时间要长得多。 


&nbsp;8.如果没有错误，或者已经消除了生成时错误的根源，则你可以单击“下一步”，然后在下一页中单击“连接到服务器”，以连接到你在步骤 1 中创建的 Azure SQL 数据库服务器。

![替换文字](./media/sql-database-migration-wizard/07MigrationWizard.png)


&nbsp;9.下一步是选择要对其执行脚本的数据库。将列出服务器上的所有现有数据库。选择步骤 1 中创建的数据库。该数据库应该是空的，并使用适用于此操作的相应定价层。如果需要，此时你可以使用向导创建数据库。为此，请单击“创建数据库”以配置并创建该数据库。有关在迁移过程中选择要使用的性能级别的指导，请参阅“入门”部分。在创建数据库后，请选择并继续。


&nbsp;10.在选择了一个空数据库的情况下，单击“下一步”并确认你要执行该脚本，以完成迁移。

<!---HONumber=69-->