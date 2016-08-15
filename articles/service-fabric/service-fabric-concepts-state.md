<properties
   pageTitle="定义和管理状态 | Microsoft Azure"
   description="如何定义和管理 Service Fabric 中的服务状态"
   services="service-fabric"
   documentationCenter=".net"
   authors="appi101"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="01/20/2016"
   wacn.date="07/04/2016"/>

# 服务状态
**服务状态**指的是服务正常运行所必需的数据。这包括服务为正常工作而读取和写入的数据结构和变量。

例如，请考虑一个简单的计算器服务。此服务需要两个数字并返回总和。这是没有任何数据与之关联的纯无状态服务。

现在，考虑相同的计算器，但除了计算总和，它还有方法返回上次所计算的总和。此服务现在有状态 -- 它包含写入的状态（计算新的总和时）和读取的状态（返回上次计算出的总和时）。

在 Azure Service Fabric 中，第一个服务称为无状态服务。第二个服务则称为有状态服务。

## 存储服务状态
状态可以外部化或与操作状态的代码共存。通常使用外部的数据库或存储区进行状态的外部化。在我们的计算器示例中，这可能是 SQL 数据库（其中当前结果存储在表中）。计算总和的每个请求均将在此行上执行更新。

状态也可以与操作此代码的代码共存。Service Fabric 中的有状态服务是使用此模型生成的。Service Fabric 提供了确保此状态高度可用和容错（出错时）的基础结构。

## 后续步骤

有关 Service Fabric 概念的详细信息，请参阅以下内容：

- [Service Fabric 服务的可用性](/documentation/articles/service-fabric-availability-services/)

- [Service Fabric 服务的可伸缩性](/documentation/articles/service-fabric-concepts-scalability/)

- [Service Fabric 服务分区](/documentation/articles/service-fabric-concepts-partitioning/)
 

<!---HONumber=Mooncake_0307_2016-->