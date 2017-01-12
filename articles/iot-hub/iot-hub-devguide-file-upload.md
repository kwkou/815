<properties
 pageTitle="开发人员指南 - 文件上载 | Azure"
 description="Azure IoT 中心开发人员指南 - 将文件从设备上载到 IoT 中心"
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


# 从设备上载文件

## 概述

如 [IoT Hub endpoints][lnk-endpoints]（IoT 中心终结点）一文所述，设备可以通过面向设备的终结点 (**/devices/{deviceId}/files**) 发送通知以启动文件上载。当设备通知 IoT 中心已完成上载时，IoT 中心将生成文件上载通知，你可以通过面向服务的终结点 (**/messages/servicebound/filenotifications**) 将其作为消息接收。

IoT 中心本身不中转消息，而是充当关联 Azure 存储帐户的调度程序。设备请求来自 IoT 中心的存储令牌，该令牌特定于设备要上传的文件。设备使用 SAS URI 将文件上传到存储空间，上传完成后，设备将完成通知发送到 IoT 中心。IoT 中心验证该文件已上传，然后将文件上传通知添加到新的面向服务的文件通知消息传递终结点。

从设备将文件上载到 IoT 中心之前，必须对中心进行配置，为其[关联 Azure 存储][lnk-associate-storage]帐户。

然后，设备就能[初始化上载][lnk-initialize]，并在上载完成后[通知 IoT 中心][lnk-notify]。（可选）当设备通知 IoT 中心上载完成以后，服务就可以生成[通知消息][lnk-service-notification]。

### 使用时机
使用文件上载功能发送连接的设备间歇性地上载的媒体文件和大型遥测批文件（或者是经过压缩的文件以节省带宽）。

如果在使用报告属性、设备到云消息或文件上载方面有任何疑问，请参阅[设备到云通信指南][lnk-d2c-guidance]。

## <a name="associate-an-azure-storage-account-with-iot-hub"></a> 将 Azure 存储帐户与 IoT 中心相关联
若要使用文件上传功能，必须首先将 Azure 存储帐户链接到 IoT 中心。可以通过 [Azure 门户预览][lnk-management-portal]实现此操作，或通过 [IoT 中心资源提供程序 REST API][lnk-resource-provider-apis] 以编程方式实现此操作。将 Azure 存储帐户与 IoT 中心关联后，当设备启动文件上传请求时，此服务将向该设备返回 SAS URI。

> [AZURE.NOTE] [Azure IoT 中心 SDK][lnk-sdks] 自动处理检索 SAS URI、上载文件和通知 IoT 中心已完成上载。

## <a name="initialize-a-file-upload"></a> 初始化文件上传

IoT 中心有一个终结点，专供设备在上载文件时请求用于存储的 SAS URI。设备可以使用以下 JSON 正文向 IoT 中心的 `{iot hub}.azure-devices.cn/devices/{deviceId}/files` 发送 POST，从而启动文件上载过程：


        {
            "blobName": "{name of the file for which a SAS URI will be generated}"
        }


IoT 中心返回以下内容，供设备用来上载文件：


        {
            "correlationId": "somecorrelationid",
            "hostname": "contoso.azure-devices.cn",
            "containerName": "testcontainer",
            "blobName": "test-device1/image.jpg",
            "sasToken": "1234asdfSAStoken"
        }


### 已弃用：使用 GET 初始化文件上载

> [AZURE.NOTE] 本部分介绍已弃用的功能，这些功能用于从 IoT 中心接收 SAS URI。请使用上述 POST 方法。

IoT 中心有两个 REST 终结点支持文件上传，一个用于获取存储空间的 SAS URI，另一个用于通知 IoT 中心已完成上传。设备通过在 `{iot hub}.azure-devices.cn/devices/{deviceId}/files/{filename}` 向 IoT 中心发送 GET 来启动文件上载过程。该中心将返回特定于要上载的文件的 SAS URI，以及上载完成时要使用的相关性 ID。

## <a name="notify-iot-hub-of-a-completed-file-upload"></a> 通知 IoT 中心已完成文件上传

设备负责使用 Azure 存储 SDK 将文件上传到存储空间。上载完成后，设备会使用以下 JSON 正文向 IoT 中心的 `{iot hub}.azure-devices.cn/devices/{deviceId}/files/notifications` 发送 POST：


        {
            "correlationId": "{correlation ID received from the initial request}",
            "isSuccess": bool,
            "statusCode": XXX,
            "statusDescription": "Description of status"
        }


`isSuccess` 的值为布尔值，表示文件是否上载成功。`statusCode` 的状态代码表示将文件上载到存储时的状态，`statusDescription` 对应于 `statusCode`。

## 参考主题：

以下参考主题详细介绍了如何从设备上载文件。

## <a name="file-upload-notifications"></a> 文件上传通知

当设备上传文件并通知 IoT 中心上传完成时，该服务将选择性地生成包含该文件名称和存储位置的通知消息。

如[终结点][lnk-endpoints]中所述，IoT 中心通过面向服务的终结点 (**/messages/servicebound/feedback**) 以消息的形式传递文件上载通知。文件上载通知的接收语义与云到设备的消息的接收语义相同，并且具有相同的[消息生命周期][lnk-lifecycle]。从文件上传通知终结点检索到的每条消息都是具有以下属性的 JSON 记录：

| 属性 | 说明 |
| --- | --- |
| EnqueuedTimeUtc |指示通知创建时间的时间戳。 |
| DeviceId |上载文件的设备的 **DeviceId**。 |
| BlobUri |已上传文件的 URI。 |
| BlobName |已上传文件的名称。 |
| LastUpdatedTime |指示文件更新时间的时间戳。 |
| BlobSizeInBytes |已上传文件的大小。 |

**示例**。这是文件上传通知消息的示例正文。


        {
        	"deviceId":"mydevice",
        	"blobUri":"https://{storage account}.blob.core.chinacloudapi.cn/{container name}/mydevice/myfile.jpg",
        	"blobName":"mydevice/myfile.jpg",
        	"lastUpdatedTime":"2016-06-01T21:22:41+00:00",
        	"blobSizeInBytes":1234,
        	"enqueuedTimeUtc":"2016-06-01T21:22:43.7996883Z"
        }


## 文件上载通知配置选项
每个 IoT 中心都为文件上传通知公开以下配置选项：

| 属性 | 说明 | 范围和默认值 |
| --- | --- | --- |
| **enableFileUploadNotifications** |控制是否将文件上载通知写入文件通知终结点。 |布尔型。默认值：True。 |
| **fileNotifications.ttlAsIso8601** |文件上传通知的默认 TTL。 |ISO\_8601 间隔上限为 48 小时（下限为 1 分钟）。默认值：1 小时。 |
| **fileNotifications.lockDuration** |文件上传通知队列的锁定持续时间。 |5 到 300 秒（最小为 5 秒）。默认值：60 秒。 |
| **fileNotifications.maxDeliveryCount** |文件上传通知队列的最大传递计数。 |1 到 100。默认值：100。 |

## 其他参考资料

开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-endpoints]，说明了每个 IoT 中心针对运行时和管理操作公开的各种终结点。
- [限制和配额][lnk-quotas]，说明了适用于 IoT 中心服务的配额，以及使用服务时预期会碰到的限制行为。
- [IoT 中心设备和服务 SDK][lnk-sdks]，列出了在开发与 IoT 中心交互的设备和服务应用程序时可以使用的各种语言 SDK。
- [设备孪生、方法和作业的 IoT 中心查询语言][lnk-query]介绍从 IoT 中心检索有关设备孪生、方法和作业的信息时可以使用的查询语言。
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]提供有关 IoT 中心对 MQTT 协议的支持的详细信息。

## 后续步骤

了解如何使用 IoT 中心从设备上载文件以后，可以根据兴趣参阅以下开发人员指南主题：

- [管理 IoT 中心的设备标识][lnk-devguide-identities]
- [控制 IoT 中心的访问权限][lnk-devguide-security]
- [使用设备孪生同步状态和配置][lnk-devguide-device-twins]
- [在设备上调用直接方法][lnk-devguide-directmethods]
- [Schedule jobs on multiple devices（在多台设备上计划作业）][lnk-devguide-jobs]

若要尝试本文中介绍的一些概念，可以根据兴趣学习以下 IoT 中心教程：

- [如何使用 IoT 中心将文件从设备上载到云中][lnk-fileupload-tutorial]

[lnk-resource-provider-apis]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx
[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[lnk-management-portal]: https://portal.azure.cn
[lnk-fileupload-tutorial]: /documentation/articles/iot-hub-csharp-csharp-file-upload/
[lnk-associate-storage]: /documentation/articles/iot-hub-devguide-file-upload/#associate-an-azure-storage-account-with-iot-hub
[lnk-initialize]: /documentation/articles/iot-hub-devguide-file-upload/#initialize-a-file-upload
[lnk-notify]: /documentation/articles/iot-hub-devguide-file-upload/#notify-iot-hub-of-a-completed-file-upload
[lnk-service-notification]: /documentation/articles/iot-hub-devguide-file-upload/#file-upload-notifications
[lnk-lifecycle]: /documentation/articles/iot-hub-devguide-messaging/#message-lifecycle
[lnk-d2c-guidance]: /documentation/articles/iot-hub-devguide-d2c-guidance/

[lnk-devguide-identities]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-devguide-security]: /documentation/articles/iot-hub-devguide-security/
[lnk-devguide-device-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-devguide-directmethods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/

<!---HONumber=Mooncake_1205_2016-->