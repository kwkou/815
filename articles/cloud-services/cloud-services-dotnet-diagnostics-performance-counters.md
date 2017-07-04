<properties
   pageTitle="在 Azure 诊断中使用性能计数器 | Azure"
   description="在 Azure 云服务或虚拟机中使用性能计数器查找瓶颈和优化性能。"
   services="cloud-services"
   documentationCenter=".net"
   authors="rboucher"
   manager="jwhit"
   editor="tysonn" />  

<tags
   ms.service="cloud-services"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="02/29/2016"
   wacn.date="03/17/2017"
   ms.author="robb" />

# 在 Azure 应用程序中创建和使用性能计数器

本文介绍性能计数器的好处，以及如何将其置于 Azure 应用程序中。可以使用性能计数器收集数据，查找瓶颈以及优化系统和应用程序的性能。

还可以收集适用于 Windows Server、IIS 和 ASP.NET 的性能计数器，并用于确定 Azure Web 角色、辅助角色和虚拟机的运行状况。此外，还可以创建和使用自定义性能计数器。

可以检查性能计数器数据
1. 直接在应用程序主机上进行，使用远程桌面访问性能计数器工具
2. 通过 System Center Operations Manager 使用 Azure Management Pack 进行
3. 使用其他监视工具访问传输到 Azure 存储的诊断数据。有关详细信息，请参阅[在 Azure 存储中存储和查看诊断数据](https://msdn.microsoft.com/zh-cn/library/azure/hh411534.aspx)。  

有关在 [Azure 经典管理门户](http://manage.windowsazure.cn)中监视应用程序性能的详细信息，请参阅[如何监视云服务](/documentation/articles/cloud-services-how-to-monitor/)。

有关创建日志记录和跟踪策略以及使用诊断和其他技术排查问题及优化 Azure 应用程序的其他深入指南，请参阅[有关开发 Azure 应用程序的问题排查最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/hh771389.aspx)。


## 启用性能计数器监视

默认情况下，不启用性能计数器。应用程序或启动任务必须修改默认诊断代理配置，以包括要为每个角色实例监视的特定性能计数器。

### 适用于 Azure 的性能计数器

Azure 提供了一部分适用于 Windows Server、IIS 和 ASP.NET 堆栈的性能计数器。下表列出了一些特别适用于 Azure 应用程序的性能计数器。

|计数器类别：对象（实例）|计数器名称 |引用|
|---|---|---|
|.NET CLR Exceptions(*Global*)|引发的异常数/秒 |异常性能计数器|
|.NET CLR Memory(*Global*) |GC 中的时间百分比 |内存性能计数器|
|ASP.NET |应用程序重新启动 |ASP.NET 性能计数器|
|ASP.NET |请求执行时间 |ASP.NET 性能计数器|
|ASP.NET |断开连接的请求数 |ASP.NET 性能计数器|
|ASP.NET |工作进程重新启动 |ASP.NET 性能计数器|
|ASP.NET Applications(**Total**)|请求总数 |ASP.NET 性能计数器|
|ASP.NET Applications(**Total**)|请求数/秒 |ASP.NET 性能计数器|
|ASP.NET v4.0.30319 |请求执行时间 |ASP.NET 性能计数器|
|ASP.NET v4.0.30319 |请求等待时间 |ASP.NET 性能计数器|
|ASP.NET v4.0.30319 |当前请求 |ASP.NET 性能计数器|
|ASP.NET v4.0.30319 |排队的请求数 |ASP.NET 性能计数器|
|ASP.NET v4.0.30319 |拒绝的请求数 |ASP.NET 性能计数器|
|内存 |可用兆字节数 |内存性能计数器|
|内存 |提交的字节数 |内存性能计数器|
|Processor(\_Total) |处理器时间百分比 |ASP.NET 性能计数器|
|TCPv4 |连接失败 |TCP 对象|
|TCPv4 |已建立的连接 |TCP 对象|
|TCPv4 |重置的连接 |TCP 对象|
|TCPv4 |发送的段数/秒 |TCP 对象|
|Network Interface(*) |接收的字节数/秒 |网络接口对象|
|Network Interface(*) |发送的字节数/秒 |网络接口对象|
|Network Interface(Microsoft Virtual Machine Bus Network Adapter \_2)|接收的字节数/秒|网络接口对象|
|Network Interface(Microsoft Virtual Machine Bus Network Adapter \_2)|发送的字节数/秒|网络接口对象|
|Network Interface(Microsoft Virtual Machine Bus Network Adapter \_2)|总字节数/秒|网络接口对象|

## 创建自定义性能计数器并将其添加到应用程序

Azure 支持针对 Web 角色和辅助角色创建和修改自定义性能计数器。这些计数器可用于跟踪和监视应用程序特定行为。可以使用提升的权限通过启动任务、Web 角色或辅助角色创建和删除自定义性能计数器类别和说明符。

>[AZURE.NOTE] 用于更改自定义性能计数器的代码必须具有提升的权限才能运行。如果代码位于 Web 角色或辅助角色中，则该角色必须在 ServiceDefinition.csdef 文件中包括标记 <Runtime executionContext="elevated" /> 才能使角色正常初始化。

可以使用诊断代理将自定义性能计数器数据发送到 Azure 存储。

标准性能计数器数据由 Azure 进程生成。必须由 Web 角色或辅助角色应用程序创建自定义性能计数器数据。请参阅[性能计数器类型](https://msdn.microsoft.com/zh-cn/library/z573042h.aspx)，以了解有关可以存储在自定义性能计数器中的数据类型。

## 存储和查看性能计数器数据

Azure 缓存性能计数器数据和其他诊断信息。当正在运行的角色实例使用远程桌面访问权限查看性能监视器之类的工具时，此数据可用于进行远程监视。若要保留角色实例以外的数据，诊断代理必须将数据传输到 Azure 存储。缓存的性能计数器数据的大小限制可以在诊断代理中配置，也可以将其配置为所有诊断数据的共享限制的一部分。有关如何设置缓冲区大小的详细信息，请参阅 [OverallQuotaInMB](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitorconfiguration.overallquotainmb.aspx) 和 [DirectoriesBufferConfiguration](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.diagnostics.directoriesbufferconfiguration.aspx)。请参阅[在 Azure 存储中存储和查看诊断数据](https://msdn.microsoft.com/zh-cn/library/azure/hh411534.aspx)，了解通过设置诊断代理将数据传输到存储帐户。

每个配置的性能计数器实例均按指定的采样率进行记录，并通过计划的传输请求或按需传输请求将采样的数据传输到存储帐户。可以将自动传输计划为每分钟一次。通过诊断代理传输的性能计数器数据存储在存储帐户的 WADPerformanceCountersTable 表中。该表可以通过标准 Azure 存储 API 方法进行访问和查询。请参阅 [Azure 性能计数器示例](http://code.msdn.microsoft.com/Windows-Azure-PerformanceCo-7d80ebf9)，以获取通过 WADPerformanceCountersTable 表查询和显示性能计数器数据的示例。

>[AZURE.NOTE] 根据诊断代理传输频率和队列延迟，存储帐户中的最新性能计数器数据可能会过时数分钟。

## 使用诊断配置文件启用性能计数器

使用以下过程在 Azure 应用程序中启用性能计数器。

## 先决条件

本节假定已将诊断监视器导入应用程序，并且已将诊断配置文件（SDK 2.4 及更低版本中的 diagnostics.wadcfg，或 SDK 2.5 及更高版本中的 diagnostics.wadcfgx）添加到 Visual Studio 解决方案。请参阅[在 Azure 云服务和虚拟机中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics)中的步骤 1 和 2，以获取详细信息。

## 步骤 1：通过性能计数器收集和存储数据

在将诊断文件添加到 Visual Studio 解决方案后，可以在 Azure 应用程序中配置性能计数器数据的收集和存储。通过将性能计数器添加到诊断文件可做到这一点。首先，在实例中收集诊断数据（包括性能计数器）。然后，将该数据保留到 Azure 表服务中的 WADPerformanceCountersTable 表，因此还需要在应用程序中指定存储帐户。如果在计算模拟器中本地测试应用程序，则也可在存储模拟器中本地存储诊断数据。必须先转到 [Azure 经典管理门户](http://manage.windowsazure.cn)并创建存储帐户，然后才能存储诊断数据。最佳做法是将存储帐户定位到与 Azure 应用程序相同的地理位置，以避免支付外部带宽成本并减少延迟。

### 将性能计数器添加到诊断文件

可以使用多个计数器。下例介绍了建议用于 Web 和辅助角色监视的几个性能计数器。

打开诊断文件（在 SDK 2.4 及更低版本中为 diagnostics.wadcfg，在 SDK 2.5 及更高版本中为 diagnostics.wadcfgx），将以下代码添加到 DiagnosticMonitorConfiguration 元素：

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

       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Interop(_Global_)# of marshalling" sampleRate="PT30S" />
       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Loading(_Global_)\% Time Loading" sampleRate="PT30S" />
       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR LocksAndThreads(_Global_)\Contention Rate / sec" sampleRate="PT30S" />
       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Memory(_Global_)# Bytes in all Heaps" sampleRate="PT30S" />
       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Networking(_Global_)\Connections Established" sampleRate="PT30S" />
       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Remoting(_Global_)\Remote Calls/sec" sampleRate="PT30S" />
       <PerformanceCounterConfiguration counterSpecifier="\.NET CLR Jit(_Global_)\% Time in Jit" sampleRate="PT30S" />
    </PerformanceCounters>


bufferQuotaInMB 特性，指定可用于数据收集类型（Azure 日志、IIS 日志等）的文件系统存储的最大量。默认值为 0。在达到配额时，将删除最旧的数据，因为将添加新数据。所有 bufferQuotaInMB 属性的总和必须大于 OverallQuotaInMB 特性的值。有关确定收集诊断数据需要多少存储的更多详细讨论，请参阅[有关开发 Azure 应用程序的问题排查最佳实践](https://msdn.microsoft.com/zh-cn/library/windowsazure/hh771389.aspx)中的“设置 WAD”一节。

scheduledTransferPeriod 特性，指定计划数据传输之间的时间间隔，取整为最接近的分钟数。在下例中，将其设置为 PT30M（30 分钟）。将传输时间段设置为较小的值（如 1 分钟）会对应用程序的生产性能产生负面影响，但在测试时有助于快速查看诊断信息。计划的传输时间段应足够小以确保不会在实例上覆盖诊断数据，但也应足够大以确保不会影响应用程序的性能。

counterSpecifier 属性指定要收集的性能计数器。sampleRate 属性指定性能计数器的采样速度（此示例中为 30 秒）。

添加要收集的性能计数器后，将更改保存到诊断文件。接下来，需要指定将诊断数据保留到的存储帐户。

### 指定存储帐户

若要将诊断信息保存到 Azure 存储帐户，则必须在服务配置 (ServiceConfiguration.cscfg) 文件中指定连接字符串。

对于 Azure SDK 2.5，可在 diagnostics.wadcfgx 文件中指定存储帐户。

>[AZURE.NOTE] 这些说明仅适用于 Azure SDK 2.4 及更低版本。对于 Azure SDK 2.5，可在 diagnostics.wadcfgx 文件中指定存储帐户。

设置连接字符串：

1. 使用常用文本编辑器打开 ServiceConfiguration.Cloud.cscfg 文件，并设置存储的连接字符串。在 Azure 经典管理门户中，在存储帐户仪表板的“管理密钥”下找到 *AccountName* 和 *AccountKey* 值。

    ```
    <ConfigurationSettings>
       <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=<name>;AccountKey=<key>;EndpointSuffix=core.chinacloudapi.cn"/>
    </ConfigurationSettings>
    ```
2. 保存 ServiceConfiguration.Cloud.cscfg 文件。

3. 打开 ServiceConfiguration.Local.cscfg 文件，并确认已将 UseDevelopmentStorage 设置为 true。

    ```
    <ConfigurationSettings>
      <Settingname="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true"/>
    </ConfigurationSettings>
    ```
现已设置连接字符串，在部署应用程序时，该应用程序会将诊断数据保存到存储帐户。
4. 保存并生成项目，然后部署应用程序。

## 步骤 2：（可选）创建自定义性能计数器

除了预定义的性能计数器外，还可以添加自己的自定义性能计数器来监视 Web 角色或辅助角色。自定义性能计数器可用于跟踪和监视应用程序特定行为，并且可使用提升的权限在启动任务、Web 角色或辅助角色中创建或删除该计数器。

Azure 诊断代理会在启动后一分钟刷新 .wadcfg 文件中的性能计数器配置。如果在 OnStart 方法中创建自定义性能计数器，并且启动任务的执行时间超过 1 分钟，则自定义性能计数器不会在 Azure 诊断代理尝试加载时创建。在这种情况下，Azure 诊断可正确捕获除自定义性能计数器以外的所有诊断数据。若要解决此问题，可以在启动任务中创建性能计数器，也可以在创建性能计数器后将部分启动任务工作转移到 OnStart 方法。

执行下列步骤创建名为“\\MyCustomCounterCategory\\MyButton1Counter”的简单自定义性能计数器：

1. 打开应用程序的服务定义文件 (CSDEF)。
2. 将 Runtime 元素添加到 WebRole 或 WorkerRole 元素，可允许使用提升的权限执行：

    ```
    <runtime executioncontext="elevated"/>
    ```
3. 保存文件。
4. 打开诊断文件（在 SDK 2.4 及更低版本中为 diagnostics.wadcfg，在 SDK 2.5 及更高版本中为 diagnostics.wadcfgx），将以下代码添加到 DiagnosticMonitorConfiguration 元素： 

    ```
    <PerformanceCounters bufferQuotaInMB="0" scheduledTransferPeriod="PT30M">
     <PerformanceCounterConfiguration counterSpecifier="\MyCustomCounterCategory\MyButton1Counter" sampleRate="PT30S"/>
    </PerformanceCounters>
    ```
5. 保存文件。
6. 先使用角色的 OnStart 方法创建自定义性能计数器类别，然后再调用 base.OnStart。以下 C# 示例创建自定义类别（如果尚不存在）：


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

7. 更新应用程序中的计数器。下例更新 Button1\_Click 事件的自定义性能计数器：

    ```
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
    ```
8. 保存文件。  

现在，将由 Azure 诊断监视器收集自定义性能计数器数据。

## 步骤 3：查询性能计数器数据

部署并运行应用程序后，诊断监视器将开始收集性能计数器并将该数据保存到 Azure 存储。可使用 Cerebrata 提供的工具（如 Visual Studio 中的服务器资源管理器、[Azure 存储资源管理器](http://azurestorageexplorer.codeplex.com)或 [Azure 诊断管理器](http://www.cerebrata.com/Products/AzureDiagnosticsManager/Default.aspx)）查看 WADPerformanceCountersTable 表中的性能计数器数据。你还可以使用 [C#](/documentation/articles/storage-dotnet-how-to-use-tables/)、[Java](/documentation/articles/storage-java-how-to-use-table-storage/)、[Node.js](/documentation/articles/storage-nodejs-how-to-use-table-storage/)、[Python](/documentation/articles/storage-python-how-to-use-table-storage/)、[Ruby](/documentation/articles/storage-ruby-how-to-use-table-storage/) 或 [PHP](/documentation/articles/storage-php-how-to-use-table-storage/) 以编程方式查询表服务。

以下 C# 示例显示针对 WADPerformanceCountersTable 表的简单查询，并将诊断数据保存到 CSV 文件。将性能计数器保存到 CSV 文件后，可以使用 Microsoft Excel 或其他一些工具中的图形功能可视化数据。请务必添加对 Microsoft.WindowsAzure.Storage.dll（包含在 2012 年 10 月版的用于 .NET 的 Azure SDK 及更高版本中）的引用。程序集安装在 %Program Files%\\Microsoft SDKs\\Azure.NET SDK\\version-num\\ref\\ 目录中。

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

    // Get a reference to the storage account using the connection string.  You can also use the development
    // storage account (Storage Emulator) for local debugging.
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


实体映射到使用派生自 **TableEntity** 的自定义类的 C# 对象。以下代码定义表示 **WADPerformanceCountersTable** 表中性能计数器的实体类。


    public class PerformanceCountersEntity : TableEntity
    {
       public long EventTickCount { get; set; }
       public string DeploymentId { get; set; }
       public string Role { get; set; }
       public string RoleInstance { get; set; }
       public string CounterName { get; set; }
       public double CounterValue { get; set; }
    }




<!---HONumber=Mooncake_Quality_Review_1215_2016-->