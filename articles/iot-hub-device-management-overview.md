<properties
 pageTitle="设备管理概述 | Azure"
 description="Azure IoT 中心设备管理概述：设备克隆、设备查询、设备作业"
 services="iot-hub"
 documentationCenter=""
 authors="juanjperez"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="07/04/2016"/>

# IoT 中心设备管理概述（预览版）

Azure IoT 中心设备管理可实现标准的 IoT 设备管理，以便远程管理、配置以及更新设备。

Azure IoT 中的设备管理主要包括三个概念：

1.  **设备克隆**：物理设备在 IoT 中心的表示形式。

2.  **设备查询**：可查找设备克隆并生成多个理解设备克隆的聚合。例如，可运行查询来查找固件版本为 1.0 的所有设备克隆。

3.  **设备作业**：对一个或多个物理设备执行的操作，如固件更新、重启以及恢复出厂设置。

## 设备克隆

设备克隆是物理设备在 Azure IoT 中的表示形式。**Microsoft.Azure.Devices.Device** 对象用于表示设备克隆。

![][img-twin]

设备克隆具有以下组件：

1.  **设备字段**：设备字段是预定义的属性，用于 IoT 中心消息传递和设备管理。设备字段可帮助 IoT 中心标识和连接物理设备。设备字段不会同步到设备，只存储在设备克隆中。设备字段包括设备 ID 和身份验证信息。

2.  **设备属性**：设备属性是预定义的属性字典，用于描述物理设备。物理设备是每个设备属性的主机，是每个对应值的权威存储位置。这些属性的最终一致表示存储在云中的设备克隆中。相干性和新鲜度受同步设置影响，如[教程：如何使用设备克隆][lnk-tutorial-twin]中所述。设备属性的一些示例包括固件版本、电池剩余电量和制造商名称。

3.  **服务属性**：服务属性是开发人员添加到服务属性字典中的 **&lt;key,value&gt;** 对。这些属性扩展了设备克隆的数据模型，可让你更好地确定设备的特征。服务属性不会同步到设备，只存储在云中的设备克隆中。服务属性的其中一个示例是 **&lt;NextServiceDate, 11/12/2017&gt;**，该服务属性可用于按服务的下个日期查找设备。

4.  **标记**：标记是服务属性的子集，是任意字符串而非字典属性。标记可用于批注设备克隆或对设备分组。标记不会同步到设备，只存储在设备克隆中。例如，如果设备克隆表示物理卡车，则可为卡车中的每种货物添加标记，如**苹果**、**橘子**和**香蕉**。

## 设备查询

在上一部分，你已经学习了设备克隆的不同组件。接下来，将介绍如何基于设备属性、服务属性或标记在 IoT 中心设备注册表中查找设备克隆。其中一个示例是使用查询来查找需要更新的设备。可查询具有特定固定版本的所有设备，并将结果馈送到特定的操作中（在 IoT 中心称为设备作业，将在下一部分解释）。

可使用标记和属性进行查询：

-   若要使用标记查询设备克隆，请传递字符串数组，查询将返回一组标记了所传递全部字符串的设备。

-   若要使用服务属性或设备属性查询设备，请使用 JSON 查询表达式。下面的示例演示如何使用设备属性（键 **FirmwareVersion**，值 **1.0**）查询所有设备。可以看到属性的 **type** 为 **device**，表示基于设备属性而非服务属性进行查询：

  ```
  {                           
      "filter": {                  
        "property": {                
          "name": "FirmwareVersion",   
          "type": "device"             
        },                           
        "value": "1.0",              
        "comparisonOperator": "eq",  
        "type": "comparison"         
      },                           
      "project": null,             
      "aggregate": null,           
      "sort": null                 
  }
  ```

## 设备作业

设备管理的下一个概念是设备作业，设备作业可协调多台设备上的多步业务流程。

Azure IoT 中心设备管理当前提供 6 种类型的设备作业（如果客户需要，将增加额外的作业）：

- **固件更新**：更新物理设备上的固件（或 OS 映像）。
- **重启**：重启物理设备。
- **恢复出厂设置**：将物理设备上的固件（或 OS 映像）还原为设备上存储的出厂提供的备份映像。
- **配置更新**：配置在物理设备上运行的 IoT 中心客户端代理。
- **读取设备属性**：获取物理设备上的最新设备属性值。
- **写入设备属性**：更改物理设备上的设备属性。

有关如何使用上述每个作业的详细信息，请参阅[面向 C# 和 node.js 的 API 文档][lnk-apidocs]。

一个作业可以在多台设备上运行。启动一个作业时，将为所有这些设备创建相关的子作业。一个子作业在一台设备上运行。每个子作业都有一个指针指向其父作业。父作业只是子作业的容器，它不实现任何逻辑来区分设备类型（如，更新 Intel Edison 与更新 Raspberry Pi）。下图说明了父作业、子作业以及相关物理设备的关系。

![][img-jobs]

可查询作业历史记录以了解已启动的作业的状态。有关示例查询，请参阅[我们的查询库][lnk-query-samples]。

## 设备实现

现在，我们已经介绍了服务端概念，接下来我们将讨论如何创建托管物理设备。Azure IoT 中心 DM 客户端库可让你通过 Azure IoT 中心管理 IoT 设备。“管理”包括重启、恢复出厂设置以及更新固件等操作。我们当前提供独立于平台的 C 库，但很快就会增加对其他语言的支持。

DM 客户端库在设备管理方面主要负责以下两项任务：

- 同步物理设备上的属性与 IoT 中心中其对应的设备克隆
- 编排 IoT 中心发送到设备的设备作业

若要了解上述职责与物理设备上的实现的详细信息，请参阅[面向 C 的 Azure IoT 中心设备管理客户端库介绍][lnk-library-c]。

## 后续步骤

若要继续了解 Azure IoT 中心设备管理功能，请参阅 [Azure IoT 中心设备管理入门][lnk-get-started]教程。

<!-- Images and links -->
[img-twin]: ./media/iot-hub-device-management-overview/image1.png
[img-jobs]: ./media/iot-hub-device-management-overview/image2.png
[img-client]: ./media/iot-hub-device-management-overview/image3.png

[lnk-lwm2m]: http://technical.openmobilealliance.org/Technical/technical-information/release-program/current-releases/oma-lightweightm2m-v1-0
[lnk-library-c]: /documentation/articles/iot-hub-device-management-library/
[lnk-get-started]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-tutorial-twin]: /documentation/articles/iot-hub-device-management-device-twin/
[lnk-apidocs]: http://azure.github.io/azure-iot-sdks/
[lnk-query-samples]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/doc/get_started/dm_queries/query-samples.md

<!---HONumber=Mooncake_0523_2016-->