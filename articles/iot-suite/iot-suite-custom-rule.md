<properties
    pageTitle="在 Azure IoT 套件中创建自定义规则 | Azure"
    description="如何在 IoT 套件预配置解决方案中创建自定义规则。"
    services=""
    suite="iot-suite"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="562799dc-06ea-4cdd-b822-80d1f70d2f09"
    ms.service="iot-suite"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/09/2017"
    wacn.date="03/31/2017"
    ms.author="dobett" />  


# 在远程监视预配置解决方案中创建自定义规则

## 介绍

在预配置解决方案中，可以配置[在设备的遥测值达到特定阈值时触发的规则][lnk-builtin-rule]。[配合使用动态遥测和远程监视预配置解决方案][lnk-dynamic-telemetry]说明了如何向解决方案添加自定义遥测值，如 ExternalTemperature。本文演示如何在解决方案中为动态遥测类型创建自定义规则。

本教程使用模拟 Node.js 的简单设备来生成动态遥测，并发送到预配置解决方案后端。然后在 **RemoteMonitoring** Visual Studio 解决方案中添加自定义规则，并将此自定义后端部署到 Azure 订阅。

若要完成本教程，你需要：

* 一个有效的 Azure 订阅。如果没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk_free_trial]。
* [Node.js][lnk-node] 0.12.x 版本或更高版本，用于创建模拟设备。
* Visual Studio 2015 或 Visual Studio 2017，用于使用新规则修改预配置解决方案后端。

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

记下为部署选择的解决方案名称。本教程稍后需要此解决方案名称。

[AZURE.INCLUDE [iot-suite-send-external-temperature](../../includes/iot-suite-send-external-temperature.md)]

如果确认 Node.js 控制台应用正将 **ExternalTemperature** 遥测发送到预配置解决方案，则可以停止该应用。请保持打开控制台窗口，因为将自定义规则添加到解决方案后需再次运行此 Node.js 控制台应用。

## 规则存储位置

有关规则的信息将保留在两个位置：

* **DeviceRulesNormalizedTable** 表 - 此表存储对由解决方案门户定义的规则的规范化引用。解决方案门户显示设备规则时，它将查询此表以查找规则定义。
* **DeviceRules** blob - 此 blob 存储为所有已注册设备定义的全部规则，并定义为 Azure 流分析作业的引用输入。
 
在解决方案门户更新现有规则或定义新规则时，表和 blob 都会更新以反映所做的更改。门户中显示的规则定义来自表存储，而流分析作业引用的规则定义来自 blob。

## 更新 RemoteMonitoring Visual Studio 解决方案

以下步骤演示如何修改 RemoteMonitoring Visual Studio 解决方案，使其包含使用 **ExternalTemperature** 遥测（由模拟设备发送）的规则：

1. 如果尚未执行此操作，请使用以下 Git 命令将 **azure-iot-remote-monitoring** 存储库克隆到本地计算机上的适当位置：

		git clone https://github.com/Azure/azure-iot-remote-monitoring.git

2. 在 Visual Studio 中，从 **azure-iot-remote-monitoring** 存储库的本地副本打开 RemoteMonitoring.sln 文件。

3. 打开文件 Infrastructure\\Models\\DeviceRuleBlobEntity.cs，并按如下所示添加 **ExternalTemperature** 属性：

		public double? Temperature { get; set; }
		public double? Humidity { get; set; }
		public double? ExternalTemperature { get; set; }

4. 在同一文件中，按如下所示添加 **ExternalTemperatureRuleOutput** 属性：

	    public string TemperatureRuleOutput { get; set; }
	    public string HumidityRuleOutput { get; set; }
	    public string ExternalTemperatureRuleOutput { get; set; }

5. 打开文件 Infrastructure\\Models\\DeviceRuleDataFields.cs，并在现有 **Humidity** 属性后添加以下 **ExternalTemperature** 属性：

	    public static string ExternalTemperature
	    {
	        get { return "ExternalTemperature"; }
	    }

6. 在同一文件中，按如下所示更新 **\_availableDataFields** 方法，使其包含 **ExternalTemperature**：

	    private static List<string> _availableDataFields = new List<string>
	    {                    
	        Temperature, Humidity, ExternalTemperature
	    };

7. 打开文件 Infrastructure\\Repository\\DeviceRulesRepository.cs，并按如下所示修改 **BuildBlobEntityListFromTableRows** 方法：

	    else if (rule.DataField == DeviceRuleDataFields.Humidity)
	    {
	        entity.Humidity = rule.Threshold;
	        entity.HumidityRuleOutput = rule.RuleOutput;
	    }
	    else if (rule.DataField == DeviceRuleDataFields.ExternalTemperature)
	    {
	      entity.ExternalTemperature = rule.Threshold;
	      entity.ExternalTemperatureRuleOutput = rule.RuleOutput;
	    }

## 重新生成并重新部署解决方案。

现可将更新后的解决方案部署到 Azure 订阅。

1. 打开提升的命令提示符，并导航到 azure-iot-remote-monitoring 存储库本地副本的根目录。

2. 若要部署更新后的解决方案，请运行以下命令，将 **{deployment name}** 替换为之前记录的预配置解决方案部署的名称：

	    build.cmd cloud release {deployment name}

## 更新流分析作业

部署完成后，可以更新流分析作业，以使用新的规则定义。

1. 在 Azure 门户中，导航到包含预配置解决方案资源的资源组。此资源组的名称与部署期间指定的解决方案名称相同。

2. 导航到 {deployment name}-Rules 流分析作业。

3. 单击“停止”停止运行流分析作业。（必须等待流式处理作业停止，之后才能编辑查询）。

4. 单击“查询”。编辑查询，使其包含 **ExternalTemperature** 的 **SELECT** 语句。以下示例使用新的 **SELECT** 语句演示完整查询：

	    WITH AlarmsData AS 
	    (
	    SELECT
	         Stream.IoTHub.ConnectionDeviceId AS DeviceId,
	         'Temperature' as ReadingType,
	         Stream.Temperature as Reading,
	         Ref.Temperature as Threshold,
	         Ref.TemperatureRuleOutput as RuleOutput,
	         Stream.EventEnqueuedUtcTime AS [Time]
	    FROM IoTTelemetryStream Stream
	    JOIN DeviceRulesBlob Ref ON Stream.IoTHub.ConnectionDeviceId = Ref.DeviceID
	    WHERE
	         Ref.Temperature IS NOT null AND Stream.Temperature > Ref.Temperature
	     
	    UNION ALL
	     
	    SELECT
	         Stream.IoTHub.ConnectionDeviceId AS DeviceId,
	         'Humidity' as ReadingType,
	         Stream.Humidity as Reading,
	         Ref.Humidity as Threshold,
	         Ref.HumidityRuleOutput as RuleOutput,
	         Stream.EventEnqueuedUtcTime AS [Time]
	    FROM IoTTelemetryStream Stream
	    JOIN DeviceRulesBlob Ref ON Stream.IoTHub.ConnectionDeviceId = Ref.DeviceID
	    WHERE
	         Ref.Humidity IS NOT null AND Stream.Humidity > Ref.Humidity
	     
	    UNION ALL
	     
	    SELECT
	         Stream.IoTHub.ConnectionDeviceId AS DeviceId,
	         'ExternalTemperature' as ReadingType,
	         Stream.ExternalTemperature as Reading,
	         Ref.ExternalTemperature as Threshold,
	         Ref.ExternalTemperatureRuleOutput as RuleOutput,
	         Stream.EventEnqueuedUtcTime AS [Time]
	    FROM IoTTelemetryStream Stream
	    JOIN DeviceRulesBlob Ref ON Stream.IoTHub.ConnectionDeviceId = Ref.DeviceID
	    WHERE
	         Ref.ExternalTemperature IS NOT null AND Stream.ExternalTemperature > Ref.ExternalTemperature
	    )
	     
	    SELECT *
	    INTO DeviceRulesMonitoring
	    FROM AlarmsData
	     
	    SELECT *
	    INTO DeviceRulesHub
	    FROM AlarmsData

5. 单击“保存”，更改已更新的规则查询。

6. 单击“启动”，再次开始运行流分析作业。

## 在仪表板中添加新规则

现可在解决方案仪表板中向设备添加 **ExternalTemperature** 规则。

1. 导航到解决方案门户。

2. 导航到“设备”面板。

3. 找到创建的用于发送 **ExternalTemperature** 遥测的自定义设备，然后在“设备详细信息”面板中，单击“添加规则”。

4. 在“数据字段”中选择 **ExternalTemperature**。

5. 将“阈值”设置为 56。然后单击“保存并查看规则”。

6. 返回仪表板查看警报历史记录。

7. 在打开的控制台窗口中，启动 Node.js 控制台应用，开始发送 **ExternalTemperature** 遥测数据。

8. 请注意，触发新规则时，“警报历史记录”表将显示新警报。
 
## 其他信息

更改运算符 **>** 更复杂，操作步骤远远多于本指南中概括的步骤。尽管可将流分析作业更改为使用所需的任何运算符，但在解决方案门户中反映该运算符则是一项更复杂的任务。

## 后续步骤
现在已经了解了如何创建自定义规则，还可以了解关于预配置解决方案的更多内容：

- [远程监视预配置解决方案中的设备信息元数据][lnk-devinfo]。

[lnk-devinfo]: /documentation/articles/iot-suite-remote-monitoring-device-info/

[lnk_free_trial]: /pricing/1rmb-trial/
[lnk-node]: http://nodejs.org
[lnk-builtin-rule]: /documentation/articles/iot-suite-getstarted-preconfigured-solutions/#add-a-rule-for-the-new-device
[lnk-dynamic-telemetry]: /documentation/articles/iot-suite-dynamic-telemetry/

<!---HONumber=Mooncake_0327_2017-->