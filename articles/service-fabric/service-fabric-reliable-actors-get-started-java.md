<properties
   pageTitle="Service Fabric Reliable Actors 入门 | Azure"
   description="本教程将向你演示使用 Service Fabric Reliable Actors 创建、调试和部署简单的基于执行组件的服务的步骤。"
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="java"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/25/2016"
   wacn.date="11/28/2016"
   ms.author="vturecek"/>  


# Reliable Actors 入门

> [AZURE.SELECTOR]
- [Windows 上的 C#](/documentation/articles/service-fabric-reliable-actors-get-started/)
- [Linux 上的 Java](/documentation/articles/service-fabric-reliable-actors-get-started-java/)

本文介绍 Azure Service Fabric Reliable Actors 的基础知识，并演示如何在 Java 中创建和部署简单的 Reliable Actor 应用程序。

## 安装和设置
在开始之前，请确保已在计算机上设置 Service Fabric 开发环境。
如果需要设置环境，请转到[在 Mac 上开始使用](/documentation/articles/service-fabric-get-started-mac/)或[在 Linux 上开始使用](/documentation/articles/service-fabric-get-started-linux/)。

## 基本概念
若要开始使用 Reliable Actors，只需了解几个基本概念：

 * **执行组件服务**。可以在 Service Fabric 基础结构中部署的 Reliable Services 中封装了 Reliable Actors。执行组件服务在命名的服务实例中激活。
 
 * **执行组件注册**。与 Reliable Services 一样，Reliable Actor 服务也需要注册到 Service Fabric 运行时。此外，需要将执行组件类型注册到执行组件运行时。
 
 * **执行组件接口**。执行组件接口用于定义执行组件的强类型公共接口。在 Reliable Actor 模型术语中，执行组件接口用于定义执行组件可以理解并处理的消息类型。其他执行组件或客户端应用程序使用此执行组件接口将消息“发送”到（异步方式）此执行组件。Reliable Actors 可实现多个接口。
 
 * **ActorProxy 类**。客户端应用程序使用 ActorProxy 类调用通过执行组件接口公开的方法。ActorProxy 类提供两个重要功能：
	* 名称解析：能够在群集中找到执行组件（查找托管它的群集节点）。
	* 故障处理：例如，在需要将执行组件重新定位到群集中另一个节点的故障之后，它可以重试方法调用和重新解析执行组件的位置。

有必要提一下以下与执行组件接口有关的规则：

- 不能重载执行组件接口方法。
- 执行组件接口方法不能有 out、ref 或可选参数。
- 不支持泛型接口。

## 创建执行组件服务
首先创建新的 Service Fabric 应用程序。适用于 Linux 的 Service Fabric SDK 包括一个 Yeoman 生成器，它为包含无状态服务的 Service Fabric 应用程序提供基架。首先，请运行以下 Yeoman 命令：


	$ yo azuresfjava


按照说明创建 **Reliable Actor 服务**。本教程将应用程序命名为“HelloWorldActorApplication”，将执行组件命名为“HelloWorldActor”。 将创建以下基架：


	HelloWorldActorApplication/
	├── build.gradle
	├── HelloWorldActor
	│   ├── build.gradle
	│   ├── settings.gradle
	│   └── src
	│       └── reliableactor
	│           ├── HelloWorldActorHost.java
	│           └── HelloWorldActorImpl.java
	├── HelloWorldActorApplication
	│   ├── ApplicationManifest.xml
	│   └── HelloWorldActorPkg
	│       ├── Code
	│       │   ├── entryPoint.sh
	│       │   └── _readme.txt
	│       ├── Config
	│       │   ├── _readme.txt
	│       │   └── Settings.xml
	│       ├── Data
	│       │   └── _readme.txt
	│       └── ServiceManifest.xml
	├── HelloWorldActorInterface
	│   ├── build.gradle
	│   └── src
	│       └── reliableactor
	│           └── HelloWorldActor.java
	├── HelloWorldActorTestClient
	│   ├── build.gradle
	│   ├── settings.gradle
	│   ├── src
	│   │   └── reliableactor
	│   │       └── test
	│   │           └── HelloWorldActorTestClient.java
	│   └── testclient.sh
	├── install.sh
	├── settings.gradle
	└── uninstall.sh


## Reliable Actors 基本构建基块

前面所述的基本概念可以转化为 Reliable Actor 服务的基本构建块。

### 执行组件接口

包含执行组件的接口定义。此接口定义执行组件实现和调用执行组件的客户端所共享的执行组件协定，因此合理的做法是在独立于执行组件实现的某个位置定义接口，并且其他多个服务或客户端应用程序可以共享接口。

`HelloWorldActorInterface/src/reliableactor/HelloWorldActor.java`：


	public interface HelloWorldActor extends Actor {
	    @Readonly   
	    CompletableFuture<Integer> getCountAsync();

	    CompletableFuture<?> setCountAsync(int count);
	}


### 执行组件服务 
包含执行组件实现和执行组件注册代码。执行组件类实现执行组件接口。这是执行组件的工作位置。

`HelloWorldActor/src/reliableactor/HelloWorldActorImpl`：


	@ActorServiceAttribute(name = "HelloWorldActor.HelloWorldActorService")
	@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
	public class HelloWorldActorImpl extends ReliableActor implements HelloWorldActor {
	    Logger logger = Logger.getLogger(this.getClass().getName());

	    protected CompletableFuture<?> onActivateAsync() {
	        logger.log(Level.INFO, "onActivateAsync");

	        return this.stateManager().tryAddStateAsync("count", 0);
	    }

	    @Override
	    public CompletableFuture<Integer> getCountAsync() {
	        logger.log(Level.INFO, "Getting current count value");
	        return this.stateManager().getStateAsync("count");
	    }

	    @Override
	    public CompletableFuture<?> setCountAsync(int count) {
	        logger.log(Level.INFO, "Setting current count value {0}", count);
	        return this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value);
	    }
	}


### 执行组件注册

执行组件服务必须使用 Service Fabric 运行时中的服务类型注册。为了使执行组件服务能够运行执行组件实例，还必须向执行组件服务注册你的执行组件类型。`ActorRuntime` 注册方法将为执行组件执行此操作。

`HelloWorldActor/src/reliableactor/HelloWorldActorHost`：


	public class HelloWorldActorHost {
	
	    public static void main(String[] args) throws Exception {
		
	        try {
	            ActorRuntime.registerActorAsync(HelloWorldActorImpl.class, (context, actorType) -> new ActorServiceImpl(context, actorType, ()-> new HelloWorldActorImpl()), Duration.ofSeconds(10));

	            Thread.sleep(Long.MAX_VALUE);
			
	        } catch (Exception e) {
	            e.printStackTrace();
	            throw e;
	        }
	    }
	}


### 测试客户端

这是一个简单的测试客户端应用程序，可以单独从 Service Fabric 应用程序运行它来测试执行组件服务。这是可以使用 ActorProxy 激活执行组件实例以及与执行组件实例通信的示例场合。它不会部署到服务中。

### 应用程序 

最后，应用程序将打包执行组件服务，以及将来可能要为部署添加的其他任何服务。其中包含执行组件服务包的 *ApplicationManifest.xml* 和占位符。

## 运行应用程序

Yeoman 基架包含一个用于构建应用程序的 gradle 脚本，以及一个用于部署和取消部署应用程序的 bash 脚本。若要运行应用程序，请先使用 gradle 构建应用程序：


	$ gradle


这会生成可以使用 Service Fabric Azure CLI 部署的 Service Fabric 应用程序包。Install.sh 脚本包含用于部署应用程序包的 Azure CLI 命令。只需运行 install.sh 脚本即可部署：


	$ ./install.sh


<!---HONumber=Mooncake_1121_2016-->