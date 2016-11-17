<properties
 pageTitle="开发人员指南 - 作业 |Azure"
 description="Azure IoT 中心开发人员指南 - 计划要在连接到中心的多个设备上运行的作业"
 services="iot-hub"
 documentationCenter=".net"
 authors="juanjperez"
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
 ms.author="juanpere"/>  


# 在多台设备上计划作业（预览版）

## 概述

如先前的文章所述，Azure IoT 中心可启用多个构建基块（[设备克隆属性和标记][lnk-twin-devguide]和[云到设备的方法][lnk-dev-methods]）。通常情况下，IoT 后端应用程序允许设备管理员和操作员在计划的时间批量更新 IoT 设备并与之交互。作业在计划的时间针对一组设备封装设备克隆更新和 C2D 方法的执行。例如，操作员可使用用于启动和跟踪作业的后端应用程序在不会中断大楼运作的时间重新启动 43 号大楼第 3 层中的一组设备。

### 何时使用

在以下情况下请考虑使用作业：解决方案后端需要计划并跟踪一组设备中的以下任何活动的进度：

- 更新设备克隆所需属性
- 更新设备克隆标记
- 调用 C2D 方法

## 作业生命周期

作业由解决方案后端启动，并由 IoT 中心维护。可以通过面向服务的 URI (`{iot hub}/jobs/v2/{device id}/methods/<jobID>?api-version=2016-09-30-preview`) 启动作业，并通过面向服务的 URI (`{iot hub}/jobs/v2/<jobId>?api-version=2016-09-30-preview`) 查询正在执行的作业的进度。启动作业后，查询作业将使后端应用程序能够刷新正在运行的作业的状态。

> [AZURE.NOTE] 启动作业时，属性名称和值只能包含 US ASCII 可打印字母数字，但下列组中的任一项除外：``{'$', '(', ')', '<', '>', '@', ',', ';', ':', '\', '"', '/', '[', ']', '?', '=', '{', '}', SP, HT}``。

## 参考主题：

以下参考主题提供有关使用作业的详细信息。

## 用于执行直接方法的作业

下面是使用作业在一组设备上执行[直接方法][lnk-dev-methods]的 HTTP 1.1 请求详细信息：

    ```
    PUT /jobs/v2/<jobId>?api-version=2016-09-30-preview
    
    Authorization: <config.sharedAccessSignature>
    Content-Type: application/json; charset=utf-8
    Request-Id: <guid>
    User-Agent: <sdk-name>/<sdk-version>

    {
        jobId: '<jobId>',
        type: 'scheduleDirectRequest', 
        cloudToDeviceMethod: {
            methodName: '<methodName>',
            payload: <payload>,                 
            timeoutInSeconds: methodTimeoutInSeconds 
        },
        queryCondition: '<queryOrDevices>', // if the queryOrDevices parameter is a string
        deviceIds: '<queryOrDevices>',      // if the queryOrDevices parameter is an array
        startTime: <jobStartTime>,          // as an ISO-8601 date string
        maxExecutionTimeInSeconds: <maxExecutionTimeInSeconds>        
    }
    ```
    
## 用于更新设备克隆属性的作业

下面是使用作业更新设备克隆属性的 HTTP 1.1 请求详细信息：

    ```
    PUT /jobs/v2/<jobId>?api-version=2016-09-30-preview
    Authorization: <config.sharedAccessSignature>
    Content-Type: application/json; charset=utf-8
    Request-Id: <guid>
    User-Agent: <sdk-name>/<sdk-version>

    {
        jobId: '<jobId>',
        type: 'scheduleTwinUpdate', 
        updateTwin: <patch>                 // Valid JSON object
        queryCondition: '<queryOrDevices>', // if the queryOrDevices parameter is a string
        deviceIds: '<queryOrDevices>',      // if the queryOrDevices parameter is an array
        startTime: <jobStartTime>,          // as an ISO-8601 date string
        maxExecutionTimeInSeconds: <maxExecutionTimeInSeconds>        // format TBD
    }
    ```

## 查询作业的进度

下面是用于[查询作业][lnk-query]的 HTTP 1.1 请求详细信息：

    ```
    GET /jobs/v2/query?api-version=2016-09-30-preview[&jobType=<jobType>][&jobStatus=<jobStatus>][&pageSize=<pageSize>][&continuationToken=<continuationToken>]
    
    Authorization: <config.sharedAccessSignature>
    Content-Type: application/json; charset=utf-8
    Request-Id: <guid>
    User-Agent: <sdk-name>/<sdk-version>
    ```
    
从响应提供 continuationToken。

## 作业属性

下面是属性和相应说明的列表，在查询作业或作业结果时可使用这些属性。

| 属性 | 说明 |
| -------------- | -----------------|
| **jobId** | 应用程序提供的作业 ID。 |
| **startTime** | 应用程序提供的作业开始时间(ISO-8601)。 |
| **endTime** | IoT 中心提供的作业完成时的日期(ISO-8601)。只有在作业达到“完成”状态后才有效。 | 
| **type** | 作业的类型： |
| | **scheduledUpdateTwin**：用于更新一组克隆所需属性或标记的作业。 |
| | **scheduledDeviceMethod**：用于对一组克隆调用设备方法的作业。 |
| **status** | 作业的当前状态。可能的状态值： |
| | **挂起**：已计划并等待作业服务选取。 |
| | **已计划**：计划在将来某个时间。 |
| | **正在运行**：当前活动的作业。 |
| | **已取消**：已取消作业。 |
| | **失败**：作业失败。 |
| | **完成**：作业已完成。 |
| **deviceJobStatistics** | 有关作业执行的统计信息。 |

在预览期间，deviceJobStatistics 对象仅在作业完成后才可用。

| 属性 | 说明 |
| -------------- | -----------------|
| **deviceJobStatistics.deviceCount** | 作业中的设备数。 |
| **deviceJobStatistics.failedCount** | 作业失败的设备数。 |
| **deviceJobStatistics.succeededCount** | 作业成功的设备数。 |
| **deviceJobStatistics.runningCount** | 当前正在运行作业的设备数。 |
| **deviceJobStatistics.pendingCount** | 等待运行作业的设备数。 |


### 其他参考资料

开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-endpoints]说明每个 IoT 中心针对运行时和管理操作公开的各种终结点。
- [限制和配额][lnk-quotas]说明适用于 IoT 中心服务的配额，以及使用服务时可预期的限制行为。
- [IoT 中心设备和服务 SDK][lnk-sdks] 列出在开发与 IoT 中心交互的设备和服务应用程序时使用的各种语言 SDK。
- [克隆、方法和作业的查询语言][lnk-query]描述可用于从 IoT 中心检索有关设备克隆、方法和作业的信息的查询语言。
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]提供有关 IoT 中心对 MQTT 协议的支持的详细信息。

## 后续步骤

如果要尝试本文中介绍的一些概念，你可能对以下 IoT 中心教程感兴趣：

- [计划和广播作业][lnk-jobs-tutorial]

<!-- links and images -->


[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[lnk-jobs-tutorial]: /documentation/articles/iot-hub-schedule-jobs/
[lnk-c2d-methods]: /documentation/articles/iot-hub-c2d-methods/
[lnk-dev-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-get-started-twin]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-twin-devguide]: /documentation/articles/iot-hub-devguide-device-twins/

<!---HONumber=Mooncake_1031_2016-->