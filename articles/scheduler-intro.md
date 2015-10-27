<properties 
 pageTitle="计划程序是什么？" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags
 ms.service="scheduler"
 ms.date="08/04/2015"
 wacn.date="09/16/2015"/>

# 计划程序是什么？

Azure 计划程序允许你以声明方式描述要在云中运行的操作。然后，它自动计划并运行这些操作。Azure 计划程序使用 [Azure 门户](scheduler-get-started-portal)、代码、[REST API](https://msdn.microsoft.com/zh-cn/library/dn528946) 或 PowerShell 执行此操作。

Azure 计划程序维护、管理、计划并调用计划的工作。Azure 计划程序不托管任何工作负荷或运行任何代码。它仅_调用_别处托管的代码；该代码可以托管在 Azure 中、本地或与其他提供程序一起托管。它通过 HTTP、HTTPS 或存储队列调用。

Azure 计划程序计划作业、保留用户可以询问的作业执行结果历史记录，并确切可靠地计划要运行的工作负荷。Azure 移动服务计划脚本、Azure Web 应用程序 WebJobs 及其他 Azure 计划功能在后台使用 Azure 计划程序。[计划程序 REST API](https://msdn.microsoft.com/zh-cn/library/dn528946) 可帮助管理这些操作的通信。因此，计划程序轻松地支持[复杂的计划以及高级重复执行](scheduler-advanced-complexity)。

有几种方案适合使用 Azure 计划程序。例如：

+ _重复执行应用程序操作_：定期从 Twitter 收集数据并将数据收集到源。
+ _日常维护_：每天删改日志、执行备份和其他维护任务。例如，管理员可以选择在接下来的 9 个月中每天凌晨 1 点备份其数据库。

计划程序允许你以编程方式、使用脚本以及在门户中创建、更新、删除、查看和管理[“作业集合”和“作业”](/documentation/articles/scheduler-concepts-terms)。

## 另请参阅

 [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms)

 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal)

 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing)

 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity)

 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-cn/library/dn528946)

 [计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference)

 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability)

 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors)

 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication)
 
 

<!---HONumber=69-->