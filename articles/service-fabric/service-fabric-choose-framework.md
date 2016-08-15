<properties
   pageTitle="Service Fabric 编程模型概述 | Azure"
   description="Service Fabric 提供了两个框架用于构建服务：执行组件框架和服务框架。它们在简单性和控制力方面具有截然不同的取舍。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="04/18/2016"
   wacn.date="07/04/2016"/>

# Service Fabric 编程模型概述

Service Fabric 提供了多种方法来编写和管理服务。服务可以选择使用 Service Fabric API 来充分利用平台的功能和应用程序框架，或者服务可以只是采用任何语言编写的任何已编译的可执行程序，并且只是托管在 Service Fabric 群集上。

## 来宾可执行文件

来宾可执行文件是采用任何语言编写的任意可执行文件，因此可以将现有应用程序托管在 Service Fabric 群集上。来宾可执行文件可在应用程序中打包，并且和其他服务一起托管。Service Fabric 可处理可执行文件的业务流程和简单的执行管理，确保其始终按照服务说明启动并运行。但是，因为来宾可执行文件不直接与 Service Fabric API 集成，所以它们不会从平台所提供的完整功能集中获益，例如自定义健康和负载报告、服务终结点注册和有状态计算。

从部署第一个[来宾可执行文件应用程序](/documentation/articles/service-fabric-deploy-existing-app/)开始使用来宾可执行文件。

## Reliable Services

Reliable Services 是一个用于编写与 Service Fabric 平台集成的服务的轻型框架，并且受益于完整的平台功能集。Reliable Services 提供最小 API 集合，该集合允许 Service Fabric 运行时管理服务的生命周期，以及允许服务与运行时进行交互。此应用程序框架虽然很小，但是通过它你可以完全控制设计和实现选择，并且可以用来托管其他任何应用程序框架，例如 ASP.NET MVC 或 Web API。

Reliable Services 可以是无状态的，类似于大多数服务平台，例如 Web 服务器或 Azure 云服务中的辅助角色，这些角色中创建的每个服务实例都是平等的，并且状态保存在外部解决方案中，例如 Azure DB 或 Azure 表存储。

Reliable Services 也可以是有状态的，专门用于 Service Fabric，其状态使用 Reliable Collections 直接保存在服务中。通过复制使状态具有高可用性，以及通过分区来分布状态，所有状态由 Service Fabric 自动管理。

[了解有关 Reliable Services 的详细信息](/documentation/articles/service-fabric-reliable-services-introduction/)或者通过[编写第一个 Reliable Service](/documentation/articles/service-fabric-reliable-services-quick-start/) 帮助你入门

## Reliable Actors

Reliable Actor 框架在 Reliable Services 的基础上构建，是根据执行组件设计模式实现虚拟执行组件模式的应用程序框架。Reliable Actor 框架使用称为执行组件的单线程执行的独立的计算单元和状态。Reliable Actor 为执行组件提供内置通信，以及提供预设的状态暂留和扩展配置。

由于 Reliable Actors 自身是在 Reliable Services 基础上构建的应用程序框架，所以它完全与 Service Fabric 平台集成，并且获益于平台所提供的完整功能集。

## 后续步骤
[了解有关 Reliable Actors 的详细信息](/documentation/articles/service-fabric-reliable-actors-introduction/)或通过[编写第一个 Reliable Actor 服务](/documentation/articles/service-fabric-reliable-actors-get-started/)帮助你入门

<!---HONumber=Mooncake_0523_2016-->