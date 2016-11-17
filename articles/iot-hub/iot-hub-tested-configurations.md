<properties
	pageTitle="OS 平台和硬件兼容性 | Azure"
	description="总结 IoT 设备 SDK 与 OS 平台和设备硬件的兼容性。"
	services="iot-hub"
	documentationCenter=""
	authors="hegate"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="na"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="10/04/2016"
     wacn.date="11/07/2016"
     ms.author="hegate"/>

# OS 平台和硬件与设备 SDK 的兼容性

本文档介绍了 SDK 与不同 OS 平台的兼容性。如果你不确定要使用的设备，请查看本文中的 [OS 平台和库](#os-platforms)兼容性部分。



## <a name="os-platforms"></a> OS 平台

Azure IoT 库在以下操作系统平台上进行了测试：


|Linux/Unix OS 平台 | 版本|
|:---------------|:------------:|
|Debian Linux| 7\.5|
|Fedora Linux|20|
|Raspbian Linux| 3\.18 |
|Ubuntu Linux| 14\.04 |
|Yocto Linux|2\.1 |

|Android OS 平台 | 版本|
|:---------------|:------------:|
|Android| > 4.2|

|Windows OS 平台 | 版本|
|:---------------|:------------:|
|Windows 桌面| 10 |
|Windows IoT Core| 10 |
|Windows Server| 2012 R2|

|其他平台 | 版本|
|:---------------|:------------:|
|Arduino | IDE 1.6.8 |
|mbed | 2\.0 |
|TI-RTOS | 2\.x |



## C 库

[适用于 C 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/c/readme.md) 在以下配置上进行了测试：

|OS 平台| 版本|协议|
|:---------|:----------:|:----------:|
|Arduino| IDE 1.6.8 | HTTPS |
|Debian Linux| 7\.5 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Fedora Linux| 20 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|mbed OS| 2\.0 | HTTPS、AMQP、MQTT |
|TI-RTOS| 2\.x | HTTPS |
|Ubuntu Linux| 14\.04 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Windows 桌面| 10 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Yocto Linux|2\.1 | HTTPS、AMQP|



## Node.js 库

[适用于 Node.js 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/node/device/readme.md) 在以下配置上进行了测试：


|运行时| 版本|协议|
|:---------|:----------:|:----:|
|Node.js| 4\.1.0 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |

适用于 Node.js 的 Azure IoT 服务 SDK 在以下配置上进行了测试：

|运行时| 版本|
|:---------|:----------:|
|Node.js| 4\.1.0 |

## Java 库

[适用于 Java 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/java/device/readme.md) 在以下配置上进行了测试：

|运行时| 版本|协议|
|:---------|:----------:|----|
|Java SE (Windows)| 1\.8 | HTTPS、AMQP、MQTT |
|Java SE (Linux)| 1\.8 | HTTPS、AMQP、MQTT|
|Android| API 15 | HTTPS、MQTT |

适用于 Java 的 Azure IoT 服务 SDK 在以下配置上进行了测试：

|运行时| 版本|协议|
|:---------|:----------:|:-----|
|Java SE| 1\.8 | HTTPS、AMQP、MQTT |


## CSharp

[适用于 .NET 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/csharp/device/readme.md) 在以下配置上进行了测试：

|OS 平台| 版本|协议|
|:---------|:----------:|:----------:|
|Windows 桌面| 10 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Windows IoT Core|10 | HTTPS |
|PCL（Xamarin、Mono、UWP、WP8.1、 Win8.1）| | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |

适用于 CSharp 的 Azure IoT 服务 SDK 在以下配置上进行了测试：

|运行时| 版本|
|:---------|:----------:|
|.Net| 4\.5 |
|UWP| |

托管代理代码需要 Microsoft .NET Framework 4.5


## Python 库

[适用于 Python 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/python/device/readme.md) 在以下配置上进行了测试：

|运行时| 版本|协议|
|:---------|:----------:|:----------:|
|Python | 2\.7.x、3.4.x、3.5.x | HTTPS、AMQP、MQTT |



## 后续步骤


若要深入了解如何规划 IoT 中心部署，请参阅：

- [支持其他协议][lnk-protocols]
- [与事件中心比较][lnk-compare]
- [缩放、HA 和 DR][lnk-scaling]

若要进一步探索 IoT 中心的功能，请参阅：

- [开发人员指南][lnk-devguide]
- [使用网关 SDK 模拟设备][lnk-gateway]


[lnk-iot-suite]: /documentation/services/iot-suite/

[lnk-protocols]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-compare]: /documentation/articles/iot-hub-compare-event-hubs/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-dmui]: /documentation/articles/iot-hub-device-management-ui-sample/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-portal]: /documentation/articles/iot-hub-manage-through-portal/

<!---HONumber=Mooncake_0725_2016-->