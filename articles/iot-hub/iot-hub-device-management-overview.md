<properties
 pageTitle="IoT 中心设备管理概述 | Azure"
 description="本文概述了 Azure IoT 中心的设备管理：企业设备生命周期、重新启动、恢复出厂设置、固件更新、配置、设备克隆、查询、作业"
 services="iot-hub"
 documentationCenter=""
 authors="bzurcher"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="10/03/2016"
 ms.author="bzurcher"
 wacn.date="12/16/2016"/>  


# IoT 中心设备管理概述

## 介绍
Azure IoT 中心提供功能和可扩展性模型，使设备和后端开发人员可以构建功能强大的设备管理解决方案。IoT 设备的范围从受约束的传感器和单一用途微控制器到功能强大的路由设备组通信的网关。此外，在不同行业中，IoT 操作员的用例和要求也显著不同。尽管有此不同，但 Azure IoT 中心设备管理提供了功能、模式和代码库，以满足不同设备和最终用户的需要。

创建成功的企业 IoT 解决方案的一个重要部分，是提供操作员如何处理其设备集合的日常管理的策略。IoT 操作员需要简单且可靠的工具和应用程序，使他们能够重点处理其工作的更具战略意义方面。本文将提供：

- Azure IoT 中心设备管理方法的简要概述。
- 常见设备管理原则的说明。
- 设备生命周期的说明。
- 常见设备管理模式的概述。

## 设备管理原则
IoT 带来了一系列独特的设备管理难题，每个企业级解决方案必须满足以下原则：

![设备管理原则图形][img-dm_principles]  


- **规模和自动化**：IoT 解决方案需要可以自动执行日常任务的简单工具，使相对数量少的操作人员可以管理数百万台设备。每天，操作员希望远程批量处理设备操作，并且仅在出现需要直接干预的问题时才收到通知。

- **开放性和兼容性**：IoT 设备生态系统是截然不同的。管理工具必须进行定制以适应多种设备类、平台和协议。操作员必须能够支持许多类型的设备，从最受限制的嵌入式单进程芯片到功能强大且全功能的计算机。

- **上下文感知**：IoT 环境是动态的、不断变化的。服务可靠性极为重要。设备管理操作必须考虑到 SLA 维护时段、网络和电源状态、正在使用情况和设备地理位置，以确保维护停机时间不会影响关键的业务操作或创造危险情况。

- **为许多角色提供服务**：支持 IoT 操作角色的独特工作流和进程至关重要。操作人员必须与给定约束的内部 IT 部门协调工作。他们还必须找到可持续方法将实时设备操作信息传递给主管和其他业务管理角色。

## 设备生命周期
有一组所有企业 IoT 项目通用的常规设备管理阶段。在 Azure IoT 中，IoT 设备生命周期有五个阶段：

![Azure IoT 设备生命周期的五个阶段：计划、预配、配置、监视、停用][img-device_lifecycle]  


在上述五个阶段的每个阶段中，都有几项应满足以提供完整解决方案的设备操作员要求：

- **计划**：使操作员能够创建可让他们轻松且精确地查询的设备元数据方案，并针对一组设备进行批量管理操作。可以使用设备克隆以标记和属性的形式存储此设备元数据。

    *其他阅读材料*：[设备克隆入门][lnk-twins-getstarted]、[了解设备克隆][lnk-twins-devguide]、[如何使用克隆属性][lnk-twin-properties]

- **预配**：安全地为 IoT 中心预配新设备，使操作员立即发现设备功能。使用 IoT 中心设备注册表创建灵活的设备标识和凭据，并使用作业成批执行此操作。通过设备克隆中的设备属性构建设备来报告其功能和情况。

    *其他阅读材料*：[管理设备标识][lnk-identity-registry]、[批量管理设备标识][lnk-bulk-identity]、[如何使用克隆属性][lnk-twin-properties]

- **配置**：在保持正常运行和安全性的同时便于对设备进行批量配置更改和固件更新。使用所需属性或通过直接方法和广播作业成批执行这些设备管理操作。

    *其他阅读材料*：使用直接方法、[对设备调用直接方法][lnk-methods-devguide]、[如何使用克隆属性][lnk-twin-properties]、[在多台设备上计划作业][lnk-jobs-devguide]

- **监视**：监视总体设备集合运行状况、正在进行的操作的状态并针对可能需要操作员注意的问题向操作员发出警报。应用设备克隆以允许设备报告实时操作情况和更新操作的状态。使用设备克隆查询生成显示最直接问题的功能强大的仪表板报告。

    *其他阅读材料*：[如何使用设备克隆属性][lnk-twin-properties]、[设备克隆和作业的 IoT 中心查询语言][lnk-query-language]

- **停用**：在发生故障、升级周期后，或在服务生存期结束时更换或停用设备。使用设备克隆来维护设备信息（如果正在更换物理设备），或者将设备信息存档（如果正在停用设备）。使用 IoT 中心设备注册表来安全地撤销设备标识和凭据。

    *其他阅读材料*：[如何使用设备克隆属性][lnk-twin-properties]、[管理设备标识][lnk-identity-registry]

## 设备管理模式
IoT 中心启用以下设备管理模式集。

- **重新启动** - 后端应用程序通过直接方法通知设备它已启动重新启动。设备使用设备克隆报告属性来更新设备的重新启动状态。

    ![设备管理重启模式图形][img-reboot_pattern]  


- **恢复出厂设置** - 后端应用程序通过直接方法通知设备它已启动恢复出厂设置。设备使用设备克隆报告属性来更新设备的恢复出厂设置状态。

    ![设备管理恢复出厂设置模式图形][img-facreset_pattern]  


- **配置** - 后端应用程序使用设备克隆所需属性来配置设备上运行的软件。设备使用设备克隆报告属性来更新设备的配置状态。

    ![设备管理配置模式图形][img-config_pattern]  


- **固件更新** - 后端应用程序通过直接方法通知设备它已启动固件更新。设备将启动一个多步骤过程，用于下载固件包、应用固件包，最后重新连接到 IoT 中心服务。在整个多步骤过程中，设备使用设备克隆报告属性来更新设备的进度和状态。

    ![设备管理固件更新模式图形][img-fwupdate_pattern]  


- **报告进度和状态** - 应用程序后端在一组设备上运行设备克隆查询，以报告设备上运行的操作的状态和进度。

    ![设备管理报告进度和状态模式图形][img-report_progress_pattern]  


## 后续步骤
可以使用 IoT 中心为设备管理提供的功能、模式和代码库，在每个设备生命周期阶段创建满足企业 IoT 操作员需求的 IoT 应用程序。


<!-- Images and links -->

[img-dm_principles]: ./media/iot-hub-device-management-overview/image4.png
[img-device_lifecycle]: ./media/iot-hub-device-management-overview/image5.png
[img-config_pattern]: ./media/iot-hub-device-management-overview/configuration-pattern.png
[img-facreset_pattern]: ./media/iot-hub-device-management-overview/facreset-pattern.png
[img-fwupdate_pattern]: ./media/iot-hub-device-management-overview/fwupdate-pattern.png
[img-reboot_pattern]: ./media/iot-hub-device-management-overview/reboot-pattern.png
[img-report_progress_pattern]: ./media/iot-hub-device-management-overview/report-progress-pattern.png

[lnk-twins-devguide]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-get-started]: /documentation/articles/iot-hub-node-node-device-management-get-started/
[lnk-twins-getstarted]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-twin-properties]: /documentation/articles/iot-hub-node-node-twin-how-to-configure/
[lnk-hub-getstarted]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-identity-registry]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-bulk-identity]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-query-language]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-c2d-methods]: /documentation/articles/iot-hub-node-node-direct-methods/
[lnk-methods-devguide]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-jobs-devguide]: /documentation/articles/iot-hub-devguide-jobs/

<!---HONumber=Mooncake_1205_2016-->