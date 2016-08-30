<properties 
	pageTitle="如何监视 Azure Redis 缓存 | Azure" 
	description="了解如何监视 Azure Redis 缓存实例的运行状况和性能" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="06/15/2016"
	wacn.date="08/29/2016"/>

# 如何监视 Azure Redis 缓存

Azure Redis 缓存提供了用于监视缓存实例的几个选项。你可以查看度量值、将度量值图表固定到启动面板、自定义监视图表的日期和时间范围、在图表中添加和删除度量值，以及设置符合特定条件时发出的警报。借助这些工具，你可以监视 Azure Redis 缓存实例的运行状况，以及管理缓存应用程序。

启用缓存诊断后，将约每 30 秒收集一次 Azure Redis 缓存实例的度量值并进行存储，以便将它们显示在度量值图表中并根据警报规则进行评估。

使用 Redis [INFO](http://redis.io/commands/info) 命令收集缓存度量值。有关用于每个缓存度量值的不同 INFO 值的详细信息，请参阅[可用度量值和报告间隔](#available-metrics-and-reporting-intervals)。

若要查看缓存度量值，请[浏览](/documentation/articles/cache-configure/)到 [Azure 门户预览](https://portal.azure.cn)中的缓存实例。Azure Redis 缓存实例的度量值可在“Redis 指标”边栏选项卡上访问。

![Redis 指标][redis-cache-redis-metrics-blade]

>[AZURE.IMPORTANT] 如果“Redis 指标”边栏选项卡中显示以下消息，请按照[启用缓存诊断](#enable-cache-diagnostics)部分中的步骤启用缓存诊断。
>
>`Monitoring may not be enabled. Click here to turn on Diagnostics.`

“Redis 指标”边栏选项卡上有显示缓存度量值的“监视”图表。通过添加或删除度量值以及更改报告间隔可以自定义每个图表。若要查看和配置操作和警报，“Redis 缓存”边栏选项卡上有“操作”部分，该部分显示缓存“事件”和“警报规则”。

## <a name="enable-cache-diagnostics"></a>启用缓存诊断

>[AZURE.NOTE] 想要在 Azure 中国启用缓存诊断，必须得使用命令行设置 `rdb-storage-connection-string`。

Azure Redis 缓存提供如下功能：将诊断数据存储在存储帐户中，以便你可以使用希望访问的任何工具以及直接处理数据。为了收集、存储缓存诊断并将其显示在 Azure 门户预览中，必须配置一个存储帐户。同一区域和订阅中的缓存共享相同的诊断存储帐户，并且在配置更改时适用于该区域订阅中的所有缓存。

若要启用和配置缓存诊断，请导航到缓存实例的“Redis 缓存”边栏选项卡。如果尚未启用诊断，则将显示一条消息，而不会显示诊断图。

![启用缓存诊断][redis-cache-enable-diagnostics]

单击该消息以显示“度量值”边栏选项卡，然后单击“诊断设置”，以启用和配置缓存服务实例的诊断设置。

![诊断设置][redis-cache-diagnostic-settings]

![配置诊断][redis-cache-configure-diagnostics]

单击“打开”按钮，启用缓存诊断并显示诊断配置。

单击“存储帐户”右侧的箭头，选择一个用于保存诊断数据的存储帐户。为了获得最佳性能，请在与你的缓存相同的区域中选择一个存储帐户。

诊断设置配置完成后，单击“保存”以保存配置。注意，更改可能需要几分钟才能生效。

>[AZURE.IMPORTANT] 同一区域和订阅中的缓存共享相同的诊断存储设置，并且在配置更改（诊断启用/禁用或更改存储帐户）时适用于该区域订阅中的所有缓存。

若要查看存储的度量值，请检查名称开头为 `WADMetrics` 的存储帐户中的表。有关访问 Azure 门户预览外部的存储度量值的详细信息，请参阅[访问 Redis 缓存监视数据](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)示例。

>[AZURE.NOTE] Azure 门户预览仅显示存储在所选存储帐户中的度量值。如果更改存储帐户，以前配置的存储帐户中的数据仍可供下载，但并不会显示在 Azure 门户预览中。

## <a name="available-metrics-and-reporting-intervals"></a>可用度量值和报告间隔

缓存度量值将使用多个报告间隔进行报告，其中包括“前一小时”、“今天”、“前一周”和“自定义”。每个度量值图表的“度量值”边栏选项卡将在图表中显示每个度量值的平均值、最小值和最大值，并且一些度量值将显示总报告间隔。

每个度量值均包含两个版本。一个度量值测量整个缓存的性能以及使用[群集](/documentation/articles/cache-how-to-premium-clustering/)的缓存的性能，名称中包含 `(Shard 0-9)` 的另一个度量值版本则测量缓存中单个分片的性能。例如，如果缓存有 4 个分片，`Cache Hits` 就是整个缓存的命中总数，而 `Cache Hits (Shard 3)` 就只是该缓存分片的命中数。

>[AZURE.NOTE] 即使在缓存处于空闲状态，且没有连接的活动客户端应用程序时，也可能会看到一些缓存活动，例如，连接的客户端、内存使用率以及正在执行的操作。Azure Redis 缓存实例的操作期间，此活动是正常情况。

| 度量值 | 说明 |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 缓存命中数 | 在指定的报告间隔期间，成功的键查找次数。此值将映射到 Redis [INFO](http://redis.io/commands/info) 命令中的 `keyspace_hits`。 |
| 缓存未命中数 | 在指定的报告间隔期间，失败的键查找次数。此值将映射到 Redis INFO 命令中的 `keyspace_misses`。缓存未命中并不一定意味着缓存出现了问题。例如，在使用缓存端编程模式时，应用程序会首先查找缓存中的项。如果该项不存在（缓存未命中），则将从数据库中检索该项并将其添加到下一次缓存中。对于缓存端编程模式，缓存未命中是正常行为。如果缓存未命中数大于预期值，请检查从缓存中填充并读取的应用程序逻辑。如果由于内存压力而导致项目从缓存中逐出，则可能存在一些缓存未命中的情况，但度量值 `Used Memory` 或 `Evicted Keys` 可以更好的监视内存压力。 |
| 连接的客户端数 | 指定的报告间隔期间，客间户端与缓存的连接数。此值将映射到 Redis INFO 命令中的 `connected_clients`。一旦达到了[连接限制](/documentation/articles/cache-configure/#default-redis-server-configuration)，则对缓存的后续连接尝试将失败。注意，即使没有任何活动的客户端应用程序，由于内部进程和连接，仍可能存在一些连接的客户端的实例。 |
| 逐出的密钥数 | 由于 `maxmemory` 限制，指定的报告间隔期间从缓存中逐出的项目数。此值将映射到 Redis INFO 命令中的 `evicted_keys`。 |
| 过期的密钥数 | 指定的报告间隔期间，缓存中过期的项目数。此值将映射到 Redis INFO 命令中的 `expired_keys`。 |
| 获取数 | 指定的报告间隔期间，缓存中的获取操作数。此值是以下 Redis INFO 所有命令中的值的总和：`cmdstat_get`、`cmdstat_hget`、`cmdstat_hgetall`、`cmdstat_hmget`、`cmdstat_mget`、`cmdstat_getbit` 和 `cmdstat_getrange`，并且等效于报告间隔期间缓存命中和未命中数的总和。 |
| Redis 服务器负载 | Redis 服务器忙于处理消息并且非空闲等待消息的周期百分比。如果此计数器达到 100，则意味着 Redis 服务器已达到性能上限并且 CPU 无法更快地工作。如果看到高 Redis 服务器负载，则将在客户端看到超时异常。在这种情况下，你应该考虑将数据扩大或分区到多个缓存。 |
| 设置数 | 指定的报告间隔期间，对缓存的设置操作数。此值是以下 Redis INFO 所有命令中的值的总和：`cmdstat_set`、`cmdstat_hset`、`cmdstat_hmset`、`cmdstat_hsetnx`、`cmdstat_lset`、`cmdstat_mset`、`cmdstat_msetnx`、`cmdstat_setbit`、`cmdstat_setex`、`cmdstat_setrange` 和 `cmdstat_setnx`。 |
| 总操作数 | 指定的报告间隔期间，由缓存服务器处理的命令总数。此值将映射到 Redis INFO 命令中的 `total_commands_processed`。注意，当 Azure Redis 缓存纯粹用于发布/订阅时，将不存在 `Cache Hits`、`Cache Misses`、`Gets` 或 `Sets` 的度量值，但存在 `Total Operations` 度量值，该度量值反映发布/订阅操作的缓存使用情况。 |
| 已用内存 | 在指定报告间隔期间，缓存中的键/值对所用的缓存内存量（以 MB 为单位）。此值将映射到 Redis INFO 命令中的 `used_memory`。这不包括元数据或碎片。 |
| 已用内存 RSS | 指定报告间隔期间所用的缓存内存量（以 MB 为单位），包括碎片和元数据。此值将映射到 Redis INFO 命令中的 `used_memory_rss`。 |
| CPU | 指定报告间隔期间，Azure Redis 缓存服务器的 CPU 使用率（以百分比表示）。此值映射到操作系统 `\Processor(_Total)\% Processor Time` 性能计数器。 |
| 缓存读取量 | 指定报告间隔期间，从缓存中读取的数据量，以每秒兆字节数（MB/秒）为单位。此值来源于支持虚拟机的网络接口卡，该虚拟机托管缓存，但并不特定于 Redis。**此值对应于该缓存使用的网络带宽。如果你要针对服务器端网络带宽限制设置警报，则可使用此 `Cache Read` 计数器来创建这些警报。请参阅[此表](/documentation/articles/cache-faq/#cache-performance)，了解各种缓存定价层和大小所遵循的带宽限制。** |
| 缓存写入量 | 指定报告间隔期间，写入缓存中的数据量，以每秒兆字节数（MB/秒）为单位。此值来源于支持虚拟机的网络接口卡，该虚拟机托管缓存，但并不特定于 Redis。此值对应于从客户端发送到缓存的数据的网络带宽。 |


## <a name="how-to-view-metrics-and-customize-charts"></a>如何查看度量值和自定义图表

可以在“Redis 指标”边栏选项卡上查看缓存的度量值概述。若要访问“Redis 指标”边栏选项卡，请选择“所有设置”>“Redis 指标”。

![Redis 指标][redis-cache-redis-metrics]


“Redis 指标”边栏选项卡包含以下图表。

| Redis 指标图表 | 显示的度量值 |
|------------------|-------------------|
| 缓存读取和缓存写入 | 缓存读取量 |
| | 缓存写入量 |
| 连接的客户端数 | 连接的客户端数 |
| 命中数和未命中数 | 缓存命中数 |
| | 缓存未命中数 |
| 总命令数 | 总操作数 |
| 获取数和设置数 | 获取数 |
| | 设置数 |
| CPU 使用率 | CPU |
| 内存用量 | 已用内存 |
| | 已用内存 RSS |
| Redis 服务器负载 | 服务器负载 |
| 密钥计数 | 总密钥数 |
| | 逐出的密钥数 |
| | 过期的密钥数 |


有关特定图表上的度量值的更详细视图，以及若要自定义图表，请从“Redis 指标”边栏选项卡中单击所需的图表，以显示该图表的“度量值”边栏选项卡。

![“度量值”分页][redis-cache-metric-blade]

对图表中所显示度量值设置的任何警报均列于该图表的“度量值”边栏选项卡的底部。

若要添加或删除度量值或更改报告间隔，请选择“编辑图表”。

若要向图标中添加度量值或从图表中删除度量值，请单击度量值名称旁的复选框。若要更改报告间隔，请单击所需的间隔。若要更改“图表类型”，请单击所需的样式。完成所需的更改后，请单击“保存”。

![编辑图表][redis-cache-edit-chart]

单击“保存”后，你的更改将会保留到你退出“度量值”边栏选项卡为止。当你稍后返回时，你仍会看到原始的度量值和时间范围。有关自定义图表的详细信息，请参阅[监视服务度量值](/documentation/articles/insights-how-to-customize-monitoring/)。

若要在图表上查看指定时间段的度量值，请将鼠标悬停在图表上对应于所需时间的某一特定栏或点，就会显示该间隔的度量值。

![查看图表的详细信息][redis-cache-view-chart-details]

## 如何监视含群集的高级缓存

已启用[群集](/documentation/articles/cache-how-to-premium-clustering/)的高级缓存最多可以具有 10 个分片。每个分片都有它自己的度量值，而且这些度量值会聚合在一起，作为一个整体为缓存提供度量值。每个度量值均包含两个版本。一个度量值测量整个缓存的性能，名称中包含 `(Shard 0-9)` 的另一个度量值版本则测量缓存中单个分片的性能。例如，如果缓存有 3 个分片，`Cache Hits` 就是整个缓存的命中总数，而 `Cache Hits (Shard 2)` 就只是该缓存分片的命中数。

每个监视图表均显示缓存的顶级度量值以及每个缓存分片的度量值。

![监视][redis-cache-premium-monitor]

将鼠标悬停在数据点上会显示该时间点的详细信息。

![监视][redis-cache-premium-point-summary]

较大的值通常是缓存的聚合值，而较小的值是分片的单个度量值。请注意，在此示例中有三个分片，缓存命中均匀地分布在这些分片中。

![监视][redis-cache-premium-point-shard]

若要查看更多详细信息，请单击图表在“度量值”边栏选项卡上查看展开的视图。

![监视][redis-cache-premium-chart-detail]

默认情况下，每个图表均包含顶级缓存性能计数器以及单个分片的性能计数器。你可以在“编辑图表”边栏选项卡上自定义这些项。

![监视][redis-cache-premium-edit]

有关可用性能计数器的详细信息，请参阅[可用度量值和报告间隔](#available-metrics-and-reporting-intervals)。


## <a name="operations-and-alerts"></a>操作和警报

“Redis 缓存”边栏选项卡上的“操作”部分有“事件”和“警报规则”部分。

![操作][redis-cache-operations-events]

若要查看最近的缓存操作的列表，请单击“事件”图表，显示“事件”边栏选项卡。操作示例包括检索和重新生成访问密钥以及警规则的激活和解析。有关每个事件的详细信息，请单击“事件”边栏选项卡中的事件。

有关事件的详细信息，请参阅[查看事件和审核日志](/documentation/articles/insights-debugging-with-events/)。

“警报规则”部分显示缓存实例的警报计数。警报规则使你可以监视缓存实例，并且每当特定度量值达到规则中所定义的阈值时都将收到一封电子邮件。

约每隔五分钟评估一次警报规则，并且当警报规则激活时，将发送任何已配置的通知。警报规则的激活和通知不会立刻进行处理；可能会延迟若干分钟，才会激活警报规则和发送通知。

警报规则可以从特定监视图表的“度量值”边栏选项卡，或从“警报规则”边栏选项卡进行查看和设置。

警报规则具有以下属性。

| 警报规则属性 | 说明 |
|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 资源 | 通过警报规则评估的资源。从 Redis 缓存创建警报规则时，缓存为资源。 |
| 名称 | 在当前缓存实例内唯一标识警报规则的名称。 |
| 说明 | 警报规则的可选说明。 |
| 度量值 | 将由警报规则监视的度量值。有关缓存度量值的列表，请参阅可用度量值和报告间隔。 |
| 条件 | 警报规则的条件运算符。可能的选项是：大于、大于或等于、小于、小于或等于 |
| 阈值 | 用于与使用条件属性指定的运算符的度量值进行比较的值。根据度量值，此值可能以字节/秒、字节、% 计或者以计数计。 |
| 周期 | 指定度量值的平均值用于警报规则比较的周期。例如，如果该周期在最近一个小时，则前一个小时间隔内度量值的平均值将用于比较。如果想要在活动中由于高峰达到该阈值时得到通知，则适用较短的周期。若要在存在高于阈值的持续活动时得到通知，则使用较长的周期。 |
| 电子邮件服务和协同管理员 | 如果为 true，则当警报被激活时，服务管理员和协同管理员将通过电子邮件得到通知。 |
| 其他管理员电子邮件 | 当警报被激活时，要通知的其他管理员的可选电子邮件地址。 |

每个警报规则激活仅发送一个通知。一旦超过规则的阈值并发送了一条通知，则该规则要在该度量值低于阈值后才会重新进行评估。如果该度量值随后超出了阈值，则将重新激活警报并发送新通知。

若要查看所有缓存实例的警报规则，请单击“Redis 缓存”边栏选项卡中的“警报规则”部分。若仅查看使用特定度量值的警报规则，请导航到包含该度量值的图表的“度量值”边栏选项卡。

![警报规则][redis-cache-alert-rules]

若要添加警报规则，请从“度量值”边栏选项卡或“警报规则”边栏选项卡，单击“添加警报”。

在“添加警报”规则边栏选项卡输入所需的规则条件，然后单击“确定”。

![添加警报规则][redis-cache-add-alert]

>[AZURE.NOTE] 通过从“度量值”边栏选项卡单击“添加警报”创建警报规则时，仅该边栏选项卡中的图表上显示的度量值将出现在“度量值”下拉列表中。通过从“警报规则”边栏选项卡单击“添加警报”创建警报规则时，所有缓存度量值均在“度量值”下拉列表中可用。

保存警报规则后，它将出现在“警报规则”边栏选项卡，以及显示警报规则中所用度量值的任何图表的“度量值”边栏选项卡上。若要编辑警报规则，请单击警报规则的名称，以显示“编辑规则”边栏选项卡。在“编辑规则”边栏选项卡中，可以编辑该规则的属性、删除或禁用警报规则，或重新启用之前禁用的规则。

>[AZURE.NOTE] 对规则的属性所做的任何更改可能会花些时间才会反映在“警报规则”边栏选项卡或“度量值”边栏选项卡上。

当激活警报规则时，将根据警报规则的配置发送一封电子邮件，并且“Redis 缓存”边栏选项卡上的“警报规则”部分将显示一个警报图标。

当警报条件评估结果不再为 true 时，警报规则将被视为已解决。解决警报规则条件之后，警报图标将替换为复选标记。有关警报激活和解析的详细信息，请单击“Redis 缓存”边栏选项卡上的“事件”部分，查看“事件”边栏选项卡上的事件。

有关 Azure 中的警报的详细信息，请参阅[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)。

## “Redis 缓存”边栏选项卡上的度量值

“Redis 缓存”边栏选项卡显示以下类别的度量值。

-	[监视图表](#monitoring-charts)
-	[图表使用情况](#usage-charts)

### <a name="monitoring-charts"></a>监视图表

“监视”部分有“命中数和未命中数”、“获取数和设置数”、“连接数”以及“总命令数”图表。

![监视图表][redis-cache-monitoring-part]

“监视”图表将显示以下度量值。

| 监视图表 | 缓存度量值 |
|------------------|-------------------|
| 命中数和未命中数 | 缓存命中数 |
| | 缓存未命中数 |
| 获取数和设置数 | 获取数 |
| | 设置数 |
| 连接 | 连接的客户端数 |
| 总命令数 | 总操作数 |

有关本部分中的查看度量值和自定义单个图表的信息，请参阅[如何查看度量值和自定义度量值图表](#how-to-view-metrics-and-customize-charts)部分。

### <a name="usage-charts"></a>图表使用情况

“使用情况”部分有“Redis 服务器负载”、“内存使用量”、“网络带宽”和“CPU 使用率”图表，并且还显示缓存实例的“定价层”。

![图表使用情况][redis-cache-usage-part]

“定价层”将显示缓存定价层，并可用于将缓存[缩放](/documentation/articles/cache-how-to-scale/)到不同的定价层。

“使用情况”图表将显示以下度量值。

| 使用情况图表 | 缓存度量值 |
|-------------------|---------------|
| Redis 服务器负载 | 服务器负载 |
| 内存用量 | 已用内存 |
| 网络带宽 | 缓存写入量 |
| CPU 使用率 | CPU |

有关本部分中的查看度量值和自定义单个图表的信息，请参阅[如何查看度量值和自定义度量值图表](#how-to-view-metrics-and-customize-charts)部分。
  
<!-- IMAGES -->

[redis-cache-enable-diagnostics]: ./media/cache-how-to-monitor/redis-cache-enable-diagnostics.png

[redis-cache-diagnostic-settings]: ./media/cache-how-to-monitor/redis-cache-diagnostic-settings.png

[redis-cache-configure-diagnostics]: ./media/cache-how-to-monitor/redis-cache-configure-diagnostics.png

[redis-cache-monitoring-part]: ./media/cache-how-to-monitor/redis-cache-monitoring-part.png

[redis-cache-usage-part]: ./media/cache-how-to-monitor/redis-cache-usage-part.png

[redis-cache-metric-blade]: ./media/cache-how-to-monitor/redis-cache-metric-blade.png

[redis-cache-edit-chart]: ./media/cache-how-to-monitor/redis-cache-edit-chart.png

[redis-cache-view-chart-details]: ./media/cache-how-to-monitor/redis-cache-view-chart-details.png

[redis-cache-operations-events]: ./media/cache-how-to-monitor/redis-cache-operations-events.png

[redis-cache-alert-rules]: ./media/cache-how-to-monitor/redis-cache-alert-rules.png

[redis-cache-add-alert]: ./media/cache-how-to-monitor/redis-cache-add-alert.png

[redis-cache-premium-monitor]: ./media/cache-how-to-monitor/redis-cache-premium-monitor.png

[redis-cache-premium-edit]: ./media/cache-how-to-monitor/redis-cache-premium-edit.png

[redis-cache-premium-chart-detail]: ./media/cache-how-to-monitor/redis-cache-premium-chart-detail.png

[redis-cache-premium-point-summary]: ./media/cache-how-to-monitor/redis-cache-premium-point-summary.png

[redis-cache-premium-point-shard]: ./media/cache-how-to-monitor/redis-cache-premium-point-shard.png

[redis-cache-redis-metrics]: ./media/cache-how-to-monitor/redis-cache-redis-metrics.png

[redis-cache-redis-metrics-blade]: ./media/cache-how-to-monitor/redis-cache-redis-metrics-blade.png

<!---HONumber=AcomDC_0718_2016-->