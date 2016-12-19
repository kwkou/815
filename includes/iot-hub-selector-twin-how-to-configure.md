> [AZURE.SELECTOR]
- [Node.js](/documentation/articles/iot-hub-node-node-twin-how-to-configure/)
- [C#](/documentation/articles/iot-hub-csharp-node-twin-how-to-configure/)

## 介绍
在 [IoT 中心设备克隆入门][lnk-twin-tutorial]中，你已学习如何使用 *标记* 通过解决方案后端设置设备元数据、如何使用 *报告属性* 通过设备应用报告设备条件以及如何使用类似 SQL 的语言查询此信息。

在本教程中，你会学习如何结合使用设备克隆的 *所需属性* 和 *报告属性* 来远程配置设备应用。具体而言，本教程演示设备克隆的报告属性和所需属性如何启用设备应用程序设置的多步配置，并跨所有设备向解决方案后端提供此操作状态的可见性。

从较高层面讲，使用设备克隆允许解决方案后端指定托管设备所需配置，而不发送特定命令。这个方法让设备负责确定更新其配置的最佳方法（对于特定设备条件影响即时执行特定命令的能力的 IoT 方案而言十分重要），同时继续向后端报告更新过程的当前状态和潜在错误条件。此模式能够帮助管理大量设备，因为它让后端能够跨所有设备完全掌握配置流程状态。

> [AZURE.NOTE] 在以更具交互性的方式控制设备的方案中（通过用户控制的应用打开风扇），请考虑使用[直接方法][lnk-methods]。

在本教程中，应用程序后端更改目标设备的遥测配置，这样一来，设备应用会执行多步流程来应用配置更新（例如，要求软件模块重启），这在本教程中以简单的延迟进行模拟。

后端采用以下方式将配置存储在设备克隆的所需属性中：

        {
            ...
            "properties": {
                ...
                "desired": {
                    "telemetryConfig": {
                        "configId": "{id of the configuration}",
                        "sendFrequency": "{config}"
                    }
                }
                ...
            }
            ...
        }

> [AZURE.NOTE] 由于配置可能会是复杂对象，通常会为它们分配唯一 ID（哈希值或 [GUID][lnk-guid]），以简化其比较。

设备应用以镜像报告属性中的所需属性 **telemetryConfig** 的形式报告其当前配置：

        {
            "properties": {
                ...
                "reported": {
                    "telemetryConfig": {
                        "changeId": "{id of the current configuration}",
                        "sendFrequency": "{current configuration}",
                        "status": "Success",
                    }
                }
                ...
            }
        }

请注意，报告的 **telemetryConfig** 拥有 **status** 这个其他属性，该属性用于报告配置更新流程的状态。

收到新的所需配置时，设备应用通过更改以下信息报告挂起的配置：

        {
            "properties": {
                ...
                "reported": {
                    "telemetryConfig": {
                        "changeId": "{id of the current configuration}",
                        "sendFrequency": "{current configuration}",
                        "status": "Pending",
                        "pendingConfig": {
                            "changeId": "{id of the pending configuration}",
                            "sendFrequency": "{pending configuration}"
                        }
                    }
                }
                ...
            }
        }

然后，在稍后的某个时间，设备应用通过更新上述属性报告此操作的成功或失败状态。请注意，后端可以在任何时候跨所有设备查询配置流程的状态。

本教程演示如何：

- 创建一个模拟设备，用于接收来自后端的配置更新，以及将多个更新作为配置更新流程的 *报告属性* 进行报告。
- 创建一个后端应用，用于更新设备的所需配置，然后查询配置更新流程。

<!-- links -->


[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-twin-tutorial]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-guid]: https://en.wikipedia.org/wiki/Globally_unique_identifier

<!---HONumber=Mooncake_1212_2016-->