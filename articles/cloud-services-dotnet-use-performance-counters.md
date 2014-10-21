<properties linkid="dev-net-commons-tasks-profiling" urlDisplayName="Performance Profiling" pageTitle="Use Performance Counters in Azure (.NET)" metaKeywords="Azure performance counters, Azure performance profiling, Azure performance counters C#, Azure performance profiling C#" description="Learn how to enable and collect data from performance counters in Azure applications. " metaCanonical="" services="cloud-services" documentationCenter=".NET" title="Using performance counters in Azure" authors="ryanwi" solutions="" manager="" editor="" />

# 在 Azure 中使用性能计数器

你可以在 Azure 应用程序中使用性能计数器来收集可帮助确定系统瓶颈以及微调系统和应用程序性能的数据。可收集适用于 Windows Server 2008、Windows Server 2012、IIS 和 ASP.NET 的性能计数器，并且可将其用于确定 Azure 应用程序的运行状况。

本主题介绍如何使用 diagnostics.wadcfg 配置文件启用应用程序中的性能计数器。有关在 [Azure 管理门户][Azure 管理门户]中监视应用程序性能的信息，请参阅[如何监视云服务][如何监视云服务]。有关创建日志记录和跟踪策略以及使用诊断和其他技术排查问题及优化 Azure 应用程序的其他深入指南，请参阅[有关开发 Azure 应用程序的问题排查最佳实践][有关开发 Azure 应用程序的问题排查最佳实践]。

此任务包括下列步骤：

-   [先决条件][先决条件]
-   [步骤 1：通过性能计数器收集和存储数据][步骤 1：通过性能计数器收集和存储数据]
-   [步骤 2：（可选）创建自定义性能计数器][步骤 2：（可选）创建自定义性能计数器]
-   [步骤 3：查询性能计数器数据][步骤 3：查询性能计数器数据]
-   [后续步骤][后续步骤]
-   [其他资源][其他资源]

## <a name="prereqs"> </a> 先决条件

本文假定你已将诊断监视器导入应用程序中，并且已将 diagnostics.wadcfg 配置文件添加到 Visual Studio 解决方案中。有关详细信息，请参阅[在 Azure 中启用诊断][在 Azure 中启用诊断]中的步骤 1 和步骤 2。

## <a name="step1"> </a>步骤 1：通过性能计数器收集和存储数据

在将 diagnostics.wadcfg 文件添加到 Visual Studio 解决方案中后，你可以在 Azure 应用程序中配置性能计数器数据的收集和存储。通过将性能计数器添加到 diagnostics.wadcfg 文件可做到这一点。首先在实例中收集诊断数据（包括性能计数器）。随后该数据将保留到 Azure 表服务中的 **WADPerformanceCountersTable** 表中，因此你还需要指定应用程序中的存储帐户。如果在计算模拟器中本地测试应用程序，则也可在存储模拟器中本地存储诊断数据。你必须先转到 [Azure 管理门户][Azure 管理门户]并创建存储帐户，然后才能存储诊断数据。最佳做法是将存储帐户置于与 Azure 应用程序相同的地理位置，从而消除外部宽带成本并减少延迟。

### 将性能计数器添加到 diagnostics.wadcfg 文件

有许多可收集的性能计数器，以下示例介绍了为 Web 角色监视和辅助角色监视建议的几个性能计数器。

打开 diagnostics.wadcfg 文件并将以下内容添加到 **DiagnosticMonitorConfiguration** 元素中：

        <PerformanceCounters bufferQuotaInMB="0" scheduledTransferPeriod="PT30M">
        <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT30S" />
        <!-- Use the Process(w3wp) category counters in a web role -->
        <PerformanceCounterConfiguration counterSpecifier="\Process(w3wp)\% Processor Time" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\Process(w3wp)\Private Bytes" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\Process(w3wp)\Thread Count" sampleRate="PT30S" />
        <!-- Use the Process(WaWorkerHost) category counters in a worker role.
        <PerformanceCounterConfiguration counterSpecifier="\Process(WaWorkerHost)\% Processor Time" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\Process(WaWorkerHost)\Private Bytes" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\Process(WaWorkerHost)\Thread Count" sampleRate="PT30S" /> 
        -->
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Interop(_Global_)\# of marshalling" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Loading(_Global_)\% Time Loading" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR LocksAndThreads(_Global_)\Contention Rate / sec" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Memory(_Global_)\# Bytes in all Heaps" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Networking(_Global_)\Connections Established" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Remoting(_Global_)\Remote Calls/sec" sampleRate="PT30S" />
        <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Jit(_Global_)\% Time in Jit" sampleRate="PT30S" />
        </PerformanceCounters>    

**bufferQuotaInMB** 特性，指定可用于数据收集类型（Azure 日志、IIS 日志等）的文件系统存储的最大量。默认值为 0。在达到配额时，将删除最旧的数据，因为将添加新数据。所有 **bufferQuotaInMB** 属性的值的和必须大于 **OverallQuotaInMB** 特性的值。有关确定收集诊断数据需要多少存储的更多详细讨论，请参阅[有关开发 Azure 应用程序的问题排查最佳实践][有关开发 Azure 应用程序的问题排查最佳实践]中的“设置 WAD”部分。

**scheduledTransferPeriod** 特性，指定计划的数据传输之间的时间间隔，取整为最接近的分钟数。在下面的示例中，将其设置为 PT30M（30 分钟）。通过将传输时间段设置为一个较小的值（例如 1 分钟），将对应用程序在生产中的性能产生负面影响，但这可能对使诊断在测试时快速运行很有用。计划的传输时间段应足够小以确保不在实例上覆盖诊断数据，但也应足够大以确保不会影响应用程序的性能。

**counterSpecifier** 特性指定要收集的性能计数器。

**sampleRate** 特性指定性能计数器的采样速度（此示例中为 30 秒）。

添加要收集的性能计数器后，将你的更改保存到 diagnostics.wadcfg 文件中。接下来，你需要指定将诊断数据保留到的存储帐户。

### 指定存储帐户

若要将诊断信息保存到 Azure 存储帐户，则必须在服务配置 (ServiceConfiguration.cscfg) 文件中指定连接字符串。利用 Azure Tools for Visual Studio 1.4 版（2011 年 8 月）和更高版本，你可以拥有针对本地和云的不同的配置文件 (ServiceConfiguration.cscfg)。多个服务配置对于诊断会很有用，因为你可以使用存储模拟器进行免费的本地测试，同时维护用于生产的单独的配置文件。

设置连接字符串：

1.  使用常用文本编辑器打开 ServiceConfiguration.Cloud.cscfg 文件并为存储帐户设置连接字符串：

        <ConfigurationSettings>
          <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=<name>;AccountKey=<key>"/>
        </ConfigurationSettings>

    将在管理门户的存储帐户仪表板的“管理密钥”下找到 **AccountName** 和 **AccountKey** 值。

2.  保存 ServiceConfiguration.Cloud.cscfg 文件。
3.  打开 ServiceConfiguration.Local.cscfg 文件并确认已将 **UseDevelopmentStorage** 设置为 true（默认值）。

        <ConfigurationSettings>
           <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
        </ConfigurationSettings>

    现已设置连接字符串，在部署应用程序时，该应用程序会将诊断数据保存到存储帐户。

4.  保存并生成项目，然后部署应用程序。

## <a name="step2"> </a>步骤 2：（可选）创建自定义性能计数器

除了预定义的性能计数器外，你还可以添加自己的自定义性能计数器来监视 Web 角色或辅助角色。自定义性能计数器可用于跟踪和监视特定于应用程序的行为，并且可使用提升的权限在启动任务、Web 角色或辅助角色中创建或删除该计数器。

执行下列步骤来创建一个名为“\\MyCustomCounterCategory\\MyButton1Counter”的简单的自定义性能计数器：

1.  打开应用程序的服务定义文件 (CSDEF)。
2.  将 **Runtime** 元素添加到 **WebRole** 或 **WorkerRole**元素可允许使用提升的权限执行：

        <Runtime executionContext="elevated" />

3.  保存文件。
4.  打开 diagnostics.wadcfg 文件并将以下内容添加到 **DiagnosticMonitorConfiguration** 元素中：

        <PerformanceCounters bufferQuotaInMB="0" scheduledTransferPeriod="PT30M">
        <PerformanceCounterConfiguration counterSpecifier="\MyCustomCounterCategory\MyButton1Counter" sampleRate="PT30S"/>
        </PerformanceCounters>      

5.  保存文件。
6.  先使用角色的 **OnStart** 方法创建自定义性能计数器类别，然后再调用 **base.OnStart**。以下 C# 示例将创建一个自定义类别（如果尚不存在）：

        public override bool OnStart()
        {
        if (!PerformanceCounterCategory.Exists("MyCustomCounterCategory"))
        {
           CounterCreationDataCollection counterCollection = new CounterCreationDataCollection();

           // add a counter tracking user button1 clicks
           CounterCreationData operationTotal1 = new CounterCreationData();
           operationTotal1.CounterName = "MyButton1Counter";
           operationTotal1.CounterHelp = "My Custom Counter for Button1";
           operationTotal1.CounterType = PerformanceCounterType.NumberOfItems32;
           counterCollection.Add(operationTotal1);

           PerformanceCounterCategory.Create(
             "MyCustomCounterCategory",
             "My Custom Counter Category",
             PerformanceCounterCategoryType.SingleInstance, counterCollection);

           Trace.WriteLine("Custom counter category created.");
        }
        else{
           Trace.WriteLine("Custom counter category already exists.");
        }

        return base.OnStart();
        }

7.  更新应用程序中的计数器。以下示例 对 **Button1\_Click** 事件更新自定义性能计数器：

        protected void Button1_Click(object sender, EventArgs e)
        {
           PerformanceCounter button1Counter = new PerformanceCounter(
              "MyCustomCounterCategory",
              "MyButton1Counter",
              string.Empty,
              false);

           button1Counter.Increment();
           this.Button1.Text = "Button 1 count: " +
              button1Counter.RawValue.ToString();
        }

8.  保存文件。

完成这些步骤后，将由 Azure 诊断监视器收集自定义性能计数器数据。

## <a name="step3"> </a>步骤 3：查询性能计数器数据

在应用程序部署完成并运行后，诊断监视器将开始收集性能计数器并将该数据保存到 Azure 存储空间。你使用 Cerebrata 提供的工具（例如 **Visual Studio 中的服务器资源管理器**、[Azure 存储资源管理器][Azure 存储资源管理器]或 [Azure 诊断管理器][Azure 诊断管理器]）查看 **WADPerformanceCountersTable** 表中的性能计数器数据。你还可以使用 [C\#][C\#]、[Java][Java]、[Node.js][Node.js]、[Python][Python] 或 [PHP][PHP] 以编程方式查询表服务。

以下 C# 示例显示针对 **WADPerformanceCountersTable** 表的简单查询并将诊断数据保存到 CSV 文件中。将性能计数器保存到 CSV 文件后，你可以使用 Microsoft Excel 中的图形功能或使用其他一些工具来使数据可视化。请务必添加对 Microsoft.WindowsAzure.Storage.dll（它包含在 2012 年 10 月版的 Azure SDK for .NET 和更高版本中）的引用。程序集安装在 %Program Files%\\Microsoft SDKs\\Azure.NET SDK\\version-num\\ref\\ 目录中。

        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Auth;
        using Microsoft.WindowsAzure.Storage.Table;

        ...

        // Get the connection string. When using Azure Cloud Services, it is recommended 
        // you store your connection string using the Azure service configuration
        // system (*.csdef and *.cscfg files). You can you use the CloudConfigurationManager type 
        // to retrieve your storage connection string.  If you're not using Cloud Services, it's
        // recommended that you store the connection string in your web.config or app.config file.
        // Use the ConfigurationManager type to retrieve your storage connection string.
        
        string connectionString = Microsoft.WindowsAzure.CloudConfigurationManager.GetSetting("StorageConnectionString");
        //string connectionString = System.Configuration.ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString;
            
        // Get a reference to the storage account using the connection string.  You can also use the development storage account (Storage Emulator)
        // for local debugging.              
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);
        //CloudStorageAccount storageAccount = CloudStorageAccount.DevelopmentStorageAccount;

        // Create the table client.
        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

        // Create the CloudTable object that represents the "WADPerformanceCountersTable" table.
        CloudTable table = tableClient.GetTableReference("WADPerformanceCountersTable");

        // Create the table query, filter on a specific CounterName, DeploymentId and RoleInstance.
        TableQuery<PerformanceCountersEntity> query = new TableQuery<PerformanceCountersEntity>()
            .Where(
                TableQuery.CombineFilters(
                    TableQuery.GenerateFilterCondition("CounterName", QueryComparisons.Equal, @"\Processor(_Total)\% Processor Time"),
                    TableOperators.And,
                    TableQuery.CombineFilters(                        
                        TableQuery.GenerateFilterCondition("DeploymentId", QueryComparisons.Equal, "ec26b7a1720447e1bcdeefc41c4892a3"),
                        TableOperators.And,
                        TableQuery.GenerateFilterCondition("RoleInstance", QueryComparisons.Equal, "WebRole1_IN_0")                        
                    )
                )
            );

        // Execute the table query.
        IEnumerable<PerformanceCountersEntity> result = table.ExecuteQuery(query);

        // Process the query results and build a CSV file.
        StringBuilder sb = new StringBuilder("TimeStamp,EventTickCount,DeploymentId,Role,RoleInstance,CounterName,CounterValue\n");
            
        foreach (PerformanceCountersEntity entity in result)
        {
            sb.Append(entity.Timestamp + "," + entity.EventTickCount + "," + entity.DeploymentId + "," 
                + entity.Role + "," + entity.RoleInstance + "," + entity.CounterName + "," + entity.CounterValue+"\n");
        }

        StreamWriter sw = File.CreateText(@"C:\temp\PerfCounters.csv");
        sw.Write(sb.ToString());
        sw.Close();

实体将映射到使用派生自 **TableEntity** 的自定义类的 C# 对象。以下代码定义一个实体类，该类表示 **WADPerformanceCountersTable** 表中的性能计数器。

        public class PerformanceCountersEntity : TableEntity
        {
            public long EventTickCount { get; set; }
            public string DeploymentId { get; set; }
            public string Role { get; set; }
            public string RoleInstance { get; set; }
            public string CounterName { get; set; }
            public double CounterValue { get; set; }                
        }

## <a name="nextsteps"> </a> 后续步骤

现在，你已了解收集性能计数器的基础知识，单击下面的链接可了解如何实现更复杂的故障排除方案。

-   [有关开发 Azure 应用程序的问题排查最佳实践][有关开发 Azure 应用程序的问题排查最佳实践]
-   [如何监视云服务][如何监视云服务]
-   [如何使用自动缩放应用程序块][如何使用自动缩放应用程序块]
-   [构建灵活的、可复原的云应用程序][构建灵活的、可复原的云应用程序]

## <a name="additional"> </a>其他资源

-   [在 Azure 中启用诊断][在 Azure 中启用诊断]
-   [使用 Azure 诊断收集日志记录数据][使用 Azure 诊断收集日志记录数据]
-   [调试 Azure 应用程序][调试 Azure 应用程序]

  [Azure 管理门户]: http://manage.windowsazure.cn
  [如何监视云服务]: https://www.windowsazure.com/zh-cn/manage/services/cloud-services/how-to-monitor-a-cloud-service/
  [有关开发 Azure 应用程序的问题排查最佳实践]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh771389.aspx
  [先决条件]: #prereqs
  [步骤 1：通过性能计数器收集和存储数据]: #step1
  [步骤 2：（可选）创建自定义性能计数器]: #step2
  [步骤 3：查询性能计数器数据]: #step3
  [后续步骤]: #nextsteps
  [其他资源]: #additional
  [在 Azure 中启用诊断]: https://www.windowsazure.com/zh-cn/develop/net/common-tasks/diagnostics/
  [Azure 存储资源管理器]: http://azurestorageexplorer.codeplex.com/
  [Azure 诊断管理器]: http://www.cerebrata.com/Products/AzureDiagnosticsManager/Default.aspx
  [Java]: http://www.windowsazure.com/zh-cn/develop/java/how-to-guides/table-service/
  [Python]: http://www.windowsazure.com/zh-cn/develop/python/how-to-guides/table-service/
  [PHP]: http://www.windowsazure.com/zh-cn/develop/php/how-to-guides/table-service/
  [如何使用自动缩放应用程序块]: http://www.windowsazure.com/zh-cn/develop/net/how-to-guides/autoscaling/
  [构建灵活的、可复原的云应用程序]: http://msdn.microsoft.com/zh-cn/library/hh680949(PandP.50).aspx
  [使用 Azure 诊断收集日志记录数据]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433048.aspx
  [调试 Azure 应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee405479.aspx
