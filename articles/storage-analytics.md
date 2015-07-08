<properties 
	pageTitle="存储分析" 
	description="如何管理 Blob、队列、表和文件服务的并发" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>
<tags ms.service="storage"
    ms.date="03/06/2015"
    wacn.date="04/15/2015"
    />










# 存储分析

Azure 存储分析执行日志记录并为存储帐户提供度量值数据。可以使用此数据跟踪请求、分析使用情况趋势以及诊断存储帐户的问题。

若要使用存储分析，必须为每个要监视的服务单独启用它。可以从 [Azure 管理门户](https://manage.windowsazure.cn)启用它；有关详细信息，请参阅[如何监视存储帐户](http://www.windowsazure.cn/manage/services/storage/how-to-monitor-a-storage-account)。还可以通过 REST API 或客户端库以编程方式启用存储分析。使用[获取 Blob 服务属性](https://msdn.microsoft.com/zh-cn/library/hh452239.aspx)、[获取队列服务属性](https://msdn.microsoft.com/zh-cn/library/hh452243.aspx)和[获取表服务属性](https://msdn.microsoft.com/zh-cn/library/hh452238.aspx)操作为每个服务启用存储分析。

聚合数据存储在众所周知的 Blob（用于日志记录）和众所周知的表（用于度量）中，可以使用 BLOB 服务和表服务 API 对其进行访问。

存储分析针对存储的数据量实施 20TB 的限制，这与存储帐户的总限制无关。有关计费和数据保留策略的详细信息，请参阅[存储分析和计费](https://msdn.microsoft.com/zh-cn/library/hh360997.aspx)。有关存储帐户限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](https://msdn.microsoft.com/zh-cn/library/dn249410.aspx)。

有关使用存储分析及其他工具来识别、诊断和排查 Azure 存储相关问题的深入指导，请参阅[监视、诊断和排查 Windows Azure 存储空间问题](/documentation/articles/storage-monitoring-diagnosing-troubleshooting)。



<!--HONumber=50-->