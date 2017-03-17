<properties
    pageTitle="流分析简介 | Azure"
    description="了解流分析，这是一种托管服务，可以帮助你分析物联网 (IoT) 实时提供的流式数据。"
    keywords="分析即服务、托管服务、流处理、流式分析、什么是流分析"
    services="stream-analytics"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="613c9b01-d103-46e0-b0ca-0839fee94ca8"
    ms.service="stream-analytics"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="01/24/2017"
    wacn.date="03/10/2017"
    ms.author="jeffstok" />  



# 什么是流分析？

Azure 流分析是完全托管的、经济高效的实时事件处理引擎，帮助开启来自数据的深入见解。通过流分析，可以针对设备、传感器、网站、社交媒体、应用程序、基础设施系统等的数据流，轻松地设置实时分析计算。

只需在 Azure 门户中单击几下，你便可创建一个流分析作业，用于指定流式数据的输入源，你的作业结果的输出接收器以及用类似 SQL 的语言表示的数据转换。你可以在 Azure 门户中监视并调整作业的规模/速度，从几千字节扩展到千兆字节或者每秒处理更多事件。

流分析利用了 Microsoft Research 在开发可调性极强且适用于时间敏感型处理的流式处理引擎方面多年的工作成果，并进行了语言集成，因此可以直观地指定各种语句。

## 流分析有什么用途？
在现今这个时代，每天都有大量的数据在缆线上高速流动。那些能够实时处理这种流式数据并采取相应行动的组织可以极大地改进效率，让自己在市场中始终处于卓尔不群的地位。各行各业都可以找到进行实时流式分析的方案：由金融服务公司提供的个性化实时股票交易分析和提醒；实时检测欺诈行为；数据和身份保护服务；对物理对象（物联网，简称 IoT）中嵌入的传感器和激励器所生成的数据进行可靠的引入和分析；Web 点击流分析；当客户体验在某个时间范围内出现下降的趋势时，客户关系管理 (CRM) 应用程序会进行提醒。为了在竞争激烈的现代商业环境中获得成功，企业需要寻找最灵活、最可靠且最经济有效的方式来执行此类实时事件流数据分析。

## 主要功能和优点
* **易用性：**流分析支持简单的声明性查询模型，用于描述各种转换。为了优化易用性，流分析使用 T-SQL 变量，不再需要客户处理在技术上复杂的流处理系统。使用浏览器内查询编辑器中的[流分析查询语言](https://msdn.microsoft.com/library/zh-cn/azure/dn834998.aspx)，可获得 intelli-sense auto-complete，它可帮助便捷地实现时序查询（包括临时联接、窗口化聚合、临时筛选器）以及其他常见的操作（例如联接、聚合、投影和筛选）。此外，依据示例数据文件进行浏览器内查询测试还可实现快速、迭代的开发。

* **可伸缩性：**流分析具有很强的事件吞吐量处理能力，最大吞吐量为 1GB/秒。与 [Azure 事件中心](/home/features/event-hubs/)和 [Azure IoT 中心](/home/features/iot-hub/)集成后，该解决方案就可以每秒引入数百万个来自已连接设备、点击流、日志文件等的事件。为此，流分析利用了事件中心的分区功能，每个分区产生的吞吐量为 1MB/秒。用户可以在查询定义中将计算分成多个逻辑步骤，每个步骤都可以进一步细分，以提高可伸缩性。

* **可靠性、可重复性和快速恢复：**流分析是云中的托管服务，可用于在出现故障的情况下，通过内置的恢复功能防止数据丢失并确保业务连续性。由于能够在内部维持相关状态，因此服务提供的可重复结果确保可以对事件进行存档并在未来重新处理，始终获得相同的结果。由此，当客户进行根原分析和假设情况分析等时便可以及时回去调查计算结果。

* **低成本：**作为一种云服务，流分析已经过优化，用户只需很少的成本即可运行和维护各种实时分析解决方案。服务经过构建，让你可以根据流式处理单元使用量和系统处理的数据量实现现用现付。使用情况取决于已处理事件的数量，以及在群集中预配的处理相应流分析作业所需的计算能力。

* **引用数据：**用户可以通过流分析来指定和使用引用数据。这些引用数据可以是历史数据，也可以只是在一段时间内更改次数较少的非流式处理数据。系统简化了引用数据的使用，将其视同其他传入事件流，可以与其他实时引入的事件流进行联接以执行各种转换。

* **用户定义函数：**流分析已与 Azure 机器学习集成，可定义机器学习服务中的函数调用作为流分析查询的一部分。这可扩展流分析的功能，以便利用现有的 Azure 机器学习解决方案。有关详细信息，请查看[机器学习集成教程](/documentation/articles/stream-analytics-machine-learning-integration-tutorial/)。

* **连接性：**流分析会直接连接到 Azure 事件中心和 Azure IoT 中心以引入流，并会连接到 Azure Blob 服务以引入历史数据。结果可以从流分析写入 Azure 存储 Blob 或表、Azure SQL 数据库、Azure Data Lake Store、DocumentDB、事件中心、Azure 服务总线主题或队列，以及 Power BI，然后从中可以对结果进行可视化，用工作流进行进一步处理，通过 [Azure HDInsight](/home/features/hdinsight/) 用于批量分析或者作为一系列事件再次处理。使用事件中心时，可以将多个流分析与其他数据源和处理引擎组合在一起，而不会失去计算的流处理本质。

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)。

## 后续步骤
我们已经向你介绍了流分析，这是一种托管服务，适用于对物联网的数据进行流式分析。若要了解有关此服务的详细信息，请参阅：

* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:update meta properties; wording update-->