<properties
    pageTitle="将 Hive 用于 Hadoop 以进行网站日志分析| Azure"
    description="了解如何通过将 Hive 与 HDInsight 配合使用来分析网站日志。我们将使用日志文件作为 HDInsight 表的输入，并使用 HiveQL 来查询数据。"
    services="hdinsight"
    documentationcenter=""
    author="nitinme"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="6fb7b5c2-8df4-40b1-a9e2-6815080004f9"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/17/2016"
    wacn.date="03/10/2017"
    ms.author="nitinme" />  


# 将 Hive 与基于 Windows 的 HDInsight 配合使用来分析网站的日志
了解如何通过将 HiveQL 与 HDInsight 配合使用来分析来自网站的日志。网站日志分析可用于根据类似活动分类受众，按人口统计分类站点访问者，以及了解他们查看的内容和这些内容来自的网站等。

> [AZURE.IMPORTANT]
本文档中的步骤仅适用于基于 Windows 的 HDInsight 群集。低于 HDInsight 3.4 的 HDInsight 版本仅在 Windows 上提供。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

在此示例中，你将使用 HDInsight 群集来分析网站日志文件，以深入了解每天从外部网站访问网站的频率。你还将生成用户遇到的网站错误的摘要。你将了解如何执行以下操作：

* 连接到 Azure Blob 存储，其中包含网站日志文件。
* 创建 HIVE 表以查询这些日志。
* 创建 HIVE 查询以分析数据。
* 使用 Microsoft Excel 连接到 HDInsight（通过使用开放的数据库连接 (ODBC)），以检索分析的数据。

![HDI.Samples.Website.Log.Analysis][img-hdi-weblogs-sample]

## 先决条件
* 必须已在 Azure HDInsight 上预配 Hadoop 群集。有关说明，请参阅[预配 HDInsight 群集][hdinsight-provision]。
* 必须已安装 Microsoft Excel 2013 或 Excel 2010。
* 必须拥有 [Microsoft Hive ODBC 驱动程序](http://www.microsoft.com/download/details.aspx?id=40886)，才能将数据从 Hive 导入 Excel。

## 运行示例
1. 在 [Azure 门户预览](https://portal.azure.cn/)中，从启动板（如果已将群集固定在此处），单击要在其中运行示例的群集磁贴。
2. 在群集边栏选项卡中的“快速链接”下，单击“群集仪表板”，然后在“群集仪表板”边栏选项卡上，单击“HDInsight 群集仪表板”。或者，也可以直接使用以下 URL 打开仪表板：
   
         https://<clustername>.azurehdinsight.cn
   
    出现提示时，通过使用设置群集时所用的管理员用户名和密码进行身份验证。
3. 在打开的网页中，单击“入门库”选项卡，然后在“使用示例数据的解决方案”类别下方，单击“网站日志分析”示例。
4. 按照网页上提供的说明完成该示例。

## 后续步骤
尝试以下示例：[通过将 Hive 与 HDInsight 配合使用分析传感器数据](/documentation/articles/hdinsight-hive-analyze-sensor-data/)。

[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-sensor-data-sample]: /documentation/articles/hdinsight-use-hive-sensor-data-analysis/

[img-hdi-weblogs-sample]: ./media/hdinsight-hive-analyze-website-log/hdinsight-weblogs-sample.png

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->