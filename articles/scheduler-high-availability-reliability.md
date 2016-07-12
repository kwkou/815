<properties 
 pageTitle="计划程序高可用性和可靠性" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags 
 ms.service="scheduler" 
 ms.date="03/09/2016"
 wacn.date="04/11/2016"/>
 
 
# 计划程序高可用性和可靠性

## Azure 计划程序高可用性

作为一个核心 Azure 平台服务，Azure 计划程序高度可用，并提供地域冗余服务部署和地理区域作业复制功能。

### 异地冗余的服务部署

Azure 计划程序已在中国的两个区域推出：中国东部和中国北部。如果托管区域中的某个数据中心服务中断，则 Azure 计划程序的故障转移功能便可发挥作用，使你能够从另一个数据中心使用该服务。

### 地理区域作业复制

不只是 Azure 计划程序前端可用于管理请求，你自己的作业也会经过地域复制。当一个区域中的服务中断时，Azure 计划程序将故障转移，并确保配对地理区域中的另一个数据中心运行作业。

例如，如果你已在美国中南部创建了一个作业，Azure 计划程序会自动在美国中北部复制该作业。如果美国中南部发生故障，Azure 计划程序可确保从美国中北部运行该作业。[有关 Azure 异地复制功能的深入说明，请参阅本主题。](/documentation/articles/sql-database-business-continuity-design/)

![][1]

因此，在 Azure 发生故障时，Azure 计划程序可确保你的数据始终位于同样广泛的地理区域内。你不需要复制作业而只是为了增加高可用性 – Azure 计划程序将自动为你的作业提供高可用性功能。

## Azure 计划程序可靠性

Azure 计划程序可保证其自身的高可用性，并提供不同的途径访问用户创建的作业。例如，你的工作可能会调用不可用的 HTTP 终结点。尽管如此，Azure 计划程序会通过为你提供处理故障的替代选项，尝试成功执行你的作业。Azure 计划程序通过两种方式实现此目的：

### 通过“retryPolicy”实现的可配置重试策略

Azure 计划程序允许你配置重试策略。默认情况下，如果某个作业失败，计划程序将在 30 秒的时间间隔内重试该作业四次。你可以将此重试策略重新配置为更频繁（例如，在 30 秒时间间隔内重试 10 次）或更松散（例如，每天重试两次）。

举例来说，如果你创建的作业每周运行一次并调用 HTTP 终结点，则此配置可能会有帮助。如果当你的作业运行时 HTTP 终结点已关闭了几个小时，则你可能不希望再等待一周来再次运行作业，因为默认重试策略将会失败。在这种情况下，你可以重新配置标准的重试策略，以便每隔三个小时（举例）而不是每隔 30 秒重试。

若要了解如何配置重试策略，请参阅 [retryPolicy](/documentation/articles/scheduler-concepts-terms/#retrypolicy)。

### 通过“errorAction”实现的备用终结点可配置性

如果 Azure 计划程序作业的目标终结点仍然无法访问，Azure 计划程序将回退到备用的错误处理终结点后重试策略。如果配置了备用错误处理终结点，则 Azure 计划程序将调用该终结点。有了备用终结点，在发生故障时你自己的作业将高度可用。

例如，在下图中，Azure 计划程序遵循其重试策略来访问位于纽约的 Web 服务。重试失败后，它会检查是否有备用终结点。然后，它将使用相同的重试策略继续向备用终结点发出请求。

![][2]

请注意，相同的重试策略适用于原始操作和备用错误操作。备用错误操作的操作类型还可以不同于主要操作的操作类型。例如，虽然主要操作可能要调用的 HTTP 终结点，错误操作而可能存储队列操作执行错误日志记录。

若要了解如何配置备用终结点，请参阅 [errorAction](/documentation/articles/scheduler-concepts-terms/#action-and-erroraction)。

## 另请参阅
 
 [计划程序是什么？](/documentation/articles/scheduler-intro/)
 
 [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)
 
 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)
 
 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity/)
 
 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)
 
 [计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)
 
 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)
 
 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)
 
 
[1]: ./media/scheduler-high-availability-reliability/scheduler-high-availability-reliability-image1.png

[2]: ./media/scheduler-high-availability-reliability/scheduler-high-availability-reliability-image2.png


<!---HONumber=Mooncake_0405_2016-->