<properties
    pageTitle="Service Fabric 和容器概述 | Azure"
    description="概述 Service Fabric，以及如何使用容器部署微服务应用程序。本文概述容器的用法以及 Service Fabric 提供的功能。"
    services="service-fabric"
    documentationcenter=".net"
    author="msfussell"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="c98b3fcb-c992-4dd9-b67d-2598a9bf8aab"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="2/6/2017"
    wacn.date="05/08/2017"
    ms.author="msfussell" />  


# 预览：Service Fabric 和容器
> [AZURE.NOTE]
>此功能在 Linux 中以预览版提供。
>   

## 介绍
Azure Service Fabric 是跨计算机群集的服务[协调器](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)。开发服务的方式多种多样：使用 [Service Fabric 编程模型](/documentation/articles/service-fabric-choose-framework/)，或者部署[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)，不一而足。默认情况下，Service Fabric 以进程形式部署和激活这些服务。进程能够以最快的速度激活、以最高的密度使用群集中的资源。Service Fabric 还可在容器映像中部署服务。重要的是，可以在同一应用程序中混合使用进程中的服务和容器中的服务。可以根据情况获得两全其美的结果。

## 什么是容器？
容器是可单独部署的封装组件，在相同内核中作为隔离的实例运行，以便利用操作系统提供的虚拟化功能。这意味着，每个应用程序及其运行时、依赖项和系统库都在容器中运行，并且对容器自身的隔离的操作系统构造拥有完全专属的访问权限。这种程度的安全性与资源隔离性，再加上可移植性，是将容器与 Service Fabric 配合使用带来的主要优势，否则就要在进程中运行服务。

容器是在应用程序中将基础操作系统虚拟化的一种虚拟化技术。容器提供固定的环境，使应用程序以不同的隔离程度运行。容器直接在内核顶层运行，并对文件系统与其他资源进行了隔离。相比于虚拟机，容器具有以下优势：

* **小型**：容器使用单个存储空间，每一层的增量较小，因此更有效率。
* **快速启动**：容器不需要启动操作系统，因此启动速度比虚拟机快，通常只需几秒。
* **可移植性**：容器化的应用程序映像可以移植到云中或本地运行、移植到虚拟机中运行，或者直接在物理机上运行。
* **资源监管**：容器可限制在其主机上消耗的物理资源。

## 容器类型
Service Fabric 支持以下类型的容器。

### Linux 上的 Docker 容器
Docker 提供高级 API 来创建和管理位于 Linux 内核容器顶层的容器。Docker 中心是一个用于存储和检索容器映像的中心存储库。

有关如何执行此操作的演示，请阅读[将 Docker 容器部署到 Service Fabric](/documentation/articles/service-fabric-deploy-container-linux/)。



## 使用容器的方案
以下是容器作为一个不错的选择的典型示例：

* **IIS 提升和转变**：如果有现成的想要继续使用的 [ASP.NET MVC](https://www.asp.net/mvc) 应用，则将它们放在一个容器里，而不是迁移到 ASP.NET Core。这些 ASP.NET MVC 应用都依赖于 Internet 信息服务 (IIS)。可以从预先创建的 IIS 映像将这些应用打包到容器映像，然后使用 Service Fabric 进行部署。
* **混合使用容器和 Service Fabric 微服务**：将现有容器映像用于应用程序的一部分。例如，对于应用程序的 Web 前端，可以使用 [NGINX 容器](https://hub.docker.com/_/nginx/)；对于更密集的后端计算，可以使用有状态服务。
* **减少“噪声邻居”服务的影响**：可以使用容器的资源监管功能来限制服务在主机上使用的资源。如果服务可能会消耗大量资源，并影响其他服务的性能（例如，长时间运行的类似于查询的操作），请考虑将这些服务放入具有资源监管功能的容器中。

## Service Fabric 对容器的支持
目前，Service Fabric 支持在 Linux 上部署 Docker 容器。将来的版本会添加对 Windows Server 以及 Hyper-V 容器的支持。

在 Service Fabric [应用程序模型](/documentation/articles/service-fabric-application-model/)中，容器代表多个服务副本所在的应用程序主机。支持以下容器方案：

* **来宾容器**：此方案与[来宾可执行文件方案](/documentation/articles/service-fabric-deploy-existing-app/)相同，可在容器中部署现有的应用程序。示例包括 Node.js、JavaScript 或任何代码（可执行文件）。
* **容器中的无状态服务**：这是一些使用 Reliable Services 和 Reliable Actors 编程模型的无状态服务。目前只有 Linux 支持此方案。
* **容器中的有状态服务**：这是 Linux 上的一些使用 Reliable Actors 编程模型的有状态服务。Linux 目前不支持 Reliable Services。

Service Fabric 提供多种容器功能，可帮助用户构建由容器化的微服务组成的应用程序。这些服务称为容器化服务。功能包括：

* 容器映像部署和激活。
* 资源调控。
* 存储库身份验证。
* 容器端口到主机端口的映射。
* 容器到容器的发现和通信。
* 能够配置和设置环境变量。

## 后续步骤
你可以在本文中了解容器，Service Fabric 是一个容器协调器，并且 Service Fabric 具有支持容器的功能。接下来，我们将演示其中的每项功能并说明其用法。


[将 Docker 容器部署到 Linux 上的 Service Fabric](/documentation/articles/service-fabric-deploy-container-linux/)

[Image1]: ./media/service-fabric-containers/Service-Fabric-Types-of-Isolation.png

<!---HONumber=Mooncake_0227_2017-->