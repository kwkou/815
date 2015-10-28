<properties
	pageTitle="演练：激活最新的 SQL 数据库 Update V12（预览版）"
	description="介绍通过新的 Azure 门户 UI 试用 Azure SQL 数据库 V12 预览版的步骤。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jhubbard, jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="04/28/2015"
	wacn.date="06/30/2015"/>


# 演练：激活最新的 SQL 数据库 Update V12（预览版）

本主题介绍你可以遵循哪些步骤来激活 Microsoft 在 2014 年 12 月首次发布的 Azure SQL 数据库 Update V12（预览版）选项。

若要试用最新的 V12，首先需要订阅 Windows Azure，或者至少获得了一个[试用](/pricing/1rmb-trial/)订阅。

可以使用 Azure 门户 ([http://manage.windowsazure.cn/](http://manage.windowsazure.cn/)) 来激活 V12 预览版。在为订阅激活 V12 预览版后，V12 预览版的创建和升级选项将在 Azure 门户中解锁。然后，用户可以从服务器边栏选项卡或数据库边栏选项卡启动[创建](/documentation/articles/sql-database-preview-create)或[升级](/documentation/articles/sql-database-preview-create)工作流。

> [AZURE.NOTE]测试数据库、数据库副本或新数据库都是适合升级到预览版的候选项。业务所依赖的生产数据库应在预览期过后再进行部署。

有关该预览版的详细信息，请参阅[规划和准备升级到 Azure SQL 数据库 V12（预览版）](/documentation/articles/sql-database-preview-plan-prepare-upgrade)。


## A.安全授权

第一步是确保你有足够的授权来为订阅激活 V12 预览版。当你尝试激活 V12 预览版选项时，系统会执行授权检查，以验证你是否在订阅中拥有足够的权限。

 若要试用预览版，你必须具有以下授权之一：

- 订阅所有者
- 订阅的协同管理员

有关 Azure 帐户的详细信息，请参阅[管理帐户、订阅和管理角色](http://msdn.microsoft.com/zh-cn/library/hh531793.aspx)。

## B.Azure 门户 UI 中的步骤

本部分介绍在激活 V12 预览版选项时，你可以在 Azure 门户 UI 中执行一次的单击序列。在激活该选项后，你仍可以使用此操作。

所有激活方案使用相同的基本思路。当你首次尝试[创建新的 SQL 数据库 服务器](/documentation/articles/sql-database-preview-create)时，将显示标记为“最新更新(预览版)”的边栏选项卡，其中提供了一个复选框让你选择激活权限以使用 V12 预览版。在你激活权限后，系统永远不会再次显示该复选框，而是显示一个“是|否”控件，让你指定是否希望新服务器使用 V12 预览版。如果你选择“否”，则会创建 V11 服务器（参阅 SELECT @@VERSION; 的返回值）。

### B.1 V12 预览版的“是|否”控件

在激活 V12 预览版权限后，你将看到“是|否”控件，如以下 Azure 门户屏幕截图中圈住的部分所示。

![YesNoOptionForTheV12Preview][Image1]


## C.后续步骤

以下主题介绍了如何使用 V12 预览版。

- [在最新的 SQL 数据库 Update V12（预览版）中创建数据库](/documentation/articles/sql-database-preview-create)
- [升级到最新的 SQL 数据库 Update V12（预览版）](/documentation/articles/sql-database-preview-upgrade)

> [AZURE.NOTE]测试数据库、数据库副本或新数据库都是适合升级到预览版的候选项。业务所依赖的生产数据库应在预览期过后再进行部署。


<!-- References, Images. -->

[Image1]: ./media/sql-database-preview-sign-up/V12Preview-YesNo-Option-New-SQLDatabase-Server-Newserver-Screenshot-e23.png


<!-- EOF -->

<!---HONumber=61-->