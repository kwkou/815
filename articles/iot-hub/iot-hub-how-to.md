<properties
    pageTitle="Azure IoT 中心操作方法 | Azure"
    description="作为开发人员，我应该如何使用各种 IoT 中心功能？"
    services="iot-hub"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="24376318-5344-4a81-a1e6-0003ed587d53"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/31/2017"
    wacn.date="04/24/2017"
    ms.author="dobett"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="e78b624d76f1d7215c6efb40164cd640f3bdddd0"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-use-azure-iot-hub"></a>如何使用 Azure IoT 中心

可以通过多种方式了解如何针对 IoT 中心服务进行开发。 可以阅读各种概念性的文章，详细了解 IoT 中心的功能，或者通过其中一个教程了解 IoT 中心的各种功能。

## <a name="the-developer-guide"></a>开发人员指南

开发人员可以在[开发人员指南][lnk-devguide]中阅读有关 IoT 中心的详细概念性指南。 本指南详细介绍了所有 IoT 中心功能，帮助用户了解如何使用这些功能，以及当多个选项可用时如何在这些功能之间进行选择

## <a name="the-tutorials"></a>教程

若要通过实际练习了解特定的 IoT 中心功能，则有多个教程可以选择。 这其中的许多教程以多种编程语言的方式提供。 这些教程包括：

- [使用路由处理 IoT 中心设备到云的消息][lnk-routes-tutorial]。 本教程介绍如何以基于配置的轻松方式，使用 IoT 中心路由规则发送设备到云的消息。

- [使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]。 本教程介绍如何通过 IoT 中心发送云到设备的消息以及如何在设备上接收云到设备的消息。

- [使用 IoT 中心将文件从设备上载到云][lnk-upload-tutorial]。 本教程介绍使用 IoT 中心的文件上载功能。

- [设备孪生入门][lnk-twin-tutorial]。 本教程介绍设备孪生、报告的属性、所需属性和标记。 可以使用设备孪生通过设备同步值。

- [使用直接方法][lnk-methods-tutorial]。 本教程介绍如何使用直接方法。 可以在模拟设备中添加直接方法的处理程序，然后从 IoT 中心调用直接方法。

- [设备管理入门][lnk-dm-tutorial]。 本教程介绍如何使用关键的设备管理功能（例如孪生和直接方法），以远程方式重新启动模拟设备。

- [使用所需属性配置设备][lnk-properties-tutorial]。 本教程介绍如何结合使用设备孪生的所需属性和报告的属性来远程配置设备。

- [使用设备作业启动设备固件更新][lnk-jobs-tutorial]。 本教程介绍如何使用关键的设备管理功能（例如孪生和直接方法），以远程方式更新设备的固件。

- [计划和广播作业][lnk-schedule-tutorial]。 本教程介绍如何使用所需属性和直接方法在计划的时间与多个设备交互。

## <a name="next-steps"></a>后续步骤

若要详细了解 IoT 中心服务，请参阅[开发人员指南][lnk-devguide]。

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-routes-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[lnk-upload-tutorial]: /documentation/articles/iot-hub-csharp-csharp-file-upload/
[lnk-twin-tutorial]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-methods-tutorial]: /documentation/articles/iot-hub-node-node-direct-methods/
[lnk-dm-tutorial]: /documentation/articles/iot-hub-node-node-device-management-get-started/
[lnk-properties-tutorial]: /documentation/articles/iot-hub-node-node-twin-how-to-configure/
[lnk-jobs-tutorial]: /documentation/articles/iot-hub-node-node-firmware-update/
[lnk-schedule-tutorial]: /documentation/articles/iot-hub-node-node-schedule-jobs/