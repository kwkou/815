<properties
    pageTitle="使用 Power Query 将 Excel 连接到 Hadoop | Azure"
    description="了解如何利用商业智能组件和使用 Power Query for Excel 访问 HDInsight 上的 Hadoop 中存储的数据。"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="01ad2f90-7520-44d9-8c16-4d936faaff9b"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 Power Query 将 Excel 连接到 Hadoop

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Microsoft 大数据解决方案的一个关键功能是，将 Microsoft 商业智能 (BI) 组件与 Azure HDInsight 中的 Hadoop 群集相集成。此集成的一个主要示例是能够使用 Microsoft Power Query for Excel 外接程序将 Excel 连接到包含与 Hadoop 群集关联的数据的 Azure 存储帐户。本文将逐步说明如何设置和使用 Power Query 来查询与 HDInsight 管理的 Hadoop 群集关联的数据。

> [AZURE.NOTE]
尽管可以对基于 Linux 或 Windows 的 HDInsight 群集使用本文中的步骤，但客户端工作站需要 Windows。
> 
> 

### 先决条件
开始阅读本文之前，必须具备以下先决条件：

* **HDInsight 群集**。若要配置 HDInsight 群集，请参阅 [Azure HDInsight 入门][hdinsight-get-started]。
* 运行 Windows 7、Windows Server 2008 R2 或更高版本操作系统的**工作站**。
* **Office 2013 Professional Plus、Office 365 ProPlus、Excel 2013 Standalone 或 Office 2010 Professional Plus**。

## <a id="InstallPowerQuery"></a> 安装 Power Query
可以使用 Power Query 将来自多种数据源的数据导入到 Microsoft Excel 中，在 Excel 中，它可以增强 PowerPivot 和 Power View 之类的 BI 工具。具体来说，Power Query 可以导入已输出的数据，或者导入由在 HDInsight 群集上运行的 Hadoop 作业生成的数据。

从 [Microsoft 下载中心][powerquery-download]下载 Microsoft Power Query for Excel 并安装它。

## <a id="ImportData"></a> 将 HDInsight 数据导入 Excel
使用用于 Excel 的 Power Query 外接程序，可以轻松地将 HDInsight 群集中的数据导入到 Excel 中，然后可以在 Excel 中使用 PowerPivot 和 Power Map 之类的 BI 工具对数据进行检查、分析和显示。

**从 HDInsight 群集导入数据**

1. 打开 Excel。
2. 创建一个新的空白工作簿。
3. 依次单击“Power Query”菜单、“来自 Azure”和“来自 Microsoft Azure HDInsight”。
   
    ![HDI.PowerQuery.SelectHdiSource][image-hdi-powerquery-hdi-source]
   
    **注意：**如果未显示“Power Query”菜单，请转到“文件”>“选项”>“外接程序”，从页面底部的下拉“管理器”框中选择“COM 外接程序”。选择“转到...”按钮，并且确认已选中 Power Query for Excel 外接程序所对应的框。
   
    **注意：**Power Query 还允许你通过单击“从其他源”从 HDFS 中导入数据。
4. 对于“帐户名称”，请输入与你的群集相关联的 Azure Blob 存储帐户的名称，然后单击“确定”。（对于 Azure 中国区，**帐户名称**必须是一个完整的 URL，如: https://<你的帐户名称>.blob.core.chinacloudapi.cn/），[这可以是默认存储帐户](/documentation/articles/hdinsight-administer-use-management-portal/#find-the-default-storage-account)或链接存储帐户。格式是 *https://<StorageAccountName>.blob.core.chinacloudapi.cn/*。
5. 对于“帐户密钥”，请输入 Blob 存储帐户的密钥，然后单击“保存”。（仅需在首次访问该存储时执行此操作。）
6. 在“查询编辑器”左侧的“导航器”窗格中，双击 Blob 存储容器名称。默认情况下，该容器名称与群集名称相同。
7. 在“名称”列中找到 **HiveSampleData.txt**（文件夹路径是 **../hive/warehouse/hivesampletable/**），然后单击 HiveSampleData.txt 左侧的“二进制”。HiveSampleData.txt 随所有群集提供。（可选）可使用自己的文件。
   
    ![HDI.PowerQuery.ImportData][image-hdi-powerquery-importdata]
8. 可根据需要重命名列名称。准备就绪后，单击“关闭并加载”。数据已加载到工作簿：
   
    ![HDI.PowerQuery.ImportedTable][image-hdi-powerquery-imported-table]  


## <a id="NextSteps"></a>后续步骤
在本文中，你已了解如何使用 Power Query 将数据从 HDInsight 检索到 Excel 中。同样地，你也可以将来自 HDInsight 的数据检索到 Azure SQL 数据库中。也可以将数据上传到 HDInsight 中。若要了解更多信息，请参阅下列文章：

* [使用 Microsoft Hive ODBC Driver 将 Excel 连接到 HDInsight][hdinsight-ODBC]
* [将数据上传到 HDInsight][hdinsight-upload-data]

[hdinsight-ODBC]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[image-hdi-powerquery-hdi-source]: ./media/hdinsight-connect-excel-power-query/HDI.PowerQuery.SelectHdiSource.png
[image-hdi-powerquery-importdata]: ./media/hdinsight-connect-excel-power-query/HDI.PowerQuery.ImportData.png
[image-hdi-powerquery-imported-table]: ./media/hdinsight-connect-excel-power-query/HDI.PowerQuery.ImportedTable.PNG

[powerquery-download]: https://www.microsoft.com/zh-cn/download/details.aspx?id=39379

<!---HONumber=Mooncake_0120_2017-->