<properties
   pageTitle="Service Fabric 中的服务远程处理 | Azure"
   description="Service Fabric 远程处理允许客户端和服务使用远程过程调用来与服务进行通信。"
   services="service-fabric"
   documentationCenter=".net"
   authors="BharatNarasimman"
   manager="timlt"
   editor="vturecek"/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# 通过 Reliable Services 进行服务远程处理
对于不依赖于特定的通信协议或堆栈的服务，如 WebAPI、Windows Communication Foundation (WCF) 或其他服务，Reliable Services 框架提供一种远程处理机制，以便快速而轻松地为这些服务设置远程过程调用。

## 为服务设置远程处理
为服务设置远程处理包括两个简单步骤：

1. 为你的服务创建要实现的接口。此接口定义可供你的服务的远程过程调用使用的方法。这些方法必须是返回任务的异步方法。此接口必须实现 `Microsoft.ServiceFabric.Services.Remoting.IService` 以表明此服务具有远程处理接口。
2. 在服务中使用远程处理侦听器。这是可以提供远程处理功能的 `ICommunicationListener` 实现。`Microsoft.ServiceFabric.Services.Remoting.Runtime` 命名空间包含一个同时适用于无状态服务和有状态服务的扩展方法 `CreateServiceRemotingListener`，可用于创建使用默认远程处理传输协议的远程处理侦听器。

例如，以下无状态服务公开了一个方法，此方法通过远程过程调用获取“Hello World”。

```csharp
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Remoting;
using Microsoft.ServiceFabric.Services.Remoting.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

public interface IMyService : IService
{
    Task<string> GetHelloWorld();
}

class MyService : StatelessService, IMyService
{
    public MyService(StatelessServiceContext context)
        : base (context)
{
    }

    public Task HelloWorld()
    {
        return Task.FromResult("Hello!");
    }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[] { new ServiceInstanceListener(context => 
            this.CreateServiceRemotingListener(context)) };
    }
}
```
> [AZURE.NOTE] 服务接口中的参数和返回类型可以是任何简单、复杂或自定义的类型，但它们必须是 .NET [DataContractSerializer](https://msdn.microsoft.com/zh-cn/library/ms731923.aspx) 可序列化的类型。


## 调用远程服务的方法
若要使用远程处理堆栈在服务上调用方法，可以通过 `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` 类对此服务使用本地代理。`ServiceProxy` 方法通过使用此服务实现的相同接口来创建本地代理。使用此代理，你只需在此接口上远程调用方法。




	IHelloWorldStateful helloWorldClient = ServiceProxy.Create<IHelloWorldStateful>(new Uri("fabric:/MyApplication/MyHelloWorldService"));

	string message = await helloWorldClient.GetHelloWorld();



此远程处理框架将服务引发的异常传播到客户端。因此，在客户端使用 `ServiceProxy` 的异常处理逻辑可直接处理服务引发的异常。

## 后续步骤

* [Reliable Services 中使用 OWIN 的 Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)

* [通过 Reliable Services 进行 WCF 通信](/documentation/articles/service-fabric-reliable-services-communication-wcf/)

* [确保 Reliable Services 的通信安全](/documentation/articles/service-fabric-reliable-services-secure-communication/)

<!---HONumber=Mooncake_0503_2016-->