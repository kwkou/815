<properties 
	pageTitle="存储分析和计费" 
	description="了解如何利用存储分析来更好地了解存储服务的计费情况" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>
<tags ms.service="storage"
    ms.date=""
    wacn.date="04/15/2015"
    />











# 存储分析和计费

本主题讨论了存储分析成本，以及如何使用度量和日志记录数据了解你的存储服务帐单。


## 存储分析成本

存储分析是由存储帐户所有者启用的；默认情况下，不会启用存储分析。所有度量数据是由存储帐户服务写入的。因此，存储分析执行的每个写入操作都是计费的。此外，度量数据使用的存储量也是计费的。

存储分析执行的以下操作都是计费的：

- 为日志记录创建 Blob 的请求

- 为度量创建表实体的请求

如果你配置了数据保留策略，在存储分析删除以前的日志记录和度量数据时，不会对删除事务进行收费。不过，从客户端中删除事务是计费的。有关保留策略的详细信息，请参阅[设置存储分析数据保留策略](https://msdn.microsoft.com/zh-cn/library/azure/hh343263.aspx)。

## 了解计费请求

向帐户的存储服务发出的每个请求是应计费或不计费的。存储分析记录向服务发出的每个请求，包括指示如何处理请求的状态消息。同样，存储分析存储服务及其 API 操作的度量数据，包括某些状态消息的百分比和计数。总之，这些功能可以帮助分析你的计费请求，对你的应用程序进行改进，以及诊断向你的服务发出的请求的问题。有关计费的详细信息，请参阅[了解 Azure 存储计费 - 带宽、事务和容量](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx)。

在查看存储分析数据时，你可以使用[存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/azure/hh343260.aspx)主题中的表来确定哪些请求是计费的。然后，你可以将日志和度量数据与状态消息进行比较，以查看是否对你的特定请求进行收费。也可以使用上述主题中的表来调查存储服务或各个 API 操作的可用性。

## 后续步骤
[存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/azure/hh343260.aspx)

[关于存储分析日志记录](https://msdn.microsoft.com/zh-cn/library/azure/hh343262.aspx)

[关于存储分析度量值](/documentation/articles/storage-about-analytics-metrics) 

<!--HONumber=50-->