<properties
	pageTitle="开始使用 Azure 流分析处理来自 IoT 设备的数据 | Azure"
	description="带流分析和实时数据处理的 IoT 传感器标记和数据流"
    keywords="iot 解决方案, iot 入门"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="jhubbard"
	editor="cgronlun"
/>  


<tags
	ms.service="stream-analytics"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.tgt_pltfrm="na"
	ms.workload="data-services"
	ms.date="09/26/2016"
	wacn.date="11/14/2016"
/>

# 开始使用 Azure 流分析处理来自 IoT 设备的数据

在本教程中，会学习如何创建流处理逻辑，以从物联网 (IoT) 设备收集数据。我们会使用真实的物联网 (IoT) 用例来演示如何经济实惠地快速生成解决方案。

## 先决条件

-   [Azure 订阅](/pricing/1rmb-trial/)
-   可从 [GitHub](https://github.com/Azure/azure-stream-analytics/tree/master/Samples/GettingStarted) 下载示例查询和数据文件

## 方案

Contoso 是一家工业自动化公司，该公司已将其制造流程完全自动化。这家工厂中的设备配有可实时发送数据流的传感器。在此方案中，产品车间经理希望通过传感器数据获得实时见解，从而找到规律并采取措施。我们将对传感器数据使用流分析查询语言 (SAQL)，查找传入数据流的有趣规律。

下图中，Texas Instruments SensorTag 设备正在生成数据。

![Texas Instruments SensorTag](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-01.jpg)  


数据的有效负载是 JSON 格式，如下所示：


	{
    	"time": "2016-01-26T20:47:53.0000000",  
	    "dspl": "sensorE",  
    	"temp": 123,  
	    "hmdt": 34  
	}  

在实际情况下，其中可能有数百个传感器以流的形式生成事件。理想情况下，网关设备会运行代码，将这些事件推送到 [Azure 事件中心](/home/features/event-hubs/)或 [Azure IoT 中心](/home/features/iot-hub/)。流分析作业将从事件中心引入这些事件，并针对流运行实时分析查询。然后，可以将结果发送到[支持的输出](/documentation/articles/stream-analytics-define-outputs/)之一。

为了方便使用，本入门指南提供从实际 SensorTag 设备中捕获的示例数据文件。可以对示例数据运行查询并查看结果。在后续教程中，你将学习如何将你的作业连接到输入和输出并将其部署到 Azure 服务。

## 创建流分析作业

1. 在 [Azure 经典管理门户](http://manage.windowsazure.cn)中，单击“流分析”，然后单击页面左下角的“新建”来创建新的分析作业。

	![创建新的流分析作业](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-02.png)  


2. 单击“快速创建”。

3. 对于“区域监视存储帐户”设置，单击“创建新存储帐户”，并为它指定唯一名称。Azure 流分析将使用此帐户来存储将来所有作业的监视信息。

	> [AZURE.NOTE] 只应为每个区域创建此存储帐户一次。跨该区域中创建的所有流分析作业共享此存储。

4. 单击页面底部的“创建流分析作业”。

	![存储帐户配置](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-03.jpg)  


## Azure 流分析查询

单击“查询”选项卡，转到查询编辑器。“查询”选项卡包含针对传入事件数据进行转换的 T-SQL 查询。

## 存档原始数据

查询的最简单形式是传递查询，该查询会将所有输入数据存档到其指定的输出。

![存档作业查询](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-04.png)  


现在，将示例数据文件从 [GitHub](https://aka.ms/azure-stream-analytics-get-started-iot) 下载到计算机中的某个位置。从 PassThrough.txt 文件粘贴查询。单击“测试”按钮，然后从下载位置选择 HelloWorldASA-InputStream.json 数据文件。

![流分析中的测试按钮](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-05.png)  


![测试输入流](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-06.png)  


可在浏览器中查看查询结果，如以下屏幕截图所示。

![测试结果](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-07.png)  


## 根据条件筛选数据

让我们尝试基于条件筛选结果。我们希望仅显示来自“sensorA”的事件的结果。 查询在 Filtering.txt 文件中。

![筛选数据流](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-08.png)  


请注意，查询区分大小写，并且会比较字符串的值。单击“重新运行”按钮执行查询。查询应返回 1,860 个事件中的 389 行。

![执行查询测试后的第二个输出结果](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-09.png)  


## 触发业务工作流的警报

让我们创建更详细的查询。对于每个类型的传感器，我们想要每 30 秒监视一次平均温度，且仅在平均温度高于 100 度的情况下显示结果。我们会编写以下查询，然后单击“重新运行”查看结果。查询在 ThresholdAlerting.txt 文件中。

![30 秒筛选查询](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-10.png)  


现在，会看到结果只包含 245 行，以及平均温度高于 100 度的传感器的名称。此查询按 **dspl**（传感器名称）以 30 秒的**轮转窗口**将事件流分组。临时查询必须声明我们所希望的时间进展方式。通过使用 **TIMESTAMP BY** 子句，我们已指定 **OUTPUTTIME** 列用于将时间与所有临时计算关联。有关详细信息，请阅读有关 [Time Management](https://msdn.microsoft.com/zh-cn/library/azure/mt582045.aspx)（时间管理）和 [Windowing functions](https://msdn.microsoft.com/zh-cn/library/azure/dn835019.aspx)（窗口化函数）的 MSDN 文章。

![温度高于 100](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-11.png)  


## 检测事件缺失

如何编写查询来确定是否缺少输入事件？ 让我们找出传感器最后一次发送数据且下一分钟未发送事件的情况。查询在 AbsenseOfEvent.txt 文件中。

![检测事件缺失](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-12.png)  


此时，使用 **LEFT OUTER** 联接到相同的数据流（自联接）。对于 **INNER** 联接，仅当找到匹配项时才返回结果。对于 **LEFT OUTER** 联接，如果联接左侧的事件不匹配，则返回右侧所有列的带 NULL 的行。这种方法对于查找事件缺失很有用。有关详细信息，请参阅 MSDN 文档 [JOIN](https://msdn.microsoft.com/zh-cn/library/azure/dn835026.aspx)（联接）。

![联接结果](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-13.png)  


## 结束语

本教程旨在演示如何编写不同的流分析查询语言查询，并在浏览器中查看结果。但是，这仅仅是一个开始。使用流分析还可以完成许多其他操作。流分析支持多种输入和输出，因而是用于分析数据流的可靠工具。有关如何编写查询的详细信息，请阅读有关[常用查询模式](/documentation/articles/stream-analytics-stream-analytics-query-patterns/)的文章。

<!---HONumber=Mooncake_1107_2016-->