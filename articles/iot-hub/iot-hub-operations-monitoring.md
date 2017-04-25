<properties
    pageTitle="Azure IoT 中心操作监视 | Azure"
    description="如何使用 Azure IoT 中心操作监视功能实时监视 IoT 中心上的操作状态。"
    services="iot-hub"
    documentationcenter=""
    author="nberdy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="a299f3a5-b14d-4586-9c3b-44aea14ed013"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="12/13/2016"
    wacn.date="04/24/2017"
    ms.author="nberdy"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="ee59d4b57440af83a89ca77b401048f41a157c44"
    ms.lasthandoff="04/14/2017" />

# <a name="iot-hub-operations-monitoring"></a>IoT 中心操作监视
IoT 中心操作监视可让你实时监视其 IoT 中心上的操作状态。 IoT 中心可跨多个类别的操作跟踪事件。 可选择将一个或多个类别的事件发送到 IoT 中心终结点进行处理。 你可以监视数据中是否有错误，或根据数据模式设置更复杂的处理行为。

IoT 中心监视 6 种类别的事件：

- 设备标识操作
- 设备遥测
- 云到设备的消息
- 连接
- 文件上载
- 消息路由

## <a name="how-to-enable-operations-monitoring"></a>如何启用操作监视
1. 创建 IoT 中心。 有关如何创建 IoT 中心的说明，请参阅 [入门][lnk-get-started] 指南。
2. 打开 IoT 中心的边栏选项卡。 在此处单击“**操作监视**”。

    ![][1]
3. 选择要监视的监视类别，然后单击“**保存**”。 可以从“**监视设置**”中所列的与事件中心兼容的终结点读取事件。 IoT 中心终结点称为 `messages/operationsmonitoringevents`。

    ![][2]

> [AZURE.NOTE]
> 为“连接”类别选择“详细”监视会导致 IoT 中心生成额外的诊断消息。 对于所有其他类别，“详细”设置会更改 IoT 中心在每个错误消息中包含的信息数量。

## <a name="event-categories-and-how-to-use-them"></a>事件类别及其用法
每种操作监视类别跟踪与 IoT 中心之间进行的不同类型的交互，每一种监视类别都有一个架构用于定义如何构建该类别的事件。

### <a name="device-identity-operations"></a>设备标识操作
设备标识操作类别跟踪你尝试在其 IoT 中心的标识注册表中创建、更新或删除条目时所发生的错误。 预配方案就很适合跟踪此类别。

    {
        "time": "UTC timestamp",
         "operationName": "create",
         "category": "DeviceIdentityOperations",
         "level": "Error",
         "statusCode": 4XX,
         "statusDescription": "MessageDescription",
         "deviceId": "device-ID",
         "durationMs": 1234,
         "userAgent": "userAgent",
         "sharedAccessPolicy": "accessPolicy"
    }

### <a name="device-telemetry"></a>设备遥测
设备遥测类别跟踪在 IoT 中心发生且与遥测管道相关的错误。 此类别包括发送遥测事件（例如限制）和接收遥测事件（例如未经授权的读取者）时发生的错误。 请注意，此类别无法捕捉设备本身运行的代码所造成的错误。

    {
         "messageSizeInBytes": 1234,
         "batching": 0,
         "protocol": "Amqp",
         "authType": "{"scope":"device","type":"sas","issuer":"iothub"}",
         "time": "UTC timestamp",
         "operationName": "ingress",
         "category": "DeviceTelemetry",
         "level": "Error",
         "statusCode": 4XX,
         "statusType": 4XX001,
         "statusDescription": "MessageDescription",
         "deviceId": "device-ID",
         "EventProcessedUtcTime": "UTC timestamp",
         "PartitionId": 1,
         "EventEnqueuedUtcTime": "UTC timestamp"
    }

### <a name="cloud-to-device-commands"></a>云到设备的命令
云到设备的命令类别跟踪在 IoT 中心发生且与云到设备的消息管道相关的错误。 此类别包括下述情况下发生的错误：发送云到设备的消息（例如未经授权的发送者）、接收云到设备的消息（例如超过传递计数），以及接收云到设备的消息反馈（例如反馈已过期）。 此类别不捕捉未正确处理云到设备的消息但却将其成功传递的设备所发生的错误。

    {
         "messageSizeInBytes": 1234,
         "authType": "{"scope":"hub","type":"sas","issuer":"iothub"}",
         "deliveryAcknowledgement": 0,
         "protocol": "Amqp",
         "time": " UTC timestamp",
         "operationName": "ingress",
         "category": "C2DCommands",
         "level": "Error",
         "statusCode": 4XX,
         "statusType": 4XX001,
         "statusDescription": "MessageDescription",
         "deviceId": "device-ID",
         "EventProcessedUtcTime": "UTC timestamp",
         "PartitionId": 1,
         "EventEnqueuedUtcTime": “UTC timestamp"
    }

### <a name="connections"></a>连接
连接类别跟踪设备与 IoT 中心连接或断开连接时发生的错误。 若要识别未经授权的连接尝试，以及在连接质量不佳的区域中的设备断开连接时进行跟踪，就很适合跟踪此类别。

    {
         "durationMs": 1234,
         "authType": "{"scope":"hub","type":"sas","issuer":"iothub"}",
         "protocol": "Amqp",
         "time": " UTC timestamp",
         "operationName": "deviceConnect",
         "category": "Connections",
         "level": "Error",
         "statusCode": 4XX,
         "statusType": 4XX001,
         "statusDescription": "MessageDescription",
         "deviceId": "device-ID"
    }

### <a name="file-uploads"></a>文件上载

文件上载类别跟踪在 IoT 中心发生且与文件上载功能相关的错误。此类别包括：

- SAS URI 发生的错误，例如它在设备就上传完毕通知中心之前到期。
- 设备报告的失败上传。
- IoT 中心通知消息创建期间未在存储中找到文件时发生的错误。

请注意，此类别不能捕获在设备将文件上载到存储空间时直接发生的错误。

    {
         "authType": "{"scope":"hub","type":"sas","issuer":"iothub"}",
         "protocol": "HTTP",
         "time": " UTC timestamp",
         "operationName": "ingress",
         "category": "fileUpload",
         "level": "Error",
         "statusCode": 4XX,
         "statusType": 4XX001,
         "statusDescription": "MessageDescription",
         "deviceId": "device-ID",
         "blobUri": "http//bloburi.com",
         "durationMs": 1234
    }

### <a name="message-routing"></a>消息路由
消息路由类别跟踪消息路由评估期间发生的错误以及 IoT 中心感知到的终结点运行状况。 此类别包括以下事件：规则评估为“未定义”、IoT 中心将终结点标记为“已停用”，以及从终结点中收到的任何其他错误。 请注意，此类别不包含有关消息本身的特定错误（如设备限制错误），这些错误在“设备遥测”类别下报告。

    {
        "messageSizeInBytes": 1234,
        "time": "UTC timestamp",
        "operationName": "ingress",
        "category": "routes",
        "level": "Error",
        "deviceId": "device-ID",
        "messageId": "ID of message",
        "routeName": "myroute",
        "endpointName": "myendpoint",
        "details": "ExternalEndpointDisabled"
    }

## <a name="view-events"></a>查看事件

可以使用 *iothub-explorer* 工具快速测试 IoT 中心是否正在生成监视事件。 若要安装该工具，请参阅 [iothub-explorer][lnk-iothub-explorer] GitHub 存储库中的说明。

1. 请确保门户中的“连接”监视类别设置为“详细”。

1. 在命令提示符下，运行以下命令以读取监视终结点：

        iothub-explorer monitor-ops --login {your iothubowner connection string}

1. 在另一个命令提示符下，运行以下命令以模拟发送设备到云消息的设备：

        iothub-explorer simulate-device {your device name} --send "My test message" --login {your iothubowner connection string}

1. 当模拟设备连接到 IoT 中心时，第一个命令提示符显示监视事件。

## <a name="next-steps"></a>后续步骤
若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links and images -->

[1]: ./media/iot-hub-operations-monitoring/enable-OM-1.png
[2]: ./media/iot-hub-operations-monitoring/enable-OM-2.png

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-diagnostic-metrics]: /documentation/articles/iot-hub-metrics/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-dr]: /documentation/articles/iot-hub-ha-dr/


[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-iothub-explorer]: https://github.com/azure/iothub-explorer

<!--Update_Description:update wording and code-->