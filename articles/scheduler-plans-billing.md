<properties 
 pageTitle="Azure 计划程序中的计划和计费方式" 
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
 
# Azure 计划程序中的计划和计费方式

## 作业集合计划

作业集合是 Azure 计划程序中的计费实体。作业集合包含许多作业，并附带三种计划 – 免费、标准和高级，下面将对此进行介绍。

|**作业集合计划**|**每个作业集合的作业数上限**|**最大重复次数**|**每个订阅的作业集合数上限**|**限制**|
|:---|:---|:---|:---|:---|
|**免费**|每个作业集合 5 个作业|每小时一次。执行作业的频率不能超过每小时一次|一个订阅最多允许 1 个免费作业集合|无法使用 [HTTP 出站授权对象](/documentation/articles/scheduler-outbound-authentication/)
|**标准**|每个作业集合 50 个作业|每分钟一次。执行作业的频率不能超过每分钟一次|一个订阅最多允许 100 个标准作业集合|访问计划程序的完整功能集|
|**高级**|每个作业集合 50 个作业|每分钟一次。执行作业的频率不能超过每分钟一次|一个订阅最多允许 10,000 个高级作业集合。有关详细信息，<a href="mailto:wapteams@microsoft.com">请联系我们</a>。|访问计划程序的完整功能集|

## 升级和降级作业集合计划

你随时可以在免费、标准和高级计划之间升级或降级作业集合计划。但是，在降级到免费作业集合时，降级可能会出于下列原因之一而失败：

- 订阅中已存在免费作业集合
- 作业集合中某个作业的重复周期高于免费作业集合中作业允许的重复周期。免费作业集合允许的最大重复周期为每小时一次
- 作业集合中有 5 个以上的作业
- 作业集合中某个作业的 HTTP 或 HTTPS 操作使用了 [HTTP 出站授权对象](/documentation/articles/scheduler-outbound-authentication/)

## 计费和 Azure 计划

不会向免费作业集合收取订阅费。如果你的标准作业集合数超过 100 个（10 个标准计费单位），则最好是将所有作业集合纳入高级计划。

如果你有一个标准作业集合和一个高级作业集合，则会根据一个标准计费单位和一个高级计费单位向你计费。计划程序服务根据设置为标准或高级的基于活动作业集合计费；接下来的两个部分将进一步对此做出解释。

## 标准计费单位

一个标准计费单位最多可以包含 10 个标准作业集合。由于每个标准作业集合最多可以包含 50 个作业，因此一个标准计费单位允许订阅最多包含 500 个作业 – 每月最多可以执行近 2200 万次作业。

如果你有 1 到 10 个标准作业集合，将会按照 1 个标准计费单位向你计费。如果你有 11 到 20 个标准作业集合，将会按照 2 个标准计费单位向你计费。如果你有 21 到 30 个标准作业集合，将会按照 3 个标准计费单位向你计费，依次类推。

## 高级计费单位

一个高级计费单位最多可以包含 10,000 个高级作业集合。由于每个高级作业集合最多可以包含 50 个作业，因此一个高级计费单位允许订阅最多包含 500,000 个作业 – 每月最多可以执行近 220 亿次作业。

如果你有 1 到 10,000 个高级作业集合，将会按照 1 个高级计费单位向你计费。如果你有 10,001 到 20,000 个高级作业集合，将会按照 2 个高级计费单位向你计费，依次类推。

因此，高级作业集合的功能与标准作业集合相同，但是，这个价格段适合应用程序需要许多作业集合的情况，

## 计费和活动状态

作业集合始终处于活动状态，除非你的整个订阅由于计费问题而进入某种临时禁用状态。确保作业集合不被计费的唯一方法是将它设置为“免费”计划，或者删除该作业集合。

尽管你可以通过单个操作禁用作业集合中的所有作业，但这样做不会更改作业集合的计费状态 – 作业集合仍将计费。同样，空作业集合被视为活动状态，因此将被计费。

## 定价

有关定价详细信息，请参阅[计划程序定价](/home/features/scheduler/pricing/)。

## 另请参阅
 

 [计划程序是什么？](/documentation/articles/scheduler-intro/)
 [Azure 计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)

 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)

 [Azure 计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)

 [Azure 计划程序 PowerShell Cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)
 
 [Azure 计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)

 [Azure 计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)

 [Azure 计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)
 
  

  

<!---HONumber=Mooncake_0405_2016-->