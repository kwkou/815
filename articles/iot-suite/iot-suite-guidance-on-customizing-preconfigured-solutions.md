<properties
	pageTitle="自定义预配置解决方案 | Azure"
	description="提供有关如何自定义 Azure IoT 套件预配置解决方案的指导。"
	services=""
    suite="iot-suite"
	documentationCenter=".net"
	author="dominicbetts"
	manager="timlt"
	editor=""/>


<tags
     ms.service="iot-suite"
     ms.devlang="dotnet"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="02/15/2017"
     wacn.date="03/28/2017"
     ms.author="corywink"/>  



# 自定义预配置解决方案
Azure IoT 套件提供的预配置解决方案演示了套件中的服务如何协力提供端到端解决方案。从这个起点开始，有多个地方可以针对特定应用场景扩展和自定义解决方案。以下各节描述了这些常见的自定义点。

## 查找源代码
预配置解决方案的源代码可在以下 GitHub 存储库获得：

- 远程监控：[https://www.github.com/Azure/azure-iot-remote-monitoring](https://github.com/Azure/azure-iot-remote-monitoring)
- 预测性维护：[https://github.com/Azure/azure-iot-predictive-maintenance](https://github.com/Azure/azure-iot-predictive-maintenance)

提供预配置解决方案源代码的目的，在于演示实现使用 Azure IoT 套件的 IoT 解决方案的端到端功能时所采用的模式和做法。你可以找到有关如何在 GitHub 存储库中生成和部署解决方案的详细信息。

## 更改预配置规则

远程监控解决方案包含三个 [Azure 流分析](/home/features/stream-analytics)作业，这些作业可处理解决方案中的设备信息、遥测数据及规则逻辑。

[远程监控预配置解决方案演练](/documentation/articles/iot-suite-remote-monitoring-sample-walkthrough/)深入介绍了这三个流分析作业及其语法。

你可以直接编辑这些作业以更改逻辑，或添加特定于你的方案的逻辑。你可以按以下方式查找流分析作业：
 
1. 转到 [Azure 门户预览](https://portal.azure.cn)。
2. 导航到名称与 IoT 解决方案相同的资源组。
3. 选择要修改的 Azure 流分析作业。
4. 在命令集中选择“停止”以停止作业。
5. 编辑输入、查询及输出。

    简单修改的目的在于更改**规则**作业的查询，以便使用**“<”**而不是**“>”**。编辑规则时，解决方案门户仍会显示“>”，但请注意行为如何由于基础作业中的更改而翻转。

6. 启动作业

> [AZURE.NOTE] 远程监控仪表板依赖特定数据，因此更改作业可能会导致仪表板出现故障。

## 添加自己的规则

除了更改预配置的 Azure 流分析作业，也可以使用 Azure 门户预览添加新作业或添加对现有作业的新查询。

## 自定义设备
最常见的扩展活动之一是使用方案特定的设备。使用设备的方法有数种。这些方法包括更改模拟设备以符合你的方案，或使用 [IoT 设备 SDK][IoT Device SDK] 将物理设备连接到解决方案。

有关添加设备的分步指南，请参阅 [Iot 套件连接设备](/documentation/articles/iot-suite-connecting-devices/)一文和[远程监控 C SDK 示例](https://github.com/Azure/azure-iot-sdk-c/tree/master/serializer/samples/remote_monitoring)。本示例旨在配合远程监控预配置解决方案使用。

### 创建自己的模拟设备
[远程监控解决方案源代码](https://github.com/Azure/azure-iot-remote-monitoring)中包含 .NET 模拟器。此模拟器是解决方案中预配的模拟器，可以对其进行更改以发送不同的元数据、遥测数据和响应不同的命令和方法。

远程监控预配置解决方案模拟器中的预配置模拟器是发出温度和湿度遥测的冷却设备。派生 GitHub 存储库后，可以修改 [Simulator.WebJob](https://github.com/Azure/azure-iot-remote-monitoring/tree/master/Simulator/Simulator.WebJob) 项目中的模拟器。

### 模拟设备的可用位置
默认的位置集为中国北京。可以 [SampleDeviceFactory.cs][lnk-sample-device-factory] 中更改这些位置。

### 将所需的属性更新处理程序添加到模拟器
可在解决方案门户中设置设备所需属性的值。当设备检索所需的属性值时，由设备负责处理属性更改请求。若要通过所需的属性添加属性值更改支持，需要将一个处理程序添加到模拟器。

模拟器包含 **SetPointTemp** **和TelemetryInterval** 的处理程序，可以通过在解决方案门户中设置所需值来更新这些属性。

以下示例演示了 **CoolerDevice** 类中 **SetPointTemp** 所需属性的处理程序：


	protected async Task OnSetPointTempUpdate(object value)
	{
	    var telemetry = _telemetryController as ITelemetryWithSetPointTemperature;
	    telemetry.SetPointTemperature = Convert.ToDouble(value);

	    await SetReportedPropertyAsync(SetPointTempPropertyName, telemetry.SetPointTemperature);
	}


此方法更新遥测点温度，然后通过设置报告的属性向 IoT 中心报告更改。

可以遵循前一示例中的模式，为自己的属性添加自己的处理程序。

此外，必须按以下示例中所示，通过 **CoolerDevice** 构造函数将所需的属性绑定到处理程序：


	_desiredPropertyUpdateHandlers.Add(SetPointTempPropertyName, OnSetPointTempUpdate);


请注意，**SetPointTempPropertyName** 是定义为“Config.SetPointTemp”的常量。

### <a name="add-support-for-a-new-method-to-the-simulator"></a> 将新方法支持添加到模拟器
可以自定义模拟器，以添加对新[方法（直接方法）][lnk-direct-methods]的支持。需要执行两个重要步骤：

- 模拟器必须在预配置解决方案中向 IoT 中心告知方法的详细信息。
- 模拟器必须包含相应的代码，以便在通过解决方案资源管理器中的“设备详细信息”面板或者通过作业调用该方法时，能够处理方法调用。

远程监控预配置解决方案使用*报告的属性*向 IoT 中心发送受支持方法的详细信息。解决方案后端维护每个设备支持的所有方法的列表，以及方法调用的历史记录。可在解决方案门户中查看有关设备的这些信息以及调用方法。

为了告知 IoT 中心某个设备支持某个方法，设备必须将该方法的详细信息添加到报告的属性中的 **SupportedMethods** 节点：


	"SupportedMethods": {
	  "<method signature>": "<method description>",
	  "<method signature>": "<method description>"
	}


方法签名采用以下格式：`<method name>--<parameter #0 name>-<parameter #1 type>-...-<parameter #n name>-<parameter #n type>`。例如，若要指定 **InitiateFirmwareUpdate** 方法需要名为 **FwPackageURI** 的字符串参数，请使用以下方法签名：


	InitiateFirmwareUpate--FwPackageURI-string: "description of method"


有关受支持的参数类型的列表，请参阅 Infrastructure 项目中的 **CommandTypes** 类。

若要删除某个方法，请在报告的属性中将方法签名设置为 `null`。

> [AZURE.NOTE]
> 从设备接收*设备信息*消息时，解决方案后端只会更新有关受支持方法的信息。
> 
> 

以下代码示例摘自 Common 项目中的 **SampleDeviceFactory** 类，演示如何将方法添加到设备发送的报告属性中的 **SupportedMethods** 列表：


	device.Commands.Add(new Command(
	    "InitiateFirmwareUpdate",
	    DeliveryType.Method,
	    "Updates device Firmware. Use parameter 'FwPackageUri' to specifiy the URI of the firmware file, e.g. https://iotrmassets.blob.core.chinacloudapi.cn/firmwares/FW20.bin",
	    new[] { new Parameter("FwPackageUri", "string") }
	));


此代码片段将添加 **InitiateFirmwareUpdate** 方法的详细信息，包括要在解决方案门户中显示的文本，以及所需方法参数的详细信息。

模拟器在启动时，将向 IoT 中心发送报告的属性，包括支持的方法列表。

将模拟器支持的每个方法的处理程序添加到模拟器代码。可以在 Simulator.WebJob 项目的 **CoolerDevice** 类中查看现有的处理程序。以下示例演示了 **InitiateFirmwareUpdate** 方法的处理程序：


	public async Task<MethodResponse> OnInitiateFirmwareUpdate(MethodRequest methodRequest, object userContext)
	{
	    if (_deviceManagementTask != null && !_deviceManagementTask.IsCompleted)
	    {
	        return await Task.FromResult(BuildMethodRespose(new
	        {
	            Message = "Device is busy"
	        }, 409));
	    }

	    try
	    {
	        var operation = new FirmwareUpdate(methodRequest);
	        _deviceManagementTask = operation.Run(Transport).ContinueWith(async task =>
	        {
	            // after firmware completed, we reset telemetry
	            var telemetry = _telemetryController as ITelemetryWithTemperatureMeanValue;
	            if (telemetry != null)
	            {
	                telemetry.TemperatureMeanValue = 34.5;
	            }

	            await UpdateReportedTemperatureMeanValue();
	        });

	        return await Task.FromResult(BuildMethodRespose(new
	        {
	            Message = "FirmwareUpdate accepted",
	            Uri = operation.Uri
	        }));
	    }
	    catch (Exception ex)
	    {
	        return await Task.FromResult(BuildMethodRespose(new
	        {
	            Message = ex.Message
	        }, 400));
	    }
	}


方法处理程序的名称必须以 `On` 开头，后接方法的名称。**methodRequest** 参数包含通过解决方案后端使用方法调用传递的所有参数。返回值的类型必须是 **Task&lt;MethodResponse&gt;**。可以借助 **BuildMethodResponse** 实用工具方法创建返回值。

在方法处理程序中，可以：

- 启动异步任务。
- 从 IoT 中心的*设备孪生*中检索所需的属性。
- 使用 **CoolerDevice** 类中的 **SetReportedPropertyAsync** 方法更新单个报告属性。
- 通过创建 **TwinCollection** 实例并调用 **Transport.UpdateReportedPropertiesAsync** 方法更新多个报告属性。

上面的固件更新示例将执行以下步骤：

- 检查设备是否能够接受固件更新请求。
- 以异步方式启动固件更新操作，完成该操作时重置遥测。
- 立即返回“FirmwareUpdate 已接受”消息，指出设备已接受请求。

### 构建并使用自己的（物理）设备
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

7. 此过程会将一个 .json 文件下载到本地计算机。在所选的文本编辑器中打开此文件进行编辑。
8. 在 .json 文件的第三行中，可看到：
 
		"appRoles" : [],
 
      将此行替换为以下代码：

	
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

10.  在 Azure 经典管理门户中，选择页面底部的“管理清单”，然后选择“上载清单”上载在上一步中保存的 .json 文件。

11. 现在已将 **Admin** 和 **ReadOnly** 角色添加到应用程序中。

12. 若要将其中一个角色分配给你目录中的用户，请参阅 [azureiotsuite.cn 站点上的权限][lnk-permissions]。

## 反馈
本文档是否涵盖你感兴趣的自定义内容？ 请在[用户之声](https://feedback.azure.com/forums/321918-azure-iot)中添加功能建议，或对本文发表评论。

## 后续步骤

若要详细了解自定义预配置解决方案的选项，请参阅：

- [配合使用动态遥测和远程监控预配置解决方案][lnk-dynamic]
- [远程监控预配置解决方案中的设备信息元数据][lnk-devinfo]

[lnk-dynamic]: /documentation/articles/iot-suite-dynamic-telemetry/
[lnk-devinfo]: /documentation/articles/iot-suite-remote-monitoring-device-info/

[IoT Device SDK]: /documentation/articles/iot-hub-sdks-summary/
[lnk-permissions]: /documentation/articles/iot-suite-permissions/
[lnk-dashboard-controller]: https://github.com/Azure/azure-iot-remote-monitoring/blob/3fd43b8a9f7e0f2774d73f3569439063705cebe4/DeviceAdministration/Web/Controllers/DashboardController.cs#L27
[lnk-telemetry-api-controller-01]: https://github.com/Azure/azure-iot-remote-monitoring/blob/3fd43b8a9f7e0f2774d73f3569439063705cebe4/DeviceAdministration/Web/WebApiControllers/TelemetryApiController.cs#L27
[lnk-telemetry-api-controller-02]: https://github.com/Azure/azure-iot-remote-monitoring/blob/e7003339f73e21d3930f71ceba1e74fb5c0d9ea0/DeviceAdministration/Web/WebApiControllers/TelemetryApiController.cs#L25
[lnk-sample-device-factory]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Common/Factory/SampleDeviceFactory.cs#L40
[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-direct-methods]: /documentation/articles/iot-hub-devguide-direct-methods/

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description:update wording and link references-->