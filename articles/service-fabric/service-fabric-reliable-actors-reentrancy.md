<properties
    pageTitle="基于执行组件的 Azure 微服务中的重新进入 | Azure"
    description="Service Fabric Reliable Actors 的可重入性简介"
    services="service-fabric"
    documentationcenter=".net"
    author="vturecek"
    manager="timlt"
    editor="amanbha" />
<tags
    ms.assetid="be23464a-0eea-4eca-ae5a-2e1b650d365e"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/10/2017"
    wacn.date="06/15/2017"
    ms.author="vturecek" />  

# Reliable Actors 可重入性
默认情况下，Reliable Actors 运行时允许基于逻辑调用上下文的可重入性。这使执行组件在处于相同调用上下文链中时可重入。例如，如果执行组件 A 将消息发送给执行组件 B，而后者将消息发送给执行组件 C。在处理消息的过程中，如果执行组件 C 调用执行组件 A，则允许消息可重入。属于不同调用上下文的任何其他消息会在执行组件 A 上受阻，直到它完成处理。

有两个选项可用于 `ActorReentrancyMode` 枚举中定义的执行组件重新进入：

 - `LogicalCallContext`（默认行为）
 - `Disallowed` - 禁用重新进入


	public enum ActorReentrancyMode
	{
	    LogicalCallContext = 1,
	    Disallowed = 2
	}


可在注册过程中在 `ActorService` 的设置中配置重新进入。该设置适用于执行组件服务中创建的所有执行组件实例。

以下示例演示了将重入模式设置为 `ActorReentrancyMode.Disallowed` 的执行组件服务。在这种情况下，如果执行组件向另一个执行组件发送可重入消息，则会引发类型为 `FabricException` 的异常。


	static class Program
	{
	    static void Main()
	    {
	        try
	        {
	            ActorRuntime.RegisterActorAsync<Actor1>(
	                (context, actorType) => new ActorService(
	                    context, 
	                    actorType, () => new Actor1(), 
	                    settings: new ActorServiceSettings()
	                    {
	                        ActorConcurrencySettings = new ActorConcurrencySettings()
	                        {
	                            ReentrancyMode = ActorReentrancyMode.Disallowed
	                        }
	                    }))
	                .GetAwaiter().GetResult();

	            Thread.Sleep(Timeout.Infinite);
	        }
	        catch (Exception e)
	        {
	            ActorEventSource.Current.ActorHostInitializationFailed(e.ToString());
	            throw;
	        }
	    }
	}


## 后续步骤
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)

<!---HONumber=Mooncake_0227_2017-->