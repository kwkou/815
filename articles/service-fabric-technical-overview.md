<properties
   pageTitle="Service Fabric 术语概述 | Azure"
   description="Service Fabric 的术语概述。讨论本文档其余部分所用的重要术语概念和术语。"
   services="service-fabric"
   documentationCenter=".net"
   authors="msfussell"
   manager="timlt"
   editor="chackdan;subramar"/>

<tags
   ms.service="service-fabric"
   ms.date="04/05/2016"
   wacn.date="07/04/2016"/>

# Service Fabric 术语概述

Service Fabric 是一种分布式系统平台，可让你轻松打包、部署和管理可缩放、可靠的微服务。本主题详细说明 Service Fabric 所使用的术语，以帮助你了解文档中其他位置使用的术语。

## 基础结构概念
**群集**：一组通过网络连接在一起的虚拟机或物理计算机，你的微服务将在其中部署和管理。群集可以扩展到成千上万台计算机。

**节点**：属于群集一部分的计算机或 VM 称为节点。需为每个节点分配节点名称（字符串）。节点具有各种特征，如放置属性。每个计算机或 VM 都有一个自动启动 Windows 服务 `FabricHost.exe`，此服务在引导时开始运行，然后启动两个可执行文件：`Fabric.exe` 和 `FabricGateway.exe`。这两个可执行文件构成了节点。在测试方案中，你可以通过运行 `Fabric.exe` 和 `FabricGateway.exe` 的多个实例，在单台电脑或 VM 上托管多个节点。

## 应用程序概念
**应用程序类型**：分配给服务类型集合的名称/版本。此信息在 `ApplicationManifest.xml` 文件中定义，在应用程序包目录嵌入，并复制到 Service Fabric 群集的映像存储中。然后，你可以基于此应用程序类型，在群集内创建命名的应用程序。

有关详细信息，请阅读 [Application Model（应用程序模型）](/documentation/articles/service-fabric-application-model/)一文。

**应用程序包**：一个磁盘目录，其中包含应用程序类型的 `ApplicationManifest.xml` 文件。此文件引用构成应用程序类型的每个服务类型的服务包。应用程序包目录中的文件将复制到 Service Fabric 群集的映像存储。例如，电子邮件应用程序类型的应用程序包可能包含对队列服务包、前端服务包和数据库服务包的引用。

**命名应用程序**：将应用程序包复制到映像存储后，可以通过指定应用程序包的应用程序类型（使用其名称/版本）在群集内创建应用程序的实例。将为每个应用程序类型实例分配一个类似于下面的 URI 名称：`"fabric:/MyNamedApp"`。在群集中，你可以从单个应用程序类型创建多个命名应用程序。还可以从不同的应用程序类型创建命名应用程序。可单独管理每个命名应用程序并设置其版本。

**服务类型**：分配给服务的代码包、数据包、配置包的名称/版本。此信息在 `ServiceManifest.xml` 文件中定义，在服务包目录中嵌入，然后，应用程序包的 `ApplicationManifest.xml` 文件将引用该服务包目录。在群集中创建命名应用程序后，你可以从应用程序类型的服务类型之一创建命名服务。服务类型的 `ServiceManifest.xml` 文件描述该服务。

有关详细信息，请阅读 [Application Model](/documentation/articles/service-fabric-application-model/)（应用程序模型）一文。

有两种类型的服务：

- **无状态：**当服务的持久状态存储在 Azure 存储空间、Azure SQL 数据库、Azure DocumentDB 等外部存储服务中时，请使用无状态服务。当服务根本没有持久性存储时，也应使用无状态服务。以计算器服务为例，首先要将值传递给该服务，然后服务使用这些值执行计算并返回结果。

- **有状态：**如果你要让 Service Fabric 通过其 Reliable Collections 或 Reliable Actors 编程模型管理服务状态，请使用有状态服务。创建命名服务时，需指定想要让状态分散到多少个分区（提供扩展性），并指定要在所有节点上复制状态几次（提供可靠性）。每个命名服务都有一个主副本和多个辅助副本。可通过写入主副本来修改命名服务的状态；然后，Service Fabric 会将此状态复制到所有辅助副本，使状态保持同步。当主副本失败时，Service Fabric 会自动检测到此状态，并将现有的辅助副本升级为主副本，然后创建新的辅助副本。

**服务包**：一个磁盘目录，其中包含服务类型的 `ServiceManifest.xml` 文件。此文件引用服务类型的代码、静态数据和配置包。应用程序类型的 `ApplicationManifest.xml` 文件引用服务包目录中的文件。例如，服务包可能引用构成数据库服务的代码、静态数据和配置包。

**命名服务**：创建命名应用程序后，可以通过指定服务类型（使用其名称/版本），在群集中创建应用程序服务类型之一的实例。需为每个服务类型实例分配一个 URI 名称，该名称归并到实例的命名应用程序的 URI 之下。例如，如果在命名应用程序“MyNamedApp”中创建命名服务“MyDatabase”，则 URI 将类似于：`"fabric:/MyNamedApp/MyDatabase"`。在一个命名应用程序中可以创建多个命名服务。每个命名服务可以有自身的分区方案和实例/副本计数。

**代码包**：一个磁盘目录，其中包含服务类型的可执行文件（通常是 EXE/DLL 文件）。服务类型的 `ServiceManifest.xml` 文件引用代码包目录中的文件。创建命名服务时，代码包中的文件将复制到 Service Fabric 选择用来运行命名服务的节点，然后代码将开始运行。有两种类型的代码包可执行文件：

- **来宾可执行文件**：在主机操作系统（Windows 或 Linux）上按原样运行的可执行文件。也就是说，这些可执行文件不会链接到或引用任何 Service Fabric 运行时文件，因此不会使用任何 Service Fabric 编程模型。这些可执行文件无法利用某些 Service Fabric 功能（例如，用于终结点发现的命名服务），并且无法报告特定于每个服务实例的负载指标。

- **服务主机可执行文件**：这是一些通过链接到 Service Fabric 运行时文件来使用 Service Fabric 编程模型的可执行文件。这会将可执行文件的代码部分绑定到 Service Fabric，从而启用其他功能。例如，命名服务实例可在 Service Fabric 命名服务中注册终结点，还可以报告负载指标。

**数据包**：一个磁盘目录，其中包含服务类型的静态只读数据文件（通常是照片、声音和视频文件）。服务类型的 `ServiceManifest.xml` 文件引用数据包目录中的文件。创建命名服务时，数据包中的文件将复制到 Service Fabric 选择用来运行命名服务的节点，然后代码将开始运行；代码现在可以访问这些数据文件。

**配置包**：一个磁盘目录，其中包含服务类型的静态只读配置文件（通常是文本文件）。服务类型的 `ServiceManifest.xml` 文件引用配置包目录中的文件。创建命名服务时，配置包中的文件将复制到 Service Fabric 选择用来运行命名服务的节点，然后代码将开始运行；代码现在可以访问这些配置文件。

**分区方案**：创建命名服务时，需要指定一个分区方案。包含大量状态的服务将跨分区拆分数据，从而将数据分散在群集的节点上。这样，命名服务的状态便可缩放。在分区中，无状态命名服务具有实例，而有状态命名服务具有副本。通常，无状态命名服务只有 1 个分区，因为它们有没有内部状态。分区实例提供可用性；如果一个实例失败，其他实例可继续正常运行，然后 Service Fabric 将创建新的实例。有状态命名服务在副本中保持其状态，每个分区都有自身的副本集，其中包含保持同步的所有状态。如果某个副本失败，Service Fabric 将从现有副本创建新副本。

有关详细信息，请阅读 [Service Fabric Reliable Services 分区](/documentation/articles/service-fabric-concepts-partitioning/)一文。

## 系统服务
系统在每个群集中创建了一些系统服务，用于提供 Service Fabric 的平台功能。

**命名服务**：每个 Service Fabric 群集有一个命名服务，该服务将服务名称解析到群集中的某个位置，并允许你管理服务名称和属性。这就类似于群集的 Internet 域名服务 (DNS)。使用命名服务，客户端可以安全地与群集中的任何节点通信，来解析服务名称及其位置（亦即掌握实际计算机 IP 地址及其当前运行所在的端口）。使用通信客户端 API 时，尽管应用程序会在群集内移动（例如因为故障、资源平衡或重设群集大小），你也可以开发能够解析当前网络位置的服务和客户端。

有关使用搭配命名服务运行的客户端与服务通信 API 的详细信息，请阅读[与服务通信](/documentation/articles/service-fabric-connect-and-communicate-with-services/)一文。

**映像存储服务**：每个 Service Fabric 群集都有一个映像存储服务，其中保存已部署且版本化的应用程序包。你必须将应用程序包的内容复制到映像存储服务，然后注册该应用程序包内包含的应用程序类型。接着，在预配应用程序类型后，可以创建它的命名应用程序。在删除某个应用程序类型的所有命名应用程序之后，可以从映像存储服务中注销该应用程序类型。

有关将应用程序部署到映像存储服务的详细信息，请阅读[部署应用程序](/documentation/articles/service-fabric-deploy-remove-applications/)一文。

## 内置编程模型
有一些 .NET Framework 编程模型可用于生成 Service Fabric 服务：

**Reliable Services**：用于构建无状态和有状态服务的 API。有状态服务将其状态存储在可靠集合（例如字典或队列）中。你也可以插入各种通信堆栈，如 Web API 和 Windows Communication Foundation (WCF)。

**Reliable Actors** - 用于通过虚拟执行组件编程模型构建无状态和有状态对象的 API。如果你有大量的独立计算/状态单位，此模型可能十分有用。由于此模型使用基于轮次的线程模型，因此最好避免使用向外调用其他执行组件或服务的代码，原因是只有在单个执行组件的所有出站请求都已完成后，该执行组件才能处理其他传入请求。

有关详细信息，请阅读[为服务选择编程模型](/documentation/articles/service-fabric-choose-framework/)一文。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤
了解有关 Service Fabric 的详细信息

- [Service Fabric 概述](/documentation/articles/service-fabric-overview/)
- [为什么通过微服务的方法构建应用程序？](/documentation/articles/service-fabric-overview-microservices/)
- [应用程序方案](/documentation/articles/service-fabric-application-scenarios/)

<!---HONumber=Mooncake_0523_2016-->