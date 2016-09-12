<properties 
   pageTitle="流分析限制表"
   description="描述流分析组件和连接的系统限制和建议大小。"
   services="stream-analytics"
   documentationCenter="NA"
   authors="jeffstokes72"
   manager="paulettm"
   editor="cgronlun" />
<tags 
   ms.service="stream-analytics"
   ms.date="07/13/2015"
   wacn.date="" />

| 限制标识符 | 限制 | 注释 |
|----------------- | ------------|--------- |
| 每个区域每个订阅的最大流式处理单位数 | 50 | 增加订阅的流式处理单位数超过 50 的请求可通过联系[技术支持](/support/contact/)发出。 |
| 流式处理单位的最大吞吐量 | 1MB/秒* | 每个 SU 的最大吞吐量取决于方案。实际的吞吐量可能较低，并取决于查询复杂性和分区。可在[扩展 Azure 流分析作业以增加吞吐量](/documentation/articles/stream-analytics-scale-jobs/)一文中找到更多详细信息。 |
| SELECT 语句查询限制 | 每个查询 5 个输出 | 将来可能会增加此限制。 |
| SELECT 语句子查询限制 | 每个子查询 14 个聚合 | 将来可能会增加此限制。 |

<!---HONumber=69-->