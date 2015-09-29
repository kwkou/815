<properties 
   pageTitle="在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库" 
   description="时间点还原, Windows Azure SQL 数据库, 还原数据库, 恢复数据库, Azure 管理门户, Azure 门户" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库

> [AZURE.SELECTOR]
- [时间点还原 - PowerShell](/documentation/articles/sql-database-point-in-time-restore-tutorial-powershell)
- [时间点还原 - REST API](/documentation/articles/sql-database-point-in-time-restore-tutorial-rest) 

## 概述

本教程说明如何在 [Azure 门户](http://manage.windowsazure.cn/)中使用时间点还原来还原 Azure SQL 数据库。Azure SQL Database 针对基本、标准和高级服务层提供内置备份，以支持自助时间点还原。

时间点还原会创建一个新的数据库。服务会根据还原时间点使用的备份自动选择服务层。请确保你在逻辑服务器上具有创建另一个数据库所需的可用配额。如果你想要请求增加配额，请联系 [Azure 支持](/support/contact/)。

## 限制和安全性

* 已针对基本、标准和高级服务层启用时间点还原。

* 备份保留期为：基本服务层 7 天、标准服务层 14 天、高级服务层 35 天。
 
* 只有订阅的管理员或协同管理员才能提交还原请求。

* 服务器级主体登录名将是还原的数据库的所有者。

* Web 和企业版服务层不支持时间点还原。
 
	* 如果你有 Web 或企业版数据库，可以使用数据库复制来获取数据库的事务一致副本，然后将复制的数据库导出到 Windows Azure 存储帐户。有关更多信息，请参阅[如何：使用数据库复制 (Azure SQL 数据库) ](http://msdn.microsoft.com/zh-cn/library/azure/ff951631.aspx) 和[如何：在 Azure SQL 数据库 中使用导入和导出服务](http://msdn.microsoft.com/zh-cn/library/azure/hh335292.aspx)。

	* Web 和企业版将在 2015 年 9 月停用。有关详细信息，请参阅 [Web 和企业版停用常见问题](http://msdn.microsoft.com/zh-cn/library/azure/dn741330.aspx)。

## 如何：在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库


1. 使用你的 Microsoft 帐户登录到 [Azure 门户](http://manage.windowsazure.cn)。

2. 在左侧导航窗格中，单击“SQL 数据库”。
  
3. 在“数据库”列表中，单击你要还原的数据库。

4. 在页面底部的命令栏中，单击“还原”。此时将启动“指定还原设置”对话框。

5. 选择“数据库名称”。默认情况下，系统会为你选择数据库名称，但你可以根据需要更改。

6. 选择要将数据库还原到的时间点。该时间点必须在数据库的保留期限内。
	
7. 单击复选标记以提交还原请求。

还原可能需要一段时间才能完成。若要监视还原状态，请在页面右下角的命令栏中单击“状态”图标，然后单击“详细信息”。

## 后续步骤

有关详细信息，请参阅以下主题：

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

[Azure SQL 数据库时间点还原（博客）](http://azure.microsoft.com/blog/2014/10/01/azure-sql-database-point-in-time-restore/)

<!---HONumber=69-->