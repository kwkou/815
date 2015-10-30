<properties
	pageTitle="Azure Batch 中的有效列表查询 | Windows Azure"
	description="了解如何减少列表中返回的 Azure Batch 项目数，以及如何减少针对每个项目返回的信息量。"
	services="批处理( batch )"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="Batch"
	ms.date="08/04/2015"
	wacn.date="10/30/2015"/>

# 高效的列表查询

下面的方法是几乎每个使用 Azure 批处理( Batch )的应用程序都需要执行且通常必须频繁执行的操作的示例：

- [ListTasks](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listtasks.aspx)
- [ListJobs](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx)
- [ListPools](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.listpools.aspx)
- [ListCertificates](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.certificateoperations.listcertificates.aspx)

监视是一种常用的操作；例如，确定池的容量和状态将需要对池中的所有计算节点 (VM) 执行查询。另一个示例是查询任务中是否存在某个作业，以便确定是否有任务仍在排队。有时需要种类繁多的数据，但有时则不需要考虑种类，只需对项目的总数或特定状态项目的总数进行计算。

请务必认识到，可能返回的项目数以及表示项目列表所需的数据大小都可能会很大。直接查询大量项目会导致大量响应，从而导致许多问题：

- 批处理( Batch ) API 响应在时间上可能会变得很慢。项目数越大，批处理( Batch )服务所需的查询时间越长。必须将大量项目划分为区块，因此可能必须由客户端库向服务发出多个服务 API 调用，以便获取整个列表的所有项目。
- 需要处理的项目越多，通过应用程序调用批处理( Batch )来完成 API 处理所需的时间也就越长。
- 由于项目数量更多且/或大小更大，应用程序调用批处理( Batch )时将耗用更多的内存。
- 项目数量更多且/或大小更大将导致网络流量增加。这将需要更长的时间来传输，而且可能导致对在批处理( Batch )帐户区域外传输的数据的网络计费增加，具体取决于应用程序体系结构。

批处理( Batch ) API 既可减少列表中返回的项目数，也可以减少针对每个项目返回的信息量。进行列表操作时，可以指定类型为 [DetailLevel](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.detaillevel.aspx) 的参数。DetailLevel 是一个抽象基类，而 [ODATADetailLevel](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.odatadetaillevel.aspx) 对象则实际上需要以参数的形式进行创建和传递。

所有 API 均适用以下原则：

- 每个属性名称都是一个将映射到对象属性的字符串。
- 所有属性名称均区分大小写，但属性值不区分大小写
- 日期/时间字符串可以采用两种格式中的一种，并且需要在前面加上 DateTime
	- W3CDTF（例如 creationTime gt DateTime’2011-05-08T08:49:37Z’）
	- RFC1123（例如 creationTime gt DateTime’Sun, 08 May 2011 08:49:37 GMT’）
- 布尔值字符串为“true”或“false”
- 如果指定了无效的属性或运算符，则会生成一个异常，并显示“400 (错误的请求)”内部异常。
- 也可将带有 Select 和 Expand 子句的 DetailLevel 参数传递到相应的“Get”方法，例如 PoolOperations.GetPool()。

ODataDetailLevel 对象有三个可以在构造函数中指定或直接设置的公共属性。这三个属性都是字符串：

- [FilterClause](#filter) – 进行筛选，可能会减少返回项的数目
- [SelectClause](#select) – 指定返回特定属性值，减小项目和响应的大小
- [ExpandClause](#expand) – 一次调用即返回所有需要的数据，不需多次调用

### <a id="filter"></a> FilterClause

可以通过筛选器字符串来降低返回项的数目。若要确保只返回必需的项，可以指定一个或多个属性值。下面是几个示例：只列出作业的运行中任务、只列出已做好运行任务准备的计算节点。

[FilterClause](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.odatadetaillevel.filterclause.aspx) 是一个包含一个或多个表达式的字符串，其中一个表达式包含属性名称、运算符和值。能够指定哪些属性取决于每个 API 调用，每个属性所支持的运算符也是这样。可以使用逻辑运算符“和”与“或”将多个表达式组合到一起。

例如，用于任务列表的筛选器可以是：

	startswith(name, 'MyTask') and (state eq 'Running')

### <a id="select"></a> SelectClause

可以通过使用 select 字符串限制为每个项返回的属性值。可以指定某个项的属性列表，然后就会只返回这些属性值。

[SelectClause](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.odatadetaillevel.selectclause.aspx) 是一个字符串，包含以逗号分隔的属性名称的列表。可以指定列表操作所返回的项中的所有属性。

	"name, state, stateTransitionTime"

### <a id="expand"></a> ExpandClause

可以使用 expand 字符串来减少 API 调用数。可以通过对整个列表进行 API 调用来获取每个列表项的更多详细信息，不需先获取列表，然后再针对列表中的每个项目进行调用。

[ExpandClause](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.odatadetaillevel.expandclause.aspx) 类似于 Select 子句，而 Expand 子句则控制是否在结果中返回特定数据。仅工作项列表、任务列表、池列表和作业列表支持 Expand 子句；该子句目前只支持统计信息。当所有属性都是必需属性时，将没有 select 子句，此时必须使用 expand 子句来获取统计信息。如果使用了 select 子句来获取属性的子集，则可在 select 子句中指定统计项，将 expand 子句保留为 null。

> [AZURE.NOTE]建议你始终使用筛选器和 select子句进行列表 API 调用，以确保获得最大效率并实现应用程序的最佳性能。

<!---HONumber=69-->