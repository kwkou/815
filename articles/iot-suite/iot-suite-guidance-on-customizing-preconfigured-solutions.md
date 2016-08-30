<properties
	pageTitle="自定义预配置解决方案 | Azure"
	description="提供有关如何自定义 Azure IoT 套件预配置解决方案的指导。"
	services=""
    suite="iot-suite"
	documentationCenter=".net"
	authors="stevehob"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-suite"
     ms.date="06/27/2016"
     wacn.date="08/22/2016"/>  


# 自定义预配置解决方案

Azure IoT 套件提供的预配置解决方案演示了套件中的服务如何协力提供端到端解决方案。从这个起点开始，有好几个地方可以针对特定应用场景扩展和自定义解决方案。以下各节描述了这些常见的自定义点。

## 查找源代码

预配置解决方案的源代码可在以下 GitHub 存储库获得：

- 远程监视：[https://www.github.com/Azure/azure-iot-remote-monitoring](https://github.com/Azure/azure-iot-remote-monitoring)
- 预见性维护：[https://github.com/Azure/azure-iot-predictive-maintenance](https://github.com/Azure/azure-iot-predictive-maintenance)

提供预配置解决方案源代码的目的，在于演示实现使用 Azure IoT 套件的 IoT 解决方案的端到端功能时所采用的模式和做法。你可以找到有关如何在 GitHub 存储库中生成和部署解决方案的详细信息。

## 更改预配置规则

远程监视解决方案包含三个 [Azure 流分析](/home/features/stream-analytics)作业，这些作业可实现针对解决方案显示的设备信息、遥测数据及规则逻辑。

[远程监视预配置解决方案演练](/documentation/articles/iot-suite/iot-suite-remote-monitoring-sample-walkthrough/)深入介绍了这三个流分析作业及其语法。

你可以直接编辑这些作业以更改逻辑，或添加特定于你的方案的逻辑。你可以按以下方式查找流分析作业：
 
1. 转到 [Azure 门户预览](https://portal.azure.cn)。
2. 导航到名称与 IoT 解决方案相同的资源组。
3. 选择要修改的 Azure 流分析作业。
4. 在命令集中选择“停止”以停止作业。
5. 编辑输入、查询及输出。

    简单修改的目的在于更改**规则**作业的查询，以便使用**“<”**而不是**“>”**。编辑规则时，解决方案门户仍会显示**“>”**，不过因为基础作业中的更改，你会发现行为已翻转。

6. 启动作业

> [AZURE.NOTE] 远程监视仪表板依赖特定数据，因此更改作业可能会导致仪表板出现故障。

## 添加你自己的规则

除了更改预配置的 Azure 流分析作业，也可以使用 Azure 门户预览添加新作业或添加对现有作业的新查询。

## 自定义设备

最常见的扩展活动之一是使用方案特定的设备。使用设备的方法有数种。这些方法包括更改模拟设备以符合你的方案，或使用 [IoT 设备 SDK][] 将物理设备连接到解决方案。

有关向远程监视预配置解决方案添加设备的分步指南，请参阅 [Iot 套件连接设备](/documentation/articles/iot-suite/iot-suite-connecting-devices/)和[远程监视 C SDK 示例](https://github.com/Azure/azure-iot-sdks/tree/master/c/serializer/samples/remote_monitoring)（旨在搭配远程监视预配置解决方案）。

### 创建你自己的模拟设备

之前提及的远程监视解决方案源代码中包含 .NET 模拟器。此模拟器是解决方案中预配的模拟器，并且可以更改以发送不同的元数据、遥测数据或响应不同的命令。

远程监视预配置解决方案中的预配置模拟器是发出温度和湿度遥测的冷却设备，当你分叉 GitHub 存储库后，可以在 [Simulator.WebJob](https://github.com/Azure/azure-iot-remote-monitoring/tree/master/Simulator/Simulator.WebJob) 项目中修改模拟器。

### 模拟设备的可用位置

默认的位置集为美国华盛顿州西雅图/雷德蒙德。可以 [SampleDeviceFactory.cs][lnk-sample-device-factory] 中更改这些位置。


### 生成并使用你自己的（物理）设备

[Azure IoT SDK](https://github.com/Azure/azure-iot-sdks) 提供用于将各种设备类型（语言和操作系统）连接到 IoT 解决方案中的库。

## 修改仪表板限制

### 仪表板下拉列表中显示的设备数

默认值为 200。可以在 [DashboardController.cs][lnk-dashboard-controller] 中更改此数字。

### 必应地图控件中显示的图钉数

默认值为 200。可以在 [TelemetryApiController.cs][lnk-telemetry-api-controller-01] 中更改此数字。

### 遥测图形的时间段

默认值为 10 分钟。可以在 [TelmetryApiController.cs][lnk-telemetry-api-controller-02] 中更改此值。

## 手动设置应用程序角色

以下过程描述如何将 **Admin** 和 **ReadOnly** 应用程序角色添加到预配置解决方案中。请注意，从 azureiotsuite.cn 站点预配的预配置解决方案已经包含 **Admin** 和 **ReadOnly** 角色。

**ReadOnly** 角色的成员可以看到仪表板和设备列表，但不能添加设备、更改设备属性或发送命令。**Admin** 角色的成员具有对解决方案中所有功能的完全访问权限。

1. 转到 [Azure 经典管理门户][lnk-classic-portal]。

2. 选择“Active Directory”。

3. 单击在预配解决方案时所使用的 AAD 租户名称。

4. 单击“应用程序”。

5. 单击与预配置解决方案名称匹配的应用程序名称。如果在列表中看不到你的应用程序，请选择“显示”下拉列表中的“我公司拥有的应用程序”，然后单击复选标记。

6.  在页面底部，单击“管理清单”，然后单击“下载清单”。

7. 这会将一个 .json 文件下载到本地计算机。在所选的文本编辑器中打开此文件进行编辑。

8. 在 .json 文件的第三行，你会看到：
 
		"appRoles" : [],
 
      将此代码替换为以下代码：

	
		  "appRoles": [
		  {
		  "allowedMemberTypes": [
		  "User"
		  ],
		  "description": "Administrator access to the application",
		  "displayName": "Admin",
		  "id": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
		  "isEnabled": true,
		  "value": "Admin"
		  },
		  {
		  "allowedMemberTypes": [
		  "User"
		  ],
		  "description": "Read only access to device information",
		  "displayName": "Read Only",
		  "id": "e5bbd0f5-128e-4362-9dd1-8f253c6082d7",
		  "isEnabled": true,
		  "value": "ReadOnly"
		  } ],


9. 保存更新后的 .json 文件（可以覆盖现有文件）。

10.  在 Azure 经典管理门户中，选择页面底部的“管理清单”，然后选择“上载清单”上载你在上一步保存的 .json 文件。

11. 现在已将 **Admin** 和 **ReadOnly** 角色添加到应用程序中。

12. 若要将其中一个角色分配给你目录中的用户，请参阅 [azureiotsuite.cn 站点上的权限][lnk-permissions]。

## 反馈

本文档是否涵盖你感兴趣的自定义内容？ 请在[用户心声](https://feedback.azure.com/forums/321918-azure-iot)中添加功能建议，或在本文下方留言。

## 后续步骤

有关 IoT 设备的详细信息，请参阅 [Azure IoT 开发人员站点](/develop/iot)以查找链接和文档。

[IoT 设备 SDK]: /documentation/articles/iot-hub-sdks-summary/
[lnk-permissions]: /documentation/articles/iot-suite-permissions/
[lnk-dashboard-controller]: https://github.com/Azure/azure-iot-remote-monitoring/blob/3fd43b8a9f7e0f2774d73f3569439063705cebe4/DeviceAdministration/Web/Controllers/DashboardController.cs#L27
[lnk-telemetry-api-controller-01]: https://github.com/Azure/azure-iot-remote-monitoring/blob/3fd43b8a9f7e0f2774d73f3569439063705cebe4/DeviceAdministration/Web/WebApiControllers/TelemetryApiController.cs#L27
[lnk-telemetry-api-controller-02]: https://github.com/Azure/azure-iot-remote-monitoring/blob/e7003339f73e21d3930f71ceba1e74fb5c0d9ea0/DeviceAdministration/Web/WebApiControllers/TelemetryApiController.cs#L25
[lnk-sample-device-factory]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Common/Factory/SampleDeviceFactory.cs#L40
[lnk-classic-portal]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0815_2016-->