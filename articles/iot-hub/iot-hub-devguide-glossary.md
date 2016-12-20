<properties
 pageTitle="开发人员指南 - 术语表 | Azure"
 description="与 IoT 中心相关的常见术语表"
 services="iot-hub"
 documentationCenter=".net"
 authors="dominicbetts"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="multiple"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="09/30/2016"
 wacn.date="12/12/2016" 
 ms.author="dobett"/>  


# IoT 中心术语表
本文列出了一些在 IoT 中心文章中使用的常用术语。

## <a name="advanced-message-queue-protocol"></a>高级消息队列协议
[高级消息队列协议 (AMQP)](https://www.amqp.org/) 是 [IoT 中心](#iot-hub)支持的一种消息传送协议，适用于与设备通信。若要详细了解 IoT Hub 支持的消息传送协议，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## Azure CLI
[Azure 命令行接口 (Azure CLI)](/documentation/articles/xplat-cli-install/) 是一个跨平台、开源、基于 shell 的命令工具，适用于在 Azure 中创建和管理资源。

## <a name="azure-iot-device-sdks"></a> Azure IoT 设备 SDK
提供了多种语言的_设备 SDK_，以便于用户创建与 IoT 中心交互的[设备应用](#device-app)。IoT 中心教程介绍了如何使用这些设备 SDK。可以在此 GitHub [存储库](https://github.com/Azure/azure-iot-sdks)中找到有关设备 SDK 的源代码和进一步信息。

## <a name="azure-iot-gateway-sdk"></a> Azure IoT 网关 SDK
用户可以使用此 SDK 编写应用程序，使和网关连接的设备能够与 [IoT 中心](#iot-hub)通信。IoT 中心网关教程介绍了如何使用此 SDK。可以在此 GitHub [存储库](https://github.com/Azure/azure-iot-gateway-sdk)中找到有关 Azure IoT 网关 SDK 的源代码和进一步信息。

## <a name="azure-iot-service-sdks"></a> Azure IoT 服务 SDK
提供了多种语言的_服务 SDK_，以便于用户创建与 IoT 中心交互的[后端应用](#back-end-app)。IoT 中心教程介绍了如何使用这些服务 SDK。可以在此 GitHub [存储库](https://github.com/Azure/azure-iot-sdks)中找到有关服务 SDK 的源代码和进一步信息。

## <a name="azure-portal"></a> Azure 门户
[Microsoft Azure 门户](https://portal.azure.cn)是一个中心位置，可在其中预配和管理 Azure 资源。该门户使用_边栏选项卡_组织其内容。在某些 IoT 中心教程中，可能会要求使用 [Azure 经典管理门户](https://manage.windowsazure.cn)。

## Azure PowerShell
[Azure PowerShell](/documentation/articles/powershell-install-configure/) 是一个 cmdlet 集合，可用于通过 Windows PowerShell 管理 Azure。你可以使用 cmdlet 来创建、测试、部署和管理通过 Azure 平台传送的解决方案和服务。

## <a name="azure-resource-manager"></a>Azure 资源管理器
使用 [Azure Resource Manager](/documentation/articles/resource-group-overview/) 能够将解决方案中的资源作为组进行处理。可以通过一个协调操作为解决方案部署、更新或删除资源。

## Azure 服务总线
[服务总线](/documentation/services/service-bus/)提供使用企业消息传送的云端通信和有助于将本地解决方案与云端连接的中继通信。某些 IoT 中心教程使用服务总线[队列](/documentation/articles/service-bus-messaging-overview/)。

## Azure 存储空间
[Azure 存储](/documentation/articles/storage-introduction/)是云存储解决方案。它包含可用于存储非结构化的对象数据的 Blob 存储服务。某些 IoT 中心教程使用 blob 存储。

## <a name="back-end-app"></a>后端应用
在 [IoT 中心](#iot-hub)环境中，后端应用是指连接到 IoT 中心的一个面向服务的终结点的应用。例如，后端应用可能检索[设备到云](#device-to-cloud)消息或管理[标识注册表](#identity-registry)。通常，后端应用在云中运行，但在许多教程中，后端应用是在本地开发计算机上运行的控制台应用。

## <a name="cloud-gateway"></a> 云网关
云网关使不能直接连接到 [IoT 中心](#iot-hub)的设备能建立连接。和在设备本地运行的[现场网关](#field-gateway)相反，云网关在云中托管。云网关的一个典型用例是实现设备的协议转换。

## <a name="cloud-to-device"></a> 云到设备
指从 IoT 中心发送到已连接设备的消息。这些消息通常是命令，用于指示设备采取某项操作。有关详细信息，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## <a name="connection-string"></a> 连接字符串
使用应用程序代码中的连接字符串来封装连接到终结点所需的信息。连接字符串通常包含终结点的地址和安全信息，但连接字符串的格式因服务而异。

## <a name="custom-gateway"></a> 自定义网关
网关使不能直接连接到 [IoT 中心](#iot-hub)的设备能建立连接。可以使用 [Azure IoT 网关 SDK](#azure-iot-gateway-sdk) 生成自定义网关，以便使用自定义逻辑处理消息和自定义协议转换。

## <a name="data-point-message"></a> 数据点消息
数据点消息是指[设备到云](#device-to-cloud)的消息，内含[遥测](#telemetry)数据（例如风速或温度）。

## 所需配置
在与[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)相关的使用中，所需配置是指设备克隆中要与设备同步的完整的属性和元数据集。

## <a name="desired-properties"></a> 所需属性
在与[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)相关的使用中，所需属性是设备克隆的一部分，和[报告属性](#reported-properties)一起用于同步设备配置或条件。所需属性只能由[后端应用](#back-end-app)设置，并由[设备应用](#device-app)遵守。

## <a name="device-to-cloud"></a>设备到云
指从已连接设备发送到 [IoT 中心](#iot-hub)的消息。这些消息可以是[数据点](#data-point-message)消息，也可以是[交互式](#interactive-message)消息。有关详细信息，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## <a name="device"></a> 设备
在 IoT 上下文中，设备通常是指小型、独立的计算设备，可用于收集数据或控制其他设备。例如，设备可以是环境监视设备，也可以是控制器，控制温室中的浇水和通风系统。 

## <a name="device-app"></a>设备应用
设备应用在用户的[设备](#device)上运行，并处理与 [IoT 中心](#iot-hub)的通信。通常情况下，实现设备应用时会使用一个 [Azure IoT 设备 SDK](#azure-iot-device-sdks)。在许多 IoT 教程中，为方便起见使用[模拟设备](#simulated-device)。

## <a name="device-condition"></a> 设备条件
指[设备应用](#device-app)报告的设备状态信息，例如当前正在使用的连接方法。[设备应用](#device-app)还可以报告其功能。可以使用设备克隆查询条件和功能的信息。

## <a name="device-data"></a> 设备数据
设备数据是指存储在 IoT 中心[标识注册表](#identity-registry)中的每个设备数据。可以导入和导出此数据。

## 设备资源管理器
设备资源管理器是一个在 Windows 上运行的工具，用户可使用该工具管理[标识注册表](#identity-registry)中的设备，以及发送和接收设备的消息。

## 设备标识 REST API
通过[设备标识 REST API](https://docs.microsoft.com/rest/api/iothub/device-identities-rest) 可使用 REST API管理在[标识注册表](#identity-registry)中注册的设备。通常情况下，使用 IoT 中心教程中演示的一种较高级别的[服务 SDK](#azure-iot-service-sdks)。

## <a name="device-identity"></a> 设备标识
设备标识是分配给在[标识注册表](#identity-registry)中注册的每个设备的唯一标识符。

## 设备管理
设备管理包含在 IoT 解决方案中管理设备的完整生命周期，包括规划、预配、配置、监视和停用设备。

## 设备管理模式
[IoT 中心](#iot-hub)支持常见的设备管理模式，包括重新启动、执行恢复出厂设置，以及执行设备的固件更新。

## 设备消息传送 REST API
可以在设备上使用[设备消息传送 REST API](https://docs.microsoft.com/rest/api/iothub/device-messaging-rest-apis)，以便将设备到云消息发送到 IoT 中心，以及从 IoT 中心接收[云到设备](#cloud-to-device)消息。通常情况下，使用 IoT 中心教程中演示的一种较高级别的[设备 SDK](#azure-iot-device-sdks)。

## 设备预配
设备预配是将初始[设备数据](#device-data)添加到解决方案中的存储的过程。若要使新设备能够连接到中心，必须将新设备 ID 和密钥添加到 IoT 中心的[标识注册表](#identity-registry)。在预配过程中，你可能需要初始化其他解决方案存储中的设备特定数据。

## 设备克隆
[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)是存储设备状态信息（如元数据、配置和条件）的 JSON 文档。[IoT 中心](#iot-hub)为在 IoT 中心预配的每台设备保留一个设备克隆。借助设备克隆可以在设备和解决方案后端之间同步[设备条件](#device-condition)和配置。可以通过查询设备克隆来定位特定设备和查询长时间运行的操作状态。

## 设备克隆查询
[设备克隆查询](/documentation/articles/iot-hub-devguide-query-language/)使用类似 SQL 的查询语言从设备克隆中检索信息。可以使用相同的查询语言检索在 IoT 中心运行的[作业](#job)的信息。

## 设备克隆同步
设备克隆同步使用设备克隆中的[所需属性](#desired-properties)配置设备并检索设备中的[报告属性](#reported-properties)，以将其存储在设备克隆中。

## <a name="direct-method"></a> 直接方法
可以通过调用 IoT 中心的 API，使用[直接方法](/documentation/articles/iot-hub-devguide-direct-methods/)触发在设备上执行的方法。

## 终结点
IoT 中心公开了多个[终结点](/documentation/articles/iot-hub-devguide-endpoints/)，以便使应用能够连接到中心。有面向设备的终结点，通过此终结点设备可以执行一些操作，例如发送[设备到云](#device-to-cloud)消息和接收[云到设备](#cloud-to-device)消息。有面向服务的终结点，通过此终结点[后端应用程序](#back-end-app)可以执行一些操作，如[设备标识](#device-identity)管理和设备克隆管理。

## 事件中心服务
[事件中心](/documentation/articles/event-hubs-what-is-event-hubs/)是高度可扩展的数据传入服务，每秒可以引入数以百万计的事件。该服务使用户能够处理和分析连接设备和应用程序产生的大量数据。有关该服务与 IoT 中心服务的比较的信息，请参阅 [Azure IoT 中心和 Azure 事件中心的比较](/documentation/articles/iot-hub-compare-event-hubs/)。

## 与事件中心兼容的终结点
若要读取发送到 IoT 中心的[设备到云](#device-to-cloud)的消息，可以先连接到中心的终结点，然后使用与事件中心兼容的任何方法读取这些消息。与事件中心兼容的方法包括使用[事件中心 SDK](/documentation/articles/event-hubs-programming-guide/) 和 [Azure 流分析](/documentation/articles/stream-analytics-introduction/)。

## <a name="field-gateway"></a>现场网关
无法直接连接到 [IoT 中心](#iot-hub)的设备可以通过现场网关进行连接，而现场网关通常与设备一起部署在本地。有关详细信息，请参阅[什么是 Azure IoT 中心？](/documentation/articles/iot-hub-what-is-iot-hub/)。

## 免费帐户
你可以创建[免费的 Azure 帐户](/pricing/1rmb-trial/)，以便使用 IoT 中心服务（及其他 Azure 服务）完成 IoT 中心教程和试验。

## 网关
网关使不能直接连接到 [IoT 中心](#iot-hub)的设备能建立连接。另请参阅[现场网关](#field-gateway)、[云网关](#cloud-gateway)和[自定义网关](#custom-gateway)。

## <a name="identity-registry"></a>标识注册表
[标识注册表](/documentation/articles/iot-hub-devguide-identity-registry/)是 IoT 中心的内置组件，用于存储允许连接到中心的单个设备的信息。

## <a name="interactive-message"></a> 交互式消息
交互式消息是[云到设备](#cloud-to-device)的消息，可在应用程序后端触发即时操作。例如，设备可能会发送故障警报，而该故障会自动记录到 CRM 系统中。

## <a name="iot-hub"></a> IoT 中心
IoT 中心是一项完全托管的 Azure 服务，可在数百万个 IoT 设备和一个解决方案后端之间实现安全可靠的双向通信。有关详细信息，请参阅[什么是 Azure IoT 中心？](/documentation/articles/iot-hub-what-is-iot-hub/) 使用 [Azure 订阅](#subscription)可以创建 IoT 中心来处理 IoT 消息传送工作负荷。

## IoT 中心指标
[IoT 中心指标](/documentation/articles/iot-hub-metrics/)向用户提供有关 [Azure 订阅](#subscription)中的 IoT 中心的状态数据。可以使用度量值评估服务以及连接到服务的设备的总体运行状况。指标可以帮助用户了解 IoT 中心发生的情况，并调查根本原因，而无需联系 Azure 支持部门。

## IoT 中心查询语言
[IoT 中心查询语言](/documentation/articles/iot-hub-devguide-query-language/)是一种类似 SQL 的语言，用于查询[作业](#job)和设备克隆。

## IoT 中心资源提供程序 REST API
可以使用 [IoT 中心资源提供程序 REST API](https://docs.microsoft.com/rest/api/iothub/resourceprovider/iot-hub-resource-provider-rest) 管理 [Azure 订阅](#subscription)中的 IoT 中心，以便执行一些操作，例如创建、更新和删除中心。

## IoT 套件
Azure IoT 套件将多个 Azure 服务和预配置解决方案打包在一起。有了这些预配置解决方案，用户就可以快速启动常见 IoT 方案的端到端实现。有关详细信息，请参阅[什么是 Azure IoT 套件？](/documentation/articles/iot-suite-overview/)

## iothub-explorer
iothub-explorer 是跨平台的命令行工具。使用该工具可以管理[标识注册表](#identity-registry)中的设备、向设备发送消息和文件和接收来自设备的消息和文件，以及监视 IoT 中心的操作。

## <a name="job"></a> 作业
解决方案后端可以使用[作业](/documentation/articles/iot-hub-devguide-jobs/)计划和跟踪在 IoT 中心注册的一组设备的活动。活动包括更新设备克隆的[所需属性](#desired-properties)、更新设备克隆[标记](#tags)，以及调用[直接方法](#direct-method)。[IoT 中心](#iot-hub)还使用作业在[标识注册表](#identity-registry)中[导入和导出](/documentation/articles/iot-hub-devguide-identity-registry/#import-and-export-device-identities)。

## 模块
在 [Azure IoT 网关 SDK](/documentation/articles/iot-hub-linux-gateway-sdk-get-started/) 中，[模块](/documentation/articles/iot-hub-linux-gateway-sdk-get-started/#azure-iot-gateway-sdk-concepts)是执行特定任务的组件。任务可能包括从设备引入消息、转换消息，或者将消息发送到 IoT 中心。中转站负责在模块之间转发消息。Azure IoT 网关 SDK 包括一组示例模块。用户还可以创建自己的自定义模块。

## MQTT
[MQTT](http://mqtt.org/) 是 [IoT 中心](#iot-hub)支持的一种消息传送协议，适用于与设备通信。若要详细了解 IoT Hub 支持的消息传送协议，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## 操作监视
IoT 中心[操作监视](/documentation/articles/iot-hub-operations-monitoring/)功能可让用户实时监视其 IoT 中心的操作状态。[IoT 中心](#iot-hub)可以跨多个类别的操作跟踪事件。可以选择将一个或多个类别的事件发送到 IoT 中心终结点进行处理。你可以监视数据中是否有错误，或根据数据模式设置更复杂的处理行为。

## <a name="physical-device"></a> 物理设备
物理设备是真实的设备，如连接到 IoT 中心的 Raspberry Pi。为方便起见，许多 IoT 中心教程使用[模拟设备](#simulated-device)，以便在本地计算机上运行示例。

## <a name="primary-and-secondary-keys"></a> 主要和次要密钥
连接到 IoT 中心的面向设备或面向服务的终结点时，[连接字符串](#connection-string)包含密钥以授予用户访问权限。向[标识注册表](#identity-registry)中添加设备或向中心添加[共享访问策略](#shared-access-policy)时，服务生成主要和次要密钥。拥有两个密钥能够在更新密钥时从一个密钥切换到另一个密钥，而不丢失对中心的访问。

## 协议网关
协议网关通常部署在云中，为连接到 [IoT 中心](#iot-hub)的设备提供协议转换服务。有关详细信息，请参阅[什么是 Azure IoT 中心？](/documentation/articles/iot-hub-what-is-iot-hub/)。

## 配额和限制
各种[配额](/documentation/articles/iot-hub-devguide-quotas-throttling/)可用于 [IoT 中心](#iot-hub)，其中许多配额因所在的中心的层而异。[IoT 中心](#iot-hub)在运行时也对服务的使用执行一些[限制](/documentation/articles/iot-hub-devguide-quotas-throttling/)。

## 报告的配置
在与[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)相关的使用中，报告的配置是指设备克隆中的完整的属性和元数据集，该配置由设备报告给解决方案后端。

## <a name="reported-properties"></a> 报告属性
在与[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)相关的使用中，报告属性是设备克隆的一部分，和[所需属性](#desired-properties)一起用于同步设备配置或条件。报告属性只能由[设备应用](#device-app)设置，可由[后端应用](#back-end-app)读取和查询。

## 资源组
[Azure Resource Manager](#azure-resource-manager) 使用资源组将相关的资源组合在一起。通过使用资源组，可以对组中的所有资源同时执行操作。

## 重试策略
连接到云服务时使用重试策略来处理 [暂时性错误][]。

## SASL PLAIN
SASL PLAIN 是一种协议， [AMQP](#advanced-message-queue-protocol) 协议使用它传输安全令牌。

## <a name="shared-access-signature"></a> 共享访问签名
共享访问签名 (SAS) 是基于 SHA–256 安全哈希或 URI 的身份验证机制。SAS 身份验证有两个组件：_共享访问策略_和_共享访问签名_（通常称为令牌）。设备使用 SAS 在 IoT 中心进行身份验证。[后端应用](#back-end-app)也使用 SAS 在 IoT 中心的面向服务的终结点上进行身份验证。通常，在[连接字符串](#connection-string)中包含 SAS 令牌，应用使用此令牌建立与 IoT 中心的连接。

## <a name="shared-access-policy"></a> 共享访问策略
共享访问策略定义向具有有效的[主要密钥或次要密钥](#primary-and-secondary-keys)（与该策略相关联）的任何人授予的权限。用户可以在[门户预览](#azure-portal)中管理中心的共享访问策略和密钥。

## <a name="simulated-device"></a> 模拟设备
为方便起见，许多 IoT 中心教程使用模拟设备，以便在本地计算机上运行示例。相反，[物理设备](#physical-device)是真实的设备，如连接到 IoT 中心的 Raspberry Pi。

## 解决方案
*解决方案* 可以是包含一个或多个项目的 Visual Studio 解决方案。 *解决方案* 也可能是包括诸如设备、[设备应用](#device-app)、IoT 中心、其他 Azure 服务和[后端应用](#back-end-app)等元素的 IoT 解决方案。

## <a name="subscription"></a> 订阅
Azure 订阅是发生计费的地方。用户创建的每个 Azure 资源或使用的 Azure 服务均与单个订阅关联。许多配额也在订阅级别应用。

## 系统属性
在与[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)相关的使用中，系统属性为只读，其中包括与设备使用情况相关的信息，例如上次活动时间和连接状态。

## <a name="tags"></a> 标记
在与[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)相关的使用中，标记是指由应用程序后端以 JSON 文档形式存储和检索的设备元数据。标记对设备上的应用不可见。

## <a name="telemetry"></a> 遥测
设备收集遥测数据，如风速或温度，并使用[数据点消息](#data-point-message)将遥测数据发送到 IoT 中心。

## 令牌服务
可以使用令牌服务对设备实施身份验证机制。它使用包含 **DeviceConnect** 权限的 IoT 中心[共享访问策略](#shared-access-policy) 创建*设备范围的* 令牌。这些令牌可让设备连接到 IoT 中心。设备通过令牌服务使用自定义的身份验证机制进行身份验证。如果设备成功通过身份验证，那么令牌服务向设备颁发 SAS 令牌用于访问 IoT 中心。

## X.509 客户端证书
设备可以使用 X.509 证书在 [IoT 中心](#iot-hub)进行身份验证。使用 X.509 证书是使用 [SAS 令牌](#shared-access-signature)的替代方案。



[暂时性错误]: https://msdn.microsoft.com/library/hh680901(v=pandp.50).aspx
<!---HONumber=Mooncake_1205_2016-->