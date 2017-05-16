<properties
    pageTitle="使用 ASP.NET Core 与服务通信 | Azure"
    description="了解如何在无状态和有状态 Reliable Services 中使用 ASP.NET Core。"
    services="service-fabric"
    documentationcenter=".net"
    author="vturecek"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="8aa4668d-cbb6-4225-bd2d-ab5925a868f2"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="required"
    ms.date="03/22/2017"
    wacn.date="05/15/2017"
    ms.author="vturecek"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="d259a0a42948c9a9b3caaf88dc66fa1e9cb581a8"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="aspnet-core-in-service-fabric-reliable-services"></a>Service Fabric Reliable Services 中的 ASP.NET Core

ASP.NET Core 是新的开源跨平台框架，用于构建现代基于云的连接 Internet 的应用程序，如 Web 应用、IoT 应用和移动后端。 虽然 ASP.NET Core 应用可在 .NET Core 或完整的 .NET Framework 上运行，但 Service Fabric 服务当前只能在完整的 .NET Framework 上运行。 这意味着在构建 ASP.NET Core Service Fabric 服务时，仍必须以完整的 .NET Framework 为目标。

在 Service Fabric 中可通过两种不同方法使用 ASP.NET Core：
 - **作为来宾可执行文件托管**。 这主要用于在 Service Fabric 上运行现有 ASP.NET Core 应用程序，无需更改代码。
 - **在 Reliable Service 内部运行**。 这可改善与 Service Fabric 运行时的集成，实现有状态的 ASP.NET Core 服务。

本文的其余部分说明如何借助 Service Fabric SDK 提供的 ASP.NET Core 集成组件在 Reliable Service 内部使用 ASP.NET Core。 

> [AZURE.NOTE]
>本文的其余部分假定你熟悉 ASP.NET Core 中的托管。 若要了解有关在 ASP.NET Core 中托管的详细信息，请参阅：[在 ASP.NET Core 中托管的简介](https://docs.microsoft.com/aspnet/core/fundamentals/hosting)。

> [AZURE.NOTE]
> 若要在 Visual Studio 2015 中使用 ASP.NET Core 开发 Reliable Services，则需要安装 [.NET Core VS 2015 Tooling Preview 2](https://www.microsoft.com/net/download/core)。

## <a name="service-fabric-service-hosting"></a>Service Fabric 服务托管
在 Service Fabric 中，服务的一个或多个实例和/或副本在*服务主机进程*（运行服务代码的可执行文件）中运行。 服务作者拥有服务主机进程，Service Fabric 将为服务作者激活并监视此进程。

传统的 ASP.NET（最高为 MVC 5）通过 System.Web.dll 与 IIS 紧密耦合。 ASP.NET Core 在 Web 服务器和 Web 应用程序之间提供分隔。 这使 Web 应用程序可在不同 Web 服务器之间移植，并且还允许 Web 服务器*自托管*，这意味着你可以在自己的进程（而不是由 IIS 等专用 Web 服务器软件拥有的进程）中启动 Web 服务器。 

若要合并 Service Fabric 服务和 ASP.NET，无论是作为来宾可执行文件或是在 Reliable Service 中，必须能够在服务主机进程内启动 ASP.NET。 可借助 ASP.NET Core 的自托管功能执行此操作。

## <a name="hosting-aspnet-core-in-a-reliable-service"></a>在 Reliable Service 中托管 ASP.NET Core
通常情况下，自托管 ASP.NET Core 应用程序将在应用程序的入口点创建 WebHost，如 `Program.cs` 中的 `static void Main()` 方法。 在这种情况下，WebHost 的生命周期绑定到进程的生命周期中。

![在进程中托管 ASP.NET Core][0]

但是，应用程序入口点并不是在 Reliable Service 中创建 WebHost 的正确位置，因为应用程序入口点仅用于向 Service Fabric 运行时注册服务类型，以便它能创建该服务类型的实例。 应在 Reliable Service 中创建 WebHost。 在服务主机进程中，服务实例和/或副本可以完成多个生命周期。 

Reliable Service 实例由派生自 `StatelessService` 或 `StatefulService` 的服务类表示。 服务的通信堆栈包含在服务类中的 `ICommunicationListener` 实现内。 `Microsoft.ServiceFabric.Services.AspNetCore.*` NuGet 包包含 `ICommunicationListener` 的实现，这些实现可启动和管理 Reliable Service 中 Kestrel 或 WebListener 的 ASP.NET Core WebHost。

![在 Reliable Service 中托管 ASP.NET Core][1]

## <a name="aspnet-core-icommunicationlisteners"></a>ASP.NET Core ICommunicationListeners
`Microsoft.ServiceFabric.Services.AspNetCore.*` NuGet 包中 Kestrel 和 WebListener 的 `ICommunicationListener` 实现具有类似的使用模式，但针对每个 Web 服务器所执行的操作略有不同。 

这两种通信侦听器都能提供采用以下参数的构造函数：
 - **`ServiceContext serviceContext`**：包含有关运行中的服务信息的 `ServiceContext` 对象。
 - **`string endpointName`**：ServiceManifest.xml 中 `Endpoint` 配置的名称。 以下是两种通信侦听器的主要区别：WebListener **需要** `Endpoint` 配置，而 Kestrel 不需要。
 - **`Func<string, AspNetCoreCommunicationListener, IWebHost> build`**：你实现的 lambda，在其中创建和返回 `IWebHost`。 这允许你按通常在 ASP.NET Core 应用程序中使用的方法配置 `IWebHost`。 Lambda 提供为你生成的 URL，具体取决于你使用的 Service Fabric 集成选项和你提供的 `Endpoint` 配置。 然后可修改 URL 或直接将其按原样用于启动 Web 服务器。

## <a name="service-fabric-integration-middleware"></a>Service Fabric 集成中间件
`Microsoft.ServiceFabric.Services.AspNetCore` NuGet 包包含添加 Service Fabric 可识别的中间件的 `IWebHostBuilder` 上的 `UseServiceFabricIntegration` 扩展方法。 此中间件将 Kestrel 或 WebListener `ICommunicationListener` 配置为向 Service Fabric 命名服务注册唯一的服务 URL，然后验证客户端请求，以确保客户端连接到了正确的服务。 在 Service Fabric 等共享主机环境中，多个 Web 应用程序可能在同一物理计算机或虚拟机上运行，但不使用唯一的主机名，为了防止客户端错误地连接到错误的服务，此操作是必需的。 后续部分将对此方案进行详细说明。

### <a name="a-case-of-mistaken-identity"></a>错误标识示例
服务副本（无论哪种协议）侦听唯一的 IP:port 组合。 服务副本开始侦听 IP:port 终结点后，它将向 Service Fabric 命名服务报告该终结点地址，并被该命名服务中的客户端或其他服务发现。 如果服务使用动态分配的应用程序端口，服务副本可能恰巧使用同一物理计算机或虚拟机上的以前其他服务所使用的相同 IP:port 终结点。 这可能会导致客户端错误地连接到错误的服务。 出现以下事件序列时可能会出现此情况：

 1. 服务 A 通过 HTTP 侦听 10.0.0.1:30000。 
 2. 客户端解析服务 A 并获取地址 10.0.0.1:30000
 3. 服务 A 移动到其他节点。
 4. 服务 B 放置在 10.0.0.1 并恰巧使用了同一端口 30000。
 5. 客户端尝试使用缓存地址 10.0.0.1:30000 连接到服务 A。
 6. 客户端现已成功连接到服务 B，但未意识到已连接到错误的服务。

这可能导致在随机时间出现 bug，并且很难诊断。 

### <a name="using-unique-service-urls"></a>使用唯一的服务 URL
若要防止此情况，服务可向具有唯一标识符的命名服务发布终结点，并在客户端请求期间验证该唯一标识符。 这是非恶意租户受信任环境中的服务之间的协作操作。 这不会在恶意租户环境中提供安全的服务身份验证。

在受信任的环境中，由 `UseServiceFabricIntegration` 方法自动添加的中间件可对已发布到命名服务的地址追加唯一标识符，并在每次请求时验证该标识符。 如果标识符不匹配，该中间件将立即返回 HTTP 410 Gone 响应。

使用动态分配的端口的服务应使用此中间件。

使用固定唯一端口的服务在协作环境中不存在此问题。 固定唯一端口通常用于面向外部的服务，此类服务需要可供客户端应用程序连接到的已知端口。 例如，大多数面向 Internet 的 Web 应用程序将使用端口 80 或 443 进行 Web 浏览器连接。 在此情况下，不应启用唯一标识符。

下图显示了启用中间件时的请求流：

![Service Fabric ASP.NET Core 集成][2]

Kestrel 和 WebListener `ICommunicationListener` 实现以完全相同的方式使用此机制。 尽管 WebListener 可使用基本 *http.sys* 端口共享功能基于唯一 URL 路径内部区分请求，但 WebListener `ICommunicationListener` 实现*不会*使用此功能，因为这会导致上述方案中出现 HTTP 503 和 HTTP 404 错误状态代码。 这进而使客户端很难确定错误原因，因为 HTTP 503 和 HTTP 404 通常用于指示其他错误。 因此，Kestrel 和 WebListener `ICommunicationListener` 实现将在 `UseServiceFabricIntegration` 扩展方法提供的中间件上执行标准化，使客户端只需对 HTTP 410 响应执行服务终结点重新解析操作。

## <a name="weblistener-in-reliable-services"></a>Reliable Services 中的 WebListener
通过导入 **Microsoft.ServiceFabric.AspNetCore.WebListener** NuGet 包，可在 Reliable Service 中使用 WebListener。 此包包含 `WebListenerCommunicationListener` - `ICommunicationListener` 的实现，此实现允许使用 WebListener Web 服务器在 Reliable Service 内部创建 ASP.NET Core WebHost。

将在 [Windows HTTP Server API](https://msdn.microsoft.com/zh-cn/library/windows/desktop/aa364510(v=vs.85).aspx) 上构建 WebListener。 这将使用 IIS 所用的 *http.sys* 内核驱动程序处理 HTTP 请求，并将其路由到运行 Web 应用程序的进程。 这可允许同一物理计算机或虚拟机上的多个进程在同一端口上托管 Web 应用程序，通过唯一 URL 路径或主机名来消除歧义。 Service Fabric 在同一群集中托管多个网站时，这些功能非常有用。

下图说明了 WebListener 如何在 Windows 上使用 *http.sys* 内核驱动程序进行端口共享：

![http.sys][3]

### <a name="weblistener-in-a-stateless-service"></a>无状态服务中的 WebListener
若要在无状态服务中使用 `WebListener`，需替代 `CreateServiceInstanceListeners` 方法并返回 `WebListenerCommunicationListener` 实例：

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new ServiceInstanceListener[]
        {
            new ServiceInstanceListener(serviceContext =>
                new WebListenerCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                    new WebHostBuilder()
                        .UseWebListener()
                        .ConfigureServices(
                            services => services
                                .AddSingleton<StatelessServiceContext>(serviceContext))
                        .UseContentRoot(Directory.GetCurrentDirectory())
                        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                        .UseStartup<Startup>()
                        .UseUrls(url)
                        .Build()))
        };
    }

### <a name="weblistener-in-a-stateful-service"></a>有状态服务中的 WebListener

由于基本 *http.sys* 端口共享功能所具有的的复杂性，当前不能在有状态服务中使用 `WebListenerCommunicationListener`。 有关详细信息，请参阅以下关于 WebListener 动态端口分配的部分。 对于有状态服务，建议使用 Kestrel Web 服务器。

### <a name="endpoint-configuration"></a>终结点配置

使用 Windows HTTP Server API 的 Web 服务器（包括 WebListener）需要使用 `Endpoint` 配置。 使用 Windows HTTP Server API 的 Web 服务器首先必须保留带有 *http.sys* 的 URL（通常可使用 [netsh](https://msdn.microsoft.com/zh-cn/library/windows/desktop/cc307236(v=vs.85).aspx) 工具实现）。 此操作需要提升的权限，默认情况下你的服务不具备此权限。 用于 *ServiceManifest.xml* 中 `Endpoint` 配置的 `Protocol` 属性的“Http”或“https”选项，可专门用于指示 Service Fabric 运行时使用[*强通配符*](https://msdn.microsoft.com/zh-cn/library/windows/desktop/aa364698(v=vs.85).aspx) URL 前缀代表你注册带有 *http.sys* 的 URL。

例如，若要保留服务的 `http://+:80`，则应在 ServiceManifest.xml 中使用以下配置：

    <ServiceManifest ... >
        ...
        <Resources>
            <Endpoints>
                <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" />
            </Endpoints>
        </Resources>

    </ServiceManifest>

并且必须将终结点名称传递到 `WebListenerCommunicationListener` 构造函数：

     new WebListenerCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
     {
         return new WebHostBuilder()
             .UseWebListener()
             .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
             .UseUrls(url)
             .Build();
     })

#### <a name="use-weblistener-with-a-static-port"></a>将 WebListener 与静态端口配合使用
若要将 WebListener 与静态端口配合使用，需在 `Endpoint` 配置中提供端口号：

      <Resources>
        <Endpoints>
          <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
        </Endpoints>
      </Resources>

#### <a name="use-weblistener-with-a-dynamic-port"></a>将 WebListener 与动态端口配合使用
若要将 WebListener 与动态分配端口配合使用，需在 `Endpoint` 配置中省略 `Port` 属性：

      <Resources>
        <Endpoints>
          <Endpoint Protocol="http" Name="ServiceEndpoint" />
        </Endpoints>
      </Resources>

请注意，`Endpoint` 配置分配的动态端口仅为*每个主机进程*提供一个端口。 当前的 Service Fabric 托管模型允许在同一进程中托管多个服务实例和/或副本，这意味着当通过 `Endpoint` 配置分配时，每个实例/副本将共享相同的端口。 多个 WebListener 实例可使用基本 *http.sys* 端口共享功能共享一个端口，但 `WebListenerCommunicationListener` 不支持此做法，因为这会增加客户端请求的复杂性。 对于使用动态端口，建议使用 Kestrel Web 服务器。

## <a name="kestrel-in-reliable-services"></a>Reliable Services 中的 Kestrel
通过导入 **Microsoft.ServiceFabric.AspNetCore.Kestrel** NuGet 包，可在 Reliable Service 中使用 Kestrel。 此包包含 `KestrelCommunicationListener` - `ICommunicationListener` 的实现，此实现允许使用 Kestrel Web 服务器在 Reliable Service 内部创建 ASP.NET Core WebHost。

Kestrel 是基于 libuv 的 ASP.NET Core 的跨平台 Web 服务器，libuv 是跨平台异步 I/O 库。 与 WebListener 不同，Kestrel 不使用集中式终结点管理器，如 *http.sys*。 与 WebListener 的另一个区别在于，Kestrel 不支持多个进程之间共享端口。 Kestrel 的每个实例必须使用唯一端口。

![Kestrel][4]

### <a name="kestrel-in-a-stateless-service"></a>无状态服务中的 Kestrel
若要在无状态服务中使用 `Kestrel`，需替代 `CreateServiceInstanceListeners` 方法并返回 `KestrelCommunicationListener` 实例：

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new ServiceInstanceListener[]
        {
            new ServiceInstanceListener(serviceContext =>
                new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                    new WebHostBuilder()
                        .UseKestrel()
                        .ConfigureServices(
                            services => services
                                .AddSingleton<StatelessServiceContext>(serviceContext))
                        .UseContentRoot(Directory.GetCurrentDirectory())
                        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                        .UseStartup<Startup>()
                        .UseUrls(url)
                        .Build();
                ))
        };
    }

### <a name="kestrel-in-a-stateful-service"></a>有状态服务中的 Kestrel
若要在有状态服务中使用 `Kestrel`，需替代 `CreateServiceReplicaListeners` 方法并返回 `KestrelCommunicationListener` 实例：

    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        return new ServiceReplicaListener[]
        {
            new ServiceReplicaListener(serviceContext =>
                new KestrelCommunicationListener(serviceContext, (url, listener) =>
                    new WebHostBuilder()
                        .UseKestrel()
                        .ConfigureServices(
                             services => services
                                 .AddSingleton<StatefulServiceContext>(serviceContext)
                                 .AddSingleton<IReliableStateManager>(this.StateManager))
                        .UseContentRoot(Directory.GetCurrentDirectory())
                        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                        .UseStartup<Startup>()
                        .UseUrls(url)
                        .Build();
                ))
        };
    }

此示例中为 WebHost 依赖关系注入容器提供 `IReliableStateManager` 的单一实例。 这不是必需的，但通过此操作，可在 MVC 控制器操作方法中使用 `IReliableStateManager` 和 Reliable Collections。

请注意，有状态服务中**不会**为 `KestrelCommunicationListener` 提供`Endpoint` 配置名称。 后续部分会对此进行详细说明。

### <a name="endpoint-configuration"></a>终结点配置
使用 Kestrel 时不需要 `Endpoint` 配置。 

Kestrel 是简单的独立 Web 服务器；与 WebListener（或 HttpListener）不同，它不需要 *ServiceManifest.xml* 中的 `Endpoint` 配置，因为它在启动前不需要注册 URL。 

#### <a name="use-kestrel-with-a-static-port"></a>将 Kestrel 和静态端口配合使用
可在 ServiceManifest.xml 的 `Endpoint` 配置中配置静态端口，以使其与 Kestrel 配合使用。 虽然这不是必需的，但这样做有两个潜在好处：
 1. 如果此端口不在应用程序端口范围内，则会由 Service Fabric 通过 OS 防火墙将其打开。
 2. 通过 `KestrelCommunicationListener` 提供的 URL 将使用此端口。

      <Resources>
        <Endpoints>
          <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
        </Endpoints>
      </Resources>

如果已配置 `Endpoint`，则其名称必须传递到 `KestrelCommunicationListener` 构造函数： 

    new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) => ...

如果不使用 `Endpoint` 配置，则在 `KestrelCommunicationListener` 构造函数中省略此名称。 在此情况下将使用动态端口。 有关详细信息，请参阅下一部分。

#### <a name="use-kestrel-with-a-dynamic-port"></a>将 Kestrel 和动态端口配合使用
Kestrel 无法使用 ServiceManifest.xml 中 `Endpoint` 配置的自动端口分配，因为 `Endpoint` 配置中的自动端口分配会为每个*主机进程*分配唯一端口，并且单个主机进程可能包含多个 Kestrel 实例。 由于 Kestrel 不支持端口共享，并且每个 Kestrel 实例必须在唯一端口上打开，因此此方案不可行。

若要将 Kestrel 和动态端口分配配合使用，只需完全省略ServiceManifest.xml 中的 `Endpoint` 配置，并且不要将终结点名称传递到 `KestrelCommunicationListener` 构造函数：

    new KestrelCommunicationListener(serviceContext, (url, listener) => ...

在此配置中，`KestrelCommunicationListener` 将自动从应用程序端口范围中选择未使用的端口。

## <a name="scenarios-and-configurations"></a>方案和配置
本部分介绍以下方案，并提供 Web 服务器、端口配置、Service Fabric 集成选项和其他设置的建议组合方式，以使服务正常工作：
 - 外部公开的 ASP.NET Core 无状态服务
 - 仅限内部的 ASP.NET Core 无状态服务
 - 仅限内部的 ASP.NET Core 有状态服务

**外部公开**的服务公开可从群集外部到达的终结点（通常通过负载均衡器）。

**仅限内部**的服务的终结点只能从群集内部到达。

> [AZURE.NOTE]
> 通常不应将有状态服务终结点公开到 Internet。 位于无法识别 Service Fabric 服务解析的负载均衡器（如 Azure 负载均衡器）后的群集将无法公开有状态服务，因为负载均衡器无法找到流量并将其路由到相应有状态服务副本。 

### <a name="externally-exposed-aspnet-core-stateless-services"></a>外部公开的 ASP.NET Core 无状态服务
对于在 Windows 上公开外部、面向 Internet 的 HTTP 终结点的前端服务，建议使用 WebListener Web 服务器。 它能针对攻击提供更好的防护并支持 Kestrel 不支持的功能，例如 Windows 身份验证和端口共享。 

此时不支持将 Kestrel 用作边缘（面向 Internet）服务器。 必须使用 IIS 或 Nginx 等反向代理服务器处理来自公共 Internet 的流量。
 
向 Internet 公开时，无状态服务应使用可通过负载均衡器到达的已知稳定终结点。 这是将为应用程序的用户提供的 URL。 建议采用以下配置：

|  |  | **说明** |
| --- | --- | --- |
| Web 服务器 | WebListener | 如果仅向 Intranet 等受信任的网络公开服务，则可以使用 Kestrel。 否则，WebListener 是首选选项。 |
| 端口配置 | 静态 | 应在 ServiceManifest.xml 的 `Endpoints` 配置中配置已知静态端口，例如为 HTTP 配置 80 或为 HTTPS 配置 443。 |
| ServiceFabricIntegrationOptions | 无 | 配置 Service Fabric 集成中间件时应使用 `ServiceFabricIntegrationOptions.None` 选项，以使服务不会验证传入请求是否具有唯一标识符。 应用程序的外部用户不会知道中间件使用的唯一标识信息。 |
| 实例计数 | -1 | 通常使用情况下，应将实例计数设置设置为“-1”，以使实例在从负载均衡器接收流量的所有节点上可用。 |

如果多个外部公开的服务共享相同的节点集，则应使用唯一但稳定的 URL 路径。 这可以通过修改配置 IWebHost 时提供的 URL 来实现。 请注意这仅适用于 WebListener。

     new WebListenerCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
     {
         url += "/MyUniqueServicePath";
 
         return new WebHostBuilder()
             .UseWebListener()
             ...
             .UseUrls(url)
             .Build();
     })

### <a name="internal-only-stateless-aspnet-core-service"></a>仅限内部的无状态 ASP.NET Core 服务
仅从群集内部调用的无状态服务应使用唯一的 URL 和动态分配的端口，以确保多个服务之间的协作正常进行。 建议采用以下配置：

|  |  | **说明** |
| --- | --- | --- |
| Web 服务器 | Kestrel | 尽管 WebListener 可用于内部无状态服务，但建议使用 Kestrel 服务器，以便允许服务实例共享主机。  |
| 端口配置 | 动态分配 | 有状态服务的多个副本可能会共享主机进程或主机操作系统，因此将需要唯一端口。 |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | 通过动态端口分配，此设置可以防止前面所述的错误标识问题。 |
| InstanceCount | 任意 | 可根据操作服务的需要将实例计数设置设置为任何值。 |

### <a name="internal-only-stateful-aspnet-core-service"></a>仅限内部的有状态 ASP.NET Core 服务
仅从群集内部调用的有状态服务应使用动态分配的端口，以确保多个服务之间的协作正常进行。 建议采用以下配置：

|  |  | **说明** |
| --- | --- | --- |
| Web 服务器 | Kestrel | `WebListenerCommunicationListener` 不能用于副本在其中共享主机进程的有状态服务。 |
| 端口配置 | 动态分配 | 有状态服务的多个副本可能会共享主机进程或主机操作系统，因此将需要唯一端口。 |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | 通过动态端口分配，此设置可以防止前面所述的错误标识问题。 |

## <a name="next-steps"></a>后续步骤
[使用 Visual Studio 调试 Service Fabric 应用程序](/documentation/articles/service-fabric-debugging-your-application/)

<!--Image references-->
[0]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-standalone.png
[1]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-servicefabric.png
[2]:./media/service-fabric-reliable-services-communication-aspnetcore/integration.png
[3]:./media/service-fabric-reliable-services-communication-aspnetcore/httpsys.png
[4]:./media/service-fabric-reliable-services-communication-aspnetcore/kestrel.png