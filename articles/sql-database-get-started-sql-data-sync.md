<properties linkid="manage-services-sql-databases-datasync" urlDisplayName="How to sync data" pageTitle="Getting started with SQL Databases Data Sync" metaKeywords="" description="" metaCanonical="" services="sql-database" documentationCenter="" title="Getting Started with Azure SQL Data Sync" authors="" solutions="" manager="" editor="" />

# Azure SQL 数据同步入门

在本教程中，你将了解使用 Azure（预览版）门户的 Azure SQL 数据同步的基础知识。

本教程假定你之前未使用过 SQL Server 和 Azure SQL Database。在本教程中，你将创建一个完全配置且按既定计划同步的混合（SQL Server 和 SQL Database 实例）同步组。

## 目录

-   [步骤 1：连接到 Azure SQL Database][步骤 1：连接到 Azure SQL Database]
-   [步骤 2：添加客户端代理 ()][步骤 2：添加客户端代理 ()]
-   [步骤 3：使用客户端代理注册 SQL Server 数据库][步骤 3：使用客户端代理注册 SQL Server 数据库]
-   [步骤 4：创建同步组][步骤 4：创建同步组]
-   [步骤 5：定义要同步的数据][步骤 5：定义要同步的数据]
-   [步骤 6：配置同步组][步骤 6：配置同步组]

## <span id="Connect"></span></a>步骤 1：连接到 Azure SQL Database

1.  登录到[管理门户][管理门户]。

2.  在左窗格中，单击“SQL Database”。

3.  单击页面底部的“同步”。单击“同步”后，将出现一个显示可供添加内容的列表 –“新建同步组”和“新建同步代理”。

4.  若要启动“新建 SQL 数据同步代理”向导，请单击“新建同步代理”。

5.  如果你之前没有添加代理，请“单击此处进行下载”。

    ![图像 1][图像 1]

## <span id="AddAgent"></span></a>步骤 2：添加客户端代理

仅当你要在同步组中包含本地 SQL Server 数据库时，才需要执行此步骤。你可以跳到步骤 4：创建同步组（如果你的同步组只具有 SQL Database 实例）。

### <span id="InstallRequiredSoftware"></span></a>步骤 2a：安装必要的软件

确保你安装客户端代理的计算机上安装有下列软件。

-   **.NET Framework 4.0**

你可以从[此处][此处]安装 .NET Framework 4.0。

-   **Microsoft SQL Server 2008 R2 SP1 System CLR Types (x86)**

你可以从[此处][1]安装 Microsoft SQL Server 2008 R2 SP1 System CLR Types (x86)

-   **Microsoft SQL Server 2008 R2 SP1 共享管理对象 (x86)**

你可以从 [此处][1]安装 Microsoft SQL Server 2008 R2 SP1 Shared Management Objects (x86)

### <span id="InstallClient"></span></a>步骤 2b：安装新的客户端代理

按照[安装客户端代理（SQL 数据同步）][安装客户端代理（SQL 数据同步）]中的说明来安装代理。

### <span id="RegisterSSDb"></span></a>步骤 2c：完成新建 SQL 数据同步代理向导

1.  返回到新建 SQL 数据同步代理向导。
2.  为代理提供一个有意义的名称。
3.  从下拉列表中，选择托管此代理的“地区”（数据中心）。
4.  从下拉列表中，选择托管此代理的“订阅”。
5.  单击右箭头。

## <span id="RegisterSSDB"></span></a>步骤 3：使用客户端代理注册 SQL Server 数据库

安装客户端代理后，向该代理注册你要包含在同步组中的每个本地 SQL Server 数据库。
若要向该代理注册数据库，请按照[向客户端代理注册 SQL Server 数据库][向客户端代理注册 SQL Server 数据库]中的说明操作。

## <span id="CreateSG"></span></a>步骤 4：创建同步组

### <span id="StartNewSGWizard"></span></a>步骤 4a：启动新建同步组向导

1.  返回[管理门户][管理门户]。
2.  单击“SQL Database”。
3.  单击页面底部的“添加同步”，然后从下拉列表中选择“新建同步组”。

    ![图像 2][图像 2]

### <span></span></a>步骤 4b：输入基本设置

1.  为该同步组输入一个有意义的名称。
2.  从下拉列表中，选择托管此同步组的“地区”（数据中心）。
3.  单击右箭头。

    ![图像 3][图像 3]

### <span id="DefineHubDB"></span></a>步骤 4c：定义同步中心

1.  从下拉列表中，选择 SQL Database 实例以用作同步组中心。
2.  输入此 SQL Database 实例的凭据 –“中心用户名”和“中心密码”。
3.  等待 SQL 数据同步确认该用户名和密码。在凭据被确认后，密码右侧将出现一个绿色复选标记。
4.  从下拉列表中，选择“冲突解决”策略。

**中心 Wins** – 写入中心数据库的任何更改将写入引用数据库，以覆盖同一引用数据库记录中的更改。从功能上看，这意味着写入中心的首次更改会传播到其他数据库。

**客户端 Wins** – 写入中心的更改将被引用数据库中的更改覆盖。从功能上看，这意味着写入中心的最后一次更改会被保留并传播到其他数据库。

1.  单击右箭头。

    ![图像 4][图像 4]

### <span id="AddRefDB"></span></a>步骤 4d：添加引用数据库

对你要额外添加到同步组中的每个数据库重复此步骤。

1.  从下拉列表中，选择要添加的数据库。

    下拉列表中的数据库包含已向代理进行注册的 SQL Server 数据库，以及 SQL Database 实例。

2.  输入此数据库的凭据 –“用户名”和“密码”。
3.  从下拉列表中，选择此数据库的“同步方向”。

    **双向** – 引用数据库中的更改将写入中心数据库，对中心数据库的更改将写入引用数据库。

    **从中心同步** - 数据库从中心接收更新，而不将更改发送到中心。

    **同步到中心** - 数据库将更新发送到中心。中心的更改不会写入此数据库。

4.  若要完成创建同步组，请单击向导右下方的复选标记。等待 SQL 数据同步确认凭据。绿色复选标记表示凭据已被确认。

5.  再次单击复选标记。你将会返回到 SQL Database 下的“同步”页面。现在，此同步组将与其他同步组和代理一同列出。

    ![图像 5][图像 5]

## <span id="SyncRules"></span></a>步骤 5：定义要同步的数据

利用 Azure SQL 数据同步，你可以选择要同步的表和列。如果你还希望对列进行筛选以便仅同步具有特定值（如 Age\>=65）的行，请使用 Azure 的 SQL 数据同步门户以及“选择要同步的表、列和行”文档，来定义要同步的数据。

1.  返回[管理门户][管理门户]。
2.  单击“SQL Database”。
3.  单击“同步”选项卡。
4.  单击此同步组的名称。
5.  单击“同步规则”选项卡。
6.  选择你要提供同步组架构的数据库。
7.  单击右箭头。
8.  单击“刷新架构”。
9.  对于数据库中的每个表，选择要包含在同步中的列。

    -   不能选择包含不受支持数据类型的列。
    -   如果表中没有列被选中，则该表将不会包含在同步组中。
    -   若要选择/取消选择全部表，请单击屏幕底部的“选择”。

10. 单击“保存”，然后等待同步组完成配置。
11. 若要返回到数据同步登陆页，请单击屏幕左上角（同步组名称的上方）的后退箭头。

    ![图像 6][图像 6]

## <span id="Configure"></span></a>步骤 6：配置同步组

你可以始终通过单击数据同步登录页底部的“同步”来对同步组执行同步操作。
如果你希望某个同步组按照计划同步，请配置该同步组。

1.  返回[管理门户][管理门户]。
2.  单击“SQL Database”。
3.  单击“同步”选项卡。
4.  单击此同步组的名称。
5.  单击“配置”选项卡。
6.  **自动同步**

    -   若要将同步组配置为按照设定频率进行同步，请单击“打开”。还可以通过单击“同步”来按需进行同步。
    -   单击“关闭”可将同步组配置为仅在单击“同步”时才同步。

7.  **同步频率**

    -   如果“自动同步”处于开启状态，可设置同步频率。频率必须介于 5 分钟到 1 个月之间。

8.  单击“保存”。

![图像 7][图像 7]

祝贺你！你已创建了一个包含 SQL Database 实例和 SQL Server 数据库的同步组。

## <span id="NextSteps"></span></a> 后续步骤

有关 SQL Database 和 SQL 数据同步的其他信息，请参阅：

-   [注册 SQL Database 的高级版产品/服务] (../sign-up-for-sql-database-premium/)
-   [SQL 数据同步（WA 门户）][SQL 数据同步（WA 门户）]
-   [Azure SQL Database 入门][Azure SQL Database 入门]
-   [SQL Server 数据库生命周期][SQL Server 数据库生命周期]

  [步骤 1：连接到 Azure SQL Database]: #Connect
  [步骤 2：添加客户端代理 ()]: #AddAgent
  [步骤 3：使用客户端代理注册 SQL Server 数据库]: #RegisterSSDB
  [步骤 4：创建同步组]: #CreateSG
  [步骤 5：定义要同步的数据]: #SyncRules
  [步骤 6：配置同步组]: #Configure
  [管理门户]: http://manage.windowsazure.cn
  [图像 1]: ./media/sql-database-get-started-data-sync/SQLDatabaseScreen-Figure1.PNG
  [此处]: http://www.microsoft.com/zh-cn/download/details.aspx?id=17851
  [1]: http://www.microsoft.com/zh-cn/download/details.aspx?id=26728
  [安装客户端代理（SQL 数据同步）]: http://msdn.microsoft.com/zh-cn/library/jj823137.aspx
  [向客户端代理注册 SQL Server 数据库]: http://msdn.microsoft.com/zh-cn/library/jj823138.aspx
  [图像 2]: ./media/sql-database-get-started-data-sync/NewSyncGroup-Figure2.png
  [图像 3]: ./media/sql-database-get-started-data-sync/NewSyncGroupName-Figure3.PNG
  [图像 4]: ./media/sql-database-get-started-data-sync/NewSyncGroupHub-Figure4.PNG
  [图像 5]: ./media/sql-database-get-started-data-sync/NewSyncGroupReference-Figure5.PNG
  [图像 6]: ./media/sql-database-get-started-data-sync/NewSyncGroupSyncRules-Figure6.PNG
  [图像 7]: ./media/sql-database-get-started-data-sync/NewSyncGroupConfigure-Figure7.PNG
  [SQL 数据同步（WA 门户）]: http://msdn.microsoft.com/zh-cn/library/azure/jj856263.aspx
  [Azure SQL Database 入门]: ../getting-started-w-sql-databases/
  [SQL Server 数据库生命周期]: http://msdn.microsoft.com/zh-cn/library/hh213416.aspx
