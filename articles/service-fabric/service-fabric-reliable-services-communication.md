<properties
    pageTitle="Reliable Services 通信概述 | Azure"
    description="概述 Reliable Services 通信模型，包括在服务上打开侦听器、解析终结点，以及在服务之间进行通信。"
    services="service-fabric"
    documentationcenter=".net"
    author="vturecek"
    manager="timlt"
    editor="BharatNarasimman" />
<tags
    ms.assetid="36217988-420e-409d-b0a4-e0e875b6eac8"
    ms.service="service-fabric"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="required"
    ms.date="04/07/2017"
    wacn.date="05/15/2017"
    ms.author="vturecek"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="0954b8fa696777db43008bd15f4312f376bf6e8c"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="how-to-use-the-reliable-services-communication-apis"></a>如何使用 Reliable Services 通信 API
“Azure Service Fabric 即平台”完全不受服务间通信的影响。 所有协议和堆栈（从 UDP 到 HTTP）都可接受。 至于服务应以哪种方式通信，完全由服务开发人员选择。 Reliable Services 应用程序框架提供了一些内置的通信堆栈和 API，可用于生成自定义通信组件。

## <a name="set-up-service-communication"></a>设置服务通信
Reliable Services API 为服务通信使用一个简单的接口。 若要打开服务的终结点，只需实现此接口即可：



	public interface ICommunicationListener
	{
	    Task<string> OpenAsync(CancellationToken cancellationToken);

	    Task CloseAsync(CancellationToken cancellationToken);

	    void Abort();
	}


<br/>

    public interface CommunicationListener {
        CompletableFuture<String> openAsync(CancellationToken cancellationToken);

        CompletableFuture<?> closeAsync(CancellationToken cancellationToken);

        void abort();
    }

然后，可以通过在基于服务的类方法重写中返回通信侦听器实现来添加该实现。

对于无状态服务：


	class MyStatelessService : StatelessService
	{
	    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
	    {
	        ...
	    }
	    ...
	}

<br/>

    public class MyStatelessService extends StatelessService {

        @Override
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ...
        }
        ...
    }

对于有状态服务：

> [AZURE.NOTE]
> Java 目前不支持有状态的 Reliable Services。
>
>

	class MyStatefulService : StatefulService
	{
	    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
	    {
	        ...
	    }
	    ...
	}

在这两种情况下，都将返回侦听器的集合。 这可让你的服务通过多个侦听器，可能使用不同的协议在多个终结点上侦听。 例如，你可能有一个 HTTP 侦听器和一个单独的 WebSocket 侦听器。 当客户端请求服务实例或分区的侦听地址时，每个侦听器将获取一个名称，生成的“名称 : 地址”对集合以 JSON 对象的形式表示。

在无状态服务中，重写将返回 ServiceInstanceListeners 的集合。 `ServiceInstanceListener` 包含一个函数，用于创建 `ICommunicationListener(C#) / CommunicationListener(Java)` 并为其提供名称。 对于有状态服务，重写将返回 ServiceReplicaListeners 集合。 这与无状态服务稍有不同，因为 `ServiceReplicaListener` 可以选择在辅助副本上打开 `ICommunicationListener`。 你不仅可以在服务中使用多个通信侦听器，而且还可以指定哪些侦听器要在辅助副本上接受请求，以及哪些侦听器只能在主副本上进行侦听。

例如，你可以创建一个只在主要副本上接受 RPC 调用的 ServiceRemotingListener，并创建另一个可通过 HTTP 在次要副本上接受读取请求的自定义侦听器：


	protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
	{
	    return new[]
	    {
	        new ServiceReplicaListener(context =>
	            new MyCustomHttpListener(context),
	            "HTTPReadonlyEndpoint",
	            true),

	        new ServiceReplicaListener(context =>
	            this.CreateServiceRemotingListener(context),
	            "rpcPrimaryEndpoint",
	            false)
	    };
	}

> [AZURE.NOTE]
> 为一个服务创建多个侦听器时， **必须** 为每个侦听器指定一个唯一名称。
>
>

最后，在[服务清单](/documentation/articles/service-fabric-application-model/)中有关终结点的节下面描述服务所需的终结点。


	<Resources>
	    <Endpoints>
	      <Endpoint Name="WebServiceEndpoint" Protocol="http" Port="80" />
	      <Endpoint Name="OtherServiceEndpoint" Protocol="tcp" Port="8505" />
	    <Endpoints>
	</Resources>



通信侦听器可以从 `ServiceContext` 中的 `CodePackageActivationContext` 访问分配给它的终结点资源。然后侦听器在打开时开始侦听请求。


	var codePackageActivationContext = serviceContext.CodePackageActivationContext;
	var port = codePackageActivationContext.GetEndpoint("ServiceEndpoint").Port;


<br/>

    CodePackageActivationContext codePackageActivationContext = serviceContext.getCodePackageActivationContext();
    int port = codePackageActivationContext.getEndpoint("ServiceEndpoint").getPort();


> [AZURE.NOTE]
> 终结点资源通用于整个服务包，由 Service Fabric 在激活服务包时分配。 托管在同一 ServiceHost 中的多个服务副本可能共享同一个端口。 这意味着通信侦听器应支持端口共享。 实现此目标的一种推荐方法是通信侦听器在生成侦听地址时使用分区 ID 和副本/实例 ID。
>
>

### <a name="service-address-registration"></a>服务地址注册
名为 *命名服务* 的系统服务在 Service Fabric 群集上运行。 命名服务是服务及其地址（服务的每个实例或副本正在其上侦听）的注册机构。 当 `ICommunicationListener(C#) / CommunicationListener(Java)` 的 `OpenAsync(C#) / openAsync(Java)` 方法完成时，它的返回值会在命名服务中注册。 这个在命名服务中发布的返回值是一个字符串，其值完全可以是任何内容。 此字符串值是客户端向命名服务请求服务的地址时看到的内容。

	public Task<string> OpenAsync(CancellationToken cancellationToken)
	{
	    EndpointResourceDescription serviceEndpoint = serviceContext.CodePackageActivationContext.GetEndpoint("ServiceEndpoint");
	    int port = serviceEndpoint.Port;

	    this.listeningAddress = string.Format(
	                CultureInfo.InvariantCulture,
	                "http://+:{0}/",
	                port);
                        
	    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
            
	    this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));
    
	    // the string returned here will be published in the Naming Service.
	    return Task.FromResult(this.publishAddress);
	}

<br/>

    public CompletableFuture<String> openAsync(CancellationToken cancellationToken)
    {
        EndpointResourceDescription serviceEndpoint = serviceContext.getCodePackageActivationContext.getEndpoint("ServiceEndpoint");
        int port = serviceEndpoint.getPort();

        this.publishAddress = String.format("http://%s:%d/", FabricRuntime.getNodeContext().getIpAddressOrFQDN(), port);

        this.webApp = new WebApp(port);
        this.webApp.start();

        /* the string returned here will be published in the Naming Service.
         */
        return CompletableFuture.completedFuture(this.publishAddress);
    }

Service Fabric 提供许多 API，使客户端和其他服务随后可以通过服务名称请求此地址。 这一点很重要，因为服务地址不是静态的。 服务为了资源平衡和可用性目的在群集中移动。 这是允许客户端为服务解析侦听地址的机制。

> [AZURE.NOTE]
> 有关如何用 C# 编写通信侦听器的完整演练，请参阅 [Service Fabric Web API 服务与 OWIN 自托管](/documentation/articles/service-fabric-reliable-services-communication-webapi/)，而在 Java 中，你可以编写自己的 HTTP 服务器实现，请参阅 https://github.com/Azure-Samples/service-fabric-java-getting-started 中的 EchoServer 应用程序示例。
>
>

## <a name="communicating-with-a-service"></a>与服务通信
Reliable Services API 提供以下库，编写与服务通信的客户端。

### <a name="service-endpoint-resolution"></a>服务终结点解析
与服务通信的第一步是解析你想要通信的服务的分区或实例的终结点地址。 `ServicePartitionResolver(C#) / FabricServicePartitionResolver(Java)` 实用工具类是一个基本基元，可帮助客户端在运行时确定服务的终结点。 确定服务终结点的过程在 Service Fabric 术语中称为*服务终结点解析*。

若要连接到群集内的服务，可使用默认设置创建 ServicePartitionResolver。 这是针对大多数情况的建议用法：


	ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();

<br/>

    FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();

若要连接到不同群集中的服务，可利用一组群集网关终结点来创建 ServicePartitionResolver。 请注意，网关终结点只是可用来连接到相同群集的不同终结点。 例如：

    ServicePartitionResolver resolver = new  ServicePartitionResolver("mycluster.cloudapp.chinacloudapi.cn:19000", "mycluster.cloudapp.chinacloudapi.cn:19001");

<br/>

    FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver("mycluster.cloudapp.chinacloudapi.cn:19000", "mycluster.cloudapp.chinacloudapi.cn:19001");

或者，可为 `ServicePartitionResolver` 指定一个函数来创建 `FabricClient`，以便在内部使用：

	public delegate FabricClient CreateFabricClientDelegate();

<br/>

    public FabricServicePartitionResolver(CreateFabricClient createFabricClient) {
    ...
    }

    public interface CreateFabricClient {
        public FabricClient getFabricClient();
    }

`FabricClient` 是用于与 Service Fabric 群集通信以便在群集上实现各种管理操作的对象。 当需要更好地控制服务分区解析程序与群集交互的方式时，这非常有用。 `FabricClient` 会在内部执行缓存，但创建成本通常很高，因此，一定要尽可能重复使用 `FabricClient` 实例。

	ServicePartitionResolver resolver = new  ServicePartitionResolver(() => CreateMyFabricClient());

<br/>

    FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver(() -> new CreateFabricClientImpl());

接下来，使用解析方法来检索服务的地址或已分区服务的服务分区的地址。


	ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();

	ResolvedServicePartition partition =
	    await resolver.ResolveAsync(new Uri("fabric:/MyApp/MyService"), new ServicePartitionKey(), cancellationToken);

<br/>

    FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();

    CompletableFuture<ResolvedServicePartition> partition =
        resolver.resolveAsync(new URI("fabric:/MyApp/MyService"), new ServicePartitionKey());

服务地址可以使用 ServicePartitionResolver 轻松解析，但需要执行更多操作，才能确保解析的地址可正确使用。 客户端必须检测连接尝试是因暂时性错误而失败且可重试（例如，服务已移动或暂时不可用），还是因永久错误而失败（例如，已删除服务或请求的资源不再存在）。 服务实例或副本可出于多种原因随时在节点之间移动。 通过 ServicePartitionResolver 解析的服务地址可能会在客户端代码尝试连接之前过时。 再回到这种情况，客户端必须重新解析地址。 如果提供先前的 `ResolvedServicePartition`，则表示解析程序需要再试一次，而不只是检索缓存的地址。

客户端代码通常不需要直接处理 ServicePartitionResolver。 它在创建后即会传递给 Reliable Services API 中的通信客户端工厂。 这些工厂会在内部使用解析程序，生成可用来与服务通信的客户端对象。

### <a name="communication-clients-and-factories"></a>通信客户端和工厂
通信工厂库可实现典型的错误处理重试模式，从而可以更轻松地重试连接已解析服务终结点。 尽管你提供错误处理程序，工厂库还是会提供重试机制。

`ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` 定义由通信客户端工厂实现的基接口，该通信客户端工厂生成可与 Service Fabric 服务通信的客户端。 CommunicationClientFactory 的实现将取决于客户端要与之进行通信的 Service Fabric 服务所使用的通信堆栈。 Reliable Services API 提供 `CommunicationClientFactoryBase<TCommunicationClient>`。 这提供了 CommunicationClientFactory 接口的基实现，并可执行所有通信堆栈共有的任务。 （这些任务包括使用 ServicePartitionResolver 来确定服务终结点）。 客户端通常实现抽象 CommunicationClientFactoryBase 类来处理通信堆栈特定的逻辑。

通信客户端只接收地址，并使用它连接到服务。客户端可以使用它想要的任何协议。


	class MyCommunicationClient : ICommunicationClient
	{
	    public ResolvedServiceEndpoint Endpoint { get; set; }

	    public string ListenerName { get; set; }

	    public ResolvedServicePartition ResolvedServicePartition { get; set; }
	}

<br/>

    public class MyCommunicationClient implements CommunicationClient {

        private ResolvedServicePartition resolvedServicePartition;
        private String listenerName;
        private ResolvedServiceEndpoint endPoint;

        /*
         * Getters and Setters
         */
    }

客户端工厂主要负责创建通信客户端。 对于不会维持持续连接的客户端（例如 HTTP 客户端），工厂只需创建并返回客户端。 其他会维持持续连接的协议（例如某些二进制协议）也应该由工厂验证，以确定是否需要重新创建连接。  

	public class MyCommunicationClientFactory : CommunicationClientFactoryBase<MyCommunicationClient>
	{
	    protected override void AbortClient(MyCommunicationClient client)
	    {
	    }

	    protected override Task<MyCommunicationClient> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
	    {
	    }

	    protected override bool ValidateClient(MyCommunicationClient clientChannel)
	    {
	    }

	    protected override bool ValidateClient(string endpoint, MyCommunicationClient client)
	    {
	    }
	}

<br/>

    public class MyCommunicationClientFactory extends CommunicationClientFactoryBase<MyCommunicationClient> {

        @Override
        protected boolean validateClient(MyCommunicationClient clientChannel) {
        }

        @Override
        protected boolean validateClient(String endpoint, MyCommunicationClient client) {
        }

        @Override
        protected CompletableFuture<MyCommunicationClient> createClientAsync(String endpoint) {
        }

        @Override
        protected void abortClient(MyCommunicationClient client) {
        }
    }

最后，异常处理程序负责确定发生异常时要采取的操作。 异常分为**可重试**和**不可重试**两种类型。

* **不可重试**的异常只会重新引发回调用方。
* **可重试**的异常进一步分为**暂时性**和**非暂时性**两种类型。
  * **暂时性**异常是只会重试而不会重新解析服务终结点地址的异常。 这类异常包括暂时性网络问题或服务错误响应，但不包括指出服务终结点地址不存在的异常。
  * **非暂时性** 异常是需要重新解析服务终结点地址的异常。 这类异常包括指出无法访问服务终结点（表示服务已移至其他节点）的异常。

`TryHandleException` 针对给定异常做出决定。 如果它**不知道**要对异常做出哪些决定，则应返回 **false**。 如果它**知道**如何做决定，则应相应地设置结果并返回 **true**。

	class MyExceptionHandler : IExceptionHandler
	{
	    public bool TryHandleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings, out ExceptionHandlingResult result)
	    {
	        // if exceptionInformation.Exception is known and is transient (can be retried without re-resolving)
	        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, true, retrySettings, retrySettings.DefaultMaxRetryCount);
	        return true;


	        // if exceptionInformation.Exception is known and is not transient (indicates a new service endpoint address must be resolved)
	        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, false, retrySettings, retrySettings.DefaultMaxRetryCount);
	        return true;

	        // if exceptionInformation.Exception is unknown (let the next IExceptionHandler attempt to handle it)
	        result = null;
	        return false;
	    }
	}

<br/>

    public class MyExceptionHandler implements ExceptionHandler {

        @Override
        public ExceptionHandlingResult handleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings) {        

            /* if exceptionInformation.getException() is known and is transient (can be retried without re-resolving)
             */
            result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), true, retrySettings, retrySettings.getDefaultMaxRetryCount());
            return true;


            /* if exceptionInformation.getException() is known and is not transient (indicates a new service endpoint address must be resolved)
             */
            result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), false, retrySettings, retrySettings.getDefaultMaxRetryCount());
            return true;

            /* if exceptionInformation.getException() is unknown (let the next ExceptionHandler attempt to handle it)
             */
            result = null;
            return false;

        }
    }

### <a name="putting-it-all-together"></a>汇总
使用以通信协议生成的 `ICommunicationClient(C#) / CommunicationClient(Java)`、`ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` 和 `IExceptionHandler(C#) / ExceptionHandler(Java)`，`ServicePartitionClient(C#) / FabricServicePartitionClient(Java)` 会将它全部包装在一起，并为这些组件提供错误处理和服务分区地址解析循环。

	private MyCommunicationClientFactory myCommunicationClientFactory;
	private Uri myServiceUri;

	var myServicePartitionClient = new ServicePartitionClient<MyCommunicationClient>(
	    this.myCommunicationClientFactory,
	    this.myServiceUri,
	    myPartitionKey);

	var result = await myServicePartitionClient.InvokeWithRetryAsync(async (client) =>
	   {
	      // Communicate with the service using the client.
	   },
	   CancellationToken.None);


<br/>

    private MyCommunicationClientFactory myCommunicationClientFactory;
    private URI myServiceUri;

    FabricServicePartitionClient myServicePartitionClient = new FabricServicePartitionClient<MyCommunicationClient>(
        this.myCommunicationClientFactory,
        this.myServiceUri,
        myPartitionKey);

    CompletableFuture<?> result = myServicePartitionClient.invokeWithRetryAsync(client -> {
          /* Communicate with the service using the client.
           */
       });


## <a name="next-steps"></a>后续步骤
* 请参阅 [GitHUb 上的 C# 示例项目](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Services/WordCount)或 [GitHUb 上的 Java 示例项目](https://github.com/Azure-Samples/service-fabric-java-getting-started/tree/master/Services/WatchDog)中服务之间的 HTTP 通信示例。
* [使用 Reliable Services 远程控制执行远程过程调用](/documentation/articles/service-fabric-reliable-services-communication-remoting/)
* [Reliable Services 中使用 OWIN 的 Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)
* [使用 Reliable Services 的 WCF 通信](/documentation/articles/service-fabric-reliable-services-communication-wcf/)
<!--Update_Description:update most code snippet-->