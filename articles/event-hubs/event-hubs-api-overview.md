<properties
    pageTitle="Azure 事件中心 API 概述 | Azure"
    description="可用 Azure 事件中心 API 的概述"
    services="event-hubs"
    documentationcenter="na"
    author="jtaubensee"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3f221a0c-182d-4e39-9f3d-3a3c16c5c6ed"
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/31/2017"
    wacn.date="03/24/2017"
    ms.author="jotaub" />  


# 可用的事件中心 API

## 运行时 API

下面列出了当前可用的所有事件中心运行时客户端。虽然其中某些库也包含受限的管理功能，但也有专用于管理操作的[特定库](#management-apis)。这些库的核心功能是向事件中心发送消息和从事件中心接收消息。

有关每个运行时库的当前状态的更多详细信息，请参阅[其他信息](#additional-information)。

| 语言/平台 | 客户端程序包 | EventProcessorHost 包 | 存储库 |
| --- | --- | --- | --- |
| .NET Standard | [NuGet ](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) | [NuGet ](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/) | [GitHub](https://github.com/azure/azure-event-hubs-dotnet) |
| .NET framework | [NuGet ](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) | [NuGet ](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/) | 不适用 |
| Java | [Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22) | [Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22) | [GitHub](https://github.com/Azure/azure-event-hubs-java) |
| 节点 | [NPM](https://www.npmjs.com/package/azure-event-hubs) | 不适用 | [GitHub](https://github.com/Azure/azure-event-hubs-node) |
| C | 不适用 | 不适用 | [GitHub](https://github.com/Azure/azure-event-hubs-c) |

### <a id="additional-information"></a> 其他信息

#### .NET
.NET 生态系统具有多个运行时，因此事件中心有多个 .NET 库。可以使用 .NET Core 或 .NET Framework 运行 .NET Standard 库，但 .NET Framework 库只能在 .NET Framework 环境中运行。有关 .NET Framework 的详细信息，请参阅[框架版本](https://docs.microsoft.com/dotnet/articles/standard/frameworks#framework-versions)。

#### 节点

Node.js 库目前处于预览状态，由 Microsoft 员工和外部参与者作为副项目进行维护。包括源代码在内的所有贡献都欢迎并将对其进行审查。

## <a id="management-apis"></a> 管理 API

下面列出了当前可用的所有特定于管理的库。这些库不包含运行时操作，管理事件中心实体是其唯一的用途。

| 语言/平台 | 管理包 | 存储库 |
| --- | --- | --- | --- |
| .NET Standard | [NuGet ](https://www.nuget.org/packages/Microsoft.Azure.Management.EventHub) | [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/ResourceManagement/EventHub) |

## 后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:update meta properties;new feature of runtime and management for api-->