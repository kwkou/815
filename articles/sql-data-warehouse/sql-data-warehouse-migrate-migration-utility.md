<properties
   pageTitle="迁移：数据仓库迁移实用程序 | Azure"
   description="迁移到 SQL 数据仓库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="04/20/2016"
   wacn.date="06/20/2016"/>


# 数据仓库迁移实用程序（预览版）

> [AZURE.SELECTOR]
- [下载迁移实用程序][]

数据仓库迁移实用程序是专门用于将架构和数据从 SQL Server 与 Azure SQL 数据库迁移到 Azure SQL 数据仓库的工具。在迁移架构期间，该工具会自动将相应架构从源映射到目标。完成迁移架构后，用户还能选择以自动生成的脚本来移动数据。

除了迁移架构和数据以外，此工具还能让用户选择生成兼容性报告，汇总目标与源实例之间可能会妨碍迁移操作的不兼容问题。

## 入门
安装的先决条件之一是，你需要使用 BCP 命令行实用程序来运行迁移脚本和 Office，这样才能查看兼容性报告。启动下载的可执行文件后，系统将提示你在安装该工具之前接受标准 EULA。

此外，若要运行此迁移实用程序，你需要以下某种针对你要迁移的数据库的权限：CREATE DATABASE、ALTER ANY DATABASE 或 VIEW ANY DEFINITION。

### 启动工具并连接
单击在安装后显示的桌面图标即可启动该工具。打开工具时，系统会显示初始连接页面，提示你选择迁移工具的源和目标。目前我们支持将 SQL Server 和 Azure SQL 数据库用作源，将 SQL 数据仓库用作目标。选择源和目标后，系统会要求你填写服务器名称并执行身份验证，然后单击“连接”以连接到源服务器。

完成身份验证后，该工具将显示连接到的服务器中的数据库列表。你可以通过选择想要迁移的数据库，然后单击“迁移选定项目”开始迁移。

## 迁移报告
在工具中选择“检查数据库兼容性”会生成一份报告，其中汇总了请求迁移的数据库中所有的对象不兼容情况。未列在 SQL 数据仓库中的部分 SQL Server 功能详细列表可以在[迁移文档][]中找到。生成报告后，可以将它保存并在 Excel 中打开。

请注意，生成迁移架构时，大多数识别为“对象”的问题将经过调整，以便立即迁移该数据。请检查这些更改，以确保在应用架构之前不需要做出其他调整。

## 迁移架构

建立连接后，选择“迁移架构”生成所选表的架构迁移脚本。此脚本可以移植表的结构、将不兼容的数据类型映射为兼容性较高的格式，并创建安全凭据和架构（如果用户已在迁移设置中指定）。可以针对目标 SQL 数据仓库实例运行此代码，或将它保存到文件、复制到剪贴板，甚至先以内联方式进行编辑，然后再执行其他操作。

如前所述，迁移架构时，请仔细检查工具所做的迁移更改，以确保完全了解这些更改。

## 迁移数据

单击“迁移数据”选项即可生成 BCP 脚本，这些脚本会先将数据移到服务器上的平面文件，然后直接移入 SQL 数据仓库。建议使用此过程来迁移少量数据，因为系统本身不会进行重试，如果网络连接中断，则可能会发生故障。若要运行此操作，需要安装 BCP 命令行实用程序，并事先创建数据的架构。

填写上述参数后，只需单击“运行迁移”，系统就会在指定的位置生成两个包。运行导出文件，以将迁移源中的数据导出到平面文件，然后运行导入文件，以将数据导入 SQL 数据仓库。

## 后续步骤
现在你已迁移一些数据，请继续了解如何[开发][]。

<!--Image references-->

<!--Article references-->
[迁移文档]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
<!--Other Web references--> 
[下载迁移实用程序]: https://migrhoststorage.blob.core.windows.net/sqldwsample/DataWarehouseMigrationUtility.zip

<!---HONumber=Mooncake_0613_2016-->