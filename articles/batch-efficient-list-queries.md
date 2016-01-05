<properties
	pageTitle="Azure 批处理( batch ) 中的有效列表查询 | Windows Azure"
	description="在查询 Azure 批处理( batch ) 实体（例如池、作业、任务和计算节点）时通过减少返回的数据量来提高性能。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="Batch"
	ms.date="10/12/2015"
	wacn.date="12/31/2015"/>
# 有效地查询 Azure 批处理 ( batch ) 服务

在本文中，你将了解在使用 [批处理 ( batch ) .NET][api_net] API 查询批处理 ( batch ) 服务以获取作业、任务、计算节点等项的列表时，如何减少返回的项数和数据量。

Azure 批处理 ( batch ) 属于大计算，在生产环境中，作业、任务和计算节点等实体的数目成千上万。因此，获取这些项的信息时，可能会生成大量的数据，这些数据在每次查询时都必须进行传输。限制每次查询时返回的项数和信息类型将会提高查询速度，因此也会提高应用程序的性能。

列出作业、任务、计算节点--这些是几乎每个使用 Azure 批处理( batch ) 的应用程序都会执行的操作示例，而且这些操作的执行频率很高。常见的用例是监视。例如，确定池的容量和状态需要查询该池中的所有计算节点。另一个示例是查询作业的任务，以便确定是否有任务仍在排队。

此 [批处理( batch ) .NET][api_net] API 代码段检索所有与作业关联的任务，以及这些任务的全部属性：

```
// Get a collection of all of the tasks and all of their properties for job-001
IPagedEnumerable<CloudTask> allTasks = batchClient.JobOperations.ListTasks("job-001");
```

不过，可以通过将 [ODATADetailLevel][odata] 提供给 [JobOperations.ListTasks][net_list_tasks] 方法，来执行效率要高得多的列表查询。此代码段仅返回已完成任务的 ID、命令行和计算节点信息属性：

```
// Configure an ODATADetailLevel specifying a subset of tasks and their properties to return
ODATADetailLevel detailLevel = new ODATADetailLevel();
detailLevel.FilterClause = "state eq 'completed'";
detailLevel.SelectClause = "id,commandLine,nodeInfo";

// Supply the ODATADetailLevel to the ListTasks method
IPagedEnumerable<CloudTask> completedTasks = batchClient.JobOperations.ListTasks("job-001", detailLevel);
```

在上面的示例方案中，如果作业中存在数以千计的任务，则通常情况下，第二个查询的结果的返回速度将远远快于第一个查询。有关通过批处理( batch ) .NET API 列出项时如何使用 ODATADetailLevel 的更多信息如下所示。

> [AZURE.IMPORTANT]
> 强烈建议你**始终**将 ODATADetailLevel 提供给 .NET API 列表调用，以确保最大程度地提高应用程序的效率和性能。指定详细程度有助于缩短批处理( batch ) 服务响应时间、提高网络利用率，以及最大程度减少客户端应用程序的内存使用量。

## 高效查询工具

[批处理 ( batch ) .NET][api_net] 和 [批处理 ( batch ) REST][api_rest] API 可以在执行列表查询时指定 *filter*、*select* 和 *expand* 字符串，因此能够减少列表中返回的项数以及针对每个查询返回的信息量。

- **filter** - *filter 字符串* 是一个表达式，用于减少返回的项数。例如，只列出作业的运行中任务，或者只列出已做好运行任务准备的计算节点。
  - filter 字符串包含一个或多个表达式，其中一个表达式包含属性名称、运算符和值。能够指定哪些属性取决于每个 API 调用类型，每个属性所支持的运算符也是这样。
  - 可以使用逻辑运算符 `and` 和 `or` 将多个表达式组合到一起。
  - 仅列出正在运行的呈现任务的 filter 字符串示例：`startswith(id, 'renderTask') and (state eq 'running')`
- **select** - *select 字符串* 用于限制为每个项返回的属性值。可以在 select 字符串中指定项的属性列表，然后系统就会只返回每个项中那些包含列表查询结果的属性值。
  - select 字符串包含逗号分隔的属性名称列表。可以指定列表操作所返回项的任意属性。
  - 示例 select 字符串（仅指定每个任务要返回的三项属性）：`id, state, stateTransitionTime`
- **expand** - *expand 字符串*用于减少获取特定信息所需的 API 调用数。可以通过对单个列表进行 API 调用来获取每个列表项的更多详细信息，不需先获取列表，然后再针对列表中的每个项进行调用。
  - 与 select 字符串类似，expand 字符串用于控制是否允许某些数据包括在列表查询结果中。
  - 在列出作业、作业计划、任务和池时，仅支持 expand 字符串，而且目前仅支持统计信息。
  - 示例 expand 字符串（指定每个项都应返回统计信息）：`stats`
  - 当所有属性都是必需属性且没有指定 select 字符串时，*必须* 使用 expand 字符串来获取统计信息。如果使用了 select 字符串来获取属性的子集，则可在 select 字符串中指定 `stats`，不需指定 expand 字符串。

> [AZURE.NOTE]构造这三种查询字符串类型（filter、select、expand）中的任意一种类型时，必须确保属性名称和大小写与其 REST API 元素的对应项相匹配。例如，在使用 .NET [CloudTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask) 时，你必须指定 **state** 而非 **State**，即使 .NET 属性为 [CloudTask.State](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.state)。请参阅下表中 .NET 和 REST API 之间的属性映射。

### Filter、select 和 expand 字符串规范

- 在 filter、select 和 expand 字符串中指定的属性与 [Batch REST][api_rest] API 中出现的属性名称等同--使用 [Batch .NET][api_net] 库时也是如此。
- 所有属性名称均区分大小写，但属性值不区分大小写
- 日期/时间字符串可以采用两种格式中的一种，并且必须在前面加上 `DateTime`
  - W3CDTF 格式示例：`creationTime gt DateTime'2011-05-08T08:49:37Z'`
  - RFC1123 格式示例：`creationTime gt DateTime'Sun, 08 May 2011 08:49:37 GMT'`
- 布尔值字符串为 `true` 或 `false`
- 如果指定了无效的属性或运算符，则会导致 `400 (Bad Request)` 错误

## 在 批处理( batch ) .NET 中进行高效查询

在 [批处理( batch ) .NET][api_net] API 中，将通过 [ODATADetailLevel][odata] 来提供 filter、select 和 expand 字符串以列出相应操作。一个 ODataDetailLevel 对象有三个公共字符串属性，这些属性可以在构造函数中指定，也可以直接设置，然后，系统会将此对象作为参数传递给不同的列表操作，例如 [ListPools][net_list_pools]、[ListJobs][net_list_jobs] 和 [ListTasks][net_list_tasks]。

- [ODATADetailLevel.FilterClause][odata_filter] – 限制返回的项数
- [ODATADetailLevel.SelectClause][odata_select] – 指定随每个项返回的部分属性值
- [ODATADetailLevel.ExpandClause][odata_expand] – 在单个 API 调用中检索项数据，不必针对每个项发出调用

以下代码段使用 Batch .NET API 对 Batch 服务进行有效的查询，查询其中是否存在特定池集的统计信息。在此方案中，Batch 用户既有测试池又有生产池，测试池 ID 具有“test”前缀，生产池 ID 具有“prod”前缀。在代码段中，*myBatchClient* 是正确初始化的 [BatchClient](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient) 实例。

	// First we need an ODATADetailLevel instance on which to set the expand, filter, and select
	// clause strings
	ODATADetailLevel detailLevel = new ODATADetailLevel();

	// We want to pull only the "test" pools, so we limit the number of items returned by using a
	// FilterClause and specifying that the pool IDs must start with "test"
	detailLevel.FilterClause = "startswith(id, 'test')";

	// To further limit the data that crosses the wire, configure the SelectClause to limit the
	// properties returned on each CloudPool object to only CloudPool.Id and CloudPool.Statistics
	detailLevel.SelectClause = "id, stats";

	// Specify the ExpandClause so that the .NET API pulls the statistics for the CloudPools in a single
	// underlying REST API call. Note that we use the pool's REST API element name "stats" here as opposed
	// to "Statistics" as it appears in the .NET API (CloudPool.Statistics)
	detailLevel.ExpandClause = "stats";

	// Now get our collection of pools, minimizing the amount of data returned by specifying the
	// detail level we configured above
	List<CloudPool> testPools = await myBatchClient.PoolOperations.ListPools(detailLevel).ToListAsync();

> [AZURE.TIP]使用 Select 和 Expand 子句配置的 [ODATADetailLevel][odata] 实例也可以传递给相应的 Get 方法（例如 [PoolOperations.GetPool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getpool.aspx)），以便限制返回的数据量。

## 批处理 ( batch ) REST 到 .NET API 映射

filter、select 和 expand 字符串中的属性名称*必须*反映其 REST API 对应项，不管是名称本身还是大小写。下表提供了 .NET 和 REST API 的对应项之间的映射。

### filter 字符串的映射

- **.NET 列表方法** - 此列中的每个 .NET API 方法都接受 [ODATADetailLevel][odata] 对象作为参数。
- **REST 列表请求** - 此列中的每个 REST API 页面都包含一个表，该表指定了 *filter* 字符串中允许的属性和操作。在构造 [ODATADetailLevel.FilterClause][odata_filter] 字符串时，你会使用这些属性名称和操作。

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

- **批处理( batch ) .NET 类型** - 批处理( batch ) .NET API 类型
- **REST API 实体** - 此列中的每一页都包含一个或多个表，其中列出了类型的 REST API 属性名称。在构造 *select* 字符串时使用这些属性名称。在构造 [ODATADetailLevel.SelectClause][odata_select] 字符串时，你会使用这些相同的属性名称。

| 批处理( batch ) .NET 类型 | REST API 实体 |
|---|---|
| [证书][net_cert] | [获取有关证书的信息][rest_get_cert] |
| [CloudJob][net_job] | [获取有关作业的信息][rest_get_job] |
| [CloudJobSchedule][net_schedule] | [获取有关作业计划的信息][rest_get_schedule] |
| [ComputeNode][net_node] | [获取有关节点的信息][rest_get_node] |
| [CloudPool][net_pool] | [获取有关池的信息][rest_get_pool] |
| [CloudTask][net_task] | [获取有关任务的信息][rest_get_task] |

### 示例：构造 filter 字符串

针对 [ODATADetailLevel.FilterClause][odata_filter] 构造 filter 字符串时，请查阅上表，在 *filter 字符串的映射*下找到与你所希望执行的列表操作相对应的 REST API 文档页。你会在该页第一个多行表中找到可筛选属性及其支持的运算符。例如，如果你希望检索其退出代码不为零的所有任务，则可查看[列出与作业相关联的任务][rest_list_tasks]上的此行，此行指定了相应的属性字符串以及允许的运算符：

| 属性 | 允许的操作 | 类型 |
| :--- | :--- | :--- |
| `executionInfo/exitCode` | `eq, ge, gt, le , lt` | `Int` |

因此，若要列出退出代码不为零的所有任务，可使用以下 filter 字符串：

`(executionInfo/exitCode lt 0) or (executionInfo/exitCode gt 0)`

### 示例：构造 select 字符串

若要构造 [ODATADetailLevel.SelectClause][odata_select]，请查阅上表，在 *select 字符串的映射*下导航到与所列实体类型相对应的 REST API 页。你会在该页第一个多行表中找到可选择属性及其支持的运算符。例如，如果你希望仅检索列表中每个任务的 ID 和命令行，则可在[获取有关任务的信息][rest_get_task]的相应表中找到这些行：

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `id` | `String` | `The id of the task.` |
| `commandLine` | `String` | `The command line of the task.` |

因此，若要只包括每个所列任务的 ID 和命令行，可使用以下 select 字符串：

`id, commandLine`

## 后续步骤

请查看 GitHub 上的 [EfficientListQueries][efficient_query_sample] 示例项目，了解列表查询如何有效地影响应用程序的性能。此 C# 控制台应用程序创建大量的任务并将其添加到作业，然后使用不同的 [ODATADetailLevel][odata] 规范来查询批处理( batch ) 服务，所显示的输出如下所示：

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

<!---HONumber=Mooncake_1221_2015-->