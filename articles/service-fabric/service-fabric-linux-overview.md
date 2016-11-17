<properties
   pageTitle="Linux 上的 Azure Service Fabric | Azure"
   description="Service Fabric 群集支持 Linux 和 Java，这意味着，你可以在 Linux 上部署和托管以 Java 编写的 Service Fabric 应用程序。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="Java"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/14/2016"
   wacn.date="10/24/2016"
   ms.author="SubramaR"/>  


# Linux 上的 Service Fabric

Service Fabric 目前在 Linux 上以受限预览版提供，可让你在 Linux 环境中构建、部署和管理高度可用、高度可缩放的应用程序，就像在 Windows 上一样。此外，现在可以在 Java 中构建高级 Service Fabric 框架（Reliable Services 和 Reliable Actors）。

## 支持操作系统和编程语言

受限预览版支持创建单机开发群集，以及在 Azure 中运行 Ubuntu Server 16.04 的多计算机群集。

可以使用任何语言或框架来构建[来宾可执行服务](/documentation/articles/service-fabric-deploy-existing-app/)。还可以使用 Java 或 C# 来构建基于 Reliable Services 和 Reliable Actor 框架的服务。

>[AZURE.NOTE] Linux 尚不支持 Reliable 集合。

Linux 上的 Service Fabric 在概念上等同于 Windows 上的 Service Fabric（OS 细节和编程语言支持除外）。因此，我们的大部分[现有文档](/documentation/services/service-fabric/)都能帮助你熟悉该技术。

## 后续步骤

熟悉 [Reliable Actors](/documentation/articles/service-fabric-reliable-actors-introduction/) 和 [Reliable Services](/documentation/articles/service-fabric-reliable-services-introduction/) 编程框架。

<!---HONumber=Mooncake_1017_2016-->