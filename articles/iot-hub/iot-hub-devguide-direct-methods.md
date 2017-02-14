<properties
    pageTitle="了解 Azure IoT 中心直接方法 | Azure"
    description="开发人员指南 - 使用直接方法从服务应用调用设备上的代码。"
    services="iot-hub"
    documentationcenter=".net"
    author="nberdy"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="9f0535f1-02e6-467a-9fc4-c0950702102d"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/11/2017"
    wacn.date="02/10/2017"
    ms.author="nberdy" />  


# 直接方法
## 概述
借助 IoT 中心，用户可以从云中对设备调用直接方法。直接方法表示与设备进行的请求-答复式交互，类似于会立即成功或失败（在用户指定的超时时间后）的 HTTP 调用。这适用于即时操作过程取决于设备能否响应的情况，例如，如果设备脱机，则向设备发送短信以唤醒设备（短信的开销比方法调用更大）。

每个设备方法针对一个设备。[作业][lnk-devguide-jobs]用于对多个设备调用直接方法，并为已断开连接的设备安排方法的调用。

只要拥有 IoT 中心的“服务连接”权限，任何人都可以调用设备上的方法。

### 使用时机
直接方法遵循请求-响应模式，适用于需要立即确认其结果的通信，通常是对设备的交互式控制，例如，要打开风扇。

如果在使用所需属性、直接方法或云到设备消息方面有任何疑问，请参阅[云到设备通信指南][lnk-c2d-guidance]。

## 方法生命周期
直接方法在设备上实现，可能需要在方法有效负载中进行 0 次或 0 次以上的输入才能正确地实例化。可以通过面向服务的 URI (`{iot hub}/twins/{device id}/methods/`) 调用直接方法。设备通过特定于设备的 MQTT 主题 (`$iothub/methods/POST/{method name}/`) 接收直接方法。将来可能会支持在更多的设备端网络协议上使用直接方法。

> [AZURE.NOTE]
调用设备上的直接方法时，属性名称和值只能包含 US ASCII 可打印字母数字，但下列组中的任一项除外：``{'$', '(', ')', '<', '>', '@', ',', ';', ':', '\', '"', '/', '[', ']', '?', '=', '{', '}', SP, HT}``。
> 
> 

直接方法是同步的，在超时期限（默认：30 秒，最长可设置为 3600 秒）过后，其结果不是成功就是失败。直接方法适用于交互式场景，即当且仅当设备处于联机状态且可接收命令时，用户希望设备做出响应，例如打开手机的灯。在此类方案中，用户需要立即看到结果是成功还是失败，以便云服务可以尽快根据结果进行操作。设备可能返回某些消息正文作为方法的结果，但系统不会要求方法一定这样做。无法保证基于方法调用的排序或者任何并发语义。

从云端来看，设备方法调用仅限 HTTP，从设备端来看，则仅限 MQTT。

## 参考主题：
以下参考主题详细介绍了如何使用直接方法。

## 从后端应用调用直接方法
### 方法调用
在设备上直接调用方法属于 HTTP 调用，其中包括：

* *URI*，特定于设备 \(`{iot hub}/twins/{device id}/methods/`\)
* POST *方法*
* *标头*，其中包含身份验证、请求 ID、内容类型和内容编码
* 透明的 JSON *正文*，采用以下格式：


    	{
    	    "methodName": "reboot",
    	    "responseTimeoutInSeconds": 200,
    	    "payload": {
    	        "input1": "someInput",
    	        "input2": "anotherInput"
    	    }
    	}


    超时以秒为单位。如果未设置超时，则默认为 30 秒。

### 响应
由后端应用接收响应，其中包括：

* *HTTP 状态代码*，用于 IoT 中心发出的错误，包括 404 错误（针对当前未连接的设备）
* *标头*，其中包含 ETag、请求 ID、内容类型和内容编码
* JSON *正文*，采用以下格式：


    	{
    	    "status" : 201,
    	    "payload" : {...}
    	}


    `status` 和 `body` 均由设备提供，用于响应，其中包含设备自身的状态代码和/或描述。

## 处理针对设备的直接方法
### 方法调用
设备通过 MQTT 主题接收直接方法请求：`$iothub/methods/POST/{method name}/?$rid={request id}`

设备接收的正文采用以下格式：


	{
	    "input1": "someInput",
	    "input2": "anotherInput"
	}


方法请求为 QoS 0。

### 响应
设备将响应发送到 `$iothub/methods/res/{status}/?$rid={request id}`，其中：

* `status` 属性是设备提供的方法执行状态。
* `$rid` 属性是从 IoT 中心接收的方法调用中的请求 ID。

正文由设备设置，可以是任何状态。

## 其他参考资料
IoT 中心开发人员指南中的其他参考主题包括：

* [IoT 中心终结点][lnk-endpoints]，介绍了每个 IoT 中心针对运行时和管理操作公开的各种终结点。
* [限制和配额][lnk-quotas]，说明了适用于 IoT 中心服务的配额，以及使用服务时预期会碰到的限制行为。
* [Azure IoT 设备和服务 SDK][lnk-sdks]，列出了在开发与 IoT 中心交互的设备和服务应用时可使用的各种语言 SDK。
* [设备孪生和作业的 IoT 中心查询语言][lnk-query]，介绍了在 IoT 中心检索设备孪生和作业相关信息时可使用的 IoT 中心查询语言。
* [IoT 中心 MQTT 支持][lnk-devguide-mqtt]提供有关 IoT 中心对 MQTT 协议的支持的详细信息。

## 后续步骤
了解如何使用直接方法后，可根据兴趣参阅以下 IoT 中心开发人员指南主题：

* [Schedule jobs on multiple devices（在多台设备上计划作业）][lnk-devguide-jobs]

若要尝试本文中介绍的一些概念，可以根据兴趣学习以下 IoT 中心教程：

* [使用直接方法][lnk-methods-tutorial]

<!-- links and images -->


[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-methods-tutorial]: /documentation/articles/iot-hub-node-node-direct-methods/
[lnk-devguide-messages]: /documentation/articles/iot-hub-devguide-messaging/
[lnk-c2d-guidance]: /documentation/articles/iot-hub-devguide-c2d-guidance/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update meta properties-->