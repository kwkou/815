<properties
    pageTitle="在 Azure 门户预览中启用存储指标 | Azure"
    description="如何为 Blob、队列、表和文件服务启用存储度量值"
    services="storage"
    documentationcenter=""
    author="robinsh"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="0407adfc-2a41-4126-922d-b76e90b74563"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="02/14/2017"
    wacn.date="04/24/2017"
    ms.author="robinsh"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="75c2817998d6c1f1e5fb6b35381dcd5ef688549e"
    ms.lasthandoff="04/14/2017" />

# <a name="enabling-azure-storage-metrics-and-viewing-metrics-data"></a>启用 Azure 存储指标并查看指标数据
[AZURE.INCLUDE [storage-selector-portal-enable-and-view-metrics](../../includes/storage-selector-portal-enable-and-view-metrics.md)]

## <a name="overview"></a>概述
创建新的存储帐户时默认情况下会启用存储度量。 可通过 [Azure 门户预览](https://portal.azure.cn)或 Windows PowerShell 配置监视，也可通过一个存储客户端库以编程方式配置监视。

可以为度量数据配置保留期：此期限用于确定存储服务保留度量并针对存储度量所需的空间向你收费的时长。 通常，由于分钟度量值需要大量额外的空间，因此，应对分钟度量值而非小时度量值使用较短的保留期。 你应该选择恰当的保留期，以便有足够的时间分析数据，并下载任何需要保留下来进行脱机分析或报告的度量值。 请记住，从存储帐户下载度量值数据时，你也需要付费。

## <a name="how-to-enable-metrics-using-the-azure-portal-preview"></a>如何通过 Azure 门户预览启用指标
请按照下列步骤在 [Azure 门户预览](https://portal.azure.cn)中启用指标：

1. 导航到存储帐户。
1. 在“菜单”边栏选项卡上选择“诊断”
1. 确保“状态”设置为“打开”。
1. 选择你希望监视的服务的度量值。
1. 指定用来指示保留度量值和日志数据的时间长度的保留期策略。
1. 选择“其他安全性验证” 。

请注意，[Azure 门户预览](https://portal.azure.cn)目前不允许在存储帐户中配置分钟指标；必须通过 PowerShell 或以编程方式启用分钟指标。

## <a name="how-to-enable-metrics-using-powershell"></a>如何通过 PowerShell 启用指标
可以使用本地计算机上的 PowerShell 在存储帐户中配置存储指标，具体方法是：使用 Azure PowerShell cmdlet Get-AzureStorageServiceMetricsProperty 检索当前设置，然后使用 cmdlet Set-AzureStorageServiceMetricsProperty 更改当前设置。

控制存储指标的 cmdlet 使用以下参数：

* MetricsType：可能值是 Hour 和 Minute。
* ServiceType：可能值为 Blob、Queue 和 Table。
* MetricsLevel：可能的值为 None、Service 和 ServiceAndApi。

例如，以下命令在默认存储帐户中为 Blob 服务打开分钟指标，并将保留期设为 5 天：

    Set-AzureStorageServiceMetricsProperty -MetricsType Minute -ServiceType Blob -MetricsLevel ServiceAndApi  -RetentionDays 5`

以下命令在默认存储帐户中为 Blob 服务检索当前的小时度量值级别和保留天数：

    Get-AzureStorageServiceMetricsProperty -MetricsType Hour -ServiceType Blob

若要了解如何配置 Azure PowerShell cmdlet 来使用 Azure 订阅并了解如何选择要使用的默认存储帐户，请参阅：[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

## <a name="how-to-enable-storage-metrics-programmatically"></a>如何以编程方式启用存储度量值
下面的 C# 代码段演示了如何使用 .NET 的存储客户端库为 Blob 服务启用度量值和日志记录：

    //Parse the connection string for the storage account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key;EndpointSuffix=core.chinacloudapi.cn";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create service client for credentialed access to the Blob service.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Enable Storage Analytics logging and set retention policy to 10 days.
    ServiceProperties properties = new ServiceProperties();
    properties.Logging.LoggingOperations = LoggingOperations.All;
    properties.Logging.RetentionDays = 10;
    properties.Logging.Version = "1.0";

    // Configure service properties for metrics. Both metrics and logging must be set at the same time.
    properties.HourMetrics.MetricsLevel = MetricsLevel.ServiceAndApi;
    properties.HourMetrics.RetentionDays = 10;
    properties.HourMetrics.Version = "1.0";

    properties.MinuteMetrics.MetricsLevel = MetricsLevel.ServiceAndApi;
    properties.MinuteMetrics.RetentionDays = 10;
    properties.MinuteMetrics.Version = "1.0";

    // Set the default service version to be used for anonymous requests.
    properties.DefaultServiceVersion = "2015-04-05";

    // Set the service properties.
    blobClient.SetServiceProperties(properties);

## <a name="viewing-storage-metrics"></a>查看存储指标
在将存储分析指标配置为监视存储帐户后，存储分析将使用存储帐户在一组已知表中记录指标。可以将图表配置为每小时查看 [Azure 门户预览](https://portal.azure.cn)中的指标：

1. 在 [Azure 门户预览](https://portal.azure.cn)中导航到存储帐户。
1. 在要查看其指标的服务的“菜单”边栏选项卡中，选择“指标”。
1. 在要配置的图表上选择“编辑”。
1. 在“编辑图表”边栏选项卡中，选择“时间范围”、“图表类型”，以及想要在图表中显示的指标。
1. 选择“确定”

若要下载指标供长期存储或本地分析，需执行以下操作：

* 使用识别这些表并允许查看和下载这些表的工具。
* 编写自定义应用程序或脚本来读取和存储表。

很多第三方存储浏览工具可识别这些表，并可用于直接查看这些表。
有关可用工具的列表，请参阅 [Azure 存储客户端工具](/documentation/articles/storage-explorers/)。

> [AZURE.NOTE]
> 从 [Azure 存储资源管理器](http://storageexplorer.com/) 0.8.0 版本开始，可以查看和下载分析指标表。
> 
> 

若要以编程方式访问分析表，请注意如果存储帐户中列出这些表，将不显示它们。 可按名称直接访问它们，也可使用 .NET 客户端库中的 [CloudAnalyticsClient API](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.analytics.cloudanalyticsclient.aspx) 查询表名。

### <a name="hourly-metrics"></a>小时指标
* $MetricsHourPrimaryTransactionsBlob
* $MetricsHourPrimaryTransactionsTable
* $MetricsHourPrimaryTransactionsQueue

### <a name="minute-metrics"></a>分钟度量值
* $MetricsMinutePrimaryTransactionsBlob
* $MetricsMinutePrimaryTransactionsTable
* $MetricsMinutePrimaryTransactionsQueue

### <a name="capacity"></a>容量
* $MetricsCapacityBlob

有关这些表的完整架构详细信息，请参阅[存储分析指标表架构](https://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx)。以下示例行仅显示一部分可用列，但也说明了存储度量值在采用相应方式保存这些度量值时展现的一些重要功能：

| PartitionKey | RowKey | Timestamp | TotalRequests | TotalBillableRequests | TotalIngress | TotalEgress | Availability | AverageE2ELatency | AverageServerLatency | PercentSuccess |
| --- |:---:| ---:| --- | --- | --- | --- | --- | --- | --- | --- |
| 20140522T1100 |user;All |2014-05-22T11:01:16.7650250Z |7 |7 |4003 |46801 |100 |104.4286 |6.857143 |100 |
| 20140522T1100 |user;QueryEntities |2014-05-22T11:01:16.7640250Z |5 |5 |2694 |45951 |100 |143.8 |7.8 |100 |
| 20140522T1100 |user;QueryEntity |2014-05-22T11:01:16.7650250Z |1 |1 |538 |633 |100 |3 |3 |100 |
| 20140522T1100 |user;UpdateEntity |2014-05-22T11:01:16.7650250Z |1 |1 |771 |217 |100 |9 |6 |100 |

在这个分钟度量值数据示例中，分区键按分钟使用时间。 行键可识别行中存储的信息的类型，其中包含两条信息，即访问类型和请求类型：

* 访问类型是 user 或 system，其中 user 是指用户对存储服务发出的所有请求，而 system 是指存储分析发出的请求。
* 请求类型是 all（在这种情况下是摘要行）或可识别的特定 API，如 QueryEntity 或 UpdateEntity。

上面的示例数据显示一分钟的所有记录（从上午 11:00 开始），因此 QueryEntities 请求数加 QueryEntity 请求数再加 UpdateEntity 请求数的和为 7，这是显示在 user:All 行上的总数。 同样，通过计算 ((143.8 * 5) + 3 + 9)/7，可以在 user:All 行得到平均端到端延迟为 104.4286。

## <a name="metrics-alerts"></a>度量警报
应考虑在 [Azure 门户预览](https://portal.azure.cn)中设置警报，以便存储指标可以自动向你通知存储服务行为的重要更改。如果使用存储资源管理器工具下载这种采用分隔格式的指标数据，则可以使用 Microsoft Excel 分析数据。有关可用存储资源管理器工具的列表，请参阅 [Azure 存储客户端工具](/documentation/articles/storage-explorers/)。 可以在“警报规则”边栏选项卡（可在存储帐户菜单边栏选项卡中的“监视”下进行访问）。

> [AZURE.IMPORTANT]
> 在存储事件与记录对应每小时或分钟度量数据的时间之间可能存在延迟。 对于分钟度量，可能会一次写入几分钟的数据。 这可能会导致将前面几分钟的事务聚合到当前分钟的事务中。 发生此情况时，警报服务可能没有已配置警报间隔内的所有可用度量数据，这可能会导致意外触发警报。
>

## <a name="accessing-metrics-data-programmatically"></a>以编程方式访问度量值数据
以下列表显示示例 C# 代码，该代码用于访问分钟范围的分钟度量值，并在控制台窗口中显示结果。 它使用 Azure 存储库版本 4，其中包括 CloudAnalyticsClient 类，用于简化访问存储中的度量值表的过程。

    private static void PrintMinuteMetrics(CloudAnalyticsClient analyticsClient, DateTimeOffset startDateTime, DateTimeOffset endDateTime)
    {
        // Convert the dates to the format used in the PartitionKey
        var start = startDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");
        var end = endDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");

        var services = Enum.GetValues(typeof(StorageService));
        foreach (StorageService service in services)
        {
            Console.WriteLine("Minute Metrics for Service {0} from {1} to {2} UTC", service, start, end);
            var metricsQuery = analyticsClient.CreateMinuteMetricsQuery(service, StorageLocation.Primary);
            var t = analyticsClient.GetMinuteMetricsTable(service);
            var opContext = new OperationContext();
            var query =
              from entity in metricsQuery
              // Note, you can't filter using the entity properties Time, AccessType, or TransactionType
              // because they are calculated fields in the MetricsEntity class.
              // The PartitionKey identifies the DataTime of the metrics.
              where entity.PartitionKey.CompareTo(start) >= 0 && entity.PartitionKey.CompareTo(end) <= 0
            select entity;

            // Filter on "user" transactions after fetching the metrics from Table Storage.
            // (StartsWith is not supported using LINQ with Azure table storage)
            var results = query.ToList().Where(m => m.RowKey.StartsWith("user"));
            var resultString = results.Aggregate(new StringBuilder(), (builder, metrics) => builder.AppendLine(MetricsString(metrics, opContext))).ToString();
            Console.WriteLine(resultString);
        }
    }

    private static string MetricsString(MetricsEntity entity, OperationContext opContext)
    {
        var entityProperties = entity.WriteEntity(opContext);
        var entityString =
          string.Format("Time: {0}, ", entity.Time) +
          string.Format("AccessType: {0}, ", entity.AccessType) +
          string.Format("TransactionType: {0}, ", entity.TransactionType) +
          string.Join(",", entityProperties.Select(e => new KeyValuePair<string, string>(e.Key.ToString(), e.Value.PropertyAsObject.ToString())));
        return entityString;
    }

## <a name="what-charges-do-you-incur-when-you-enable-storage-metrics"></a>在启用存储度量值时，你需要支付多少费用？
为度量值创建表实体的写入请求，按适用于所有 Azure 存储操作的标准费率收费。

客户端针对度量值数据的读取和删除请求也按标准费率收费。 如果你已配置数据保留策略，则当 Azure 存储空间删除旧的度量值数据时，你不用付费。 但是，如果你删除分析数据，则会针对删除操作向你的帐户收费。

度量值表使用的容量也要付费：你可以依据以下内容估算用于存储度量值数据的容量：

* 如果某个服务每小时都会利用每个服务的每个 API，则在启用服务和 API 级别摘要后，每小时约有 148KB 的数据存储在度量值事务表中。
* 如果某个服务每小时都会利用每个服务的每个 API，则在仅启用服务级别摘要后，每小时约有 12KB 的数据存储在度量值事务表中。
* Blob 的容量表每天添加两行（如果用户已为日志选择加入）：这表示，此表的大小每天最多以约 300 字节的幅度增加。

## <a name="next-steps"></a>后续步骤
[启用存储日志记录和访问日志数据](https://docs.microsoft.com/zh-cn/rest/api/storageservices/fileservices/Enabling-Storage-Logging-and-Accessing-Log-Data)
<!--Update_Description: wording update;add anchors to H2 titles-->