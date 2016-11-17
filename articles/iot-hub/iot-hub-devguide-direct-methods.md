<properties
 pageTitle="开发人员指南 - 直接方法 | Azure"
 description="Azure IoT 中心开发人员指南 - 使用直接方法调用设备上的代码"
 services="iot-hub"
 documentationCenter=".net"
 authors="nberdy"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="multiple"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="09/30/2016"
 wacn.date="11/07/2016" 
 ms.author="nberdy"/>  


# 在设备上调用直接方法（预览版）

## 概述

用户可以通过 IoT 中心从云调用设备上的方法。这些方法表示与设备进行的请求-答复式交互，类似于 HTTP 调用，因为它们不管是成功还是失败，速度都非常快（在用户指定的超时过后），会让用户知道调用的状态。这适用于即时操作过程取决于设备能否响应的情况，例如，如果设备脱机，则向设备发送短信以唤醒设备（短信的开销比方法调用更大）。

可以将方法视为直接对设备进行的一种远程过程调用。只能从云调用已在设备上实现的方法。如果云尝试调用设备上的方法，而该设备并没有定义该方法，则方法调用会失败。

每个设备方法针对一个设备。[作业][lnk-devguide-jobs]用于调用多个设备上的方法，并对已断开连接的设备的方法调用排队。

只要拥有 IoT 中心的“服务连接”权限，任何人都可以调用设备上的方法。

### 使用时机

设备方法类似于[云到设备的消息][lnk-devguide-messages]，二者都允许云后端将信息传递到设备，但基本方式存在区别。从概念上来说，方法是同步性的，不具有持久性，而云到设备消息则是异步性的，其持久性长达 48 小时。

方法遵循请求-响应模式，不具持久性。向设备发出命令时，不具持久性具有两项立即会显示的优势：

- **执行方法时立即提供反馈**，也就是说，用户无需管理请求和答复之间的相关性。
- **更高的吞吐量**，这意味着操作执行速度更快，因为 IoT 中心不提供任何持久性。IoT 中心允许的每个单元的方法调用数多于云到设备的消息。

云到设备的消息不一定是对设备的命令，而是代表云服务将某些信息传递给设备，让设备自由选取，设备可以响应，也可以不响应。云到设备的消息的超时时间较长（最长可达 48 小时），而方法的过期时间则要快得多。

若要对设备上的命令进行计划的调用，可以使用设备方法在设备和作业上进行即时的命令调用。

## 方法生命周期

方法在设备上实现，可能需要在方法有效负载中进行 0 次或 0 次以上的输入才能正确地实例化。可以通过面向服务的 URI (`{iot hub}/twins/{device id}/methods/`) 调用直接方法。设备通过特定于设备的 MQTT 主题 (`$iothub/methods/POST/{method name}/`) 接收直接方法。将来可能会支持在更多的设备端网络协议上使用方法。

> [AZURE.NOTE] 调用设备上的直接方法时，属性名称和值只能包含 US ASCII 可打印字母数字，但下列组中的任一项除外：``{'$', '(', ')', '<', '>', '@', ',', ';', ':', '\', '"', '/', '[', ']', '?', '=', '{', '}', SP, HT}``。

方法是同步的，在超时期限（默认：30 秒，最长可设置为 3600 秒）过后，其结果不是成功就是失败。方法适用于交互式方案，即用户需要设备进行操作，但前提是设备处于联机状态且可接收命令（例如打开手机的灯）。在此类方案中，用户需要立即看到结果是成功还是失败，以便云服务可以尽快根据结果进行操作。设备可能返回某些消息正文作为方法的结果，但系统不会要求方法一定这样做。无法保证基于方法调用的排序或者任何并发语义。

从云端来看，设备方法调用仅限 HTTP，从设备端来看，则仅限 MQTT。

## 参考主题：

以下参考主题详细介绍了如何使用直接方法。

## 从后端应用调用直接方法

### 方法调用

在设备上直接调用方法属于 HTTP 调用，其中包括：

- *URI* ，特定于设备 (`{iot hub}/twins/{device id}/methods/`)
- POST *方法* 
- *标头* ，其中包含身份验证、请求 ID、内容类型和内容编码
- 透明的 JSON *正文* ，采用以下格式：

```
{
    "methodName": "reboot",
    "timeoutInSeconds": 200,
    "payload": {
        "input1": "someInput",
        "input2": "anotherInput"
    }
}
```

  超时以秒为单位。如果未设置超时，则默认为 30 秒。
  
### 响应

由后端接收响应，其中包括：

- *HTTP 状态代码* ，用于 IoT 中心发出的错误，包括 404 错误（针对当前未连接的设备）
- *标头* ，其中包含 ETag、请求 ID、内容类型和内容编码
- JSON *正文* ，采用以下格式：

```
{
    "status" : "OK",
    "payload" : {...}
}
```
  
   `status` 和 `body` 均由设备提供，用于响应，其中包含设备自身的状态代码和/或描述。

## 在设备上处理直接方法

### 方法调用

设备通过 MQTT 主题接收直接方法请求：`$iothub/methods/POST/{method name}/?$rid={request id}`

设备接收的正文采用以下格式：

```
{
    "input1": "someInput",
    "input2": "anotherInput"
}
```

方法请求为 QoS 0。

### 响应

设备将响应发送到 `$iothub/methods/res/{status}/?$rid={request id}`，其中：

 - `status` 属性是设备提供的方法执行状态。
 - `$rid` 属性是从 IoT 中心接收的方法调用中的请求 ID。

正文由设备设置，可以是任何状态。

## 其他参考资料

开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-endpoints]，说明了每个 IoT 中心针对运行时和管理操作公开的各种终结点。
- [限制和配额][lnk-quotas]，说明了适用于 IoT 中心服务的配额，以及使用服务时预期会碰到的限制行为。
- [IoT 中心设备和服务 SDK][lnk-sdks]，列出了在开发与 IoT 中心交互的设备和服务应用程序时可以使用的各种语言 SDK。
- [克隆、方法和作业的查询语言][lnk-query]，说明了从 IoT 中心检索有关设备克隆、方法和作业的信息时可以使用的查询语言。
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]，详细介绍了 IoT 中心对 MQTT 协议的支持。

## 后续步骤

了解如何使用直接方法以后，可以根据兴趣参阅以下开发人员指南主题：

- [Schedule jobs on multiple devices][lnk-devguide-jobs]（在多台设备上计划作业）

若要尝试本文中介绍的一些概念，可以根据兴趣学习以下 IoT 中心教程：

- [Use cloud-to-device methods][lnk-methods-tutorial]（使用云到设备的方法）

<!-- links and images -->


[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-methods-tutorial]: /documentation/articles/iot-hub-c2d-methods/
[lnk-devguide-messages]: /documentation/articles/iot-hub-devguide-messaging/

<!---HONumber=Mooncake_1031_2016-->