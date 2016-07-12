<properties 
 pageTitle="计划程序的概念、术语和实体 | Azure"
 description="Azure 计划程序的概念、术语和实体层次结构，包括作业和作业集合。显示了一个计划作业的综合示例。"
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags
 ms.service="scheduler"
 ms.date="03/09/2016"
 wacn.date="04/11/2016"/>

# 计划程序的概念、术语和实体层次结构

## 计划程序实体层次结构

下表描述了计划程序 API 所公开或使用的主要资源：

|资源 | 说明 |
|---|---|
|**作业集合**|一个作业集合包含一组作业，并且维护该集合内各作业共享的设置、配额和限制。作业集合由订阅所有者创建，并基于使用情况或应用程序边界将作业组合在一起。它被约束到一个区域。通过作业集合，还可以强制执行配额以便约束对该集合中所有作业的使用。配额包括 MaxJobs 和 MaxRecurrence。|
|**作业**|一个作业通过用于执行的简单或复杂策略定义单个重复发生的操作。操作可以包含 HTTP 请求或存储队列请求。|
|**作业历史记录**|作业历史记录表示用于执行作业的详细信息。它包含成功与失败信息以及任何响应详细信息。|

## 计划程序实体管理

大体而言，计划程序和服务管理 API 公开资源上的以下操作：

|功能|说明和 URI 地址|
|---|---|
|**作业集合管理**|针对创建和修改作业集合和其中包含的作业的 GET、PUT 和 DELETE 支持。作业集合是针对作业以及指向配额和共享设置的映射的容器。配额（在后面介绍）的例子包括最大作业数和最小重复间隔 <p>PUT 和 DELETE：`https://management.core.chinacloudapi.cn/{subscriptionId}/cloudservices/{cloudServiceName}/resources/scheduler/jobcollections/{jobCollectionName}`</p><p>GET：`https://management.core.chinacloudapi.cn/{subscriptionId}/cloudservices/{cloudServiceName}/resources/scheduler/~/jobcollections/{jobCollectionName}`</p>
|**作业管理**|针对创建和修改作业的 GET、PUT、POST、PATCH 和 DELETE 支持。所有作业都必须属于某一已存在的作业集合，因此没有显式创建 <p>`https://management.core.chinacloudapi.cn/{subscriptionId}/cloudservices/{cloudServiceName}/resources/scheduler/~/jobcollections/{jobCollectionName}/jobs/{jobId}`</p>|
|**作业历史记录管理**|针对用于获取 60 天的作业执行历史记录（例如作业占用时间和作业执行结果）的 GET 支持。添加基于状态进行筛选的查询字符串参数支持 <P>`https://management.core.chinacloudapi.cn/{subscriptionId}/cloudservices/{cloudServiceName}/resources/scheduler/~/jobcollections/{jobCollectionName}/jobs/{jobId}/history`</p>|

## 作业类型

有两种类型的作业：HTTP 作业（包括支持 SSL 的 HTTPS 作业）和存储队列作业。HTTP 作业适用于现有工作负载或服务的终结点。可以使用存储队列作业将消息发布到存储队列，因此这些作业适合使用存储队列的工作负荷。

## “作业”实体详述

在基本级别上，一个计划的作业具有若干部分：

- 在作业计时器引发时要执行的操作  

- （可选）运行作业的时间

- （可选）重复作业的时间和频率

- （可选）主操作失败时要触发的操作

在内部，一个计划的作业还包含系统提供的数据，例如下次计划的执行时间。

以下代码提供了一个计划作业的综合示例。后面的部分中将提供详细信息。

	{
		"startTime": "2012-08-04T00:00Z",               // optional
		"action":
		{
			"type": "http",
			"retryPolicy": { "retryType":"none" },
			"request":
			{
				"uri": "http://contoso.com/foo",        // required
				"method": "PUT",                        // required
				"body": "Posting from a timer",         // optional
				"headers":                              // optional

				{
					"Content-Type": "application/json"
				},
			},
		   "errorAction":
		   {
			   "type": "http",
			   "request":
			   {
				   "uri": "http://contoso.com/notifyError",
				   "method": "POST",
			   },
		   },
		},
		"recurrence":                                   // optional
		{
			"frequency": "week",                        // can be "year" "month" "day" "week" "minute"
			"interval": 1,                              // optional, how often to fire (default to 1)
			"schedule":                                 // optional (advanced scheduling specifics)
			{
				"weekDays": ["monday", "wednesday", "friday"],
				"hours": [10, 22]
			},
			"count": 10,                                 // optional (default to recur infinitely)
			"endTime": "2012-11-04",                     // optional (default to recur infinitely)
		},
		"state": "disabled",                           // enabled or disabled
		"status":                                       // controlled by Scheduler service
		{
			"lastExecutionTime": "2007-03-01T13:00:00Z",
			"nextExecutionTime": "2007-03-01T14:00:00Z ",
			"executionCount": 3,
											    "failureCount": 0,
												"faultedCount": 0
		},
	}

如上面的示例计划作业中所示，一个作业定义具有若干部分：

- 开始时间（“startTime”）  

- 操作（“action”），包括错误操作（“errorAction”）

- 重复周期（“recurrence”）

- 状况（“state”）

- 状态（“status”）

- 重试策略（“retryPolicy”）

让我们仔细看看其中每一项：

## startTime

“startTime”是开始时间，允许调用方以 [ISO-8601 格式](http://zh.wikipedia.org/wiki/ISO_8601)在线指定时区偏移量。

##<a id="action-and-erroraction"></a> action 和 errorAction

“action”是每次执行时调用的操作，并且描述服务调用的类型。操作是将按提供的计划执行的内容。计划程序支持 HTTP 和存储队列操作。

上例中的操作是一个 HTTP 操作。下面是存储队列操作的示例：

	{
			"type": "storageQueue",
			"queueMessage":
			{
				"storageAccount": "myStorageAccount",  // required
				"queueName": "myqueue",                // required
				"sasToken": "TOKEN",                   // required
				"message":                             // required
					"My message body",
			},
	}

“errorAction”是错误处理程序，在主操作失败时调用的操作。你可以使用此变量调用错误处理终结点或发送用户通知。这可用于在主终结点不可用时（例如，在终结点的站点上出现灾难情形时）访问辅助终结点，或者可用于通知错误处理终结点。与主操作相似，错误操作可以是基于其他操作的简单或复合逻辑。若要了解如何创建一个 SAS 令牌，请参阅[创建和使用共享访问签名](/documentation/articles/storage-manage-access-to-resources/)。

## recurrence

重复周期具有若干部分：

- 频率：分钟、小时、天、周、月、年之一  

- 间隔：针对重复周期的按给定频率的间隔

- 规定的计划：指定重复周期的分钟、小时、星期几、月和月中第几天

- 计数：重复周期的计数

- 结束时间：在指定的结束时间后将不执行任何作业

如果某一作业具有在其 JSON 定义中指定的重复执行的对象，将重复执行该作业。如果计数和结束时间均指定，则遵循首先发生的完成规则。

## state

作业的状态是以下四个值之一：enabled、disabled、completed 或 faulted。你可以 PUT 或 PATCH 作业以便将它们更新到已启用或已禁用状态。如果某一作业已完成或已出错，则该作业是不能更新的最终状态（尽管该作业仍可以是 DELETED）。如下所示是状态属性的示例：


    	"state": "disabled", // enabled, disabled, completed, or faulted
60 天后删除完成的作业和出错的作业。

## status

一旦启动了某一计划程序作业后，将返回与该作业的当前状态有关的信息。用户无法设置该对象 – 该对象是由系统设置的。但是，该对象包含在作业对象中（而不是单独的链接资源中），因此，可以轻松地获取某一作业的状态。

作业状态包含前一个执行的时间（如果有）、下一个计划执行的时间（对于正在进行中的作业）以及作业的执行计数。

##<a id="retrypolicy"></a> retryPolicy

如果计划程序作业失败，可以指定重试策略来确定是否以及如何重试该操作。这由 **retryType** 对象确定 - 如果没有重试策略，它将设为 **none**，如上面所示。如果有重试策略，请将其设为 **fixed**。

若要设置重试策略，可指定两个附加设置：重试间隔 (**retryInterval**) 和重试次数 (**retryCount**)。

重试间隔使用 **retryInterval** 对象指定，表示两次重试之间的时间间隔。其默认值为 1 分钟，其最小值为 1 分钟，其最大值为 18 个月。它使用 ISO 8601 格式定义。同样，重试次数的值使用 **retryCount** 对象指定；它是尝试重试的次数。其默认值为 5，其最大值为 20。**retryInterval** 和 **retryCount** 都是可选的。如果 **retryType** 设为 **fixed** 并且未为它们显式指定任何值，则为它们赋予默认值。

## 另请参阅

 [计划程序是什么？](/documentation/articles/scheduler-intro/)

 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)

 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)

 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity/)

 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)

 [计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)

 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)

 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)

 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)
 

<!---HONumber=Mooncake_0405_2016-->