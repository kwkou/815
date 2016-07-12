<properties
   pageTitle="将 Elasticsearch 用作 Service Fabric 应用程序跟踪存储 | Azure"
   description="介绍 Service Fabric 应用程序如何使用 Elasticsearch 和 Kibana 来存储、索引和搜索应用程序跟踪（日志）"
   services="service-fabric"
   documentationCenter=".net"
   authors="karolz-ms"
   manager="adegeo"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/16/2016"
   wacn.date="07/04/2016"/>

# 将 Elasticsearch 用作 Service Fabric 应用程序跟踪存储
## 介绍
本文介绍 [Azure Service Fabric](/documentation/services/service-fabric/) 应用程序如何使用 **Elasticsearch** 和 **Kibana** 来存储、索引和搜索应用程序跟踪。[Elasticsearch](https://www.elastic.co/guide/index.html) 是开源的、分布式和可缩放的实时搜索和分析引擎，很适合执行此任务。它可以安装在 Azure 中运行的 Windows 和 Linux 虚拟机上。Elasticsearch 可以非常高效地处理使用**Windows 事件跟踪 (ETW)** 之类的技术所生成的*结构化*跟踪。

Service Fabric 运行时会使用 ETW 来获取诊断信息（跟踪）。它也是 Service Fabric 应用程序获取其诊断信息的建议方法。这可让运行时提供和应用程序提供的跟踪之间相互关联，使故障排除更轻松。Visual Studio 中的 Service Fabric 项目模板包含日志记录 API（基于 .NET **EventSource** 类），该 API 默认情况下会发出 ETW 跟踪。有关使用 ETW 的 Service Fabric 应用程序跟踪的一般概述，请参阅[在本地计算机开发安装过程中监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)。

需要在 Service Fabric 群集节点上实时（当应用程序正在运行时）捕获跟踪并发送至 Elasticsearch 终结点，Elasticsearch 中才会显示跟踪。跟踪捕获有两个主要选项：

+ **进程内跟踪捕获**  
应用程序（更准确来说是服务进程）负责将诊断数据送出到跟踪存储 (Elasticsearch)。

+ **进程外跟踪捕获**  
单独的代理捕获来自一个或多个服务进程的跟踪，并发送至跟踪存储。

下面介绍如何在 Azure 上设置 Elasticsearch，讨论这两种捕获选项的优缺点，并说明如何配置 Service Fabric 服务以将数据发送到 Elasticsearch。


## 在 Azure 上设置 Elasticsearch
若要在 Azure 上设置 Elasticsearch 服务，最直接的方法是通过 [**Azure Resource Manager 模板**](/documentation/articles/resource-group-overview/)。Azure 快速入门模板存储库提供完整的 [Elasticsearch 快速入门 Azure Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates/tree/master/elasticsearch)。此模板会针对缩放单位使用不同的存储帐户（节点组）。它也可以设置具有不同配置和附加各种数量的数据磁盘的个别客户端与服务器节点。

我们在此处将使用另一个模板（名为 **ES-MultiNode**，来自 [Azure 诊断工具存储库](https://github.com/Azure/azure-diagnostics-tools)）。此模板比较容易使用，会创建受 HTTP 基本身份验证保护的 Elasticsearch 群集。在继续操作之前，请先从 GitHub 将存储库下载到你的计算机（通过克隆存储库或下载 zip 文件）。ES-MultiNode 模板位于具有相同名称的文件夹中。

### 准备计算机以运行 Elasticsearch 安装脚本
若要使用 ES-MultiNode 模板，最简单的方式是通过提供的 Azure PowerShell 脚本，称为 `CreateElasticSearchCluster`。若要使用此脚本，你需要安装 PowerShell 模块和名为 **openssl** 的工具。需要后者，才能创建可用于远程管理 Elasticsearch 群集的 SSH 密钥。

请注意，`CreateElasticSearchCluster` 脚本主要是为了从 Windows 计算机轻松使用 ES-MultiNode 模板。可以在非 Windows 计算机上使用此模板，但这已超出本文的范围。

1. 如果你尚未安装它们，请安装 [**Azure PowerShell 模块**](http://aka.ms/webpi-azps)。出现提示时，请单击“运行”，再单击“安装”。需要 Azure PowerShell 1.3 或更高版本。

2. [**Git for Windows**](http://www.git-scm.com/downloads) 的分发版中包含 **openssl** 工具。如果你尚未安装 [Git for Windows](http://www.git-scm.com/downloads)，请立即安装。（可使用默认安装选项）。

3. 假设 Git 已安装，但未包含在系统路径中，请打开 Azure PowerShell 窗口并运行下列命令：

    ```powershell
    $ENV:PATH += ";<Git installation folder>\usr\bin"
    $ENV:OPENSSL_CONF = "<Git installation folder>\usr\ssl\openssl.cnf"
    ```

    用你计算机上的 Git 位置替换 `<Git installation folder>`；默认值为 **"C:\\Program Files\\Git"**。请注意第一个路径开头的分号字符。

4. 确保你已登录到 Azure（通过 [Add-AzureRmAccount](https://msdn.microsoft.com/zh-cn/library/mt619267.aspx) cmdlet），并且已选择应该用来创建弹性搜索群集的订阅。你可以使用 `Get-AzureRmContext` 和 `Get-AzureRmSubscription` cmdlet 来验证是否选择了正确的订阅。

5. 如果你尚未将当前目录更改为 ES-MultiNode 文件夹，请更改。

### 运行 CreateElasticSearchCluster 脚本
运行脚本之前，打开 `azuredeploy-parameters.json` 文件，确认或提供脚本参数的值。提供下列参数：

|参数名称 |说明|
|-----------------------  |--------------------------|
|dnsNameForLoadBalancerIP |此名称用于为弹性搜索群集创建公开可见 DNS 名称（通过将 Azure 区域域附加到提供的名称）。例如，如果此参数值为“myBigCluster”，而所选的 Azure 区域是美国西部，则为群集产生的 DNS 名称会是 myBigCluster.chinaeast.chinacloudapp.cn。<br /><br />对于与弹性搜索群集相关联的许多项目的名称，例如数据节点名称，此名称也会成为这些名称的根。|
|adminUsername |用于管理弹性搜索群集的管理员帐户的名称（将自动生成对应的 SSH 密钥）。|
|dataNodeCount |弹性搜索群集中的节点数。当前版本的脚本不会区分数据与查询节点；所有节点会同时扮演这两种角色。默认为 3 个节点。|
|dataDiskSize |将分配给每个数据节点的数据磁盘大小（以 GB 为单位）。每个节点将会收到 4 个数据磁盘，专供弹性搜索服务使用。|
|region |应该放置弹性搜索群集的 Azure 区域的名称。|
|esUserName |将配置为可访问 ES 群集的用户的用户名（受限于 HTTP 基本身份验证）。密码不是参数文件的一部分，必须在调用 `CreateElasticSearchCluster` 脚本时提供。|
|vmSizeDataNodes |弹性搜索群集节点的 Azure 虚拟机大小。默认为 Standard\_D2。|

你现在可以开始运行脚本。发出以下命令：

```powershell
CreateElasticSearchCluster -ResourceGroupName <es-group-name> -Region <azure-region> -EsPassword <es-password>
```

其中

|脚本参数名称 |说明|
|-----------------------  |--------------------------|
|`<es-group-name>` |将包含所有弹性搜索群集资源的 Azure 资源组的名称|
|`<azure-region>` |应该放置弹性搜索群集的 Azure 区域的名称|         
|`<es-password>` |弹性搜索用户的密码|

>[AZURE.NOTE] 如果从 Test-AzureResourceGroup cmdlet 收到 NullReferenceException，表示你忘记登录到 Azure (`Add-AzureRmAccount`)。

如果运行脚本时发生错误，而且你确定错误起因于错误的模板参数值，请更正参数文件，然后以不同资源组名再次运行脚本。你也可以将 `-RemoveExistingResourceGroup` 参数添加到脚本调用，以重复使用相同的资源组名，并让脚本清理旧的资源组。

### 运行 CreateElasticSearchCluster 脚本的结果
运行 `CreateElasticSearchCluster` 脚本之后会创建下列主要项目。为清楚起见，我们假设你已使用“myBigCluster”作为 `dnsNameForLoadBalancerIP` 参数的值，而且你创建群集的区域为美国西部。

|项目|名称、位置及备注|
|----------------------------------|----------------------------------|
|用于远程管理的 SSH 密钥 |myBigCluster.key 文件（在运行 CreateElasticSearchCluster 的目录中）。<br /><br />这是可用于连接到群集中的管理节点及（通过管理节点）连接到数据节点的密钥。|
|管理节点 |myBigCluster-admin.chinaeast.chinacloudapp.cn <br /><br />这是用于远程 Elasticsearch 群集管理的专用 VM，也是唯一允许外部 SSH 连接的 VM。它与所有 Elasticsearch 群集节点一样在相同的虚拟网络上运行，但不会运行 Elasticsearch 服务。|
|数据节点 |myBigCluster1 … myBigCluster*N* <br /><br />运行 Elasticsearch 和 Kibana 服务的数据节点。你可以通过 SSH 连接到每个节点，但只能通过管理节点。|
|Elasticsearch 群集 |http://myBigCluster.chinaeast.chinacloudapp.cn/es/<br /><br />以上是 Elasticsearch 群集的主要终结点（请注意 /es 后缀）。它由基本 HTTP 身份验证提供保护（凭据为 ES-MultiNode 模板的指定 esUserName/esPassword 参数）。群集也已安装头插件 (http://myBigCluster.chinaeast.chinacloudapp.cn/es/_plugin/head) 来进行基本群集管理。|
|Kibana 服务 |http://myBigCluster.chinaeast.chinacloudapp.cn<br /><br />Kibana 服务设置为显示已创建的 Elasticsearch 群集的数据。它由与群集本身相同的身份验证凭据提供保护。|

## 进程内与进程外跟踪捕获
在简介中，我们曾提过收集诊断数据的两种基本方法：进程内和进程外。这两种方法各有优缺点。

**进程内跟踪捕获**的优点包括：

1. “易于配置及部署”

    * 诊断数据收集配置只是应用程序配置的一部分。将其与应用程序的其余部分始终保持“同步”很简单。

    * 可轻松基于每个应用程序或每个服务进行配置。

    * 进程外跟踪捕获通常需要单独部署和配置诊断代理，这是额外的管理任务，也是潜在的错误来源。特定代理技术通常只允许每个虚拟机（节点）有一个代理实例。这表示该节点上运行的所有应用程序和服务会共享诊断配置集合的配置。

2. “灵活性”

    * 只要客户端库支持目标数据存储系统，应用程序就可以将数据发送至任何需要的地方。可以根据需要添加新的接收器。

    * 可以实现复杂的捕获、筛选和数据聚合规则。

    * 进程外跟踪捕获通常受限于代理支持的数据接收器。有些代理是可扩展的。

3. “访问内部应用程序数据与上下文”

    * 应用程序/服务进程内运行的诊断子系统可以轻松地随着上下文信息而扩展跟踪。

    * 在进程外方法中，必须通过 Windows 事件跟踪之类的某种进程间通信机制将数据发送至代理。这可能会造成额外的限制。

**进程外跟踪捕获**的优点包括：

1. “能够监视应用程序并收集故障转储”

    * 如果应用程序启动失败或崩溃，进程内跟踪捕获可能不会成功。独立代理有更好的机会捕获重要的故障排除信息。<br /><br />

2. “成熟度、稳定性和已证实的性能”

    * 平台供应商所开发的代理（例如 Azure 诊断代理）都经过严格的测试和实战锻炼。

    * 执行进程内跟踪捕获时，必须确保从应用程序程序发送诊断数据的活动不会干扰应用程序的主要任务，或引起计时或性能问题。独立运行的代理不太容易发生这些问题，通常也特别设计为限制它对系统的影响。

当然，你可以结合这两种方法的优点。事实上，它可能是许多应用程序的最佳解决方案。

此处我们将使用 **Microsoft.Diagnostic.Listeners 库**和进程内跟踪捕获，将数据从 Service Fabric 应用程序发送到 Elasticsearch 群集。

## 使用 Listeners 库将诊断数据发送到 Elasticsearch
Microsoft.Diagnostic.Listeners 库是 PartyCluster 示例 Service Fabric 应用程序的一部分。使用方式：

1. 从 GitHub 下载 [PartyCluster 示例](https://github.com/Azure-Samples/service-fabric-dotnet-management-party-cluster)。

2. 从 PartyCluster 示例目录中，将 Microsoft.Diagnostics.Listeners 和 Microsoft.Diagnostics.Listeners.Fabric 项目（整个文件夹）复制到应将数据发送到 Elasticsearch 的应用程序的解决方案文件夹。

3. 打开目标解决方案，右键单击“解决方案资源管理器”中的解决方案节点，然后选择“添加现有项目”。将 Microsoft.Diagnostics.Listeners 项目添加到解决方案中。针对 Microsoft.Diagnostics.Listeners.Fabric 项目重复相同的步骤。

4. 添加从服务项目到两个已添加项目的项目引用。（应将数据发送到 Elasticsearch 的每个服务都应该引用 Microsoft.Diagnostics.EventListeners 和 Microsoft.Diagnostics.EventListeners.Fabric）。

    ![对 Microsoft.Diagnostics.EventListeners 和 Microsoft.Diagnostics.EventListeners.Fabric 库的项目引用][1]

### Service Fabric 正式版和 Microsoft.Diagnostics.Tracing NuGet 包
使用 Service Fabric 正式版（2016 年 3 月 31 日发布的 2.0.135）生成的应用程序面向 **.NET Framework 4.5.2**。这是 Azure 在 GA 版发布时支持的最高 .NET Framework 版本。可惜，这个版本的框架缺少 Microsoft.Diagnostics.Listeners 库所需的某些 EventListener API。因为 EventSource（Fabric 应用程序中形成日志记录 API 基础的组件）和 EventListener 紧密结合，每个使用 Microsoft.Diagnostics.Listeners 库的项目都必须使用替代的 EventSource 实现。此实现由 Microsoft 创作的 **Microsoft.Diagnostics.Tracing NuGet 包**所提供。此包与框架中包含的 EventSource 完全后向兼容，因此，除了更改引用的命名空间，应该不需要更改任何代码。

若要开始使用 Microsoft.Diagnostics.Tracing 的 EventSource 类实现，请针对需要将数据发送到 Elasticsearch 的每个服务项目执行下列步骤：

1. 右键单击服务项目，然后选择“管理 NuGet 包”。

2. 切换到 nuget.org 包源（如果尚未选择），搜索“Microsoft.Diagnostics.Tracing”。

3. 安装 `Microsoft.Diagnostics.Tracing.EventSource` 包（及其依赖项）。

4. 打开服务项目中的 **ServiceEventSource.cs** 或 **ActorEventSource.cs** 文件，并将文件顶端的 `using System.Diagnostics.Tracing` 指令替换为 `using Microsoft.Diagnostics.Tracing` 指令。

当 Azure 支持 **.NET Framework 4.6** 时，就不需要这些步骤。

### Elasticsearch 侦听器实例化和配置
将诊断数据发送到 Elasticsearch 所需的最后一个步骤，就是创建 `ElasticSearchListener` 的实例并使用 Elasticsearch 连接数据来配置它。侦听器会自动捕获所有通过服务项目中定义的 EventSource 类引发的事件。它必须在服务的生存期内保持活动状态，所以创建它的最佳位置是在服务初始化代码中。以下是无状态服务的初始化代码在完成必要更改后的样子（以 `****` 开头的注释指出新增的部分）：

```csharp
using System;
using System.Diagnostics;
using System.Fabric;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.ServiceFabric.Services.Runtime;

// **** Add the following directives
using Microsoft.Diagnostics.EventListeners;
using Microsoft.Diagnostics.EventListeners.Fabric;

namespace Stateless1
{
    internal static class Program
    {
        /// <summary>
        /// This is the entry point of the service host process.
        /// </summary>        
        private static void Main()
        {
            try
            {
                // **** Instantiate ElasticSearchListener
                var configProvider = new FabricConfigurationProvider("ElasticSearchEventListener");
                ElasticSearchListener esListener = null;
                if (configProvider.HasConfiguration)
                {
                    esListener = new ElasticSearchListener(configProvider);
                }

                // The ServiceManifest.XML file defines one or more service type names.
                // Registering a service maps a service type name to a .NET type.
                // When Service Fabric creates an instance of this service type,
                // an instance of the class is created in this host process.

                ServiceRuntime.RegisterServiceAsync("Stateless1Type", 
                    context => new Stateless1(context)).GetAwaiter().GetResult();

                ServiceEventSource.Current.ServiceTypeRegistered(Process.GetCurrentProcess().Id, typeof(Stateless1).Name);

                // Prevents this host process from terminating so services keep running.
                Thread.Sleep(Timeout.Infinite);

                // **** Ensure that the ElasticSearchListner instance is not garbage-collected prematurely
                GC.KeepAlive(esListener);
            }
            catch (Exception e)
            {
                ServiceEventSource.Current.ServiceHostInitializationFailed(e);
                throw;
            }
        }
    }
}
```

Elasticsearch 连接数据应该放在服务配置文件 (**PackageRoot\\Config\\Settings.xml**) 中的单独节中。该节的名称必须与传递给 `FabricConfigurationProvider` 构造函数的值对应，例如：


	<Section Name="ElasticSearchEventListener">
  		<Parameter Name="serviceUri" Value="http://myBigCluster.chinaeast.chinacloudapp.cn/es/" />
  		<Parameter Name="userName" Value="(ES user name)" />
  		<Parameter Name="password" Value="(ES user password)" />
  		<Parameter Name="indexNamePrefix" Value="myapp" />
	</Section>

`serviceUri`、`userName` 和 `password` 的值分别对应于 Elasticsearch 群集终结点地址、Elasticsearch 用户名和密码。`indexNamePrefix` 是 Elasticsearch 索引的前缀；Microsoft.Diagnostics.Listeners 库会每天创建其数据的新索引。

### 验证
就这么简单！ 现在每当服务运行时，就会开始将跟踪发送至配置中指定的 Elasticsearch 服务。如果要验证这一点，你可以打开与目标 Elasticsearch 实例相关联的 Kibana UI（在示例中，网页地址是 http://myBigCluster.chinaeast.chinacloudapp.cn/）， 检查 `ElasticSearchListener` 实例的索引（具有所选的名称前缀）已确实创建并填充了数据。

![显示 PartyCluster 应用程序事件的 Kibana][2]

## 后续步骤
- [深入了解诊断和监视 Service Fabric 服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)

<!--Image references-->
[1]: ./media/service-fabric-diagnostics-how-to-use-elasticsearch/listener-lib-references.png
[2]: ./media/service-fabric-diagnostics-how-to-use-elasticsearch/kibana.png

<!---HONumber=Mooncake_0523_2016-->