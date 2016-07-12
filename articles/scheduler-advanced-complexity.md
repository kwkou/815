<properties 
 pageTitle="如何使用 Azure 计划程序生成复杂的计划和高级循环" 
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

# 如何使用 Azure 计划程序生成复杂的计划和高级循环  

## 概述

Azure 计划程序作业的核心是计划。计划确定了计划程序何时以及如何执行作业。

Azure 计划程序允许你为作业指定不同的一次性计划和重复性计划。一次性计划在指定的时间触发一次 - 实际上，它们是只执行一次的重复性计划。重复性计划根据预先确定的频率触发。

由于具有这种灵活性，Azure 计划程序可让你支持各种业务方案：

-	周期性数据清理 – 例如，每日删除 3 个月以前的所有推文
-	存档 – 例如，每月向备份服务推送发票历史记录
-	请求外部数据 – 例如，每隔 15 分钟从 NOAA 提取新的滑雪天气报告
-	图像处理 – 例如，在每个工作日的非高峰时间，使用云计算来压缩当天上载的图像


在本文中，我们将会演练你可以使用 Azure 计划程序创建的示例作业。我们将提供用于描述每个计划的 JSON 数据。如果你熟悉[计划程序 REST API](https://msdn.microsoft.com/zh-CN/library/azure/dn528946.aspx)，可以使用与此相同的 JSON 来[创建 Azure 计划程序作业](https://msdn.microsoft.com/zh-CN/library/azure/dn528937.aspx)。

## 支持的方案

本主题中的许多示例演示了 Azure 计划程序支持的各种方案。从广义上讲，这些示例演示了如何为多种行为模式创建计划，这些行为模式包括：

-	在特定的日期和时间运行一次
-	运行并重复明确的次数
-	立即运行，然后重复
-	运行并每隔 *n* 分钟、小时、天、周、月在特定的时间开始重复
-	运行并根据每周或每月的频率重复，不过，只会在特定的日期、特定的星期或特定的月份日次重复
-	运行并在某个时间段中的多个时间重复 - 例如，每月的最后一个星期五和星期一，或者每天的 5:15AM 和 5:15PM

## 日期和日期时间

Azure 计划程序作业中的日期遵循 [ISO-8601 规范](http://en.wikipedia.org/wiki/ISO_8601)并只包括日期。

Azure 计划程序作业中的日期时间引用遵循 [ISO-8601 规范](http://zh.wikipedia.org/wiki/ISO_8601)并包括日期和时间部分。未指定 UTC 偏差的日期时间将假定为 UTC。  

## 如何：使用 JSON 和 REST API 来创建计划

若要使用 [Azure 计划程序 REST API](https://msdn.microsoft.com/zh-cn/library/mt629143) 创建简单计划，将首先[使用资源提供程序注册订阅](https://msdn.microsoft.com/zh-cn/library/azure/dn790548.aspx)（计划程序的提供程序名称是 Microsoft.Scheduler），然后[创建作业集合](https://msdn.microsoft.com/zh-cn/library/mt629159.aspx)，最后[创建作业](https://msdn.microsoft.com/zh-cn/library/mt629145.aspx)。在创建作业时，可以使用类似于以下摘录内容的 JSON 指定计划和循环：

	{
	    "startTime": "2012-08-04T00:00Z", // optional
	     …
	    "recurrence":                     // optional
	    {
	        "frequency": "week",     // can be "year" "month" "day" "week" "hour" "minute"
	        "interval": 1,                // optional, how often to fire (default to 1)
	        "schedule":                   // optional (advanced scheduling specifics)
	        {
	            "weekDays": ["monday", "wednesday", "friday"],
	            "hours": [10, 22]                      
	        },
	        "count": 10,                  // optional (default to recur infinitely)
	        "endTime": "2012-11-04",      // optional (default to recur infinitely)
	    },
	    …
	}

## 概述：作业架构基础知识

下表提供了与作业中的循环和计划相关的主要元素的高级概述：

|**JSON 名称**|**说明**|
|:--|:--|
|**startTime**|startTime 是日期时间。对于简单的计划，startTime 是第一次循环；对于复杂的计划，一旦达到 startTime，作业就会启动。|
|**recurrence**|recurrence 对象指定作业的循环规则，以及执行作业时遵循的循环计划。recurrence 对象支持元素 frequency、interval、endTime、count 和 schedule。如果定义了 recurrence，则必须定义 frequency；recurrence 的其他元素是可选的。|
|**frequency**|frequency 表示重复作业的频率单位的频率字符串。支持的值为“minute”、“hour”、“day”、“week”或“month”。|
|**interval**|interval 是一个正整数，表示确定作业运行频率的间隔。例如，如果 interval 为 3，frequency 为“week”，则作业每隔 3 周重复一次。对于每月频率，Azure 计划程序支持最长 18 个月的间隔；对于每周频率，支持最长 78 周；对于每日频率，最长支持 548 天。对于小时和分钟频率，支持的范围为 1 <= interval <= 1000。|
|**endTime**|endTime 字符串指定一个日期时间，如果超过此日期时间，则不应执行作业。发生在过去的 endTime 是无效的。如果未指定 endTime 或 count，作业将无限期运行。不能为同一个作业同时指定 endTime 和 count。|
|**count**|<p>count 是一个正整数（大于零），指定此作业在完成之前应运行的次数。</p><p>count 表示作业在被确定为已完成之前要运行的次数。例如，对于 count 为 5 并且开始日期为星期一的每日执行的作业，该作业将在星期五执行后完成。如果开始时间发生在过去，则基于创建时间计算第一次执行时间。</p><p>如果未指定 endTime 或 count，作业将无限期运行。不能为同一个作业同时指定 endTime 和 count。</p>|
|**schedule**|指定了频率的作业将根据循环执行更改其循环。schedule 包含基于分钟、小时、星期、月份日次和周次的修改。|

## 概述：作业架构默认值、限制和示例

在了解概述后，让我们详细讨论其中的每个元素。

|**JSON 名称**|**值类型**|**必需？**|**默认值**|**有效值**|**示例**|
|:---|:---|:---|:---|:---|:---|
|**startTime**|字符串|否|无|ISO-8601 日期时间|<code>"startTime" :"2013-01-09T09:30:00-08:00"</code>|
|**recurrence**|对象|否|无|Recurrence 对象|<code>"recurrence" : { "frequency" : "monthly", "interval" : 1 }</code>|
|**frequency**|字符串|是|无|"minute"、"hour"、"day"、"week"、"month"|<code>"frequency" :"hour"</code> |
|**interval**|数字|否|1|1 到 1000。|<code>"interval":10</code>|
|**endTime**|字符串|否|无|代表将来某个时间的日期时间值|<code>"endTime" :"2013-02-09T09:30:00-08:00"</code> |
|**count**|数字|否|无|>= 1 |<code>"count":5</code>|
|**schedule**|对象|否|无|Schedule 对象|<code>"schedule" : { "minute" : [30], "hour" : [8,17] }</code>|

## 深入探讨：startTime

下表说明了 startTime 如何控制作业的运行方式。

|**startTime 值**|**无循环**|**循环。无计划**|**按计划循环**|
|:--|:--|:--|:--|
|**无开始时间**|立即运行一次|立即运行一次。在从上次执行时间进行计算后运行后续执行|<p>立即运行一次</p><p>根据循环计划运行后续执行</p>|
|**开始时间在过去**|立即运行一次|<p>计算开始时间后的第一个将来执行时间，并在该时间运行</p><p>在从上次执行时间进行计算后运行后续执行</p><p>有关进一步说明，请参阅此表后面的示例</p>|<p>一旦达到指定的开始时间，作业就会立即启动。第一次循环基于从开始时间计算的计划</p><p>根据循环计划运行后续执行</p>|
|**开始时间在将来或当前**|在指定的开始时间运行一次|<p>在指定的开始时间运行一次</p><p>在从上次执行时间进行计算后运行后续执行</p>|<p>一旦达到指定的开始时间，作业就会立即启动。第一次循环基于从开始时间计算的计划</p><p>根据循环计划运行后续执行</p>|

让我们查看 startTime 在过去，并且指定了 recurrence 但未指定 schedule 的情况下会发生的情况示例。假设当前时间为 2015-04-08 13:00，startTime 为 2015-04-07 14:00，recurrence 为每隔 2 天（定义方式为 frequency: day，interval: 2）。 请注意， startTime 在过去，即发生在当前时间以前

在这种情况下，第一次执行将发生在 2015-04-09 14:00。计划程序引擎将从开始时间计算执行循环。过去的所有实例将被丢弃。引擎将使用将来发生的下一个实例。在本例中，startTime 为 2015-04-07 2:00pm，因此，下一个实例为从该时间算起的 2 天，即 2015-04-09 2:00pm。

请注意，不管 startTime 是 2015-04-05 14:00 还是 2015-04-01 14:00，第一次执行时间都是相同的。在第一次执行后，将使用计划循环计算后续执行 - 依次为 2015-04-11 2:00pm、2015-04-13 2:00pm 和 2015-04-15 2:00pm，等等。

最后，如果为作业指定了计划，但未在计划中设置小时和/或分钟，则小时和分钟分别默认为第一次执行的小时和/或分钟。

## 深入探讨：schedule

在一方面，schedule 可以限制作业执行的次数。例如，如果定义了 "month" 频率的作业遵循的计划只会在 31 号运行，则该作业只会在包含 31号 的月份运行<sup></sup>。

另一方面，schedule 还可以增加作业执行的次数。例如，如果定义了 "month" 频率的作业遵循的计划在 1 号和 2 号运行，则该作业将在每月的 1 号 和 2号 运行，而不只是在每月运行一次<sup></sup><sup></sup>。

如果指定了多个计划元素，则求值顺序为大到小 – 周次、月份日次、星期、小时和分钟。

下表详细描述了 schedule 元素。

|**JSON 名称**|**说明**|**有效值**|
|:---|:---|:---|
|**minutes**|运行作业的小时中的分钟|<ul><li>整数，或</li><li>整数数组</li></ul>|
|**hours**|运行作业的日期中的小时|<ul><li>整数，或</li><li>整数数组</li></ul>|
|**weekDays**|运行作业的星期日期。只能配合每周频率指定。|<ul><li>"Monday"、"Tuesday"、"Wednesday"、"Thursday"、"Friday"、"Saturday" 或 "Sunday"</li><li>上述任意值的数组（最大数组大小为 7）</li></ul>不区分大小写|
|**monthlyOccurrences**|确定运行作业的月份日期。只能配合每月频率指定。|<ul><li>MonthlyOccurence 对象的数组：</li></ul> <pre>{ "day": day,<br /> "occurrence": occurence<br />}</pre><p> day 是运行作业的星期日期，例如，{Sunday} 表示月份中的每个星期日。必需。</p><p>occurrence 是月份中重复的日期，例如 {Sunday-1} 表示月份中的最后一个星期日。可选。</p>|
|**monthDays**|运行作业的月份日期。只能配合每月频率指定。|<ul><li><= -1 且 >= -31 的任何值</li><li>>= 1 且 <= 31 的任何值。</li><li>上述值的数组</li></ul>|

## 示例：循环计划

以下是循环计划的不同示例 – 着重于计划对象及其子元素。

以下计划全都假设 interval 设置为 1。此外，用户必须假设正确的频率符合 schedule - 例如，不能使用频率 "day"，并且计划中不能包含 "monthDays" 限定。上面介绍了此类限制。

|**示例**|**说明**|
|:---|:---|
|<code>{"hours":[5]}</code>|在每天的 5AM 运行。Azure 计划程序会将 "hours" 中的每个值与 "minutes" 中的每个值一一匹配，以创建运行作业的所有时间的列表。|
|<code>{"minutes":[15],"hours":[5]}</code>|在每天的 5:15AM 运行|
|<code>{"minutes":[15],"hours":[5,17]}</code>|在每天的 5:15 AM 和 5:15 PM 运行|
|<code>{"minutes":[15,45],"hours":[5,17]}</code>|在每天的 5:15AM、5:45AM、5:15PM 和 5:45PM 运行|
|<code>{"minutes":[0,15,30,45]}</code>|每隔 15 分钟运行一次|
|<code>{hours":[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23]}</code>|每隔一小时运行一次。此作业每隔一小时运行一次。分钟由 startTime（如果已指定）控制，如果未指定 startTime，则由创建时间控制。例如，如果开始时间或创建时间（以适用的为准）为 12:25 PM，则作业将在 00:25、01:25、02:25 ... 23:25 运行。该计划相当于为作业指定了频率 "hour"、间隔 1 且无计划。差别在于，也可以配合不同的频率和间隔使用此计划来创建其他作业。例如，如果频率为 "month"，则该计划将每月运行一次，而不是像频率为 "day" 时那样每天运行一次|
|<code>{minutes:[0]}</code>|每隔一小时整点运行。此作业也是每隔一小时运行一次，不过是在整点运行（例如 12AM、1AM、2AM，等等）。 这相当于作业频率为 "hour"，startTime 为零分钟，并且在频率为 "day" 时无计划。但是，如果频率为 "week" 或 "month"，则计划分别只会在一个星期日期或月份日期执行。|
|<code>{"minutes":[15]}</code>|在过去每隔一小时的第 15 分钟运行。以下作业将每隔一小时运行，从 00:15AM、1:15AM、2:15AM 等开始，在 10:15PM 和 11:15PM 结束。|
|<code>{"hours":[17],"weekDays":["saturday"]}</code>|在每周星期六的 5PM 运行|
|<code>{hours":[17],"weekDays":["monday","wednesday","friday"]}</code>|在每周星期一、星期三和星期五的 5PM 运行|
|<code>{"minutes":[15,45],"hours":[17],"weekDays":["monday","wednesday","friday"]}</code>|在每周星期一、星期三和星期五的 5:15PM 和 5:45PM 运行|
|<code>{"hours":[5,17],"weekDays":["monday","wednesday","friday"]}</code>|在每周星期一、星期三和星期五的 5AM 和 5PM 运行|
|<code>{"minutes":[15,45],"hours":[5,17],"weekDays":["monday","wednesday","friday"]}</code>|在每周星期一、星期三和星期五的 5:15AM、5:45AM、5:15PM 和 5:45PM 运行|
|<code>{"minutes":[0,15,30,45], "weekDays":["monday","tuesday","wednesday","thursday","friday"]}</code>|在工作日每隔 15 分钟运行一次|
|<code>{"minutes":[0,15,30,45], "hours": [9, 10, 11, 12, 13, 14, 15, 16] "weekDays":["monday","tuesday","wednesday","thursday","friday"]}</code>|在工作日 9AM 到 4:45PM 每隔 15 分钟运行一次|
|<code>{"weekDays":["sunday"]}</code>|在星期日的开始时间运行|
|<code>{"weekDays":["tuesday", "thursday"]}</code>|在星期二和星期四的开始时间运行|
|<code>{"minutes":[0],"hours":[6],"monthDays":[28]}</code>|在每个月 28 号的 6AM 运行（假设 frequency 为 month）|
|<code>{"minutes":[0],"hours":[6],"monthDays":[-1]}</code>|在月份最后一天的 6AM 运行。如果你要在月份的最后一天运行作业，请使用 -1 而不是日期 28、29、30 或 31。|
|<code>{"minutes":[0],"hours":[6],"monthDays":[1,-1]}</code>|在每月第一天和最后一天的 6AM 运行|
|<code>{monthDays":[1,-1]}</code>|在每月第一天和最后一天的开始时间运行|
|<code>{monthDays":[1,14]}</code>|在每月第一天和第 14 天的开始时间运行|
|<code>{monthDays":[2]}</code>|在月份第二天的开始时间运行|
|<code>{"minutes":[0], "hours":[5], "monthlyOccurrences":[{"day":"friday","occurrence":1}]}</code>|在每月第一个星期五的 5AM 运行|
|<code>{"monthlyOccurrences":[{"day":"friday","occurrence":1}]}</code>|在每月第一个星期五的开始时间运行|
|<code>{"monthlyOccurrences":[{"day":"friday","occurrence":-3}]}</code>|在每月结束后第三个星期五的开始时间运行|
|<code>{"minutes":[15],"hours":[5],"monthlyOccurrences":[{"day":"friday","occurrence":1},{"day":"friday","occurrence":-1}]}</code>|在每月第一个和最后一个星期五的 5:15AM 运行|
|<code>{"monthlyOccurrences":[{"day":"friday","occurrence":1},{"day":"friday","occurrence":-1}]}</code>|在每月第一个和最后一个星期五的开始时间运行|
|<code>{"monthlyOccurrences":[{"day":"friday","occurrence":5}]}</code>|在每月第五个星期五的开始时间运行。如果月份中没有第五个星期五，则不运行作业，因为该作业计划为只在第五个星期五运行。如果你想要在月份的最后一个星期五运行作业，可以考虑为循环使用 -1 而不是 5。|
|<code>{"minutes":[0,15,30,45],"monthlyOccurrences":[{"day":"friday","occurrence":-1}]}</code>|在月份的最后一个星期五每隔 15 分钟运行一次|
|<code>{"minutes":[15,45],"hours":[5,17],"monthlyOccurrences":[{"day":"wednesday","occurrence":3}]}</code>|在每月第三个星期三的 5:15AM、5:45AM、5:15PM 和 5:45PM 运行|

## 另请参阅
 
 [计划程序是什么？](/documentation/articles/scheduler-intro/)
 
 [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)
 
 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)
 
 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-cn/library/dn528946)
 
 [计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)
 
 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)
 
 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)
 
 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)
 
  

<!---HONumber=Mooncake_0405_2016-->