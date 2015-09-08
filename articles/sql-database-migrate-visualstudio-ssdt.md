<properties 
   pageTitle="使用 Visual Studio 和 SSDT 进行迁移" 
   description="Windows Azure SQL 数据库, 数据库迁移, 导入数据库, 导出数据库, 迁移向导" 
   services="sql-database" 
   documentationCenter="" 
   authors="kaivalyah2015" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags 
   ms.service="sql-database"
   ms.date="04/14/2015" 
   wacn.date="08/29/2015" />

#就地更新数据库，然后部署到 Azure SQL 数据库

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/01VSSSDTDiagram.png)

将数据库迁移到 Azure SQL 数据库 V12 时需要架构更改，而由于数据库使用 Azure SQL 数据库中不支持的 SQL Server 功能，导致使用 SQL Azure 迁移向导 (SAMW) 无法处理这种更改时，请使用此选项。使用此选项，将首先使用 Visual Studio 从源数据库创建数据库项目。然后将该项目目标平台设为 Azure SQL 数据库 V12，并生成项目以确定所有兼容性问题。SAMW 可解决许多兼容性问题，但不能解决全部问题，因此首先使用 SAMW 处理项目中所有脚本。使用 SAMW 是可选操作，但是强烈建议使用。使用 SAMW 处理脚本文件后生成项目可确认剩余问题，这些问题必须使用 Visual Studio 中的 Transact-SQL 编辑工具手动解决。成功生成了项目后，将架构发布回源数据库的副本（推荐）以就地更新架构和数据。然后使用选项 #1 中描述的技巧将更新的数据库部署到 Azure，可直接部署或通过导出和导入 BACPAC 文件进行部署。
 
由于此选项涉及在部署到 Azure 之前就地更新数据库的架构，所以强烈建议在数据库副本上执行此操作。可使用 Visual Studio 架构比较工具检查将在发布项目之前应用到数据库的整组更改。

使用 SQL Azure 迁移向导 (SAMW) 是可选操作，但强烈建议使用。SAMW 将检测函数主体、存储过程和触发器内的兼容性问题，如果不进行此操作，则在部署前将无法检测到这些问题。如果需要仅有架构的部署，可将更新的架构从 Visual Studio 直接发布到 Azure SQL 数据库。

## 迁移步骤

1.	在 Visual Studio 中打开“SQL Server 对象资源管理器”。使用“添加 SQL Server ”连接到包含被迁移数据库的 SQL Server 实例。在资源管理器中找到数据库，右键单击它并选择“创建新项目...” 

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/02MigrateSSDT.png)

2.	将导入设置配置为“仅导入应用程序范围对象”。取消选择导入引用的登录名、权限和数据库设置的选项。

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/03MigrateSSDT.png)

3.	单击“启动”以导入数据库并创建项目，项目将包含数据库中每个对象的 T-SQL 脚本文件。这些脚本文件嵌入到项目中的文件夹内。

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/04MigrateSSDT.png)

4.	在 Visual Studio 解决方案资源管理器中，右键单击数据库项目并选择“属性”。此操作将打开“项目设置”页，在该页上应将“目标平台”配置为“Windows Azure SQL 数据库 V12”。

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/05MigrateSSDT.png)

5.	可选：右键单击该项目并选择“生成”以生成项目（此时并非绝对必需，但现在执行此操作将为项目中的兼容性问题数和后续步骤的有效性提供一个基准。）

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/06MigrateSSDT.png)

6.	可选：右键单击该项目并选择“快照项目”。通过在转换期间开始时和后续阶段拍摄快照，可以使用架构比较工具比较每个阶段的架构。

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/07MigrateSSDT.png) - 对项目进行快照将创建一个加盖时间戳的 .dacpac 文件，它包含该项目的完整架构。你可以修改文件名称以指示该快照拍摄于进程中的哪个阶段。![替换文字](./media/sql-database-migrate-visualstudio-ssdt/08MigrateSSDT.png)

7.	使用 SQL Azure 迁移向导 (SAMW) 处理导入的脚本文件。使用文件夹选项并选择根项目文件夹。![替换文字](./media/sql-database-migrate-visualstudio-ssdt/09MigrateSSDT.png)

8.	该向导将处理此文件夹和所有子文件夹中的每个脚本文件。将在下一页上显示结果的摘要。![替换文字](./media/sql-database-migrate-visualstudio-ssdt/10MigrateSSDT.png)
9.	单击“下一步”以查看已更改的文件的摘要列表。 

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/11MigrateSSDT.png)

> [AZURE.NOTE]临时副本由处理前的原始文件和处理后受影响的文件组成，位于页面顶端指示的位置。

10.	在确认对话框中点击“覆盖”和“确定”，将以更改后文件覆盖原始文件。只有实际更改过的文件会被覆盖。
11.	可选。使用架构比较对项目和较早的快照或原始数据库进行比较，以便了解通过该向导进行了哪些更改。此时可以再次拍摄快照。 

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/12MigrateSSDT.png)

12.	再次生成项目（参阅之前的步骤）以确定剩余错误。

13.	系统地处理错误以解决每个问题。使用数据库评估每个更改对应用程序的影响。

14.	当数据库无错误后，右键单击项目并选择“发布”以生成数据库并将其发布到源数据库副本（强烈建议至少在开始时使用副本）。此步骤的目的是更新源数据库架构和对数据库中的数据进行后续更改。
- 在发布前，根据源 SQL Server 版本，可能需要重置项目的目标平台才能进行部署。如果迁移的是较早的 SQL Server 数据库，切勿向项目中引入任何不受源 SQL Server 支持的功能，除非先将数据库迁移到较新版本的 SQL Server。 
- 必须配置发布步骤才能使用适当的“发布”选项。例如，如果在项目中删除或注释去掉不支持的对象，则必须从数据库删除它们，从而必须允许“发布”删除目标数据库中的数据。 
- 如果你预计重复发布 

15.	使用选项 1 中描述的技巧将复制的数据库部署到 Azure SQL 数据库。

## 验证迁移

完成迁移后，最好将已迁移的数据库的架构与源数据库的架构进行对比，了解那些按照你的意愿做出的任何更改是符合预期的，这些更改不会在将应用程序迁移到新数据库时引发任何问题。可使用 Visual Studio 中的 SQL Server 工具包含的架构比较工具进行比较。可将 Azure SQL 数据库中的数据库与原始 SQL Server 数据库，或与首次将数据库导入到项目中时拍摄的快照进行比较。

1.	在包含已迁移数据库的 Azure SQL 数据库中连接到服务器并找到该数据库。 

2.	右键单击数据库并选择“架构比较...”，此操作将打开一个新的架构比较，其中所选 Azure 数据库作为比较的源，位于左侧。使用右侧的“选择目标”下拉菜单选择目标数据库或快照文件以进行比较。

3.	选好源与目标后，点击“比较”以触发比较。从 Azure SQL 数据库中的复杂数据库加载架构可能需要较长时间。使用较高定价层时，加载 Azure SQL 数据库上的架构和对其执行其他元数据任务将花费较少时间。

4.	比较完成后查看差异。除非有任何重大关系，通常不应对任一架构应用更改。

在以下架构比较中，对左侧 Azure SQL 数据库 V12 中的 Adventure Works 2014 数据库（通过 SQL Azure 迁移向导对其进行了转换和迁移）和右侧 SQL Server 中的源数据库进行比较。

![替换文字](./media/sql-database-migrate-visualstudio-ssdt/13MigrateSSDT.png)

<!---HONumber=67-->