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
     ms.date="06/20/2016"
     wacn.date="07/04/2016"/>

# OS 平台和硬件与设备 SDK 的兼容性

本文档介绍了 SDK 与不同 OS 平台的兼容性。如果你不确定要使用的设备，请查看本文中的 [OS 平台和库](#os-platforms)兼容性部分。



## OS 平台

Azure IoT 库在以下操作系统平台上进行了测试：


|Linux/Unix OS 平台 | 版本|
|:---------------|:------------:|
|Debian Linux| 7\.5|
|Fedora Linux|20|
|Raspbian Linux| 3\.18 |
|Ubuntu Linux| 14\.04 |
|Yocto Linux|2\.1 |

|Windows OS 平台 | 版本|
|:---------------|:------------:|
|Windows 桌面| 10 |
|Windows IoT Core| 10 |
|Windows Server| 2012 R2|

|其他平台 | 版本|
|:---------------|:------------:|
|mbed | 2\.0 |
|TI-RTOS | 2\.x |



## C 库

[适用于 C 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/c/readme.md) 在以下配置上进行了测试：

|OS 平台| 版本|协议|
|:---------|:----------:|:----------:|
|Debian Linux| 7\.5 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Fedora Linux| 20 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|mbed OS| 2\.0 | HTTPS、AMQP |
|TI-RTOS| 2\.x | HTTPS |
|Ubuntu Linux| 14\.04 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Windows 桌面| 10 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |
|Yocto Linux|2\.1 | HTTPS、AMQP|



## Node.js 库

[适用于 Node.js 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/node/device/readme.md) 在以下配置上进行了测试：


|运行时| 版本|协议|
|:---------|:----------:|:----:|
|Node.js| 4\.1.0 | HTTPS、AMQP、MQTT、基于 WebSockets 的 AMQP |



## Java 库

[适用于 Java 的 Azure IoT 设备 SDK](https://github.com/Azure/azure-iot-sdks/blob/master/java/device/readme.md) 在以下配置上进行了测试：

|运行时| 版本|协议|
|:---------|:----------:|----|
|Java SE (Windows)| 1\.8 | HTTPS、AMQP、MQTT |
|Java SE (Linux)| 1\.8 | HTTPS、AMQP、MQTT|

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

托管代理代码需要 Microsoft .NET Framework 4.5




## 后续步骤

- 了解有关使用 [IoT 认证设备](/develop/iot/)制定解决方案的详细信息。


[lnk-iot-suite]: /documentation/suites/iot-suite/
[lnk-certified]: /documentation/articles/iot-hub-certified-devices-linux-c/
<!---HONumber=Mooncake_0418_2016-->