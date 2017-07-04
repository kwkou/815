数据工厂是一种多租户服务，它实施以下默认限制以确保客户订阅不会受到彼此工作负荷的影响。你可以联系支持人员，将订阅的大多数限制调整到其最大限制。

**资源** | **默认限制** | **最大限制**
-------- | ------------- | -------------
数据工厂中的管道 | 100 | 2500
数据工厂中的数据集 | 500 | 5000
每个数据集的并发切片数 | 10 | 10
每个管道的每个对象字节数 <sup>1</sup> | 200 KB | 2000 KB
数据集和链接服务对象的每个对象字节数 <sup>1</sup> | 30 KB | 2000 KB
每个对象的字段数 | 100 | [联系支持人员](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
每个字段名称或标识符的字节数 | 2 KB | [联系支持人员](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
每个字段的字节数 | 30 KB | [联系支持人员](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
订阅中的 HDInsight 按需群集核心数 <sup>2</sup> | 48 | [联系支持人员](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
管道活动运行的重试次数 | 1000 | MaxInt（32 位）

<sup>1</sup> 管道、数据集和链接服务对象代表工作负荷的逻辑组。对这些对象的限制与可以使用 Azure 数据工厂服务移动或处理的数据量无关。可以缩放数据工厂以处理 PB 量级的数据。

<sup>2</sup> 按需 HDInsight 核心并未分配在包含数据工厂的订阅中。因此，上述限制为数据工厂针对按需 HDInsight 核心所强制的核心限制，不同于 Azure 订阅关联的核心限制。


**资源** | **默认下限** | **最小限制**
-------- | ------------------- | -------------
计划间隔 | 15 分钟 | 5 分钟
重试尝试之间的间隔 | 1 秒 | 1 秒
重试超时值 | 1 秒 | 1 秒


### Web 服务调用限制

Azure 资源管理器限制 API 调用。你可以根据 <!--[-->Azure 资源管理器 API 限制<!--](/documentation/articles/azure-subscription-service-limits/#resource-group-limits)-->中规定的频率执行 API 调用。

<!---HONumber=71-->