<properties
	pageTitle="HBase 教程：HDInsight 中的 HBase 入门 | Azure"
	description="遵循本 HBase 教程开始在 HDInsight 中将 Apache HBase 与 Hadoop 配合使用。从 HBase shell 创建表，然后使用 Hive 查询这些表。"
	keywords="apache hbase,hbase,hbase shell,hbase tutorial"
	services="hdinsight"
	documentationCenter=""
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/07/2016"
	wacn.date="04/12/2016"/>



# HBase 教程：开始在 HDInsight 中将 Apache HBase 与 Hadoop 配合使用

了解如何使用 Hive 在 HDInsight 中预配 HBase 群集、创建 HBase 表和查询表。有关 HBase 的一般信息，请参阅 [HDInsight HBase 概述][hdinsight-hbase-overview]。

> [AZURE.NOTE]HBase（版本 0.98.0）只能用于 HDInsight 上的 HDInsight 3.1 群集（基于 Apache Hadoop 和 YARN 2.4.0）。有关版本信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][hdinsight-versions]


**先决条件**

[AZURE.INCLUDE [delete-cluster-warning](../includes/hdinsight-delete-cluster-warning.md)]

在开始阅读本 HBase 教程前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 免费试用版](/pricing/1rmb-trial/)。
- 装有 Visual Studio 2013 的**工作站**：有关说明，请参阅[安装 Visual Studio](http://msdn.microsoft.com/zh-cn/library/e2h7fzkw.aspx)。

##<a name="create-hbase-cluster"></a> 设置 HBase 群集

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]

**通过使用 Azure 经典管理门户设置 HBase 群集**


1. 登录到 [Azure 经典管理门户][azure-management-portal]。
2. 单击左下方的“新建”，然后依次单击“数据服务”、“HDInsight”、“HBase”。

	>[AZURE.NOTE]你还可以使用“自定义创建”选项。
3. 输入“群集名称”、“群集大小”、HTTP 用户密码和“存储帐户”。

	![在 HDInsight 中预配 HBase 群集][img-hdinsight-hbase-cluster-quick-create]

	默认 HTTP 用户名是 admin。你可以通过使用“自定义创建”选项自定义该名称。

	若要使用默认 HBase 设置过程，你需要一个 Azure 存储帐户。请参阅[创建 Azure 存储帐户][azure-create-storageaccount]来创建一个存储帐户。“自定义创建”选项可让你使用群集预配过程创建存储帐户选项。

	> [AZURE.WARNING]为了实现 HBase 服务的高可用性，你必须设置至少包含**三个**节点的群集。这可确保，如果一个节点发生故障，则 HBase 数据区域在其他节点上可用。

	> 如果你正在学习 HBase，请始终选择 1 作为群集大小，并在每次使用后删除该群集以节省费用。

4. 单击右下方的“创建 HDInsight 群集”以创建 HBase 群集。

>[AZURE.NOTE]在删除 HBase 群集后，你可以通过使用相同的默认 Blob 容器创建另一个 HBase 群集。新群集将选取你在原始群集中创建的 HBase 表。

## 使用 HBase shell
目前，可通过两种方式访问 HBase。本部分介绍如何使用 HBase shell。下一部分介绍如何使用 .NET SDK。

对于大多数人而言，数据以表格形式显示：

![hdinsight hbase 表格数据][img-hbase-sample-data-tabular]

在 HBase（BigTable 的一种实现）中，相同的数据看起来类似于：

![hdinsight hbase bigtable 数据][img-hbase-sample-data-bigtable]

在完成下一过程后，数据将更易于理解。


**使用 HBase shell**

1. 使用 RDP 连接到 HDInsight 中的 HBase 群集。有关 RDP 的说明，请参阅[使用 Azure 经典管理门户在 HDInsight 中管理 Hadoop 群集][hdinsight-manage-portal]。
2. 在 RDP 会话中，单击桌面上的“Hadoop 命令行”快捷方式。
3. 打开 HBase shell：

		cd %HBASE_HOME%\bin
		hbase shell

4. 创建包含两个列系列的 HBase：

		create 'Contacts', 'Personal', 'Office'
		list
5. 插入一些数据：

		put 'Contacts', '1000', 'Personal:Name', 'John Dole'
		put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
		put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
		put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
		scan 'Contacts'

	![hdinsight hadoop hbase shell][img-hbase-shell]

6. 获取单个行

		get 'Contacts', '1000'

	你将看到与使用扫描命令相同的结果，因为只有一个行。

	有关 Hbase 表架构的详细信息，请参阅 [HBase 架构设计简介][hbase-schema]。有关 HBase 命令的详细信息，请参阅 [Apache HBase 参考指南][hbase-quick-start]。


6. 退出 shell

		exit

**在联系人 HBase 表中批量加载数据**

HBase 提供了多种方法用于将数据载入表中。有关详细信息，请参阅[批量加载](http://hbase.apache.org/book.html#arch.bulk.load)。


已将示例数据文件上载到公共 Azure Blob 容器 wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt。该数据文件的内容为：

	8396	Calvin Raji		230-555-0191	230-555-0191	5415 San Gabriel Dr.
	16600	Karen Wu		646-555-0113	230-555-0192	9265 La Paz
	4324	Karl Xie		508-555-0163	230-555-0193	4912 La Vuelta
	16891	Jonn Jackson	674-555-0110	230-555-0194	40 Ellis St.
	3273	Miguel Miller	397-555-0155	230-555-0195	6696 Anchor Drive
	3588	Osa Agbonile	592-555-0152	230-555-0196	1873 Lion Circle
	10272	Julia Lee		870-555-0110	230-555-0197	3148 Rose Street
	4868	Jose Hayes		599-555-0171	230-555-0198	793 Crawford Street
	4761	Caleb Alexander	670-555-0141	230-555-0199	4775 Kentucky Dr.
	16443	Terry Chander	998-555-0171	230-555-0200	771 Northridge Drive

如果需要，你可以创建一个文本文件并将该文件上载到你自己的存储帐户。有关说明，请参阅[在 HDInsight 中为 Hadoop 作业上载数据][hdinsight-upload-data]。

> [AZURE.NOTE]此过程使用你在上一个过程中创建的“联系人”HBase 表。

1. 在 RDP 会话中，单击桌面上的“Hadoop 命令行”快捷方式。
2. 更改目录：

		cd %HBASE_HOME%\bin

3. 运行以下命令，将数据文件转换成 StoreFiles 并将其存储在 Dimporttsv.bulk.output 指定的相对路径：

		hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns="HBASE_ROW_KEY,Personal:Name, Personal:Phone, Office:Phone, Office:Address" -Dimporttsv.bulk.output="/example/data/storeDataFileOutput" Contacts wasb://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt

4. 运行以下命令，将数据从 /example/data/storeDataFileOutput 上载到 HBase 表：

		hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /example/data/storeDataFileOutput Contacts

5. 你可以打开 HBase Shell，并使用扫描命令来列出表内容。

## 检查群集状态

HDInsight 中的 HBase 随附了一个 Web UI 用于监视群集。使用该 Web UI 可以请求有关区域的统计或信息。

若要打开该 Web UI，你必须通过 RDP 连接到群集，然后在桌面上单击“HMaster Info Web UI”快捷方式，或者在 Web 浏览器中使用以下 URL：

	http://zookeeper[0-2]:60010/master-status
在高可用性群集中，你将会找到要托管 WebUI 的当前活动 HBase 主节点的链接。



## 使用 Hive 查询 HBase 表

你可以使用 Hive 查询 HBase 表中的数据。本部分将创建要映射到 HBase 表的 Hive 表，并使用该表来查询 HBase 表中的数据。

**打开群集仪表板**

1. 登录到 [Azure 经典管理门户][azure-management-portal]。
2. 单击左窗格中的“HDINSIGHT”。你将会看到群集的列表，包括你在本教程中创建的群集。
3. 单击你要在其中运行 Hive 作业的群集名称。
4. 单击该页底部的“查询控制台”，以打开群集仪表板。它将在不同的浏览器标签中打开网页。
5. 输入 Hadoop 用户帐户用户名和密码。默认用户名是 **admin**，而密码是你在设置过程中输入的密码。此时将打开新浏览器标签。
6. 单击该页顶部的“Hive 编辑器”。Hive 编辑器的外观如下：

	![HDInsight 群集仪表板。][img-hdinsight-hbase-hive-editor]





























**运行 Hive 查询**

1. 在 Hive 编辑器中输入以下 HiveQL 脚本，然后单击“提交”，以创建映射到 HBase 表的 Hive 表。确保你已创建本教程中前面引用的示例表，方法是在运行此语句前使用 HBase shell。

		CREATE EXTERNAL TABLE hbasecontacts(rowkey STRING, name STRING, homephone STRING, officephone STRING, officeaddress STRING)
		STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
		WITH SERDEPROPERTIES ('hbase.columns.mapping' = ':key,Personal:Name,Personal:Phone,Office:Phone,Office:Address')
		TBLPROPERTIES ('hbase.table.name' = 'Contacts');

	等到“状态”更新为“已完成”。

2. 在 Hive 编辑器中输入以下 HiveQL 脚本，然后单击“提交”。Hive 查询会在 HBase 表中查询数据：

     	SELECT count(*) FROM hbasecontacts;

4. 若要检索 Hive 查询的结果，请在作业完成运行时，单击“作业会话”窗口中的“查看详细信息”链接。由于你将一条记录放置在 HBase 表中，因此将只有一个作业输出文件。




**浏览输出文件**

1. 在查询控制台中，单击“文件浏览器”。
2. 单击用作 HBase 群集的默认文件系统的 Azure 存储帐户。
3. 单击 HBase 群集名称。默认 Azure 存储帐户容器使用群集名称。
4. 单击“用户”，然后单击“管理员”。（这是 Hadoop 用户名。）
6. 单击具有与运行 SELECT Hive 查询的时间匹配的“上次修改时间”的作业名称。
4. 单击“stdout”。保存文件，并使用记事本打开该文件。此时将有一个输出文件。

	![HDInsight HBase Hive 编辑器文件浏览器][img-hdinsight-hbase-file-browser]

## 使用.NET HBase REST API 客户端库

你必须从 GitHub 下载适用于 .NET 的 HBase REST API 客户端库并构建项目，以便你可以使用 HBase .NET SDK。以下过程包括有关此任务的说明。

1. 创建新的 C# Visual Studio Windows 桌面控制台应用程序。
2. 打开 NuGet 包管理器控制台，方法是单击“工具”菜单 >“NuGet Package Manager”>“Package Manager Console”。
3. 在控制台中运行以下 NuGet 命令：

		Install-Package Microsoft.HBase.Client

5. 在文件的顶部添加以下 **using** 语句：

		using Microsoft.HBase.Client;
		using org.apache.hadoop.hbase.rest.protobuf.generated;

6. 将 **Main** 函数替换为以下内容：

        static void Main(string[] args)
        {
            string clusterURL = "https://<yourHBaseClusterName>.azurehdinsight.cn";
            string hadoopUsername= "<yourHadoopUsername>";
            string hadoopUserPassword = "<yourHadoopUserPassword>";

            string hbaseTableName = "sampleHbaseTable";

            // Create a new instance of an HBase client.
            ClusterCredentials creds = new ClusterCredentials(new Uri(clusterURL), hadoopUsername, hadoopUserPassword);
            HBaseClient hbaseClient = new HBaseClient(creds);

            // Retrieve the cluster version
            var version = hbaseClient.GetVersion();
            Console.WriteLine("The HBase cluster version is " + version);

            // Create a new HBase table.
            TableSchema testTableSchema = new TableSchema();
            testTableSchema.name = hbaseTableName;
            testTableSchema.columns.Add(new ColumnSchema() { name = "d" });
            testTableSchema.columns.Add(new ColumnSchema() { name = "f" });
            hbaseClient.CreateTable(testTableSchema);

            // Insert data into the HBase table.
            string testKey = "content";
            string testValue = "the force is strong in this column";
            CellSet cellSet = new CellSet();
            CellSet.Row cellSetRow = new CellSet.Row { key = Encoding.UTF8.GetBytes(testKey) };
            cellSet.rows.Add(cellSetRow);

            Cell value = new Cell { column = Encoding.UTF8.GetBytes("d:starwars"), data = Encoding.UTF8.GetBytes(testValue) };
            cellSetRow.values.Add(value);
            hbaseClient.StoreCells(hbaseTableName, cellSet);

            // Retrieve a cell by its key.
            cellSet = hbaseClient.GetCells(hbaseTableName, testKey);
            Console.WriteLine("The data with the key '" + testKey + "' is: " + Encoding.UTF8.GetString(cellSet.rows[0].values[0].data));
            // with the previous insert, it should yield: "the force is strong in this column"

		    //Scan over rows in a table. Assume the table has integer keys and you want data between keys 25 and 35.
		    Scanner scanSettings = new Scanner()
		    {
    		    batch = 10,
    		    startRow = BitConverter.GetBytes(25),
    		    endRow = BitConverter.GetBytes(35)
		    };

		    ScannerInformation scannerInfo = hbaseClient.CreateScanner(hbaseTableName, scanSettings);
		    CellSet next = null;
            Console.WriteLine("Scan results");

            while ((next = hbaseClient.ScannerGetNext(scannerInfo)) != null)
		    {
    		    foreach (CellSet.Row row in next.rows)
    		    {
                    Console.WriteLine(row.key + " : " + Encoding.UTF8.GetString(row.values[0].data));
    		    }
		    }

            Console.WriteLine("Press ENTER to continue ...");
            Console.ReadLine();
        }

7. 在 **Main** 函数中设置前三个变量。
8. 按 **F5** 运行应用程序。

## 删除集群

[AZURE.INCLUDE [delete-cluster-warning](../includes/hdinsight-delete-cluster-warning.md)]

## 后续步骤
在针对 HDInsight 的本 HBase 教程中，你已学习如何设置 HBase 群集，如何创建表，以及如何从 HBase shell 查看这些表中的数据。你还学习了如何对 HBase 表中的数据使用 Hive 查询，以及如何使用 HBase C# REST API 创建 HBase 表并从该表中检索数据。

若要了解更多信息，请参阅以下文章：

- [HDInsight HBase 概述][hdinsight-hbase-overview]：HBase 是构建于 Hadoop 上的 Apache 开源 NoSQL 数据库，用于为大量非结构化和半结构化数据提供随机访问和高度一致性。
- [在 Azure 虚拟网络上设置 HBase 群集][hdinsight-hbase-provision-vnet-v1]：通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。
- [在 HDInsight 中配置 HBase 地域复制](/documentation/articles/hdinsight-hbase-geo-replication/)：了解如何跨两个 Azure 数据中心配置 HBase 复制。


[hdinsight-manage-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hbase-reference]: http://hbase.apache.org/book.html#importtsv
[hbase-schema]: http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf
[hbase-quick-start]: http://hbase.apache.org/book.html#quickstart





[hdinsight-hbase-overview]: /documentation/articles/hdinsight-hbase-overview/
[hdinsight-hbase-provision-vnet-v1]: /documentation/articles/hdinsight-hbase-provision-vnet-v1/
[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning-v1/
[hbase-twitter-sentiment]: /documentation/articles/hdinsight-hbase-analyze-twitter-sentiment/
[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

[img-hdinsight-hbase-cluster-quick-create]: ./media/hdinsight-hbase-tutorial-get-started-v1/hdinsight-hbase-quick-create.png
[img-hdinsight-hbase-hive-editor]: ./media/hdinsight-hbase-tutorial-get-started-v1/hdinsight-hbase-hive-editor.png
[img-hdinsight-hbase-file-browser]: ./media/hdinsight-hbase-tutorial-get-started-v1/hdinsight-hbase-file-browser.png
[img-hbase-shell]: ./media/hdinsight-hbase-tutorial-get-started-v1/hdinsight-hbase-shell.png
[img-hbase-sample-data-tabular]: ./media/hdinsight-hbase-tutorial-get-started-v1/hdinsight-hbase-contacts-tabular.png
[img-hbase-sample-data-bigtable]: ./media/hdinsight-hbase-tutorial-get-started-v1/hdinsight-hbase-contacts-bigtable.png

<!---HONumber=71-->