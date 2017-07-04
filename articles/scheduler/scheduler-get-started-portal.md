<properties 
 pageTitle="Azure 门户中的 Azure 计划程序入门 | Azure"
 description="Azure 门户中的 Azure 计划程序入门"
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags
 ms.service="scheduler"
 ms.workload="infrastructure-services"
 ms.tgt_pltfrm="na"
 ms.devlang="dotnet"
 ms.topic="hero-article"
 ms.date="08/10/2016"
 wacn.date="01/03/2017"
 ms.author="deli"/>

# Azure 门户中的 Azure 计划程序入门

在 Azure 计划程序中创建计划的作业很简单。在本教程中，你将了解如何创建作业。还将学习计划程序的监视和管理功能。

## 创建作业

1.  登录到 [Azure 门户](https://portal.azure.cn/)。

2.  单击“+新建”> 在搜索框中键入“计划程序”> 在结果中选择“计划程序”> 单击“创建”。

     ![][marketplace-create]

3.  让我们来创建一个作业，这只需要使用 GET 请求访问 http://www.microsoft.com/。在“计划程序作业”屏幕上，输入以下信息：

    1.  **名称：**`getmicrosoft`

    2.  **订阅：**您的 Azure 订阅

    3.  **作业集合：**选择现有的作业集合，或者单击“新建”>“输入名称”。

4.  接下来，在“操作设置”中，定义以下值：

    1.  **操作类型：**` HTTP`

    2.  **方法：**`GET`

    3.  **URL：**` http://www.microsoft.com`

      ![][action-settings]  


5.  最后，让我们来定义一个计划。该作业可定义为一次性作业，但是我们选择了重复执行的计划：

    1. **重复周期**：`Recurring`

    2. **开始日期**：今天的日期

    3. **执行间隔**：`12 Hours`

    4. **结束日期**：从今天起两天后的日期

      ![][recurrence-schedule]  


6.  单击“创建”

## 管理和监视作业

作业创建完成后，将出现在 Azure 主仪表板中。单击该作业，将打开具有以下选项卡的新窗口：

1.  属性

2.  操作设置

3.  计划

4.  历史记录

5.  用户

    ![][job-overview]

### 属性

这些只读属性描述计划程序作业的管理元数据。

   ![][job-properties]


### 操作设置

在“作业”屏幕中单击某个作业可以配置该作业。如果未在快速创建向导中配置它们，此操作可配置高级设置。

对于所有操作类型，都可以更改重试策略和错误操作。

对于 HTTP 和 HTTPS 作业操作类型，可以将方法更改为允许的任何 HTTP 谓词。你还可以添加、删除或更改标头及基本身份验证信息。

对于存储队列操作类型，可以更改存储帐户、队列名称、SAS 令牌和正文。

对于服务总线操作类型，可以更改命名空间、主题/队列路径、身份验证设置、传输类型、消息属性和消息正文。

   ![][job-action-settings]

### 计划

如果想要更改快速创建向导中创建的计划，此处能够重新配置该计划。

这是[作业中生成复杂计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity/)的机会

可以更改开始日期与时间、重复计划以及结束日期与时间（如果该作业是重复进行的。）

   ![][job-schedule]


### 历史记录

“历史记录”选项卡显示在所选作业的系统中每次作业执行的所选度量值。这些度量值提供有关计划程序的运行状况实时值：

1.  状态

2.  详细信息

3.  重试次数

4.  发生次数：第 1 次、第 2 次、第 3 次等

5.  执行开始时间

6.  执行结束时间

   ![][job-history]  


可单击运行以查看**历史记录详情**，包括每次执行的整个响应情况。此对话框还允许你将响应复制到剪贴板。

   ![][job-history-details]

### 用户

Azure 基于角色的访问控制 (RBAC) 可用于对 Azure 计划程序进行细致的访问管理。若要了解如何使用“用户”选项卡，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)


## 另请参阅

 [计划程序是什么？](/documentation/articles/scheduler-intro/)

 [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)

 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)

 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity/)

 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)

 [计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)

 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)

 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)

 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)


[marketplace-create]: ./media/scheduler-get-started-portal/scheduler-v2-portal-marketplace-create.png
[action-settings]: ./media/scheduler-get-started-portal/scheduler-v2-portal-action-settings.png
[recurrence-schedule]: ./media/scheduler-get-started-portal/scheduler-v2-portal-recurrence-schedule.png
[job-properties]: ./media/scheduler-get-started-portal/scheduler-v2-portal-job-properties.png
[job-overview]: ./media/scheduler-get-started-portal/scheduler-v2-portal-job-overview-1.png
[job-action-settings]: ./media/scheduler-get-started-portal/scheduler-v2-portal-job-action-settings.png
[job-schedule]: ./media/scheduler-get-started-portal/scheduler-v2-portal-job-schedule.png
[job-history]: ./media/scheduler-get-started-portal/scheduler-v2-portal-job-history.png
[job-history-details]: ./media/scheduler-get-started-portal/scheduler-v2-portal-job-history-details.png


[1]: ./media/scheduler-get-started-portal/scheduler-get-started-portal001.png
[2]: ./media/scheduler-get-started-portal/scheduler-get-started-portal002.png
[3]: ./media/scheduler-get-started-portal/scheduler-get-started-portal003.png
[4]: ./media/scheduler-get-started-portal/scheduler-get-started-portal004.png
[5]: ./media/scheduler-get-started-portal/scheduler-get-started-portal005.png
[6]: ./media/scheduler-get-started-portal/scheduler-get-started-portal006.png
[7]: ./media/scheduler-get-started-portal/scheduler-get-started-portal007.png
[8]: ./media/scheduler-get-started-portal/scheduler-get-started-portal008.png
[9]: ./media/scheduler-get-started-portal/scheduler-get-started-portal009.png
[10]: ./media/scheduler-get-started-portal/scheduler-get-started-portal010.png
[11]: ./media/scheduler-get-started-portal/scheduler-get-started-portal011.png
[12]: ./media/scheduler-get-started-portal/scheduler-get-started-portal012.png
[13]: ./media/scheduler-get-started-portal/scheduler-get-started-portal013.png
[14]: ./media/scheduler-get-started-portal/scheduler-get-started-portal014.png
[15]: ./media/scheduler-get-started-portal/scheduler-get-started-portal015.png

<!---HONumber=Mooncake_Quality_Review_1230_2016-->