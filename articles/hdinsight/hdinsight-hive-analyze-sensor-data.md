<properties
    pageTitle="使用 Hive 和 Hadoop 分析传感器数据 | Azure"
    description="了解如何通过将 Hive 查询控制台与 HDInsight (Hadoop) 配合使用来分析传感器数据，然后通过 PowerView 在 Microsoft Excel 中可视化数据。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="a8ac160c-1cef-45d9-bf36-7beb5a439105"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/17/2017"
    wacn.date="03/10/2017"
    ms.author="larryfr" />  


# 使用 HDInsight 中 Hadoop上的 Hive 查询控制台分析传感器数据
了解如何通过将 Hive 查询控制台与 HDInsight (Hadoop) 配合使用来分析传感器数据，然后通过使用 Power View 在 Microsoft Excel 中可视化数据。

> [AZURE.IMPORTANT]
本文档中的步骤仅适用于基于 Windows 的 HDInsight 群集。低于 HDInsight 3.4 的 HDInsight 版本仅在 Windows 上提供。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

在本示例中，你将使用 Hive 来处理由采暖、通风和空调 (HVAC) 系统生成的历史数据，以识别无法可靠地保持设定温度的系统。你将了解如何执行以下操作：

* 创建 HIVE 表，以查询存储在逗号分隔值 (CSV) 文件中的数据。
* 创建 HIVE 查询以分析数据。
* 使用 Microsoft Excel 连接到 HDInsight（使用开放的数据库连接 (ODBC)），以检索分析的数据。
* 使用 Power View 可视化数据。

![解决方案体系结构关系图](./media/hdinsight-hive-analyze-sensor-data/hvac-architecture.png)  

## 先决条件
* HDInsight (Hadoop) 群集：有关创建群集的信息，请参阅[在 HDInsight 中预配 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)。
* Microsoft Excel 2013
  
    > [AZURE.NOTE]
    Microsoft Excel 用于通过 [Power View](https://support.office.com/Article/Power-View-Explore-visualize-and-present-your-data-98268d31-97e2-42aa-a52b-a68cf460472e?ui=en-us&rs=en-us&ad=US) 实现数据可视化。
    > 
    > 
* [Microsoft Hive ODBC 驱动程序](http://www.microsoft.com/download/details.aspx?id=40886)

## 运行示例
1. 通过 Web 浏览器导航到以下 URL。将 `<clustername>` 替换为 HDInsight 群集的名称。
   
         https://<clustername>.azurehdinsight.cn
   
    出现提示时，可以通过使用设置此群集时所用的管理员用户名和密码进行身份验证。
2. 在打开的网页中，单击“入门库”选项卡，然后在“使用示例数据的解决方案”类别下方，单击“传感器数据分析”示例。
   
    ![库映像入门](./media/hdinsight-hive-analyze-sensor-data/getting-started-gallery.png)  

3. 按照网页上提供的说明完成该示例。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->