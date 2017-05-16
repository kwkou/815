<properties
    pageTitle="Reliable Actors 框架中的多态性技术 | Azure"
    description="在 Reliable Actors 框架中构建 .NET 接口和类型的层次结构，以便重用功能和 API 定义。"
    services="service-fabric"
    documentationcenter=".net"
    author="seanmck"
    manager="timlt"
    editor="vturecek" />
<tags
    ms.assetid="ef0eeff6-32b7-410d-ac69-87cba8b8fd46"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/28/2017"
    wacn.date="05/15/2017"
    ms.author="seanmck"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="3ae728c87e06a436e80366b170069756c720a6c4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="polymorphism-in-the-reliable-actors-framework"></a>Reliable Actors 框架中的多态性技术
Reliable Actors 框架允许你使用许多在面向对象的设计中使用的相同技术来生成执行组件。 其中一种技术是多态性技术，它允许类型和接口从多个通用父类中继承。 Reliable Actors 框架中的继承通常遵循 .NET 模型，并会受到一些附加限制。 在 Java/Linux 中，它遵循 Java 模型。

## <a name="interfaces"></a>接口
Reliable Actors 框架要求至少定义一个要由执行组件类型实现的接口。 此接口用于生成代理类，客户端可以使用此代理类与你的执行组件进行通信。 只要由执行组件类型实现的每个接口及其所有父接口最终派生自 IActor(C#) 或 Actor(Java)，就可以从其他接口继承接口。 IActor(C#) 和 Actor(Java) 是平台定义的基接口，分别用于 .NET 和 Java 框架中的执行组件。 因此，使用形状的经典多态性示例可能如下所示：

![形状执行组件的接口层次结构][shapes-interface-hierarchy]

## <a name="types"></a>类型
你还可以创建执行组件类型的层次结构，这些类型派生自由平台提供的执行组件基类。 如果是形状，则可能具有一个 `Shape`(C#) 或 `ShapeImpl`(Java) 基类型：


	public abstract class Shape : Actor, IShape
	{
	    public abstract Task<int> GetVerticeCount();

	    public abstract Task<double> GetAreaAsync();
	}

<br/>

    public abstract class ShapeImpl extends FabricActor implements Shape
    {
        public abstract CompletableFuture<int> getVerticeCount();

        public abstract CompletableFuture<double> getAreaAsync();
    }

`Shape`(C#) 或 `ShapeImpl`(Java) 的子类型可以重写基类型的方法。

	[ActorService(Name = "Circle")]
	[StatePersistence(StatePersistence.Persisted)]
	public class Circle : Shape, ICircle
	{
	    public override Task<int> GetVerticeCount()
	    {
	        return Task.FromResult(0);
	    }

	    public override async Task<double> GetAreaAsync()
	    {
	        CircleState state = await this.StateManager.GetStateAsync<CircleState>("circle");

	        return Math.PI *
	            state.Radius *
	            state.Radius;
	    }
	}

<br/>

    @ActorServiceAttribute(name = "Circle")
    @StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
    public class Circle extends ShapeImpl implements Circle
    {
        @Override
        public CompletableFuture<Integer> getVerticeCount()
        {
            return CompletableFuture.completedFuture(0);
        }

        @Override
        public CompletableFuture<Double> getAreaAsync()
        {
            return (this.stateManager().getStateAsync<CircleState>("circle").thenApply(state->{
              return Math.PI * state.radius * state.radius;
            }));
        }
    }

请注意执行组件类型上的 `ActorService` 属性。 此属性告知 Reliable Actor 框架，它应自动创建用于托管此类型的执行组件的服务。 在某些情况下，你可能想要创建仅用于与子类型共享功能，并且始终不会用于实例化具体的执行组件的基类型。 在这些情况下，应使用 `abstract` 关键字表示你始终不会基于此类型创建执行组件。

## <a name="next-steps"></a>后续步骤
* 请参阅 [Reliable Actors 框架如何使用 Service Fabric 平台](/documentation/articles/service-fabric-reliable-actors-platform/)，以实现可靠性、可伸缩性和一致状态。
* 了解[执行组件生命周期](/documentation/articles/service-fabric-reliable-actors-lifecycle/)。

<!-- Image references -->

[shapes-interface-hierarchy]: ./media/service-fabric-reliable-actors-polymorphism/Shapes-Interface-Hierarchy.png
<!--Update_Description:wording update-->