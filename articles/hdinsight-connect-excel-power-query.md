<properties linkid="manage-services-hdinsight-connect-excel-with-power-query" urlDisplayName="HDInsight and Excel" pageTitle="Connect Excel to HDInsight with Power Query | Azure" metaKeywords="hdinsight, excel, data explorer, hive excel, hdinsight excel, power query" description="Learn how to take advantage of business intelligence components and use Excel to access data stored in Azure HDInsight using Power Query." metaCanonical="" services="hdinsight" documentationCenter="" title="Connect Excel to Azure HDInsight with Power Query" authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

# 利用 Power Query 将 Excel 连接到 HDInsight

Microsoft 的大型数据解决方案的一个关键功能是，将 Microsoft 商业智能 (BI) 组件与 HDInsight Hadoop 群集相集成。此集成的一个主要示例是能够使用 Microsoft Power Query for Excel 将 Excel 连接到包含与 HDInsight 群集关联的数据的 Azure 存储帐户。本文将说明如何从 Excel 设置和使用 Power Query 以查询与 HDInsight 群集关联的数据。

**先决条件**

在开始阅读本文前，你必须具有：

-   HDInsight 群集。若要配置一个 HDInsight 群集，请参阅 [Azure HDInsight 入门][hdinsight-get-started]。
-   运行 Windows 8、Windows 7、Windows Server 2012 或 Windows Server 2008 R2 的计算机。
-   Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus。

## 本文内容

-   [安装 Microsoft Power Query for Excel](#InstallPowerQuery)
-   [将数据导入 Excel](#ImportData)
-   [后续步骤](#NextSteps)

## 安装 Microsoft Power Query for Excel

可以使用 Power Query 将来自多种数据源的数据导入到 Microsoft Excel 中，在 Excel 中，它可以增强 PowerPivot 和 PowerView 之类的商业智能 (BI) 工具。具体来说，Power Query 可以导入已输出的数据，或者导入由在 HDInsight 群集上运行的 Hadoop 作业生成的数据。

从 [Microsoft 下载中心][powerquery-download]下载 Microsoft Power Query for Excel 并安装它。

## 将 HDInsight 数据导入 Excel

使用用于 Excel 的 Power Query 外接程序，可以轻松地将 HDInsight 群集中的数据导入到 Excel 中，然后可以在 Excel 中使用 PowerPivot 和 Power Map 之类的商业智能工具对数据进行检查、分析和显示。

**从 HDInsight 群集导入数据**

1.  打开 Excel。

2.  创建一个新的空白工作簿。

3.  依次单击“Power Query” 菜单、“从其他源” 和“从 Azure HDInsight” 。

    ![HDI.PowerQuery.SelectHdiSource][image-hdi-powerquery-hdi-source]

    注意：如果未显示“Power Query” 菜单，请转到“文件” \>“选项” \>“外接程序” ，从页面底部的下拉“管理器” 框中选择“COM 外接程序” 。选择“转到...” 按钮，并且确认已选中 Microsoft Office Power Query for Excel 外接程序所对应的框。

4.  输入与你的群集关联的 Azure Blob 存储帐户的**帐户名称**，然后单击“确定” 。

5.  输入 Blob 存储帐户的**帐户密钥**，然后单击“保存” 。（你仅需要在首次访问该存储时执行此操作。）

6.  在“查询编辑器” 左侧的“导航器” 窗格中，双击 Blob 存储容器名称。默认情况下，该容器名称与群集名称相同。

7.  在“名称”列 中找到 **HiveSampleData.txt**（文件夹路径是 **../hive/warehouse/hivesampletable/**），然后单击 HiveSampleData.txt 左侧的“二进制” 。

    ![HDI.PowerQuery.ImportData][image-hdi-powerquery-importdata]

8.  如果需要，可以重命名列名称。当你准备好时，单击“应用并关闭” 。

    ![HDI.PowerQuery.ImportedTable][image-hdi-powerquery-imported-table]

## 后续步骤

在本文中，你已了解如何使用 Power Query 将数据从 HDInsight 检索到 Excel 中。同样地，你也可以将来自 HDInsight 的数据检索到 SQL Azure 中。也可以将数据上载到 HDInsight 中。若要了解更多信息，请参阅下列文章：

-   [使用 Microsoft Hive ODBC Driver 将 Excel 连接到 HDInsight][hdinsight-excel-odbc]
-   [将数据上载到 HDInsight][hdinsight-upload-data]。

[hdinsight-excel-odbc]: /en-us/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/
[hdinsight-get-started]: /en-us/documentation/articles/hdinsight-get-started/
[hdinsight-upload-data]: /en-us/documentation/articles/hdinsight-upload-data/

[image-hdi-powerquery-hdi-source]: ./media/hdinsight-connect-excel-power-query/HDI.PowerQuery.SelectHdiSource.png
[image-hdi-powerquery-importdata]: ./media/hdinsight-connect-excel-power-query/HDI.PowerQuery.ImportData.png 
[image-hdi-powerquery-imported-table]: ./media/hdinsight-connect-excel-power-query/HDI.PowerQuery.ImportedTable.PNG 

[powerquery-download]: http://www.microsoft.com/zh-cn/download/details.aspx?id=39379 
