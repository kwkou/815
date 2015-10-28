<properties
	pageTitle="升级到 SQL 数据库 V12"
	description="介绍如何从先前版本的 Azure SQL 数据库 升级到 Azure SQL 数据库 V12。"
	services="sql-database"
	documentationCenter=""
	authors="sonalmm"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="05/15/2015"
	wacn.date="06/30/2015"/>


# 就地升级到 SQL 数据库 V12


[注册](https://manage.windowsazure.cn) SQL 数据库 V12，以利用 Windows Azure 上的下一代 SQL 数据库。你首先需要订阅 Windows Azure。你可以注册 [Azure 试用版](/pricing/1rmb-trial)并查看[价格](/home/features/sql-database/#price)信息。


## 升级到 SQL 数据库 V12 的步骤


| 升级步骤 | 屏幕截图 |
| :--- | :--- |
| 1.登录到 [http://manage.windowsazure.cn/](http://manage.windowsazure.cn/)。 | ![新的 Azure 门户][1] |
| 2.单击“浏览”。 | ![浏览服务][2] |
| 3.单击“SQL Server”。将显示 SQL Server 名称的列表。 | ![选择 SQL Server 服务][3] |
| 4.选择要将哪个服务器复制到已启用 SQL 数据库 Update 的新服务器。 | ![显示 SQL Server 的列表][4] |
| 5.单击“最新的 SQL 数据库 Update V12”。 | ![预览版最新功能][5] |
| 6.单击“升级此服务器”。 | ![将 SQL Server 升级到预览版][6] |


> [AZURE.NOTE]选择升级选项后，服务器及其内部的数据库将启用 SQL 数据库 V12 功能，而你无法撤消该操作。若要将服务器升级到 SQL 数据库 V12，你需要购买基本、标准或高级服务层。有关服务层的详细信息，请参阅[将 SQL 数据库 Web/企业数据库升级到新服务层](/documentation/articles/sql-database-upgrade-new-service-tiers)。


> [AZURE.IMPORTANT]SQL 数据库 V12（预览版）不支持地域复制。有关详细信息，请参阅[规划和准备升级到 Azure SQL 数据库 V12 预览版](/documentation/articles/sql-database-v12-plan-prepare-upgrade)。


单击“升级此服务器”选项后，打开的边栏选项卡会显示有关验证过程的消息。


- 验证过程将检查数据库的服务层，并检查是否已启用地域复制。验证完成后，该边栏选项卡将显示结果。
- 验证过程完成后，你将看到数据库名称的列表，这些数据库需要你根据升级到 SQL 数据库 V12 的要求执行相应的操作。
 - **只有针对其中的每个数据库执行了相应的操作，才能升级到 SQL 数据库 V12**。
- 单击每个数据库名称时，新的边栏选项卡将会根据你的当前使用情况提供服务定价层建议。你也可以浏览各个定价层，然后选择最适合你环境的定价层。针对地域复制进行了设置的所有数据库都需要重新配置为停止复制。
- 请注意，如果未找到足够的数据，将不会显示有关定价层的建议。


| 操作 | 屏幕截图 |
| :--- | :--- |
| 7.完成准备升级服务器的相关操作后，请键入要升级的服务器名称，然后单击“确定”。 | ![确认要升级的服务器名称][7] |
| 8.此时将启动升级过程。升级最多可能需要花费 24 小时。在此期间，此服务器上的所有数据库都保持联机，但服务器和数据库管理操作会受到限制。完成升级过程后，服务器边栏选项卡上会显示“已启用”状态。 | ![确认已启用预览版功能][8] |


## PowerShell cmdlet


可以使用 Powershell cmdlet 来启动、停止或监视从 Azure SQL 数据库 V11 或其他任何低于 V12 的版本到 V12 的升级。


有关这些 Powershell cmdlet 的参考文档，请参阅：


- [Get-AzureSqlServerUpgrade](http://msdn.microsoft.com/zh-cn/library/mt143621.aspx)
- [Start-AzureSqlServerUpgrade](http://msdn.microsoft.com/zh-cn/library/mt143623.aspx)
- [Stop-AzureSqlServerUpgrade](http://msdn.microsoft.com/zh-cn/library/mt143622.aspx)


Stop- cmdlet 表示取消，而不是暂停。你无法在中途恢复升级，而只能从头开始重新升级。Stop- cmdlet 将清理并释放所有相应的资源。


## 相关链接

-  [SQL 数据库 V12 的新增功能](/documentation/articles/sql-database-v12-whats-new)
- [规划和准备升级到 SQL 数据库 V12](/documentation/articles/sql-database-v12-plan-prepare-upgrade)


<!--Image references-->

[1]: ./media/sql-database-preview-upgrade/firstscreenportal.png
[2]: ./media/sql-database-preview-upgrade/browse.png
[3]: ./media/sql-database-preview-upgrade/sqlserver.png
[4]: ./media/sql-database-preview-upgrade/sqlserverlist.png
[5]: ./media/sql-database-preview-upgrade/latestprview.png
[6]: ./media/sql-database-preview-upgrade/upgrade.png
[7]: ./media/sql-database-preview-upgrade/typeservername.png
[8]: ./media/sql-database-preview-upgrade/enabled.png

<!---HONumber=61-->