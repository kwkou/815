<properties
	pageTitle="Azure Batch 中的有效列表查询 | Azure"
	description="在查询 Azure Batch 实体（例如池、作业、任务和计算节点）时通过减少返回的数据量来提高性能。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />
	
<tags
	ms.service="batch"
	ms.date="04/21/2016"
	wacn.date="06/06/2016"/>
# 有效地查询 Azure 批处理 ( batch ) 服务

在本文中，你将了解如何通过减少查询 Batch 服务（使用 [Batch .NET][api_net] 库）时返回的数据量来提高 Azure Batch 应用程序的性能。

Azure Batch 提供大型计算功能 -- 在生产环境中，作业、任务和计算节点等实体的数目成千上万。因此，获取这些项的信息时，可能会生成大量的数据，这些数据在每次查询时都必须从服务传输到应用程序。通过限制每次查询时返回的项数和信息类型，可以提高查询速度，因此也会提高应用程序的性能。

几乎所有使用 Azure Batch 的应用程序都会执行某类监视操作或其他查询 Batch 服务的操作（通常按固定的时间间隔）。例如，为了确定池的容量和状态，必须查询池中的每个节点。为了确定某个作业是否有任务仍处于排队状态，必须查询作业中的每个任务。本文介绍如何以最有效的方式执行这些类型的查询。

此 [Batch .NET][api_net] API 代码段检索每个与作业关联的任务，以及这些任务的全部 属性：

csharp

		// Get a collection of all of the tasks and all of their properties for job-001
		IPagedEnumerable<CloudTask> allTasks = batchClient.JobOperations.ListTasks("job-001");


但是，可以执行高效得多的列表查询。在 [JobOperations.ListTasks][net_list_tasks] 方法中提供 [ODATADetailLevel][odata] 对象即可实现此目的。此代码段仅返回已完成任务的 ID、命令行和计算节点信息属性：

csharp

		// Configure an ODATADetailLevel specifying a subset of tasks and their properties to return
		ODATADetailLevel detailLevel = new ODATADetailLevel();
		detailLevel.FilterClause = "state eq 'completed'";
		detailLevel.SelectClause = "id,commandLine,nodeInfo";
		// Supply the ODATADetailLevel to the ListTasks method
		IPagedEnumerable<CloudTask> completedTasks = batchClient.JobOperations.ListTasks("job-001", detailLevel);
		

在上面的示例方案中，如果作业中存在数以千计的任务，则通常情况下，第二个查询的结果的返回速度将远远快于第一个查询。下面提供了有关使用 Batch .NET API 列出项时使用 ODATADetailLevel 的详细信息。

> [AZURE.IMPORTANT]
强烈建议你*始终* 将 ODATADetailLevel 对象提供给 .NET API 列表调用，以确保最大程度地提高应用程序的效率和性能。指定详细程度有助于缩短 Batch 服务响应时间、提高网络利用率，以及最大程度减少客户端应用程序的内存使用量。

## 高效查询工具

[Batch .NET][api_net] 和 [Batch REST][api_rest] API 可以减少列表中返回的项数以及针对每个查询返回的信息量。在执行列表查询时可以通过指定 **filter**、**select** 和 **expand** 字符串来实现此目的。

### 筛选器
filter 字符串是一个表达式，用于减少返回的项数。例如，只列出作业的运行中任务，或者只列出已做好运行任务准备的计算节点。

- filter 字符串包含一个或多个表达式，其中一个表达式包含属性名称、运算符和值。能够指定哪些属性取决于你所查询的每个实体类型，每个属性所支持的运算符也是这样。
- 可以使用逻辑运算符 `and` 和 `or` 将多个表达式组合到一起。
- 此示例性 filter 字符串仅列出正在运行的“呈现”任务：`(state eq 'running') and startswith(id, 'renderTask')`。

### 选择
select 字符串用于限制为每个项返回的属性值。你可以指定属性名称的列表，仅在查询结果中返回项目的这些属性值。

- select 字符串包含逗号分隔的属性名称列表。你可以指定所查询实体类型的任意属性。
- 此示例性的 select 字符串指定每个任务只应返回三项属性值：`id, state, stateTransitionTime`。

### 展开
expand 字符串用于减少获取特定信息所需的 API 调用数。使用 expand 字符串时，单次 API 调用可以获取每个项目的更多信息。你不必首先获取实体的列表，然后请求列表中每个项目的信息。你可以使用 expand 字符串通过单次 API 调用获取相同的信息。API 调用数较少意味着性能较高。

- 与 select 字符串类似，expand 字符串用于控制是否允许某些数据包括在列表查询结果中。
- expand 字符串只有在列出作业、作业计划、任务和池中使用时才受支持。目前仅支持统计信息。
- 当所有属性都是必需属性且没有指定 select 字符串时，“必须” 使用 expand 字符串来获取统计信息。如果使用了 select 字符串来获取属性的子集，则可在 select 字符串中指定 `stats`，不需指定 expand 字符串。
- 此示例性 expand 字符串指定列表中的每个项都应返回统计信息：`stats`。

> [AZURE.NOTE] 构造这三种查询字符串类型（filter、select 和 expand）中的任意一种类型时，必须确保属性名称和大小写与其 REST API 元素的对应项相匹配。例如，在使用 .NET [CloudTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask) 类时，必须指定 **state** 而非 **State**，即使 .NET 属性为 [CloudTask.State](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.state)。请参阅下表中 .NET 和 REST API 之间的属性映射。

### Filter、select 和 expand 字符串规范

- 在 filter、select 和 expand 字符串中指定的属性与 [Batch REST][api_rest] API 中出现的属性名称等同--即使使用 [Batch .NET][api_net] 库也是如此。
- 所有属性名称均区分大小写，但属性值不区分大小写。
- 日期/时间字符串可以采用两种格式中的一种，并且必须在前面加上 `DateTime`。
  - W3C-DTF 格式示例：`creationTime gt DateTime'2011-05-08T08:49:37Z'`。
  - RFC 1123 格式示例：`creationTime gt DateTime'Sun, 08 May 2011 08:49:37 GMT'`。
- 布尔值字符串为 `true` 或 `false`。
- 如果指定了无效的属性或运算符，则会导致 `400 (Bad Request)` 错误。

## 在 Batch .NET 中进行高效查询

在 [Batch .NET][api_net] API 中，将通过 [ODATADetailLevel][odata] 类来提供 filter、select 和 expand 字符串以列出相应操作。ODataDetailLevel 类有三个公共字符串属性，这些属性可以在构造函数中指定，也可以直接在对象上设置。然后，你可以将 ODataDetailLevel 对象作为参数传递给不同的列表操作，例如 [ListPools][net_list_pools]、[ListJobs][net_list_jobs] 和 [ListTasks][net_list_tasks]。

- [ODATADetailLevel][odata].[FilterClause][odata_filter]：限制返回的项数。
- [ODATADetailLevel][odata].[SelectClause][odata_select]：指定随每个项返回的属性值。
- [ODATADetailLevel][odata].[ExpandClause][odata_expand]：通过单个 API 调用检索所有项的数据，不必针对每个项分别进行调用。

以下代码段使用 Batch .NET API 对 Batch 服务进行有效的查询，查询其中是否存在特定池集的统计信息。在此方案中，Batch 用户既有测试池又有生产池。测试池 ID 具有“test”前缀，生产池 ID 具有“prod”前缀。在代码段中，*myBatchClient* 是正确初始化的 [BatchClient](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient) 类实例。

csharp
		// First we need an ODATADetailLevel instance on which to set the expand, filter, and select
		// clause strings
		ODATADetailLevel detailLevel = new ODATADetailLevel();
		// We want to pull only the "test" pools, so we limit the number of items returned by using a
		// FilterClause and specifying that the pool IDs must start with "test"
		detailLevel.FilterClause = "startswith(id, 'test')";
		// To further limit the data that crosses the wire, configure the SelectClause to limit the
		// properties that are returned on each CloudPool object to only CloudPool.Id and CloudPool.Statistics
		detailLevel.SelectClause = "id, stats";
		// Specify the ExpandClause so that the .NET API pulls the statistics for the CloudPools in a single
		// underlying REST API call. Note that we use the pool's REST API element name "stats" here as opposed
		// to "Statistics" as it appears in the .NET API (CloudPool.Statistics)
		detailLevel.ExpandClause = "stats";
		// Now get our collection of pools, minimizing the amount of data that is returned by specifying the
		// detail level that we configured above
		List<CloudPool> testPools = await myBatchClient.PoolOperations.ListPools(detailLevel).ToListAsync();


> [AZURE.TIP] 使用 Select 和 Expand 子句配置的 [ODATADetailLevel][odata] 实例也可以传递给相应的 Get 方法（例如 [PoolOperations.GetPool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getpool.aspx)），以便限制返回的数据量。

## Batch REST 到 .NET API 映射

filter、select 和 expand 字符串中的属性名称“必须”反映其 REST API 对应项，不管是名称本身还是大小写。下表提供了 .NET 和 REST API 的对应项之间的映射。

### filter 字符串的映射

- **.NET 列表方法**：此列中的每个 .NET API 方法都接受 [ODATADetailLevel][odata] 对象作为参数。
- **REST 列表请求**：此列中的每个 REST API 页面都包含一个表，该表指定了 *filter* 字符串中允许的属性和操作。在构造 [ODATADetailLevel.FilterClause][odata_filter] 字符串时，你会使用这些属性名称和操作。

| .NET 列表方法 | REST 列表请求 |
|---|---|
| [CertificateOperations.ListCertificates][net_list_certs] | [列出帐户中的证书][rest_list_certs]
| [CloudTask.ListNodeFiles][net_list_task_files] | [列出与任务关联的文件][rest_list_task_files]
| [JobOperations.ListJobPreparationAndReleaseTaskStatus][net_list_jobprep_status] | [列出作业准备状态以及作业的作业版本任务][rest_list_jobprep_status]
| [JobOperations.ListJobs][net_list_jobs] | [列出帐户中的作业][rest_list_jobs]
| [JobOperations.ListNodeFiles][net_list_nodefiles] | [列出节点上的文件][rest_list_nodefiles]
| [JobOperations.ListTasks][net_list_tasks] | [列出与作业关联的任务][rest_list_tasks]
| [JobScheduleOperations.ListJobSchedules][net_list_job_schedules] | [列出帐户中的作业计划][rest_list_job_schedules]
| [JobScheduleOperations.ListJobs][net_list_schedule_jobs] | [列出与作业计划关联的作业][rest_list_schedule_jobs]
| [PoolOperations.ListComputeNodes][net_list_compute_nodes] | [列出池中的计算节点][rest_list_compute_nodes]
| [PoolOperations.ListPools][net_list_pools] | [列出帐户中的池][rest_list_pools]

### select 字符串的映射

- **Batch .NET 类型**：Batch .NET API 类型。
- **REST API 实体**：此列中的每一页都包含一个或多个表，其中列出了类型的 REST API 属性名称。在构造 *select* 字符串时使用这些属性名称。在构造 [ODATADetailLevel.SelectClause][odata_select] 字符串时，你会使用这些相同的属性名称。

| Batch .NET 类型 | REST API 实体 |
|---|---|
| [证书][net_cert] | [获取有关证书的信息][rest_get_cert] |
| [CloudJob][net_job] | [获取有关作业的信息][rest_get_job] |
| [CloudJobSchedule][net_schedule] | [获取有关作业计划的信息][rest_get_schedule] |
| [ComputeNode][net_node] | [获取有关节点的信息][rest_get_node] |
| [CloudPool][net_pool] | [获取有关池的信息][rest_get_pool] |
| [CloudTask][net_task] | [获取有关任务的信息][rest_get_task] |

### 示例：构造 filter 字符串

针对 [ODATADetailLevel.FilterClause][odata_filter] 构造 filter 字符串时，请查阅上表，在“filter 字符串的映射”下找到与你所希望执行的列表操作相对应的 REST API 文档页。你会在该页第一个多行表中找到可筛选属性及其支持的运算符。例如，如果你希望检索其退出代码不为零的所有任务，则可查看[列出与作业相关联的任务][rest_list_tasks]上的此行，此行指定了相应的属性字符串以及允许的运算符：

| 属性 | 允许的操作 | 类型 |
| :--- | :--- | :--- |
| `executionInfo/exitCode` | `eq, ge, gt, le , lt` | `Int` |

因此，若要列出退出代码不为零的所有任务，可使用以下 filter 字符串：

`(executionInfo/exitCode lt 0) or (executionInfo/exitCode gt 0)`

### 示例：构造 select 字符串

若要构造 [ODATADetailLevel.SelectClause][odata_select]，请查阅上表，在“select 字符串的映射”下导航到与所列实体类型相对应的 REST API 页。你会在该页第一个多行表中找到可选择属性及其支持的运算符。例如，如果你希望仅检索列表中每个任务的 ID 和命令行，则可在[获取有关任务的信息][rest_get_task]的相应表中找到这些行：

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `id` | `String` | `The ID of the task.` |
| `commandLine` | `String` | `The command line of the task.` |

因此，若要只包括每个所列任务的 ID 和命令行，可使用以下 select 字符串：

`id, commandLine`

## 后续步骤

### 高效列表查询代码示例

请查看 GitHub 上的 [EfficientListQueries][efficient_query_sample] 示例项目，了解列表查询如何有效地影响应用程序的性能。此 C# 控制台应用程序创建大量的任务并将其添加到作业。然后，它对 [JobOperations.ListTasks][net_list_tasks] 方法进行多次调用，并传递配置了不同属性值的 [ODATADetailLevel][odata] 对象，以改变要返回的数据量。生成的输出如下所示：

		Adding 5000 tasks to job jobEffQuery...
		5000 tasks added in 00:00:47.3467587, hit ENTER to query tasks...
		4943 tasks retrieved in 00:00:04.3408081 (ExpandClause:  | FilterClause: state eq 'active' | SelectClause: id,state)
		0 tasks retrieved in 00:00:00.2662920 (ExpandClause:  | FilterClause: state eq 'running' | SelectClause: id,state)
		59 tasks retrieved in 00:00:00.3337760 (ExpandClause:  | FilterClause: state eq 'completed' | SelectClause: id,state)
		5000 tasks retrieved in 00:00:04.1429881 (ExpandClause:  | FilterClause:  | SelectClause: id,state)
		5000 tasks retrieved in 00:00:15.1016127 (ExpandClause:  | FilterClause:  | SelectClause: id,state,environmentSettings)
		5000 tasks retrieved in 00:00:17.0548145 (ExpandClause: stats | FilterClause:  | SelectClause: )
		Sample complete, hit ENTER to continue...

如所用时间信息中所示，限制返回的属性和项数可以大大缩短查询响应时间。你可以在 GitHub 的 [azure-batch-samples][github_samples] 存储库中查找此项目和其他示例项目。

### Batch 论坛

MSDN 上的 [Azure Batch 论坛][forum]是探讨 Batch 服务以及咨询其相关问题的不错场所。欢迎前往浏览这些帮忙解决“棘手问题”的贴子，并发布你在构建 Batch 解决方案时遇到的问题。


[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_listjobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[efficient_query_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries
[github_samples]: https://github.com/Azure/azure-batch-samples
[odata]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.aspx
[odata_ctor]: https://msdn.microsoft.com/library/azure/dn866178.aspx
[odata_expand]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.expandclause.aspx
[odata_filter]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.filterclause.aspx
[odata_select]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.selectclause.aspx

[net_list_certs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificateoperations.listcertificates.aspx
[net_list_compute_nodes]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listcomputenodes.aspx
[net_list_job_schedules]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobschedules.aspx
[net_list_jobprep_status]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobpreparationandreleasetaskstatus.aspx
[net_list_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[net_list_nodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listnodefiles.aspx
[net_list_pools]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listpools.aspx
[net_list_schedule_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobs.aspx
[net_list_task_files]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[net_list_tasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listtasks.aspx

[rest_list_certs]: https://msdn.microsoft.com/library/azure/dn820154.aspx
[rest_list_compute_nodes]: https://msdn.microsoft.com/library/azure/dn820159.aspx
[rest_list_job_schedules]: https://msdn.microsoft.com/library/azure/mt282174.aspx
[rest_list_jobprep_status]: https://msdn.microsoft.com/library/azure/mt282170.aspx
[rest_list_jobs]: https://msdn.microsoft.com/library/azure/dn820117.aspx
[rest_list_nodefiles]: https://msdn.microsoft.com/library/azure/dn820151.aspx
[rest_list_pools]: https://msdn.microsoft.com/library/azure/dn820101.aspx
[rest_list_schedule_jobs]: https://msdn.microsoft.com/library/azure/mt282169.aspx
[rest_list_task_files]: https://msdn.microsoft.com/library/azure/dn820142.aspx
[rest_list_tasks]: https://msdn.microsoft.com/library/azure/dn820187.aspx

[rest_get_cert]: https://msdn.microsoft.com/library/azure/dn820176.aspx
[rest_get_job]: https://msdn.microsoft.com/library/azure/dn820106.aspx
[rest_get_node]: https://msdn.microsoft.com/library/azure/dn820168.aspx
[rest_get_pool]: https://msdn.microsoft.com/library/azure/dn820165.aspx
[rest_get_schedule]: https://msdn.microsoft.com/library/azure/mt282171.aspx
[rest_get_task]: https://msdn.microsoft.com/library/azure/dn820133.aspx

[net_cert]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificate.aspx
[net_job]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_node]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_schedule]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjobschedule.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx

<!---HONumber=Mooncake_0530_2016-->