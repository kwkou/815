<properties linkid="manage-services-hdinsight-hbase-get-started-hdinsight-hadoop" urlDisplayName="Get Started" pageTitle="Get started using HBase with Hadoop in HDInsight | Azure" metaKeywords="" description="Get started using HBase with Hadoop in HDInsight. learn how to created HBase tables and query them with Hive." metaCanonical="" services="hdinsight" documentationCenter="" title="Get started using HBase with Hadoop in HDInsight" authors="bradsev" solutions="big-data" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="08/21/2014" ms.author="bradsev"></tags>

# 开始在 HDInsight 中将 HBase 与 Hadoop 配合使用

HBase 是一种低延迟的 NoSQL 数据库，可用于对大数据进行联机事务处理。HBase 以集成到 Azure 环境中的托管群集形式提供。这些群集配置为在 Azure Blob 存储中直接存储数据，这样就降低了延迟，使客户在性能与价格方面做出选择时拥有更大的弹性。客户可以生成用于处理大型数据集的交互式网站，生成用于存储数百万个终结点的传感器数据与遥测数据的服务，并通过 Hadoop 作业来分析这些数据。

在本教程中，你将要了解如何使用 HDInsight 创建和查询 HBase 表，其中将会介绍以下过程：

-   如何使用 Azure 门户设置 HBase 群集。
-   如何启用并使用 RDP 来访问 HBase shell，然后使用 HBase shell 创建 HBase 示例表、添加行并列出表中的行。
-   如何创建要映射到现有 HBase 表的 Hive 表，并使用 HiveQL 来查询 HBase 表中的数据。
-   如何使用 .NET SDK 创建新的 HBase 表、列出帐户中的 HBase 表，以及在表中添加和检索行。

## 本教程的内容

-   [在 Azure 门户中设置 HBase 群集][]
-   [从 HBase shell 创建 HBase 示例表][]
-   [使用 Hive 查询 HBase 表][]
-   [使用 HBase C# API 创建一个 HBase 表并从该表中检索数据][]
-   [摘要][]
-   [后续步骤][]

## <a name="create-hbase-cluster"></a>在 Azure 门户中设置 HBase 群集

本部分介绍如何使用 Azure 门户设置 HBase 群集。

注意：
本文中的步骤是使用基本配置设置来创建 HDInsight 群集。有关其他群集配置设置的信息（例如，使用 Azure 虚拟网络或者 Hive 和 Oozie 的元存储），请参阅“设置 HDInsight 群集”。
[WACOM.INCLUDE [provisioningnote][]]

**在 Azure 门户中设置 HDInsight 群集**

1.  登录到 [Azure 管理门户][]。

2.  单击左侧的“HDInsight”以列出你的帐户中群集的状态，然后单击左下角的“+新建”图标。

    ![][]

3.  在左侧单击第二列中的“HDInsight”图标，然后单击下一列中的“HBase”选项。指定“群集名称”和“群集大小”的值、存储帐户的名称以及新 HBase 群集的密码。

    ![][1]

4.  单击左下角的勾选图标以创建该 HBase 群集。

## <a name="create-sample-table"></a>从 HBase shell 创建 HBase 示例表

本部分介绍如何启用并使用远程桌面协议 (RDP) 来访问 HBase shell，然后使用 HBase shell 创建 HBase 示例表、添加行并列出表中的行。

本部分的内容假设你已完成第一部分所述的过程，也就是说，你已成功创建了一个 HBase 群集。

**启用与 HBase 群集的 RDP 连接**

1.  若要启用与 HDInsight 群集的远程桌面连接，请选择你创建的 HBase 群集，然后单击“配置”选项卡。单击页底部的“启用远程”按钮，以启用与群集的 RDP 连接。
2.  在“配置远程桌面”向导中提供凭据和过期日期，然后单击右下角带圆圈的勾选图标。（可能需要几分钟时间才能完成该操作。）
3.  若要连接到 HDInsight 群集，请单击“配置”选项卡底部的“连接”按钮。

**打开 HBase Shell**

1.  在 RDP 会话中，单击桌面上的“Hadoop 命令提示符”快捷方式。

2.  从当前文件夹位置切换到 HBase 主目录：

        cd %HBASE_HOME%\bin

3.  打开 HBase shell：

        hbase shell

**创建示例表、添加数据并检索数据**

1.  创建示例表：

        create 'sampletable', 'cf1'

2.  在示例表中添加一行：

        put 'sampletable', 'row1', 'cf1:col1', 'value1'

3.  列出示例表中的行：

        scan 'sampletable'

## <a name="hive-query"></a>使用 Hive 查询 HBase 表

设置 HBase 群集并创建一个表后，可以使用 Hive 来查询该表。本部分将创建要映射到 HBase 表的 Hive 表，并使用 Hive 来查询 HBase 表中的数据。

**打开群集仪表板**

1.  登录到 [Azure 管理门户][]。
2.  单击左窗格中的“HDINSIGHT”。你将会看到所创建的群集的列表，包括你刚刚在上一部分中创建的群集。
3.  单击你要在其中运行该 Hive 作业的群集名称。
4.  单击页底部的“管理群集”以打开群集仪表板。这会在另一个浏览器选项卡中打开一个网页。
5.  输入 Hadoop 用户帐户的用户名和密码。默认的用户名为 **admin**，密码是你在设置过程中输入的密码。仪表板类似于：

    ![][2]

**运行 Hive 查询**

1.  若要创建映射到 HBase 表的 Hive 表，请在 Hive 控制台窗口中输入以下 HiveQL 脚本，然后单击“提交”按钮。在执行此语句之前，请确保已使用 HBase Shell 在 HBase 中创建此处引用的示例表。

        CREATE EXTERNAL TABLE hbasesampletable(rowkey STRING, col1 STRING, col2 STRING)
        STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
        WITH SERDEPROPERTIES ('hbase.columns.mapping' = ':key,cf1:col1,cf1:col2')
        TBLPROPERTIES ('hbase.table.name' = 'sampletable');

2.  若要针对 HBase 中的数据执行 Hive 查询，请在 Hive 控制台窗口中输入以下 HiveQL 脚本，然后单击“提交”按钮。

        SELECT count(*) FROM hbasesampletable;

3.  若要检索 Hive 查询的结果，请在作业完成执行时，在“作业会话”窗口中单击“查看详细信息”链接。

注意：单击 HBase shell 链接会将选项卡切换到“HBase Shell”。

**浏览输出文件**

1.  在群集仪表板中，单击顶部的“文件”。
2.  单击“Templeton-Job-Status”。
3.  单击其上次修改时间比你前面记下的作业开始时间稍晚的 GUID 编号。记下此 GUID。在后一个部分将要用到它。
4.  **stdout** 文件包含后一个部分需要的数据。如果需要，你可以单击“stdout”来下载该数据文件的副本。

## <a name="hbase-powershell"></a>使用 HBase C# API 创建一个 HBase 表并从该表中检索数据

Marlin 是位于 REST API 顶部的一个精简层，它可以使用 ProtoBuf（以 C# 编写）来处理与 HBase 的交互。必须从 github 下载 Marlin 项目，该项目必须已构建为使用 HBase .NET SDK。

1.  遵照所述的生成步骤。从 [Marlin 的项目页][]下载 Marlin 项目。将它解压缩到本地目录。

2.  在 Visual Studio 中打开该项目。转到“工具”菜单 -\>“库程序包管理器”，然后选择“管理解决方案的 NuGet 包...”，以打开“管理 NuGet 包”管理器向导

    ![][3]

3.  在右上角的“联机”框中搜索 protobuf-net 并安装 v2.0.0.68。在“解决方案资源管理器”中右键单击“Marlin”项目并选择“生成”，以生成 Marlin 项目。

4.  检索最终生成的 marlin.dll 并将其添加到你的 C# 项目。

5.  使用群集凭据创建 Marlin 的新实例并检索群集版本：

        var credentials = ClusterCredentials.Create("https://yourclustername.azurehdinsight.cn/", "user", "password");
            var marlin = new Marlin(credentials);
        // retrieve the version as a test
        var version = marlin.GetVersion();
        Console.WriteLine(version);

6.  若要列出群集中的表，可以使用以下代码：

        var tables = marlin.ListTables();
        foreach(var tableName in tables.name) 
                Console.WriteLine(tableName);

7.  若要创建新的 HBase 表，请使用以下代码：

        var schema = new TableSchema();
        schema.name = "sampletable";
        schema.columns.Add(new ColumnSchema() { name = "cf1" });
        schema.columns.Add(new ColumnSchema() { name = "cf2" });
        marlin.CreateTable(schema);

8.  若要按某个行的键检索该行，请指定表名称和行键以检索行值。

        var cells = marlin.GetCells("sampletable", "row1");
        var row = cells.rows[0];
            foreach(var val in row.values) 
            {
               Console.WriteLine(Encoding.UTF8.GetString(val.data));
            }

9.  若要存储新的数据行，可以使用以下代码：

        CellSet set = new CellSet();
        CellSet.Row row = new CellSet.Row() { key = BitConverter.GetBytes(1337) };
            var value = new Cell()
                    {
                        column = Encoding.UTF8.GetBytes("cf1:d"),
                        data = Encoding.UTF8.GetBytes("Hello World!")
                    };
            row.values.Add(value);
            set.rows.Add(row);
        marlin.StoreCells("sampletable", set);

## <a name="summary"></a>摘要

在本教程中，你已了解如何设置 HBase 表、如何创建表，以及如何从 HBase shell 查看这些表中的数据。你还了解了如何使用 Hive 来查询 HBase 表中的数据，以及如何使用 HBase C# API 创建一个 HBase 表并从该表中检索数据。

## <a name="next"></a>后续步骤

[HDInsight HBase 概述][]：
HBase 是构建于 Hadoop 上的 Apache 开源 NoSQL 数据库，用于为大量非结构化和半结构化数据提供随机访问和高度一致性。

[在 Azure 虚拟网络上设置 HBase 群集][]：
通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。

<!--- [azure-member-offers]: http://azure.microsoft.com/en-us/pricing/member-offers/ --->

  [在 Azure 门户中设置 HBase 群集]: #create-hbase-cluster
  [从 HBase shell 创建 HBase 示例表]: #create-sample-table
  [使用 Hive 查询 HBase 表]: #hive-query
  [使用 HBase C# API 创建一个 HBase 表并从该表中检索数据]: #hbase-powershell
  [摘要]: #summary
  [后续步骤]: #next
  [provisioningnote]: ../includes/hdinsight-provisioning.md
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: http://i.imgur.com/PmGynKZ.jpg
  [1]: http://i.imgur.com/ecxbB9K.jpg
  [2]: http://i.imgur.com/tMwXlj9.jpg
  [Marlin 的项目页]: https://github.com/thomasjungblut/marlin
  [3]: http://i.imgur.com/hUNoJDJ.jpg
  [HDInsight HBase 概述]: ../hdinsight-hbase-overview/
  [在 Azure 虚拟网络上设置 HBase 群集]: ../hdinsight-hbase-provision-vnet
