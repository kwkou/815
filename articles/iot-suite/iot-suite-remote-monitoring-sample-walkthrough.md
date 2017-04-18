<properties
 pageTitle="远程监控预配置解决方案演练 | Azure"
 description="介绍 Azure IoT 预配置解决方案远程监控及其体系结构。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>  

<tags
 ms.service="iot-suite"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="02/15/2017"
 ms.author="dobett"
 wacn.date="03/28/2017"/>  


# 远程监控预配置解决方案演练

## 介绍
IoT 套件远程监控[预配置解决方案][lnk-preconfigured-solutions]是适用于在远程位置运行的多个计算机的端到端监视解决方案实现。该解决方案结合了关键 Azure 服务来提供业务方案的通用实现。可以将其用作自己实现的起点，并可以根据特定的业务要求[自定义][lnk-customize]该解决方案。

本文将逐步讲解远程监控解决方案的一些关键要素，以帮助你了解其工作原理。该知识有助于：

- 排除解决方案中的问题。
- 规划如何定制解决方案来满足自身的特定需求。
- 自行设计使用 Azure 服务的 IoT 解决方案。

## 逻辑体系结构
下图概述该预配置解决方案的逻辑组件：

![逻辑体系结构](./media/iot-suite-remote-monitoring-sample-walkthrough/remote-monitoring-architecture.png)  


## 模拟设备
在该预配置解决方案中，模拟设备表示冷却设备（例如建筑物空调或设施空气处理单位）。部署预配置解决方案时，还会自动预配 4 个在 [Azure Web 作业][lnk-webjobs]中运行的模拟设备。模拟设备可让你轻松观测解决方案的行为，而不需要部署任何物理设备。若要部署实际的物理设备，请参阅 [Connect your device to the remote monitoring preconfigured solution][lnk-connect-rm]（将设备连接到远程监控预配置解决方案）教程。

### 设备到云的消息
每个模拟设备可将以下消息类型发送到 IoT 中心：

| 消息 | 说明 |
| --- | --- |
| 启动 |设备启动后，会向后端发送含有其自身信息的**设备信息**消息。此数据包括设备 ID，以及设备支持的命令和方法列表。 |
| 状态 |设备会定期发送**状态**消息，报告该设备能否感应到传感器的状态。 |
| 遥测 |设备会定期发送**遥测**消息，报告从设备的模拟传感器收集到的温度和湿度模拟值。 |

> [AZURE.NOTE]
解决方案将设备支持的命令列表存储在 DocumentDB 数据库中，而不是存储在设备孪生中。
> 
> 

### 属性和设备孪生
模拟设备将以下设备属性以*报告的属性*形式发送到 IoT 中心的[孪生][lnk-device-twins]中。设备在启动时以及在响应“更改设备状态”命令或方法时发送报告的属性。

| 属性 | 目的 |
| --- | --- |
| Config.TelemetryInterval | 设备发送遥测数据的频率（秒） |
| Config.TemperatureMeanValue | 指定模拟温度遥测数据的平均值 |
| Device.DeviceID |在解决方案中创建设备时所提供或分配的 ID |
| Device.DeviceState | 设备报告的状态 |
| Device.CreatedTime |在解决方案中创建设备的时间 |
| Device.StartupTime |设备的启动时间 |
| Device.LastDesiredPropertyChange |最后一项所需属性更改的版本号 |
| Device.Location.Latitude |设备的纬度位置 |
| Device.Location.Longitude |设备的经度位置 |
| System.Manufacturer |设备制造商 |
| System.ModelNumber |设备的型号 |
| System.SerialNumber |设备的序列号 |
| System.FirmwareVersion |设备上的当前固件版本 |
| System.Platform |设备的平台体系结构 |
| System.Processor |运行设备的处理器 |
| System.InstalledRAM |在设备上安装的 RAM 量 |

模拟器会以示例值在模拟设备中植入这些属性。模拟器每次初始化模拟设备时，设备将以报告的属性的形式向 IoT 中心报告预定义的元数据。报告的属性只能由设备更新。若要更改报告的属性，请在解决方案门户中设置所需的属性。设备负责：
1. 定期从 IoT 中心检索所需的属性。
2. 使用所需的属性值更新其配置。
3. 将新值以报告的属性的形式发回到中心。

在解决方案仪表板中，可以使用*所需的属性*通过[设备孪生][lnk-device-twins]在设备上设置属性。通常，设备从中心读取所需的属性值来更新其内部状态，并以报告的属性的形式来报告更改。

> [AZURE.NOTE]
模拟设备代码仅使用所需的属性 **Desired.Config.TemperatureMeanValue** 和 **Desired.Config.TelemetryInterval** 来更新发回给 IoT 中心的报告的属性。模拟设备中将忽略其他所有所需的属性更改。

### 方法
模拟设备可以处理通过 IoT 中心从解决方案门户调用的以下方法（[直接方法][lnk-direct-methods]）：

| 方法 | 说明 |
| --- | --- |
| InitiateFirmwareUpdate |指示设备执行固件更新 |
| 重新启动 |指示设备重新启动 |
| FactoryReset |指示设备执行出厂重置 |

某些方法使用报告的属性来报告进度。例如，**InitiateFirmwareUpdate** 方法模拟在设备上异步运行更新。该方法在设备上立即返回，异步任务继续使用报告的属性将状态更新发回到解决方案仪表板。

### 命令 
模拟设备可以处理通过 IoT 中心从解决方案门户发送的以下命令（云到设备的消息）：

| 命令 | 说明 |
| --- | --- |
| PingDevice |向设备发送 *ping* 以检查其是否处于活动状态 |
| StartTelemetry |使设备开始发送遥测 |
| StopTelemetry |使设备停止发送遥测 |
| ChangeSetPointTemp |更改设置点值（将围绕其生成随机数据） |
| DiagnosticTelemetry |触发设备模拟器以发送其他遥测值 (externalTemp) |
| ChangeDeviceState |更改设备的扩展状态属性，并从设备发送设备信息消息 |

> [AZURE.NOTE]
有关这些命令（设备到云的消息）和方法（直接方法）的比较，请参阅[云到设备的通信指南][lnk-c2d-guidance]。
> 
> 

## IoT 中心
[IoT 中心][lnk-iothub]将引入从设备发送到云的数据，并将其提供给 Azure 流分析 (ASA) 作业。每个流 ASA 作业使用不同的 IoT 中心使用者组从设备读取消息流。

此外，解决方案中的 IoT 中心还可以：

- 维护一个标识注册表，用于存储允许连接到门户的所有设备的 ID 和身份验证密钥。可以通过标识注册表启用和禁用设备。
- 代表解决方案门户向设备发送命令。
- 代表解决方案门户在设备上调用方法。
- 维护所有已注册设备的设备孪生。设备孪生存储设备报告的属性值。设备孪生还存储解决方案门户中设置的所需属性，供设备在后续连接时检索。
- 计划作业，为多个设备设置属性或者在多个设备上调用方法。

## Azure 流分析

在远程监控解决方案中，[Azure 流分析][lnk-asa] (ASA) 将 IoT 中心发出的设备消息分发到其他后端组件进行处理或存储。不同的 ASA 作业根据消息内容执行特定的功能。

**作业 1：设备信息**会筛选来自传入消息流的设备信息消息，并将它们发送到事件中心终结点。设备会在启动时发送设备信息消息，并且响应 **SendDeviceInfo** 命令。此作业使用以下查询定义来识别**设备信息**消息：


        SELECT * FROM DeviceDataStream Partition By PartitionId WHERE  ObjectType = 'DeviceInfo'


此作业将其输出发送到事件中心做进一步处理。

**作业 2：规则**会针对每个设备的阈值评估传入温度和湿度遥测值。阈值在解决方案门户上的规则编辑器中设置。每个设备/值对按照时间戳存储在 Blob 中，流分析将读入该对作为**参考数据**。该作业会针对设备的设置阈值比较任何非空值。如果超过“>”条件，该作业将输出**警报**事件，表示已超过阈值，并且提供设备、值和时间戳值。此作业使用以下查询定义来识别应触发警报的遥测消息：
	
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
    	)
    	
    	SELECT *
    	INTO DeviceRulesMonitoring
    	FROM AlarmsData
    	
    	SELECT *
    	INTO DeviceRulesHub
    	FROM AlarmsData

该作业将其输出发送到事件中心做进一步处理，并将每个警报的详细信息保存到 Blob 存储，解决方案门户可从该位置读取警报信息。

**作业 3：遥测**会通过两种方法来操作传入设备遥测流。第一种方法会将设备的所有遥测消息发送到永久性 Blob 存储以进行长期存储。第二种方法会通过五分钟滑动窗口计算平均、最小和最大湿度值，并将此数据发送到 Blob 存储。解决方案门户从 Blob 存储读取遥测数据来填充图表。此作业使用下列查询定义：

	
    	WITH 
    	    [StreamData]
    	AS (
    	    SELECT
    	        *
    	    FROM [IoTHubStream]
    	    WHERE
    	        [ObjectType] IS NULL -- Filter out device info and command responses
    	) 
    	
    	SELECT
    	    IoTHub.ConnectionDeviceId AS DeviceId,
    	    Temperature,
    	    Humidity,
    	    ExternalTemperature,
    	    EventProcessedUtcTime,
    	    PartitionId,
    	    EventEnqueuedUtcTime,
    	    *
    	INTO
    	    [Telemetry]
    	FROM
    	    [StreamData]
    	
    	SELECT
    	    IoTHub.ConnectionDeviceId AS DeviceId,
    	    AVG (Humidity) AS [AverageHumidity], 
    	    MIN(Humidity) AS [MinimumHumidity], 
    	    MAX(Humidity) AS [MaxHumidity], 
    	    5.0 AS TimeframeMinutes 
    	INTO
    	    [TelemetrySummary]
    	FROM [StreamData]
    	WHERE
    	    [Humidity] IS NOT NULL
    	GROUP BY
    	    IoTHub.ConnectionDeviceId,
    	    SlidingWindow (mi, 5)

## 事件中心
**设备信息**和**规则** ASA 作业将数据输出到事件中心，以便可靠地转发给 Web 作业中运行的**事件处理器**。

## Azure 存储
解决方案使用 Azure Blob 存储来保存解决方案设备中的所有原始数据和汇总的遥测数据。门户从 Blob 存储读取遥测数据来填充图表。为了显示警报，解决方案门户将从 Blob 存储读取当遥测值超过设置的阈值时所记录的数据。解决方案还使用 Blob 存储来记录在解决方案门户中设置的阈值。

## Web 作业
除了托管设备模拟器以外，解决方案中的 Web 作业还托管 Azure Web 作业中运行的、用于处理命令响应的**事件处理器**。它使用命令响应消息来更新设备命令历史记录（存储在 DocumentDB 数据库中）。

## DocumentDB
解决方案使用 DocumentDB 数据库来存储有关连接到该方案的设备的信息。此信息包括从解决方案门户发送到设备的命令以及从解决方案门户调用的方法的历史记录。

## 解决方案门户

解决方案门户是部署为预配置解决方案一部分的 Web 应用。解决方案门户中的关键页面包括仪表板和设备列表。

### 仪表板
Web 应用中的此页面使用 PowerBI javascript 控件（请参阅 [PowerBI-visuals repo（PowerBI 可视化效果存储库）](https://www.github.com/Microsoft/PowerBI-visuals)）来可视化设备发送的遥测数据。解决方案使用 ASA 遥测作业将遥测数据写入 Blob 存储。

### 列出设备
在解决方案门户的此页面中，可以：

* 预配新设备。此操作会设置唯一设备 ID 并生成身份验证密钥。它会将设备相关信息同时写入 IoT 中心标识注册表和解决方案特定的 DocumentDB 数据库。
* 管理设备属性。该操作包括查看现有属性和使用新数据进行更新。
* 将命令发送到设备。
* 查看设备的命令历史记录。
* 启用和禁用设备。

## 后续步骤

以下 TechNet 博客文章提供了有关远程监控预配置解决方案的更多详细信息：

- [IoT Suite - Under The Hood - Remote Monitoring（IoT 套件 - 幕后 - 远程监控）](http://social.technet.microsoft.com/wiki/contents/articles/32941.iot-suite-under-the-hood-remote-monitoring.aspx)
- [IoT Suite - Remote Monitoring - Adding Live and Simulated Devices（IoT 套件 - 远程监控 - 添加实时与模拟设备）](http://social.technet.microsoft.com/wiki/contents/articles/32975.iot-suite-remote-monitoring-adding-live-and-simulated-devices.aspx)

你可以通过阅读以下文章继续开始使用 IoT 套件：

- [将设备连接到远程监控预配置解决方案][lnk-connect-rm]
- [azureiotsuite.cn 站点权限][lnk-permissions]

[lnk-preconfigured-solutions]: /documentation/articles/iot-suite-what-are-preconfigured-solutions/
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/
[lnk-iothub]: /documentation/services/iot-hub/
[lnk-asa]: /documentation/services/stream-analytics/
[lnk-webjobs]: /documentation/articles/websites-webjobs-resources/
[lnk-connect-rm]: /documentation/articles/iot-suite-connecting-devices/
[lnk-permissions]: /documentation/articles/iot-suite-permissions/
[lnk-c2d-guidance]: /documentation/articles/iot-hub-devguide-c2d-guidance/
[lnk-device-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-direct-methods]: /documentation/articles/iot-hub-devguide-direct-methods/

<!---HONumber=Mooncake_0815_2016-->