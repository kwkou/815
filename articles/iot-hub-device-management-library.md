<properties
 pageTitle="设备管理库简介 |Azure"
 description="Azure IoT 中心设备管理 (DM) 客户端库"
 services="iot-hub"
 documentationCenter=""
 authors="CarlosAlayo"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="07/04/2016"/>

# 引入 Azure IoT 中心设备管理 (DM) 客户端库

## 概述

Azure IoT 中心设备管理库可让你通过 Azure IoT 中心管理 IoT 设备。“管理”包括重启、恢复出厂设置以及更新固件等操作。我们当前提供独立于平台的 C 库，但很快就会增加对其他语言的支持。如 [Azure IoT 中心设备管理概述][lnk-dm-overview]中所述，IoT 中心设备管理中有三个关键概念：

- 设备克隆
- 设备作业
- 设备查询

如果不熟悉上述内容，则建议在继续之前先阅读相关概述。所有内容都与客户端库紧密相关。

DM 客户端库在设备管理方面主要负责以下两项任务：

- 同步物理设备上的属性与 IoT 中心中其对应的设备克隆
- 编排 IoT 中心发送到设备的设备作业

物理设备的设备属性（如电池剩余电量和序列号）定期与基于云的设备克隆同步，因此 IoT 中心始终具有每个物理设备的最新视图（只要该设备已连接）。

服务与物理设备交互的主要方式是通过设备作业以及设备克隆完成。IoT 中心中提供了以下作业类型。DM 客户端库负责设备作业的大部分业务流程，但对设备执行必要的回调以支持每种作业类型还要取决于作为开发人员的你。

1.	**固件更新**：更新物理设备上的固件（或 OS 映像）
2.	**重启**：重启物理设备
3.	**恢复出厂设置**：将物理设备上的固件（或 OS 映像）还原为设备上存储的出厂提供的备份映像
4.	**配置更新**：配置在物理设备上运行的 IoT 中心客户端代理
5.	**读取设备属性**：获取物理设备上的最新设备属性值
6.	**写入设备属性**：更改物理设备上的设备属性

以下各节将引导你整体了解客户端库的体系结构，并为你提供有关如何在你的设备上实现其他设备对象的指导说明。

## 设备管理客户端库设计原则和功能概念
DM 客户端库旨在考虑可移植性和跨平台集成。这已通过以下设计决策来实现：

1.	通过 COAP 标准协议建立在 LWM2M 的基础之上，以适应各种不同设备的可扩展性。
2.	采用 ANSI C99 编写，以便于可移植到多种平台。
3.	通过 TCP/TLS 和 Azure IoT 中心身份验证（SAS 令牌）进行保护，以便它可以用于高安全性方案。
4.	基于 [Eclipse Wakaama][lnk-Wakaama] OSS 项目，以利用现有代码并回馈社区。

### 相关 LWM2M 概念
我们选择 LWM2M 标准，以适应各种不同设备的可扩展性。为了简化开发体验，我们已提取大多数协议。但是，理解库的基本原则（主要是其数据模型和数据的传输方式）也很重要。

#### 对象和资源：LWM2M 中的数据模型
LWM2M 数据模型引入了对象和资源的概念：

- **对象**描述系统中的一组一致的功能实体，如的设备和固件更新。
- **资源**描述包含在这些对象中的属性或操作，如电池剩余电量信息和重新启动操作。

请注意，此模型中有两个“1 对多”的关系：

- **设备和对象**：每个设备均可以拥有多个对象。例如，Contoso 设备必须具有“设备对象”和“服务器对象”（我们将在下一节中详细描述这些对象）。
- **对象和资源**：每个对象可以有多个资源。例如，某个对象可以包含 Contoso 设备固件更新资源，如存储新映像所在的包 URI。

#### 观察/通知模式：在 LWM2M 中传输数据的方式
除了这些概念，还务必了解数据从设备流动到服务的方式。为此，LWM2M 定义了“观察/通知”模式。当物理设备连接到服务时，IoT 中心对所选设备属性启动“观察”。然后，物理设备将设备属性的更改“通知”服务。

在客户端库中，我们已将观察/通知模式作为从设备将设备管理数据发送到 IoT 中心的方式来实现。该模式受两个参数控制：

- **最小时间段**：设备延迟发送所观察属性的更新的时间段。其设为 5 分钟。因此，设备将最多每隔 5 分钟发送所观察属性的更新，即使值会更频繁地更改也是如此。这意味着，如果属性每隔一分钟发生变化，服务也只能看到第 5 分钟的最后一次更改。

- **最大时间段**：设备不论所观察属性更改与否，发送属性值的更新的时间段。其设为 6 小时。因此，设备将至少每隔 6 小时发送所观察属性的值的更新，即使已更改也是如此。

> [AZURE.NOTE]  这些参数的唯一例外是用于固件更新作业的属性将最小时间段设为 30 秒。这是因为这些属性在作业执行期间变化非常频繁，因此我们已减小了最小时间段以便加快更新进程。

## 客户端库功能和体系结构
正如概述中所述，客户端库会负责处理两个主要功能：

- 同步物理设备与 IoT 中心中其对应的设备克隆
- 编排 IoT 中心发送到设备的设备作业。

在本节，我们将对其进行深入了解。

### 将物理设备及其设备克隆同步
客户端库使用通知观察/模式来保持设备克隆更新。请记住，设备克隆是物理设备的服务端表示形式。若要执行此同步，将发生以下过程：

1. 设备通常在初始化过程中注册到该服务。示例：“我是一个设备，我有设备 ID Contoso，使用访问令牌 Y”。
2. 该服务会观察对象上的资源。示例：“请让我了解 Contoso 设备的电池剩余电量的最新信息”。
3. 设备会将资源的值通知服务。通知的频率基于最小和最大时间段。示例：“下午 5:00 电池剩余电量为 99%，下午 5:05 电池剩余电量为 90%，等等”。

### 编排设备作业：设备克隆和设备作业如何协同工作
若要对物理设备执行操作，服务必须找到相关的设备克隆。这可以通过查询属性或搜索特定 ID 实现。知悉孪生 ID（并且因此与物理设备匹配），然后该服务准备在物理设备上启动设备作业。

设备作业表示一组表示我们要执行的特定进程的不同命令（例如：固件更新步骤）。设备作业可由多个步骤组成：

1.	设备双子星具有表示设备作业状态的属性。
2.	为了在设备作业中的不同进程之间继续，设备克隆中的属性必须是某个特定值，因此物理设备必须设置正确的属性，以便它可以被同步到设备克隆，作业也可以继续进行。

如果该进程由多个步骤组成，则此序列会不断重复（例如：固件更新期间，设备克隆将通过各种步骤多次更改其属性，然后才考虑完成该作业）。

## 在客户端中实现设备管理的指南
在前面几节，我们已经了解了设备管理客户端库的功能、其设计原则以及它与 LWM2M 的相关程度。现在我们将重点介绍如何在运行时中组装片段。

实际上，客户端库处理设备与服务之间的通信，因此只需实现设备特定逻辑即可。它由两个部分组成：

1.	**实现设备特定的详细信息**：这包括写入设备特定的逻辑以执行相应回调中的服务所需的操作（例如：下载固件包）。下面列出了针对固件更新包的此回调的示例。你可以在[此处][lnk-github1]文件夹的 .c 文件的中找到回调。

            object_firmwareupdate *obj = get_firmwareupdate_object(0);
            obj->firmwareupdate_packageuri_write_callback =     start_firmware_download;
            // platform specific code
            obj->firmwareupdate_update_execute_callback = start_firmware_update;
            //platform specific code


2.	**当属性更改时通知客户端库**：这可以通过调用[此处][lnk-github2]文件夹的 .h 文件中的相应集函数来实现。

        IOTHUB_CLIENT_RESULT set_firmwareupdate_state(uint16_t instanceId, int value);

### 在 DM 客户端库中实现所需的对象

我们刚才已说明了实现设备特定的逻辑以执行设备作业的方法。现在，我们将说明哪些对象可供你使用。

其中一些对象是必需的，这意味着你需要实现设备特定的逻辑，以便使其成为 IoT 中心设备管理的一部分。其他对象是可选的，这样你可以根据服务需求进行选择（例如：你可以选择不想要使用 IoT 中心执行固件更新）。以下是每种情况的说明：

- **设备对象（必需）**：提供设备特定的信息，如制造商信息、型号、序列号、设备时间。该服务可以读取此信息，并在某些情况下更新它。它还定义了服务可以对设备执行的两种操作：重新启动和出厂重置。
- **服务器对象（必需）**：包含用来连接到 IoT 中心的连接参数和设置，例如注册和传输绑定的生存期。该服务只能读取此信息。
- **Config 对象（可选）**：包含用户定义的配置信息，可以从设备中查询该信息或将其从服务推送到设备以便于设备的配置。该服务可以读取和更新此信息。
- **固件更新对象（可选）**：提供服务可以调用的固件更新操作。它还提供了固件包的位置以及正在进行的固件更新操作的状态等信息。

每个对象都有一组与之相关联的资源（请记住一对多关联）。在本文末尾还包括了 Azure IoT 中心设备管理中支持的 LWM2M 对象及其所有关联资源的列表。

> [AZURE.NOTE] 当前版本的系统不支持自定义设备属性、多个资源实例和多个对象实例。

### 汇总

在下面你可以找到汇总了所有的不同片段的关系图。在右侧（蓝色），你可以查看设备管理客户端库的不同组件：对象、处理程序和通知方法。左侧（绿色）表示在设备应用程序级别编写所需的逻辑。客户端库将应用程序逻辑与 IoT 中心相连接，从而确保通信和设计能够得到正确处理。

下图显示 DM 客户端库组件。

![][img-library-overview]

## 下一步：获取动手体验
本文介绍了对 C 使用 IoT 中心设备管理客户端库的基础知识。

若要获取动手体验，可以访问以下资源：

- Intel Edison 固件更新示例：通过使用 Intel Edison 设备启用的设备管理功能的示例。请参阅 [iotdm\_edison\_sample][lnk-edison-sample]。
- 模拟设备示例：在 Li
- 示例。请参阅 [iotdm\_simple\_sample][lnk-simple-sample]
- 若要了解有关 LWM2M 对象的详细信息，请参阅 [OMA LWM2M object and resource registry（OMA LWM2M 对象和资源注册表）][lnk-oma]

## 附录：目前支持的 LWM2M 对象和资源

### 设备对象

| 资源名称 | 允许在资源上执行的远程操作 | 类型 | 范围和单位 | 说明 |
|-----------------|--------------------------------------|---------|-----------------|-------------|
| 制造商 | 读取 | String | | 制造商名称 |
| ModelNumber | 读取 | String | | 模型标识符（制造商指定的字符串） |
| DeviceType | 读取 | String | | 设备的类型（制造商指定的字符串）<br/>注意：这会映射到服务器端 API **SystemPropertyNames.DeviceDescription** |
| 序列号 | 读取 | String | | 设备的序列号 |
| FirmwareVersion | 读取 | String | | 设备的当前固件版本 |
| HardwareVersion | 读取 | String | | 设备的当前硬件版本 |
| 重新启动 | 执行 | | | 重新启动设备。 |
| FactoryReset | 执行 | | | 执行设备的出厂重置，使设备具有与初始部署时相同的配置。 |
| CurrentTime | 读取<br/>写入 | 时间 | | 设备的当前 UNIX 时间。客户端应负责每隔一秒就提高此时间值。<br/>DM 服务器能够写入此资源，使客户端与服务器上的时间同步。 |
| UTCOffset | 读取<br/>写入 | String | | UTC 时差生效。 |
| 时区 | 读取<br/>写入 | String | | 指示设备所在的时区。 |
| MemoryFree | 读取 | 整数 | KB | 可存储设备中的数据和软件的当前估计可用存储空间内存 |
| MemoryTotal | 读取 | 整数 | KB | 可以存储设备中的数据和软件的存储空间的总量 |
| BatteryLevel | 读取 | 整数 | 0-100% | 当前电池剩余电量百分比（从 0 到 100） |
| BatteryStatus | 读取 | 整数 | 0-6 | **0**：电池正常运行且未接通电源。<br/>**1**：电池正在充电。<br/>**2**：电池已充满电并已介入电网。<br/>**3**：电池已损坏。<br/>**4**：电池电量低。<br/>**5**：电池不存在。<br/> **6**：电池信息不可用。 |

### 固件更新对象

| 资源名称 | 操作 | 类型 | 范围和单位 | 说明 |
|----------------|-----------|---------|-----------------|-------------|
| 程序包 | 写入 | 不透明 | | 固件包采用二进制格式。<br/>映射到服务 API：<br/>**SystemPropertyNames.FirmwarePackage** |
| PackageURI | 写入 | String | 0-255 字节 | 设备可以在其中下载固件包的 URI。<br/>映射到服务 API：**SystemPropertyNames.FirmwarePackageUri** |
| 更新 | 执行 | | | 通过使用存储在包中的固件包，或通过使用从包 URI 下载的固件更新固件。<br/>映射到服务 API：<br/>**ScheduleFirmwareUpdateAsync** |
| 状态 | 读取 | 整数 | 1-3 | 固件更新进程的状态：<br/>**1**：空闲。这可以在下载固件包之前或应用固件包之后发生。<br/>**2**：正在下载固件包。<br/>**3**：已下载固件包。<br/> 映射到服务 API：**SystemPropertyNames.FirmwareUpdateState** |
| UpdateResult | 读取 | 整数 | 0-6 | 下载或更新固件的结果<br/>**0**：默认值。<br/>**1**：固件更新成功。<br/>**2**：新固件包的存储空间不足。<br/>**3**：在固件包下载过程中用尽内存。<br/>**4**：在固件包下载过程中连接丢失。<br/>**5**：新下载的包的 CRC 检查失败。<br/>**6**：固件包类型不受支持。<br/>**7**：URI 无效。映射到服务 API：**SystemPropertyNames.FirmwareUpdateResult** |
| PkgName | 读取 | String | 0-255 字节 | **包**资源引用的固件包的描述性名称<br/>映射到服务 API：<br/>**SystemPropertyNames.FirmwarePackageName** |
| PackageVersion | 读取 | String | 0-255 字节 | **包**资源引用的固件包的版本<br/>映射到服务 API：<br/>**SystemPropertyNames.FirmwarePackageVersion** |

### LWM2M 服务器对象

| 资源名称 | 操作 | 类型 | 范围和单位 | 说明 |
|------------------------|------------|---------|-----------------|---------------|
| 默认的最小时间段 | 读取写入 | 整数 | 秒 | 设备延迟发送所观察属性的更新的时间段。例如，给定 **DefaultMinPeriod** 为 5 分钟，设备将最多每隔 5 分钟发送所观察属性的更新，即使值会更频繁地更改也是如此。映射到服务 API：**SystemPropertyNames.DefaultMinPeriod** |
| 默认的最大时间段 | 读取写入 | 整数 | 秒 | 设备不论所观察属性更改与否，发送属性值的更新的时间段（以秒为单位）。例如，给定 **DefaultMaxPeriod** 为 6 小时，观察到的属性无论资源更改与否，都会最少每隔 6 小时发送该属性的值的更新。<br/>映射到服务 API：<br/>**SystemPropertyNames.DefaultMaxPeriod** |
| 生存期 | 读取写入 | 整数 | 秒 | 设备的注册生存期。新注册或更新请求需要在此生存期内通过设备接收，否则该设备会从该服务注销。<br/>映射到服务 API：<br/>**SystemPropertyNames.RegistrationLifetime** |

### Config 对象

| 资源名称 | 操作 | 类型 | 范围和单位 | 说明 |
|---------------|------------|--------|-----------------|-------------|
| 名称 | 读取写入 | String | | 唯一地标识要读取或更新的设备配置的名称。 |
| 值 | 读取写入 | String | | 唯一地标识要读取或更新的配置值。 |
| 应用 | 执行 | | | 对设备应用配置更改。 |





[img-library-overview]: ./media/iot-hub-device-management-library/library.png
[lnk-dm-overview]: /documentation/articles/iot-hub-device-management-overview/
[lnk-get-started]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-simple-sample]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c/iotdm_client/samples/iotdm_simple_sample
[lnk-edison-sample]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c/iotdm_client/samples/iotdm_edison_sample
[Azure IoT Hub device SDK]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c
[Azure IoT Hub]: Link%20to%20DM%20Overview
[Lightweight M2M]: http://openmobilealliance.org/about-oma/work-program/m2m-enablers/
[CoAP]: https://tools.ietf.org/html/rfc7252
[Wakaama]: https://github.com/eclipse/wakaama
[OMA LWM2M Object and resource registry]: http://technical.openmobilealliance.org/Technical/technical-information/omna/lightweight-m2m-lwm2m-object-registry

[lnk-run-linux]: http://TODO
[lnk-Wakaama]: https://github.com/eclipse/wakaama
[lnk-github1]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c/iotdm_client/lwm2m_objects
[lnk-github2]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c/iotdm_client/lwm2m_objects
[lnk-oma]: http://technical.openmobilealliance.org/Technical/technical-information/omna/lightweight-m2m-lwm2m-object-registry


<!---HONumber=Mooncake_0523_2016-->