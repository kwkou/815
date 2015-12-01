<properties
	pageTitle="演练：激活最新的 SQL 数据库更新版 V12"
	description="介绍使用新的 Microsoft Azure 门户 UI 试用 Azure SQL 数据库版本 V12 的步骤。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="04/28/2015"
	wacn.date=""/>


# 演练：激活最新的 SQL 数据库更新版 V12

本主题介绍你可以遵循哪些步骤来激活 Microsoft 在 2014 年 12 月首次发布的 Azure SQL 数据库 V12 选项。

若要试用最新的 V12，首先需要订阅 Microsoft Azure，或者至少获得了一个[试用](/pricing/1rmb-trial/)订阅。

<!--You can activate V12 by using the Windows Azure portal at [https://manage.windowsazure.cn](https://manage.windowsazure.cn), instead of the [Windows Azure classic portal](http://manage.windowsazure.cn/).-->在为订阅激活 V12 后，V12 的创建和升级选项将在 Azure 门户中解锁。然后，用户可以从服务器边栏选项卡或数据库边栏选项卡启动[创建](/documentation/articles/sql-database-create)工作流。

> [AZURE.NOTE]测试数据库、数据库副本或新数据库都是适合升级到预览版的候选项。业务所依赖的生产数据库应在预览期过后再进行部署。

有关升级到 V12 的详细信息，请参阅[规划和准备升级到 Azure SQL 数据库 V12](/documentation/articles/sql-database-v12-plan-prepare-upgrade)。


## A.安全授权

第一步是确保你有足够的授权来为订阅激活 V12。当你尝试激活 V12 选项时，系统会执行授权检查，以验证你是否在订阅中拥有足够的权限。

 若要试用 V12，你必须具有以下授权之一：

- 订阅所有者
- 订阅的协同管理员

有关 Azure 帐户的详细信息，请参阅[管理帐户、订阅和管理角色](https://msdn.microsoft.com/zh-CN/library/hh531793.aspx)。

## B.Azure 门户 UI 中的步骤

本部分介绍在激活 V12 选项时，你可以在 Azure 门户 UI 中执行一次的单击序列。在激活该选项后，你仍可以使用此操作。

所有激活方案使用相同的基本思路。当你首次尝试[创建新的 SQL 数据库 服务器](/documentation/articles/sql-database-create)时，将显示标记为“最新更新(预览版)”的边栏选项卡，其中提供了一个复选框让你选择激活权限以使用 V12 版本。在你激活权限后，系统永远不会再次显示该复选框，而是显示一个“是|否”控件，让你指定是否希望新服务器使用 V12。如果你选择“否”，则会创建 V11 服务器（参阅 SELECT @@VERSION; 的返回值）。

### B.1 V12 版本的“是|否”控件

在激活 V12 选项权限后，你将看到“是|否”控件，如以下 Azure 门户屏幕截图中圈住的部分所示。

![YesNoOptionForTheV12Preview][Image1]


## C.后续步骤

以下主题介绍了如何使用 SQL 数据库 V12。

- [在 SQL 数据库更新版 V12 中创建数据库](/documentation/articles/sql-database-create)


<!-- References, Images. -->
[Image1]: ./media/sql-database-v12-sign-up/V12Preview-YesNo-Option-New-SQLDatabase-Server-Newserver-Screenshot-e23.png

 

<!---HONumber=69-->