<properties
   pageTitle="使用 ASP.NET Web API 来与服务通信 | Azure"
   description="了解如何在 Reliable Services API 中将 ASP.NET Web API 与 OWIN 自托管配合使用来实现服务通信。"
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# 入门：Service Fabric Web API 服务与 OWIN 自托管 | Microsoft Azure

Azure Service Fabric 让你有权决定你希望你的服务如何与用户以及相互之间进行通信。本教程重点介绍如何在 Service Fabric 的 Reliable Services API 中将 ASP.NET Web API 与适用于 .NET 的开放式 Web 接口 (OWIN) 自托管配置使用来实现服务通信。我们将深入探讨 Reliable Services 可插式通信 API。此外，还将在循序渐进的示例中使用 Web API 演示如何设置自定义通信侦听器。


## Service Fabric 中的 Web API 简介

ASP.NET Web API 是一个常用的强大框架，用于在 .NET Framework 之上构建 HTTP API。如果你不熟悉此框架，请参阅 [ASP.NET Web API 2 入门](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)以深入了解。

Service Fabric 中的 Web API 是你熟知并喜爱的相同 ASP.NET Web API。不同之处在于如何*托管* Web API 应用程序。你将不会使用 Microsoft Internet Information Services (IIS)。为了更好地了解区别，我们将它划分为两个部分：

 1. Web API 应用程序（包括控制器和模型）
 2. 主机（Web 服务器，通常是 IIS）

Web API 应用程序本身不会更改。它与你可能已在过去编写的 Web API 应用程序没什么不同，你应能够转移大部分应用程序代码。但是，如果要在 IIS 上托管，托管应用程序的位置可能与你的习惯稍有不同。在我们深入探讨托管部分之前，我们从更熟悉的部分开始：Web API 应用程序。


## 创建应用程序

首先在 Visual Studio 2015 中创建一个新的 Service Fabric 应用程序（具有单个无状态服务）：

![创建新的 Service Fabric 应用程序](./media/service-fabric-reliable-services-communication-webapi/webapi-newproject.png)

使用 Web API 的无状态服务有 Visual Studio 模板可供使用。在本教程中，我们将生成一个项目，这就是你选择此模板时所会得到的结果。此时，你可以从无状态服务 Web API 着手并依照指示进行，或从空的无状态服务着手并从头开始生成。

![创建单个无状态服务](./media/service-fabric-reliable-services-communication-webapi/webapi-newproject2.png)

第一步是为 Web API 拉入一些 NuGet 包。我们要使用的包是 Microsoft.AspNet.WebApi.OwinSelfHost。此包中包括所有所需的 Web API 包和宿主包。这在后面会非常重要。

![使用 NuGet 包管理器创建 Web API](./media/service-fabric-reliable-services-communication-webapi/webapi-nuget.png)

安装了这些包之后，我们可以开始构建基本的 Web API 项目结构。如果你使用过 Web API，则项目结构应看上去非常熟悉。首先添加 `Controllers` 目录和简单的值控制器：

**ValuesController.cs**

```csharp
using System.Collections.Generic;
using System.Web.Http;
    
namespace WebService.Controllers
{
    public class ValuesController : ApiController
    {
        // GET api/values 
        public IEnumerable<string> Get()
        {
            return new string[] { "value1", "value2" };
        }

        // GET api/values/5 
        public string Get(int id)
        {
            return "value";
        }

        // POST api/values 
        public void Post([FromBody]string value)
        {
        }

        // PUT api/values/5 
        public void Put(int id, [FromBody]string value)
        {
        }

        // DELETE api/values/5 
        public void Delete(int id)
        {
        }
    }
}

```

接着，在项目根目录中添加一个 Startup 类，用于注册路由、格式化程序和任何其他配置设置。这也是 Web API 插入到主机中的位置（会在以后再次重新访问）。

**Startup.cs**

```csharp
using System.Web.Http;
using Owin;

namespace WebService
{
    public static class Startup
    {
        public static void ConfigureApp(IAppBuilder appBuilder)
        {
            // Configure Web API for self-host. 
            HttpConfiguration config = new HttpConfiguration();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );

            appBuilder.UseWebApi(config);
        }
    }
}
```

应用程序部分就是这样。此时，我们只是设置了基本的 Web API 项目布局。到当前为止，看起来应该与过去可能已编写的 Web API 项目或基本的 Web API 模板有太多不同。你的业务逻辑同往常一样放入控制器和模型中。

现在我们该针对托管执行什么操作以便我们可以实际运行它？

## 服务托管

在 Service Fabric 中，服务在服务主机进程（运行服务代码的可执行文件）中运行。当你使用 Reliable Services API 编写服务时，服务项目只编译成注册服务类型并运行代码的可执行文件。当你在 .NET 中的 Service Fabric 上编写服务时，在大多数情况下都是如此。如果你打开无状态服务项目中的 Program.cs，则应该看到：

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using Microsoft.ServiceFabric.Services.Runtime;

internal static class Program
{
    private static void Main()
    {
        try
        {
            ServiceRuntime.RegisterServiceAsync("WebServiceType",
                context => new WebService(context)).GetAwaiter().GetResult();

            ServiceEventSource.Current.ServiceTypeRegistered(Process.GetCurrentProcess().Id, typeof(WebService).Name);

            // Prevents this host process from terminating so services keeps running. 
            Thread.Sleep(Timeout.Infinite);
        }
        catch (Exception e)
        {
            ServiceEventSource.Current.ServiceHostInitializationFailed(e.ToString());
            throw;
        }
    }
}

```

如果这看上去疑似控制台应用程序的入口点，这是因为它是。

有关服务主机进程和服务注册的更多详细信息已超出本文的范围。但是现在请务必了解服务代码已在它自身的进程中运行。

## 自托管 Web API 与 OWIN 主机

考虑到 Web API 应用程序代码在其自己的进程中托管，你该如何将它挂接到 Web 服务器？ 进入 [OWIN](http://owin.org/)。OWIN 只是 .NET Web 应用程序与 Web 服务器之间的协定。传统上使用 ASP.NET（最高为 MVC 5）时，Web 应用程序通过 System.Web 与 IIS 紧密耦合。但是，Web API 实现 OWIN，这使你可以编写一个与托管其自身的 Web 服务器分离的 Web 应用程序。因此，你可以使用可在自己的进程中启动的自托管 OWIN Web 服务器。这样完全符合我们前面提到的 Service Fabric 托管模型。

在本文中，我们将使用 Katana 作为 Web API 应用程序的 OWIN 主机。Katana 是一种开源 OWIN 主机实现。

> [AZURE.NOTE] 若要了解有关 Katana 的详细信息，请转到 [Katana 站点](http://www.asp.net/aspnet/overview/owin-and-katana/an-overview-of-project-katana)。有关如何使用 Katana 自托管 Web API 的快速概述，请参阅[使用 OWIN 自托管 ASP.NET Web API 2](http://www.asp.net/web-api/overview/hosting-aspnet-web-api/use-owin-to-self-host-web-api)。


## 设置 Web 服务器

Reliable Services API 提供通信入口点，可在其中插入通信堆栈，以便用户和客户端能够连接到服务：



	protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
	{
    	...
	}	



Web 服务器（以及可能在将来使用的任何其他通信堆栈，如 WebSockets）应使用 ICommunicationListener 接口与系统正确集成。这样做的原因会在后续步骤中表现得更明显。

首先创建一个名为 OwinCommunicationListener 的类，它实现 ICommunicationListener：

**OwinCommunicationListener.cs**

```csharp
using Microsoft.Owin.Hosting;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Owin;
using System;
using System.Fabric;
using System.Globalization;
using System.Threading;
using System.Threading.Tasks;

namespace WebService
{
    public class OwinCommunicationListener : ICommunicationListener
    {
        public void Abort()
        {
        }

        public Task CloseAsync(CancellationToken cancellationToken)
        {
        }

        public Task<string> OpenAsync(CancellationToken cancellationToken)
        {
        }
    }
}
```

ICommunicationListener 接口提供了三个方法来为服务管理通信侦听器：

 - OpenAsync。开始侦听请求。
 - CloseAsync。停止侦听请求，完成任何正在进行的请求，然后正常关闭。
 - Abort。取消所有内容并立即停止。

若要开始操作，请为侦听器运行所需的项目添加私有类成员。这些成员会通过构造函数初始化，并在后面设置侦听 URL 时使用。

```csharp
public class OwinCommunicationListener : ICommunicationListener
{
    private readonly ServiceEventSource eventSource;
    private readonly Action<IAppBuilder> startup;
    private readonly ServiceContext serviceContext;
    private readonly string endpointName;
    private readonly string appRoot;

    private IDisposable webApp;
    private string publishAddress;
    private string listeningAddress;

    public OwinCommunicationListener(Action<IAppBuilder> startup, ServiceContext serviceContext, ServiceEventSource eventSource, string endpointName)
        : this(startup, serviceContext, eventSource, endpointName, null)
    {
    }

    public OwinCommunicationListener(Action<IAppBuilder> startup, ServiceContext serviceContext, ServiceEventSource eventSource, string endpointName, string appRoot)
    {
        if (startup == null)
        {
            throw new ArgumentNullException(nameof(startup));
        }

        if (serviceContext == null)
        {
            throw new ArgumentNullException(nameof(serviceContext));
        }

        if (endpointName == null)
        {
            throw new ArgumentNullException(nameof(endpointName));
        }

        if (eventSource == null)
        {
            throw new ArgumentNullException(nameof(eventSource));
        }

        this.startup = startup;
        this.serviceContext = serviceContext;
        this.endpointName = endpointName;
        this.eventSource = eventSource;
        this.appRoot = appRoot;
    }
   

    ...

```

## 实现 OpenAsync

若要设置 Web 服务器，需要两项信息：

 - URL 路径前缀。尽管是可选的，不过最好现在对此进行设置，以便你可以在应用程序中安全地托管多个 Web 服务。
 - 端口。

在我们获取 Web 服务器的端口之前，请务必了解 Service Fabric 提供了一个应用程序层，该层充当应用程序与用于运行它的底层操作系统之间的缓冲区。因此，Service Fabric 提供了一种方法来为服务配置终结点。Service Fabric 可确保终结点可供服务使用。这样，你就无需在基础 OS 环境中自行配置终结点。可以轻松地在不同环境中托管 Service Fabric 应用程序，而不用对应用程序进行任何更改。（例如，可以在 Azure 或自己的数据中心内托管同一个应用程序。）

在 PackageRoot\\ServiceManifest.xml 中配置 HTTP 终结点：



	<Resources>
    	<Endpoints>
        	<Endpoint Name="ServiceEndpoint" Type="Input" Protocol="http" Port="8281" />
    	</Endpoints>
	</Resources>



此步骤很重要，因为服务主机进程要在受限制的凭据（在 Windows 上的网络服务）之下运行。这意味着服务并没有自行设置 HTTP 终结点的访问权限。通过使用终结点配置，Service Fabric 知道要为服务侦听的 URL 设置适当的访问控制列表 (ACL)。Service Fabric 还提供了一个标准位置用于配置终结点。


返回到 OwinCommunicationListener.cs 中，现在可以开始实现 OpenAsync。从此处启动 Web 服务器。首先，获取终结点信息，并创建服务将侦听的 URL。视侦听器用于无状态服务还是有状态服务而定，URL 会有所不同。如果用于有状态服务，侦听器必须针对它所侦听的每个有状态服务副本创建唯一的地址。如果用于无状态服务，此地址可以简单得多。

```csharp
public Task<string> OpenAsync(CancellationToken cancellationToken)
{
    var serviceEndpoint = this.serviceContext.CodePackageActivationContext.GetEndpoint(this.endpointName);
    int port = serviceEndpoint.Port;

    if (this.serviceContext is StatefulServiceContext)
    {
        StatefulServiceContext statefulServiceContext = this.serviceContext as StatefulServiceContext;

        this.listeningAddress = string.Format(
            CultureInfo.InvariantCulture,
            "http://+:{0}/{1}{2}/{3}/{4}",
            port,
            string.IsNullOrWhiteSpace(this.appRoot)
                ? string.Empty
                : this.appRoot.TrimEnd('/') + '/',
            statefulServiceContext.PartitionId,
            statefulServiceContext.ReplicaId,
            Guid.NewGuid());
    }
    else if (this.serviceContext is StatelessServiceContext)
    {
        this.listeningAddress = string.Format(
            CultureInfo.InvariantCulture,
            "http://+:{0}/{1}",
            port,
            string.IsNullOrWhiteSpace(this.appRoot)
                ? string.Empty
                : this.appRoot.TrimEnd('/') + '/');
    }
    else
    {
        throw new InvalidOperationException();
    }
    
    ...

```

请注意，此处使用了“http://+”。这是为了确保 Web 服务器侦听所有可用的地址，包括 localhost、FQDN 和计算机 IP。

OpenAsync 实现是为何以 ICommunicationListener 形式实现 Web 服务器（或任何通信堆栈），而不是仅仅直接在服务中从 `RunAsync()` 打开它的最重要原因之一。OpenAsync 的返回值是 Web 服务器所侦听的地址。当此地址返回到系统时，它会向服务注册此地址。Service Fabric 提供了一个 API，使客户端和其他服务随后可以通过服务名称请求此地址。这一点很重要，因为服务地址不是静态的。服务为了资源平衡和可用性目的在群集中移动。这是允许客户端为服务解析侦听地址的机制。

明确这一点后，OpenAsync 会启动 Web 服务器，并返回它所侦听的地址。请注意它侦听“http://+”，但在 OpenAsync 返回地址之前，“+”会替换为它当前所处节点的 IP 或 FQDN。此方法所返回的地址就是向系统注册的地址。它也是客户端和其他服务在请求服务地址时所看到的地址。要使客户端可以正确连接到它，它们在地址中需要实际 IP 或 FQDN。

```csharp
    ...

    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

    try
    {
        this.eventSource.ServiceMessage(this.serviceContext, "Starting web server on " + this.listeningAddress);

        this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));

        this.eventSource.ServiceMessage(this.serviceContext, "Listening on " + this.publishAddress);

        return Task.FromResult(this.publishAddress);
    }
    catch (Exception ex)
    {
        this.eventSource.ServiceMessage(this.serviceContext, "Web server failed to open. " + ex.ToString());

        this.StopWebServer();

        throw;
    }
}

```

请注意，这会引用在构造函数中传入到 OwinCommunicationListener 的 Startup 类。此启动实例由 Web 服务器用于启动 Web API 应用程序。

将来在你运行应用程序时，`ServiceEventSource.Current.ServiceMessage()` 行会出现在“诊断事件”窗口中，以确认 Web 服务器已成功启动。

## 实现 CloseAsync 和 Abort

最后，实现 CloseAsync 和 Abort 以停止 Web 服务器。可以通过释放在 OpenAsync 过程中创建的服务器句柄来停止 Web 服务器。


	public Task CloseAsync(CancellationToken cancellationToken)
	{
    	this.eventSource.ServiceMessage(this.serviceContext, "Closing web server");
            
    	this.StopWebServer();

    	return Task.FromResult(true);
	}

	public void Abort()
	{
    	this.eventSource.ServiceMessage(this.serviceContext, "Aborting web server");
    
    	this.StopWebServer();
	}

	private void StopWebServer()
	{
    	if (this.serverHandle != null)
    	{
        	try
        	{
            	this.serverHandle.Dispose();
        	}
        	catch (ObjectDisposedException)
        	{
            	// no-op
        	}
    	}
	}


在此实现示例中，CloseAsync 和 Abort 都只是停止 Web 服务器。你可以选择在 CloseAsync 中运行更妥善协调的 Web 服务器关机。例如，关机可以等待正在进行的请求在返回之前完成。

## 启动 Web 服务器

你现在已准备好创建并返回 OwinCommunicationListener 的实例以启动 Web 服务器。返回到 Service 类 (Service.cs) 中，重写 `CreateServiceInstanceListeners()` 方法：

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext => 
            new OwinCommunicationListener(Startup.ConfigureApp, serviceContext, ServiceEventSource.Current, "ServiceEndpoint"))
    };
}
```

这是 Web API 应用程序和 OWIN 主机最后相会之处。为主机 (OwinCommunicationListener) 指定应用程序实例（通过 startup 的 Web API）。然后，Service Fabric 将管理其生命周期。通常任何通信堆栈都可以遵循这一相同模式。

## 将其放在一起

在此示例中，你无需在 `RunAsync()` 方法中执行任何操作，从而可以简单地删除重写。

最终的服务实现应该非常简单。它只需创建通信侦听器：

```csharp
using System.Collections.Generic;
using System.Fabric;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

namespace WebService
{
    internal sealed class WebService : StatelessService
    {
        public WebService(StatelessServiceContext context)
            : base(context)
        { }

        protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
        {
            return new ServiceInstanceListener[]
            {
                new ServiceInstanceListener(serviceContext => 
                    new OwinCommunicationListener(Startup.ConfigureApp, serviceContext, ServiceEventSource.Current, "ServiceEndpoint"))
            };
        }
    }
}
```

完整的 `OwinCommunicationListener` 类：

```csharp
using System;
using System.Diagnostics;
using System.Fabric;
using System.Globalization;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Owin.Hosting;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Owin;

namespace WebService
{
    internal class OwinCommunicationListener : ICommunicationListener
    {
    private readonly ServiceEventSource eventSource;
    private readonly Action<IAppBuilder> startup;
    private readonly ServiceContext serviceContext;
    private readonly string endpointName;
    private readonly string appRoot;

    private IDisposable webApp;
    private string publishAddress;
    private string listeningAddress;

    public OwinCommunicationListener(Action<IAppBuilder> startup, ServiceContext serviceContext, ServiceEventSource eventSource, string endpointName)
        : this(startup, serviceContext, eventSource, endpointName, null)
    {
    }

    public OwinCommunicationListener(Action<IAppBuilder> startup, ServiceContext serviceContext, ServiceEventSource eventSource, string endpointName, string appRoot)
    {
        if (startup == null)
        {
            throw new ArgumentNullException(nameof(startup));
        }

        if (serviceContext == null)
        {
            throw new ArgumentNullException(nameof(serviceContext));
        }

        if (endpointName == null)
        {
            throw new ArgumentNullException(nameof(endpointName));
        }

        if (eventSource == null)
        {
            throw new ArgumentNullException(nameof(eventSource));
        }

        this.startup = startup;
        this.serviceContext = serviceContext;
        this.endpointName = endpointName;
        this.eventSource = eventSource;
        this.appRoot = appRoot;
    }

        public bool ListenOnSecondary { get; set; }

        public Task<string> OpenAsync(CancellationToken cancellationToken)
        {
            var serviceEndpoint = this.serviceContext.CodePackageActivationContext.GetEndpoint(this.endpointName);
            int port = serviceEndpoint.Port;

            if (this.serviceContext is StatefulServiceContext)
            {
                StatefulServiceContext statefulServiceContext = this.serviceContext as StatefulServiceContext;

                this.listeningAddress = string.Format(
                    CultureInfo.InvariantCulture,
                    "http://+:{0}/{1}{2}/{3}/{4}",
                    port,
                    string.IsNullOrWhiteSpace(this.appRoot)
                        ? string.Empty
                        : this.appRoot.TrimEnd('/') + '/',
                    statefulServiceContext.PartitionId,
                    statefulServiceContext.ReplicaId,
                    Guid.NewGuid());
            }
            else if (this.serviceContext is StatelessServiceContext)
            {
                this.listeningAddress = string.Format(
                    CultureInfo.InvariantCulture,
                    "http://+:{0}/{1}",
                    port,
                    string.IsNullOrWhiteSpace(this.appRoot)
                        ? string.Empty
                        : this.appRoot.TrimEnd('/') + '/');
            }
            else
            {
                throw new InvalidOperationException();
            }

    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

    try
    {
        this.eventSource.ServiceMessage(this.serviceContext, "Starting web server on " + this.listeningAddress);

        this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));

        this.eventSource.ServiceMessage(this.serviceContext, "Listening on " + this.publishAddress);

        return Task.FromResult(this.publishAddress);
    }
    catch (Exception ex)
    {
        this.eventSource.ServiceMessage(this.serviceContext, "Web server failed to open. " + ex.ToString());

        this.StopWebServer();

        throw;
    }
}

        public Task CloseAsync(CancellationToken cancellationToken)
        {
            this.eventSource.ServiceMessage(this.serviceContext, "Closing web server");

            this.StopWebServer();

            return Task.FromResult(true);
        }

        public void Abort()
        {
            this.eventSource.ServiceMessage(this.serviceContext, "Aborting web server");

            this.StopWebServer();
        }

        private void StopWebServer()
        {
            if (this.webApp != null)
            {
                try
                {
                    this.webApp.Dispose();
                }
                catch (ObjectDisposedException)
                {
                    // no-op
                }
            }
        }
    }
}

```

所有部分都准备就绪后，项目现在应类似于具有 Reliable Services API 入口点和 OWIN 主机的典型 Web API 应用程序：


![包含 Reliable Services API 入口点和 OWIN 主机的 Web API](./media/service-fabric-reliable-services-communication-webapi/webapi-projectstructure.png)

## 运行 Web 浏览器并通过它进行连接

如果尚未这样做，请[设置开发环境](/documentation/articles/service-fabric-get-started/)。


现在，你可以生成并部署你的服务。在 Visual Studio 中按 **F5** 以生成并部署应用程序。在“诊断事件”窗口中，你应看到一条消息，指示已在 http://localhost:8281/ 上打开了 Web 服务器。


![Visual Studio 诊断事件窗口](./media/service-fabric-reliable-services-communication-webapi/webapi-diagnostics.png)

> [AZURE.NOTE] 如果计算机上的另一个进程已经打开该端口，你可能会在此看到错误消息。这表示无法打开侦听器。如果是这种情况，请在 ServiceManifest.xml 中的终结点配置中尝试使用不同端口。


服务运行之后，打开浏览器并导航到 [http://localhost:8281/api/values](http://localhost:8281/api/values) 对它进行测试。

## 将其扩展

扩展无状态 Web 应用通常意味着添加更多计算机并在其上运行 Web 应用。每当向群集添加新节点时，Service Fabric 的业务流程引擎可以为你执行此操作。创建无状态服务的实例时，可以指定要创建的实例数。Service Fabric 将该数目的实例放置在群集中的节点上。它可以确保不会在任一节点上创建多个实例。还可以通过为实例计数指定 **-1**，指示 Service Fabric 始终在每个节点上创建一个实例。这可保证每当添加节点以扩展群集时，都会在新节点上创建无状态服务的实例。此值是服务实例的属性，因此它是在你创建服务实例时设置的：可以通过 PowerShell 设置：

```powershell

New-ServiceFabricService -ApplicationName "fabric:/WebServiceApplication" -ServiceName "fabric:/WebServiceApplication/WebService" -ServiceTypeName "WebServiceType" -Stateless -PartitionSchemeSingleton -InstanceCount -1

```

也可以在 Visual Studio 无状态服务项目中定义默认服务时设置：



	<DefaultServices>
  		<Service Name="WebService">
    		<StatelessService ServiceTypeName="WebServiceType" InstanceCount="-1">
      		<SingletonPartition />
    	</StatelessService>
  		</Service>
	</DefaultServices>



有关如何创建应用程序和服务实例的详细信息，请参阅[部署应用程序](/documentation/articles/service-fabric-deploy-remove-applications/)。

## 后续步骤

[使用 Visual Studio 调试 Service Fabric 应用程序](/documentation/articles/service-fabric-debugging-your-application/)

<!---HONumber=Mooncake_0503_2016-->