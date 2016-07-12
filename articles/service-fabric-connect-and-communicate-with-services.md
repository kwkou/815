<properties
   pageTitle="在 Azure Service Fabric 中与服务建立连接和通信 | Azure"
   description="了解如何在 Service Fabric 中解析服务、建立连接以及与之通信。"
   services="service-fabric"
   documentationCenter=".net"
   authors="kunaldsingh"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="04/18/2016"
   wacn.date="07/04/2016"/>
# 在 Service Fabric 中与服务建立连接和通信
在 Service Fabric 中，服务在 Service Fabric 群集（通常分布在多个 VM 间）中的某个位置运行。它可以从一个位置移动到另一个位置（由服务所有者移动或由 Service Fabric 自动移动）。服务不以静态方式绑定到特定计算机或地址。
 
Service Fabric 应用程序通常由许多不同服务组成，其中每个服务执行专门任务。这些服务可能会相互进行通信以形成一个完整功能，如呈现 Web 应用程序的不同部分。其中也有连接到服务并与之通信的客户端应用程序。本文档介绍如何在 Service Fabric 中设置与服务进行的通信以及服务之间的通信。

## 自带协议
Service Fabric 可帮助管理服务的生命周期，但是它不会制定有关服务执行的操作的决策。这包括通信。服务由 Service Fabric 打开时，服务可以使用所需的任何协议或通信堆栈为传入请求设置终结点。服务会使用任何寻址方案（如 URI）来侦听通常的 **IP:端口**地址。多个服务实例或副本可能会共享主机进程，在这种情况下，它们需要使用不同端口，或使用端口共享机制（例如 Windows 中的 http.sys 内核驱动程序）。在任一情况下，主机进程中的每个服务实例或副本都必须可唯一寻址。

![服务终结点][1]

## 服务发现和解析
在分布式系统中，服务可能随时间推移从一台计算机移动到另一台计算机。发生这种情况可能是由于各种原因，包括资源平衡、升级、故障转移或扩大。这意味着服务终结点地址会在服务移动到具有不同 IP 地址的节点时发生更改，并且可能在不同端口上打开（如果服务使用动态选择的端口）。

![服务分发][7]

Service Fabric 提供一种服务发现和解析服务，称为“命名服务”。命名服务维护一个表，它将命名服务实例映射到它们所侦听的终结点地址。Service Fabric 中的所有命名服务实例都具有表示为 URI 的唯一名称，例如 `"fabric:/MyApplication/MyService"`。服务的名称在服务的生存期内不会更改，只有终结点地址才能在服务移动时发生更改。这与 URL 保持不变但 IP 地址可能会更改的网站类似。与 Web 上的 DNS（将网站 URL 解析为 IP 地址）类似，Service Fabric 具有一个注册机构，它将服务名称映射到其终结点地址。

![服务终结点][2]

解析服务以及连接到服务涉及循环运行的以下步骤：

* **解析**：从命名服务获取服务发布的终结点。

* **连接**：通过服务在该终结点上使用的任何协议连接到服务。

* **重试**：连接尝试可能会由于许多原因而失败（例如，如果服务自上次解析终结点地址以来已移动）。在这种情况下，前面的解析和连接步骤需要重试，并且此循环会重复执行，直到连接成功。

## 来自外部客户端的连接

在群集内相互连接的服务通常可以直接访问其他服务的终结点，因为群集中的节点通常处于相同的本地网络上。但是在某些环境中，群集可能位于通过一组有限端口对外部传入流量进行路由的负载平衡器之后。在这些情况下，服务仍可以使用命名服务相互通信和解析地址，但必须执行额外步骤才能允许外部客户端连接到服务。

## Azure 中的 Service Fabric

Azure 中的 Service Fabric 群集位于 Azure Load Balancer 之后。发送到群集的所有外部流量都必须穿过该负载平衡器。该负载平衡器会自动在给定端口上将入站流量转发到打开了相同端口的随机“节点”。Azure Load Balancer 只了解“节点” 上打开的端口，它不了解各个*服务* 打开的端口。

![Azure Load Balancer 和 Service Fabric 拓扑][3]

例如，若要在端口 **80** 上接受外部流量，必须配置以下内容：

1. 编写侦听端口 80 的服务。在服务的 ServiceManifest.xml 中配置端口 80，并在服务中打开一个侦听器，例如自承载的 Web 服务器。
 

    	<Resources>
        	<Endpoints>
            	<Endpoint Name="WebEndpoint" Protocol="http" Port="80" />
        	</Endpoints>
    	</Resources>

    ```csharp
        class HttpCommunicationListener : ICommunicationListener
        {
            ...
            
            public Task<string> OpenAsync(CancellationToken cancellationToken)
            {
                EndpointResourceDescription endpoint = 
                    serviceContext.CodePackageActivationContext.GetEndpoint("WebEndpoint");

                string uriPrefix = $"{endpoint.Protocol}://+:{endpoint.Port}/myapp/";

                this.httpListener = new HttpListener();
                this.httpListener.Prefixes.Add(uriPrefix);
                this.httpListener.Start();

                string uriPublished = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

                return Task.FromResult(this.publishUri);
            }
            
            ...
        }
        
        class WebService : StatelessService
        {
            ...
            
            protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
            {
                return new[] {new ServiceInstanceListener(context => new HttpCommunicationListener(context))};
            }
            
            ...
        }
    ```
  
2. 在 Azure 中创建 Service Fabric 群集，并将端口 **80** 指定为将承载服务的节点类型的自定义终结点端口。如果具有多种节点类型，则可以对服务设置“放置约束”，以确保它只在打开了自定义终结点端口的节点类型上运行。

    ![在节点类型上打开端口][4]

3. 创建了群集之后，在群集的资源组中配置 Azure Load Balancer 以在端口 80 上转发流量。通过 Azure 门户预览创建群集时，会为每个已配置的自定义终结点端口自动设置此项。

    ![在 Azure Load Balancer 中转发流量][5]

4. Azure Load Balancer 使用探测来确定是否要将流量发送到特定节点。探测会定期检查每个节点上的终结点以确定节点是否正在进行响应。如果探测未能在配置的次数之后收到响应，则负载平衡器会停止将流量发送到该节点。通过 Azure 门户预览创建群集时，会为每个已配置的自定义终结点端口自动设置探测。

    ![在 Azure Load Balancer 中转发流量][8]

请务必记住，Azure Load Balancer 和探测只了解“节点”，而不了解在节点上运行的*服务*。Azure Load Balancer 始终将流量发送到响应探测的节点，因此必须格外小心以确保服务在能够响应探测的节点上可用。

## 内置通信 API 选项
Reliable Services 框架附带几个预建的通信选项。你可以根据所选的编程模型、通信框架和编写服务时使用的编程语言来决定哪个选项最适合自己。

* **无特定协议：**如果没有特定的通信框架选择，但需要可以快速启动并运行协议，则你理想之选是[服务远程处理](/documentation/articles/service-fabric-reliable-services-communication-remoting/)，此协议可对 Reliable Services 和 Reliable Actors 实现强类型远程过程调用。这是服务通信入门最简单、最快捷的方法。服务远程处理可处理服务地址的解析、连接、重试和错误处理。请注意，服务远程处理只能用于 C# 应用程序。

* **HTTP**：对于与语言无关的通信，HTTP 为行业标准选择提供了在许多不同语言（Service Fabric 全都支持）中可用的工具和 HTTP 服务器。服务可以使用可用的任何 HTTP 堆栈，包括 [ASP.NET Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)。以 C# 编写的客户端可以利用 [`ICommunicationClient` 和 `ServicePartitionClient` 类](/documentation/articles/service-fabric-reliable-services-communication/)来进行服务解析、HTTP 连接和重试循环。

* **WCF**：如果具备将 WCF 用作通信框架的现有代码，则可将 `WcfCommunicationListener` 用于服务器端，并将 `WcfCommunicationClient` 和 `ServicePartitionClient` 类用于客户端。有关更多详细信息，请参阅有关[通信堆栈的基于 WCF 的实现](/documentation/articles/service-fabric-reliable-services-communication-wcf/)的此文章。

## 使用自定义协议和其他通信框架
服务可以使用任何协议或框架进行通信，无论它是 TCP 套接字上的自定义二进制协议，还是通过 [Azure 事件中心](/home/features/event-hubs/)或 [Azure IoT 中心](/home/features/iot-hub/)实现的流式处理事件。Service Fabric 提供了通信 API，你可以将通信堆栈插入其中，同时将用于发现和连接的所有工作与你分离。有关更多详细信息，请参阅有关 [Reliable Service 通信模型](/documentation/articles/service-fabric-reliable-services-communication/)的此文章。

## 后续步骤

了解有关 [Reliable Services 通信模型](/documentation/articles/service-fabric-reliable-services-communication/)中可用的概念和 API 的详细信息，然后快速开始使用[服务远程处理](/documentation/articles/service-fabric-reliable-services-communication-remoting/)或深入了解如何使用[具有 OWIN 自承载的 Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/) 编写通信侦听器。

[1]: ./media/service-fabric-connect-and-communicate-with-services/serviceendpoints.png
[2]: ./media/service-fabric-connect-and-communicate-with-services/namingservice.png
[3]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancertopology.png
[4]: ./media/service-fabric-connect-and-communicate-with-services/nodeport.png
[5]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerport.png
[7]: ./media/service-fabric-connect-and-communicate-with-services/distributedservices.png
[8]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerprobe.png
<!---HONumber=Mooncake_0523_2016-->