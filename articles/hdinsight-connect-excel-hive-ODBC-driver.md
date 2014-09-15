<properties linkid="manage-services-hdinsight-connect-excel-with-hive-ODBC" urlDisplayName="Connect Excel to HDInsight" pageTitle="Connect Excel to HDInsight with the Hive ODBC Driver | Azure" metaKeywords="" description="Learn how to set up and use the Microsoft Hive ODBC driver for Excel to query data in an HDInsight cluster." metaCanonical="" services="hdinsight" documentationCenter="" title="Connect Excel to HDInsight with the Microsoft Hive ODBC Driver" authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

# 使用 Microsoft Hive ODBC Driver 将 Excel 连接到 HDInsight

Microsoft 的大型数据解决方案的一大功能是，将 Microsoft 商业智能 (BI) 组件与已由 Azure HDInsight 部署的 Apache Hadoop 群集相集成。此集成的一个例子是，能够使用 Microsoft Hive 开放式数据库连接 (ODBC) 驱动程序将 Excel 连接到 HDInsight Hadoop 群集的 Hive 数据仓库。

还可以使用用于 Excel 的 Microsoft Power Query 外接程序从 Excel 连接与 HDInsight 群集和其他数据源（包括其他非 HDInsight Hadoop 群集）关联的数据。有关安装和使用 Power Query 的信息，请参阅[利用 Power Query 将 Excel 连接到 HDInsight][]。

**先决条件**：

在开始阅读本文前，你必须具有：

-   HDInsight 群集。若要配置一个 HDInsight 群集，请参阅 [Azure HDInsight 入门][]。
-   运行 Windows 8、Windows 7、Windows Server 2012 或 Windows Server 2008 R2 的计算机。
-   Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus。

## 本文内容

1.  [安装 Microsoft Hive ODBC 驱动程序][]
2.  [创建 Hive ODBC 数据源][]
3.  [将数据从 HDInsight 群集导入到 Excel 中][]
4.  [后续步骤][]

## 安装 Microsoft Hive ODBC 驱动程序

从[下载中心][]下载并安装 Microsoft Hive ODBC 驱动程序。

此驱动程序可以安装在 32 位或 64 位版本的 Windows 7、Windows 8、Windows Server 2008 R2 和 Windows Server 2012 上，并将允许连接到 Azure HDInsight（1.6 及更高版本）和 Azure HDInsight Emulator（1.0.0.0 及更高版本）。你应安装与你将在其中使用 ODBC 驱动程序的应用程序版本匹配的版本。在本教程中，将通过 Office Excel 使用此驱动程序。

## 创建 Hive ODBC 数据源

下列步骤演示如何创建 Hive ODBC 数据源。

1.  在 Windows 8 中，按 Windows 键以打开“开始”屏幕，然后键入“数据源” 。
2.  单击“设置 ODBC 数据源(32 位)” 或“设置 ODBC 数据源(64 位)” ，具体取决于 Office 版本。如果你使用的是 Windows 7，请从“管理工具” 选择“ODBC 数据源(32 位)” 或“ODBC 数据源(64 位)” 。这将启动“ODBC 数据源管理器” 对话框。

    ![ODBC 数据源管理器][]

3.  从“用户 DNS”单击“添加” 以打开“新建数据源” 向导。
4.  选择“Microsoft Hive ODBC 驱动程序” ，然后单击“完成” 。这将启动“Microsoft Hive ODBC 驱动程序 DNS 安装程序” 对话框。

5.  键入或选择以下值：

<table border="1">
<tr><td><strong>属性</strong></td><td><strong>说明</strong></td></tr>
<tr><td>数据源名称</td><td>为你的数据源提供名称</td></tr>
<tr><td>Host</td><td>输入 <HDInsightClusterName>.hdinsightservice.cn. For example, myHDICluster.hdinsightservice.cn</td></tr>
<tr><td>Port</td><td>使用 <strong>443</strong>。（此端口已从 563 更改为 443。）</td></tr>
<tr><td>数据库</td><td>使用&ldquo;默认&rdquo;<strong></strong>。</td></tr>
<tr><td>Hive 服务器类型</td><td>选择&ldquo;Hive Server 2&rdquo;<strong></strong></td></tr>
<tr><td>机制</td><td>选择&ldquo;Azure HDInsight 服务&rdquo;<strong></strong></td></tr>
<tr><td>HTTP 路径</td><td>将此字段留空。</td></tr>
<tr><td>用户名</td><td>输入 HDInsight 群集用户的用户名。这是在群集设置过程中创建的用户名。如果你使用了&ldquo;快速创建&rdquo;选项，则默认用户名是&ldquo;admin&rdquo;<strong></strong>。</td></tr>
<tr><td>密码</td><td>输入 HDInsight 群集用户的密码。</td></tr>
</table>

 在单击“高级选项” 时，有一些重要参数要注意：

<table border="1">
<tr><td>使用本机查询</td><td>选择此项时，ODBC 驱动程序将不会尝试将 TSQL 转换为 HiveQL。仅当你 100% 确定提交的是纯 HiveQL 语句时，才应使用此项。连接 SQL Server 或 Azure SQL Database 时，应将此项保留为未选中状态。</td></tr>
<tr><td>每块提取的行数</td><td>提取大量记录时，可能需要调整此参数以确保最佳性能。</td></tr>
<tr><td>默认字符串列长度、<br/> 二进制列长度、<br/> 十进制列小数位数</td><td>数据类型长度和精度可能会影响返回数据的方式。由于精度损失和/或截断，可能会返回不正确的信息。</td></tr>
</table>

![高级选项][]

6.  单击“测试” 以测试数据源。正确配置数据源时，将显示“测试成功完成!”**。
7.  单击“确定” 以关闭“测试”对话框。现在，新的数据源应该在“ODBC 数据源管理器” 中列出。
8.  单击“确定” 以退出向导。

## 将 HDInsight 群集中的数据导入到 Excel 中

下列步骤介绍如何使用在上述步骤中创建的 ODBC 数据源将数据从 Hive 表中导入到 Excel 工作簿。

1.  在 Excel 中打开新工作簿或现有工作簿。
2.  在“数据” 选项卡中，依次单击“获取外部数据” 磁贴、“从其他数据源” 、“从数据连接向导” 以启动“数据连接向导” 。

    ![打开数据连接向导][]

3.  选择“ODBC DSN” 作为数据源，然后单击“下一步” 。
4.  从 ODBC 数据源中，选择你在上一步中创建的数据源名称，然后单击“下一步” 。
5.  在向导中重新输入群集的密码，然后单击“测试” 以验证配置
6.  单击“确定” 以关闭“测试”对话框。
7.  单击“确定” 。等待“选择数据库和表” 对话框打开。这可能需要几秒钟时间。
8.  选择要导入的表，然后单击“下一步” 。*hivesampletable* 是 HDInsight 群集随附的示例 Hive 表。如果你还没有创建表，则可以选择它。有关运行 Hive 查询和创建 Hive 表的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][]。
9.  单击“完成” 。
10. 在“导入数据” 对话框中，你可以更改或指定查询。为此，请单击“属性” 。这可能需要几秒钟时间。
11. 单击“定义” 选项卡，然后在“命令文本” 文本框中的 Hive select 语句后面追加“LIMIT 200” 。此修改将返回的记录集限制为 200。

    ![连接属性][]

12. 单击“确定” 以关闭“连接属性”对话框。
13. 单击“确定” 以关闭“导入数据” 对话框。
14. 重新输入密码，然后单击“确定” 。需要几秒钟时间才能将数据导入到 Excel 中。

## 后续步骤

在本文中，你已了解如何使用 Microsoft Hive ODBC 驱动程序将来自 HDInsight 服务的数据检索到 Excel 中。同样地，你也可以将来自 HDInsight 服务的数据检索到 SQL Database 中。也可以将数据上载到 HDInsight 服务中。若要了解更多信息，请参阅以下文章：

-   [使用 HDInsight 分析航班延误数据][]
-   [将数据上载到 HDInsight][]
-   [将 Sqoop 与 HDInsight 配合使用][]

  [利用 Power Query 将 Excel 连接到 HDInsight]: /en-us/documentation/articles/hdinsight-connect-excel-power-query/
  [Azure HDInsight 入门]: /en-us/documentation/articles/hdinsight-get-started/
  [安装 Microsoft Hive ODBC 驱动程序]: #InstallHiveODBCDriver
  [创建 Hive ODBC 数据源]: #CreateHiveODBCDataSource
  [将数据从 HDInsight 群集导入到 Excel 中]: #ImportData
  [后续步骤]: #nextsteps
  [下载中心]: http://go.microsoft.com/fwlink/?LinkID=286698
  [ODBC 数据源管理器]: ./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.SimbaHiveOdbc.DataSourceAdmin1.png
  [高级选项]: ./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.HiveOdbc.DataSource.AdvancedOptions1.png
  [打开数据连接向导]: ./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.SimbaHiveOdbc.Excel.DataConnection1.png
  [将 Hive 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-hive/
  [连接属性]: ./media/hdinsight-connect-excel-hive-ODBC-driver/HDI.SimbaHiveODBC.Excel.ConnectionProperties1.png
  [使用 HDInsight 分析航班延误数据]: /en-us/documentation/articles/hdinsight-analyze-flight-delay-data/
  [将数据上载到 HDInsight]: /en-us/documentation/articles/hdinsight-upload-data/
  [将 Sqoop 与 HDInsight 配合使用]: ../hdinsight-use-sqoop/
