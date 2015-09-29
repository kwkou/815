<properties 
   pageTitle="在 Azure 门户中还原已删除的 Azure SQL 数据库" 
   description="Windows Azure SQL 数据库, 还原已删除的数据库, 恢复已删除的数据库, Azure 管理门户, Azure 门户" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 在 Azure 门户中还原已删除的 Azure SQL 数据库

> [AZURE.SELECTOR]
- [还原已删除数据库 - PowerShell](/documentation/articles/sql-database-restore-deleted-database-tutorial-powershell)
- [还原已删除数据库 - REST API](/documentation/articles/sql-database-restore-deleted-database-tutorial-rest)

## 概述

本教程说明如何在 [Azure 门户](http://manage.windowsazure.cn/)中还原已删除的 Azure SQL 数据库。你可以将保留期内删除的数据库还原到删除该数据库时的时间点。保留期由数据库的服务层确定。

还原已删除的 Azure SQL 数据库会创建一个新的数据库。服务会根据还原时间点使用的备份自动选择服务层。请确保你在逻辑服务器上具有创建另一个数据库所需的可用配额。如果你想要请求增加配额，请联系 [Azure 支持](/support/contact/)。

## 限制和安全性

* 已针对基本、标准和高级服务层启用还原已删除的 Azure SQL 数据库功能。 

* 备份保留期为：基本服务层 7 天、标准服务层 14 天、高级服务层 35 天。

* 只有订阅的管理员或协同管理员才能提交还原请求。

* 服务器级主体登录名将是还原的数据库的所有者。
 
* Web 和企业版服务层不支持还原已删除的 SQL 数据库。
 
	* 如果你有 Web 或企业版数据库，可以使用数据库复制来获取数据库的事务一致副本，然后将复制的数据库导出到 Windows Azure 存储帐户。有关更多信息，请参阅[如何：使用数据库复制 (Azure SQL 数据库) ](http://msdn.microsoft.com/zh-cn/library/azure/ff951631.aspx) 和[如何：在 Azure SQL 数据库 中使用导入和导出服务](http://msdn.microsoft.com/zh-cn/library/azure/hh335292.aspx)。
	* Web 和企业版将在 2015 年 9 月停用。有关详细信息，请参阅 [Web 和企业版停用常见问题](http://msdn.microsoft.com/zh-cn/library/azure/dn741330.aspx)。

## 如何：在 Azure 门户中还原已删除的 Azure SQL 数据库


1. 使用你的 Microsoft 帐户登录到 [Azure 门户](http://manage.windowsazure.cn)。

2. 在左侧导航窗格中，单击“SQL 数据库”。

3. 在页面顶部，单击“已删除的数据库”。此时将显示“可还原的已删除数据库”列表。

4. 选择要还原的数据库。

6. 在页面底部的命令栏中，单击“还原”。此时将启动“指定还原设置”对话框。

7. 选择“数据库名称”。默认情况下，系统会为你选择数据库名称，但你可以根据需要更改。

	* 请注意，已删除的数据库只能还原到它在删除时所在的服务器，并且只能还原到删除时的时间点。   

8. 单击复选标记以提交还原请求。

还原可能需要一段时间才能完成。若要监视还原状态，请在页面右下角的命令栏中单击“状态”图标，然后单击“详细信息”。

## 后续步骤

有关详细信息，请参阅以下主题：

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

<!---HONumber=69-->