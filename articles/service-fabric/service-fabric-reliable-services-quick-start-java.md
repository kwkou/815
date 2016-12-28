<properties
   pageTitle="Reliable Services 入门 | Azure"
   description="介绍如何创建包含无状态服务和有状态服务的 Azure Service Fabric 应用程序。"
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="java"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/26/2016"
   wacn.date="11/28/2016"
   ms.author="vturecek"/>  


# Reliable Services 入门

> [AZURE.SELECTOR]
- [Windows 上的 C#](/documentation/articles/service-fabric-reliable-services-quick-start/)
- [Linux 上的 Java](/documentation/articles/service-fabric-reliable-services-quick-start-java/)

本文介绍 Azure Service Fabric Reliable Services 的基础知识，并演示如何创建和部署以 Java 编写的简单 Reliable Service 应用程序。

## 安装和设置
在开始之前，请确保已在计算机上设置 Service Fabric 开发环境。如果需要设置环境，请转到[在 Mac 上开始使用](/documentation/articles/service-fabric-get-started-mac/)或[在 Linux 上开始使用](/documentation/articles/service-fabric-get-started-linux/)。

## 基本概念
若要开始使用 Reliable Services，只需了解几个基本概念：

 - **服务类型**：这是服务实现。它由你编写的可扩展 `StatelessService` 的类、其中使用的任何其他代码或依赖项以及名称和版本号定义。

 - **命名服务实例**：若要运行服务，需要创建服务类型的命名实例，就像创建类类型的对象实例一样。事实上，服务实例是编写的服务类的对象实例化。

 - **服务宿主**：创建的命名服务实例需在宿主中运行。服务宿主是可以运行服务实例的进程。

 - **服务注册**：通过注册可将所有对象融合在一起。只有将服务类型注册到服务宿主中的 Service Fabric 运行时后，Service Fabric 才能创建该类型的可运行实例。

## 创建无状态服务

首先创建新的 Service Fabric 应用程序。适用于 Linux 的 Service Fabric SDK 包括一个 Yeoman 生成器，它为包含无状态服务的 Service Fabric 应用程序提供基架。首先，请运行以下 Yeoman 命令：


	$ yo azuresfjava


按照说明创建**可靠无状态服务**。本教程将应用程序命名为“HelloWorldApplication”，将服务命名为“HelloWorld”。结果包含 `HelloWorldApplication` 和 `HelloWorld` 的目录。


	HelloWorldApplication/
	├── build.gradle
	├── HelloWorld
	│   ├── build.gradle
	│   └── src
	│       └── statelessservice
	│           ├── HelloWorldServiceHost.java
	│           └── HelloWorldService.java
	├── HelloWorldApplication
	│   ├── ApplicationManifest.xml
	│   └── HelloWorldPkg
	│       ├── Code
	│       │   ├── entryPoint.sh
	│       │   └── _readme.txt
	│       ├── Config
	│       │   └── _readme.txt
	│       ├── Data
	│       │   └── _readme.txt
	│       └── ServiceManifest.xml
	├── install.sh
	├── settings.gradle
	└── uninstall.sh


## 实现服务

打开 **HelloWorldApplication/HelloWorld/src/statelessservice/HelloWorldService.java**。此类定义服务类型，可以运行任何代码。服务 API 为你的代码提供两个入口点：

 - 名为 `runAsync()` 的开放式入口点方法，可在其中开始执行任何工作负荷，包括长时间运行的计算工作负荷。


	@Override
	protected CompletableFuture<?> runAsync() {
	    ...
	}


 - 一个通信入口点，可在其中插入选择的通信堆栈。这就是你可以开始接收来自用户和其他服务请求的位置。


	@Override
	protected List<ServiceInstanceListener> createServiceInstanceListeners() {
	    ...
	}


在本教程中，我们将重点放在 `runAsync()` 入口点方法上。这是你可以立即开始运行代码的位置。

### RunAsync

当服务实例已放置并且可以执行时，平台将调用此方法。服务实例的打开-关闭循环可能会在服务的整个生存期内出现多次。发生这种情况的原因多种多样，包括：

- 系统可能会移动服务实例以实现资源平衡。
- 代码中发生错误。
- 应用程序或系统升级。
- 基础硬件遇到中断。

Service Fabric 将管理此业务流程，以便保持服务的高度可用和适当均衡。

#### 取消

`runAsync()` 中的代码必须能够根据 Service Fabric 的通知停止执行。当 Service Fabric 要求服务停止执行时，从 `runAsync()` 返回的 `CompletableFuture` 将被取消。以下示例演示如何处理取消事件：


	    @Override
	    protected CompletableFuture<?> runAsync() {

	        CompletableFuture<?> completableFuture = new CompletableFuture<>();
	        ExecutorService service = Executors.newFixedThreadPool(1);
        
	        Future<?> userTask = service.submit(() -> {
	            while (!Thread.currentThread().isInterrupted()) {
	            	try
	            	{
	                   logger.log(Level.INFO, this.context().serviceName().toString());
	                   Thread.sleep(1000);
	            	}
	            	catch (InterruptedException ex)
	            	{
	                    logger.log(Level.INFO, this.context().serviceName().toString() + " interrupted. Exiting");
	                    return;
	            	}
	            }
	         });
 
	        completableFuture.handle((r, ex) -> {
	            if (ex instanceof CancellationException) {
	                userTask.cancel(true);
	                service.shutdown();
	            }
	            return null;
	        });
 
	        return completableFuture;
	   }


### 服务注册

必须将服务类型注册到 Service Fabric 运行时。服务类型在 `ServiceManifest.xml` 中以及实现 `StatelessService` 的服务类中定义。服务注册在进程主入口点中执行。在本示例中，进程主入口点为 `HelloWorldServiceHost.java`：


	public static void main(String[] args) throws Exception {
	    try {
	        ServiceRuntime.registerStatelessServiceAsync("HelloWorldType", (context) -> new HelloWorldService(), Duration.ofSeconds(10));
	        logger.log(Level.INFO, "Registered stateless service type HelloWorldType.");
	        Thread.sleep(Long.MAX_VALUE);
	    } 
	    catch (Exception ex) {
	        logger.log(Level.SEVERE, "Exception in registration: {0}", ex.toString());
	        throw ex;
	    }
	}


## 运行应用程序

Yeoman 基架包含一个用于构建应用程序的 gradle 脚本，以及一个用于部署和取消部署应用程序的 bash 脚本。若要运行应用程序，请先使用 gradle 构建应用程序：


	$ gradle


这会生成可以使用 Service Fabric Azure CLI 部署的 Service Fabric 应用程序包。Install.sh 脚本包含用于部署应用程序包的 Azure CLI 命令。只需运行 install.sh 脚本即可部署：


	$ ./install.sh


<!---HONumber=Mooncake_1121_2016-->