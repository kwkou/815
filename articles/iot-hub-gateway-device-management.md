<properties
 pageTitle="在 IoT 网关后启用托管设备 | Azure"
 description="有关如何使用 IoT 网关 SDK 以及 IoT 中心托管设备的指导主题。"
 services="iot-hub"
 documentationCenter=""
 authors="chipalost"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="05/30/2016"/>
 
# 在 IoT 网关后启用托管设备

## 基本设备隔离

组织通常使用 IoT 网关来增强其 IoT 解决方案的总体安全性。一些设备需要将数据发送到云中，但不能保护自己免受 Internet 上的威胁。你可以保护这些设备免受外部线程干扰，方法是让设备通过网关与外界通信。

网关位于安全环境与开放 Internet 的边界。设备与网关对话，网关再将消息传递到正确的云目标。网关可抵御外部线程、阻止未授权的请求、允许授权的入站流量，并将入站流量转发到正确的设备。

![][1]

还可在网关后设置可保护自身的设备，以增加安全保障。在此方案中，只需保证网关 OS 修补最新的漏洞即可，而不需更新每台设备上的 OS。

## 隔离加智能

加强的路由器充当网关，足以隔离设备。但是，除隔离设备外，IoT 解决方案通常还要求网关更智能。例如，你可能希望从云管理设备。你可以使用 LWM2M（标准设备管理协议）来实现解决方案的云管理部分。但是，设备将使用不支持 TCP/IP 的协议发送遥测数据。此外，设备还会生成大量数据，而你只希望上传经过筛选的遥测数据子集。你可以生成融合 IoT 网关处理两个不同数据流的功能的解决方案。网关应：

-   理解**遥测数据**，筛选数据，然后通过网关将数据上传到云中。网关不再是仅在设备与云之间转发数据的简单路由器。

-   而只是在设备与云之间交换 **LWM2M 设备管理数据**。网关不需要理解传入的数据，只需确保在设备与云之间来回传递数据。

下图演示了此方案：

![][2]

## 解决方案：Azure IoT 设备管理和网关 SDK 

公共预览版 [Azure IoT 设备管理][lnk-device-management]和 Beta 版 [Azure IoT 网关 SDK] 支持此方案。网关处理以下每个数据流：

-   **遥测数据**：可以使用网关 SDK 来生成理解、筛选以及将遥测数据发送到云中的管道。网关 SDK 提供代表开发人员实现此管道各个部分的代码。有关 SDK 体系结构的详细信息，请参阅 [IoT 网关 SDK - 入门][lnk-gateway-get-started]教程。

-   **设备管理**：Azure 设备管理提供在设备上运行的 LWM2M 客户端以及用于向设备发出命令的云接口。
    
    不需要在网关上实现任何特殊逻辑，因为它不需要处理云与 IoT 中心之间交换的 LWM2M 数据。可以在网关上启用 Internet 连接字符串（许多现代操作系统具备的一项功能），以实现 LWM2M 数据的交换。可针对此方案选择合适的操作系统，因为网关 SDK 支持各种操作系统。以下是在 [Windows 10] 和 [Ubuntu]（支持的两种操作系统）上实现 Internet 连接共享的说明。

下图显示了用于使用 [Azure IoT 设备管理][lnk-device-management]和 [Azure IoT 网关 SDK] 实现此方案的高级体系结构。

![][3]

## 后续步骤

若要了解如何使用网关 SDK，请参阅以下教程：

- [IoT 网关 SDK - 使用 Linux 入门][lnk-gateway-get-started]
- [IoT 网关 SDK – 使用 Linux 通过模拟设备发送设备至云消息][lnk-gateway-simulated]

有关使用 IoT 中心进行设备管理的详细信息，请参阅 [Azure IoT 中心设备管理概述][lnk-device-management]。

<!-- Images and links -->
[1]: ./media/iot-hub-gateway-device-management/overview.png
[2]: ./media/iot-hub-gateway-device-management/manage.png
[Azure IoT 网关 SDK]: https://github.com/Azure/azure-iot-gateway-sdk/
[Windows 10]: http://windows.microsoft.com/zh-cn/windows/using-internet-connection-sharing#1TC=windows-7
[Ubuntu]: https://help.ubuntu.com/community/Internet/ConnectionSharing
[3]: ./media/iot-hub-gateway-device-management/manage_2.png
[lnk-gateway-get-started]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
[lnk-gateway-simulated]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-device-management]: /documentation/articles/iot-hub-device-management-overview/



<!---HONumber=Mooncake_0523_2016-->