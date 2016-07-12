<properties
	pageTitle="使用 Hive 和 Hadoop 分析传感器数据 |Azure"
	description="了解如何通过将 Hive 查询控制台与 HDInsight (Hadoop) 配合使用来分析传感器数据，然后通过 PowerView 在 Microsoft Excel 中可视化数据。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/22/2016"
	wacn.date="06/29/2016"/>

#使用 HDInsight 中 Hadoop上的 Hive 查询控制台分析传感器数据

了解如何通过将 Hive 查询控制台与 HDInsight (Hadoop) 配合使用来分析传感器数据，然后通过使用 Power View 在 Microsoft Excel 中可视化数据。

> [AZURE.NOTE]本文档中的步骤仅适用于基于 Windows 的 HDInsight 群集。

在本示例中，你将使用 Hive 来处理由采暖、通风和空调 (HVAC) 系统生成的历史数据，以识别无法可靠地保持设定温度的系统。你将了解如何执行以下操作：

- 创建 HIVE 表，以查询存储在逗号分隔值 (CSV) 文件中的数据。
- 创建 HIVE 查询以分析数据。
- 使用 Microsoft Excel 连接到 HDInsight（使用开放的数据库连接 (ODBC)），以检索分析的数据。
- 使用 Power View 可视化数据。

![解决方案体系结构关系图](./media/hdinsight-hive-analyze-sensor-data/hvac-architecture.png)

##先决条件

* HDInsight (Hadoop) 群集：有关创建群集的信息，请参阅[在 HDInsight 中预配 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)。

* Microsoft Excel 2013

	> [AZURE.NOTE]Microsoft Excel 用于通过 [Power View](https://support.office.com/Article/Power-View-Explore-visualize-and-present-your-data-98268d31-97e2-42aa-a52b-a68cf460472e?ui=zh-CN&rs=zh-CN&ad=US) 实现数据可视化。

* [Microsoft Hive ODBC 驱动程序](http://www.microsoft.com/download/details.aspx?id=40886)

##运行示例

1. 通过 Web 浏览器导航到以下 URL。将 `<clustername>` 替换为 HDInsight 群集的名称。

	 	https://<clustername>.azurehdinsight.cn

	出现提示时，可以通过使用设置此群集时所用的管理员用户名和密码进行身份验证。

2. 在打开的网页中，单击“入门库”选项卡，然后在“使用示例数据的解决方案”类别下方，单击“传感器数据分析”示例。

3. 按照网页上提供的说明完成该示例。

<!---HONumber=82-->