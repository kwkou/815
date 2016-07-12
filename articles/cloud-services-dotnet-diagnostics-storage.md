<properties
  pageTitle="在 Azure 存储空间中存储和查看诊断数据 | Azure"
  description="将 Azure 诊断数据转到 Azure 存储空间并进行查看"
  services="cloud-services"
  documentationCenter=".net"
  authors="rboucher"
  manager="jwhit"
  editor="tysonn" />
<tags
  ms.service="cloud-services"
  ms.date="02/20/2016"
  wacn.date="04/06/2016" />

# 在 Azure 存储空间中存储和查看诊断数据

诊断数据不会永久存储，除非你将其传输到 Azure 存储模拟器或 Azure 存储空间中。一旦位于存储空间中，诊断数据就可以使用提供的工具之一进行查看。

## 指定存储帐户

指定你想要在 ServiceConfiguration.cscfg 文件中使用的存储帐户。帐户信息被定义为配置设置中的连接字符串。以下示例显示的是在 Visual Studio 中针对新的云服务项目创建的默认连接字符串：


```
	<ConfigurationSettings>
	   <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
	</ConfigurationSettings>
```

你可以更改此连接字符串，为 Azure 存储帐户提供帐户信息。

根据所要收集的诊断数据的类型，Azure 诊断使用 Blob 服务或表服务。下表显示保留的数据源及其格式。

|数据源|存储格式|
|---|---|
|Azure 日志|表|
|IIS 7.0 日志|Blob|
|Azure 诊断基础结构日志|表|
|失败请求跟踪日志|Blob|
|Windows 事件日志|表|
|性能计数器|表|
|故障转储|Blob|
|自定义错误日志|Blob|

## 传输诊断数据

对于 SDK 2.5 及更高版本，可以通过配置文件来请求传输诊断数据。你可以根据配置中的指定，按计划的时间间隔来传输诊断数据。

对于 SDK 2.4 及更低版本，你可以请求通过配置文件来传输诊断数据，也可以通过编程方式来进行。编程方式还可以进行按需传输。


>[AZURE.IMPORTANT] 将诊断数据传输到 Azure 存储帐户时会产生费用，具体取决于你的诊断数据所使用的存储资源。

## 存储诊断数据

日志数据存储在使用以下名称的 Blob 或表存储中：

**表**

- **WadLogsTable** - 使用跟踪侦听器以代码编写的日志。

- **WADDiagnosticInfrastructureLogsTable** - 诊断监视器和配置更改。

- **WADDirectoriesTable** - 诊断监视器监视的目录。这包括 IIS 日志、IIS 失败请求日志和自定义目录。在“容器”字段中指定 blob 日志文件的位置，在 RelativePath 字段中指定 blob 的名称。AbsolutePath 字段指示文件的位置和名称，就像文件是存在于 Azure 虚拟机上一样。

- **WADPerformanceCountersTable** - 性能计数器。

- **WADWindowsEventLogsTable** - Windows 事件日志。

**Blob**

- **wad-control-container** -（仅适用于 SDK 2.4 及更低版本）包含用于控制 Azure 诊断的 XML 配置文件。

- **wad-iis-failedreqlogfiles** - 包含 IIS 失败请求日志中的信息。

- **wad-iis-logfiles** - 包含有关 IIS 日志的信息。

- **"custom"** - 一个自定义容器，其所基于的配置目录受诊断监视器监视。此 blob 容器的名称将在 WADDirectoriesTable 中指定。

## 用于查看诊断数据的工具
将数据传输到存储空间以后，可以使用多个工具来查看这些数据。例如：

- Visual Studio 中的服务器资源管理器 - 如果你安装了 Azure Tools for Microsoft Visual Studio，则可以在服务器资源管理器中使用“Azure 存储空间”节点从你的 Azure 存储帐户查看只读 Blob 和表数据。既可以从本地存储模拟器帐户显示数据，也可以从为 Azure 创建的存储帐户显示数据。有关详细信息，请参阅[使用服务器资源管理器浏览和管理存储资源](/documentation/articles/vs-azure-tools-storage-resources-server-explorer-browse-manage/)。

- [Neudesic 的 Azure 存储资源管理器](http://azurestorageexplorer.codeplex.com)是一个有用的图形用户界面工具，可用于检查和更改 Azure 存储项目中的数据，包括 Azure 应用程序的日志。

- [Azure Management Studio](http://www.cerebrata.com/products/azure-management-studio/introduction) 随附 Azure 诊断管理器，可让你查看、下载和管理 Azure 中运行的应用程序收集的诊断数据。

## 后续步骤

[使用 Azure 诊断跟踪云服务应用程序中的流](/documentation/articles/cloud-services-dotnet-diagnostics-trace-flow/)

<!---HONumber=Mooncake_0328_2016-->
