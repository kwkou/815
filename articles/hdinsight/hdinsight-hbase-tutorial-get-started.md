<properties
    pageTitle="HBase 教程：Hadoop 中基于 Windows 的 HBase 群集入门 | Azure"
    description="遵循本 HBase 教程开始在 HDInsight 中将 Apache HBase 与 Hadoop 配合使用。从 HBase shell 创建表，然后使用 Hive 查询这些表。"
    keywords="apache hbase,hbase,hbase shell,hbase 教程"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="fb56d00d-daf2-4a11-abd0-4005a663dd75"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/06/2017"
    wacn.date="03/10/2017"
    ms.author="jgao" />

# HBase 教程：开始在 HDInsight 中将 Apache HBase 与基于 Windows 的 Hadoop 配合使用

了解如何使用 Apache Hive 在 HDInsight 中创建 HBase 群集、创建 HBase 表和查询表。有关 HBase 的一般信息，请参阅 [HDInsight HBase 概述][hdinsight-hbase-overview]。

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。此文档中的信息特定于基于 Windows 的 HDInsight 群集。有关基于 Linux 的群集的信息，请参阅 [HBase 教程：在 HDInsight 中使用 Apache HBase 入门](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)。

## 开始之前
[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

要阅读本 HBase 教程，必须具备：

* **Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* 装有 Visual Studio 2013 或更高版本的**工作站**：有关说明，请参阅[安装 Visual Studio](http://msdn.microsoft.com/zh-cn/library/e2h7fzkw.aspx)。

### 访问控制要求
[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## <a name="create-hbase-cluster"></a>创建 HBase 群集
[AZURE.INCLUDE [provisioningnote](../../includes/hdinsight-provisioning.md)]

**使用 Azure 门户预览创建 HBase 群集**

1. 登录 [Azure 门户预览][azure-management-portal]。
2. 单击**新建**或左上角的 **+**，然后单击**数据 + 分析**、**“HDInsight”**。
3. 输入以下值：
   
    * **群集名称** - 输入用于标识此群集的名称。
    * **群集类型** - 选择 **HBase**。
    * **群集操作系统** - 选择 **Windows**。对于创建基于 Linux 的 HBase 群集，请参阅 [HBase 教程：开始在 HDInsight 中将 Apache HBase 与 Hadoop 配合使用 (Linux)](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)。
    * **版本** - 选择 HBase 版本。
    * **订阅** - 选择用于创建此群集的 Azure 订阅。
    * **资源组** - 创建新 Azure 资源组或选择现有资源组。有关详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)
    * **凭据** - 对于基于 Windows 的群集，可以创建群集用户（又称为 HTTP 用户、HTTP Web 服务用户）和远程桌面用户。单击“启用远程桌面”以添加远程桌面用户凭据。下一部分需要 RDP。
    * **数据源** - 创建新 Azure 存储帐户，或选择现有 Azure 存储空间帐户以用作群集的默认文件系统。默认存储帐户位置可确定群集位置的位置。默认存储帐户和群集必须并存于相同的数据中心。
    * **节点定价层** - 为 HBase 群集选择区域服务器数
     
        > [AZURE.WARNING]
        为了实现 HBase 服务的高可用性，必须创建至少包含**三个**节点的群集。这可确保即使一个节点发生故障，HBase 数据区域也可在其他节点上使用。
        ><p> 
        > 如果正在学习 HBase，请始终选择 1 作为群集大小，并在每次使用后删除该群集以节省成本。
        > 
        > 
    * **可选配置** - 配置 Azure 虚拟网络，配置脚本操作，以及添加其他存储帐户。
4. 单击**创建**。

> [AZURE.NOTE]
在删除 HBase 群集后，可以通过使用相同的默认存储帐户和默认 Blob 容器创建另一个 HBase 群集。新群集将选取你在原始群集中创建的 HBase 表。为了避免不一致，建议你在删除群集之前先禁用 HBase 表。
> 
> 

## <a name="create-tables-and-insert-data"></a> 创建表和插入数据
目前，可通过两种方式访问 HBase。本部分介绍如何使用 HBase shell。下一部分介绍如何使用 .NET SDK。

对于大多数人而言，数据以表格形式显示：

![HDInsight hbase 表格数据][img-hbase-sample-data-tabular]

在 HBase（BigTable 的一种实现）中，相同的数据看起来类似于：

![HDInsight hbase bigtable 数据][img-hbase-sample-data-bigtable]

在完成下一过程后，数据将更易于理解。

**使用 HBase shell**

1. 使用 RDP 连接到 HDInsight 中的 HBase 群集。有关 RDP 的说明，请参阅[使用 Azure 门户预览管理 HDInsight 中的 Hadoop 群集][hdinsight-manage-portal]。
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
   
    ![HDInsight hadoop hbase shell][img-hbase-shell]
6. 获取单个行
   
        get 'Contacts', '1000'
   
    将出现与使用扫描命令相同的结果，因为只有一个行。
   
    有关 Hbase 表架构的详细信息，请参阅 [HBase 架构设计简介][hbase-schema]。有关 HBase 命令的详细信息，请参阅 [Apache HBase 参考指南][hbase-quick-start]。
7. 退出 shell
   
        exit

**在联系人 HBase 表中批量加载数据**

HBase 提供了多种将数据载入表中的方法。有关详细信息，请参阅[批量加载](http://hbase.apache.org/book.html#arch.bulk.load)。

已将示例数据文件上载到公共 Azure Blob 容器 wasbs://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt。该数据文件的内容为：

    8396    Calvin Raji        230-555-0191    230-555-0191    5415 San Gabriel Dr.
    16600    Karen Wu        646-555-0113    230-555-0192    9265 La Paz
    4324    Karl Xie        508-555-0163    230-555-0193    4912 La Vuelta
    16891    Jonn Jackson    674-555-0110    230-555-0194    40 Ellis St.
    3273    Miguel Miller    397-555-0155    230-555-0195    6696 Anchor Drive
    3588    Osa Agbonile    592-555-0152    230-555-0196    1873 Lion Circle
    10272    Julia Lee        870-555-0110    230-555-0197    3148 Rose Street
    4868    Jose Hayes        599-555-0171    230-555-0198    793 Crawford Street
    4761    Caleb Alexander    670-555-0141    230-555-0199    4775 Kentucky Dr.
    16443    Terry Chander    998-555-0171    230-555-0200    771 Northridge Drive

如果需要，你可以创建一个文本文件并将该文件上载到自己的存储帐户。有关说明，请参阅[在 HDInsight 中为 Hadoop 作业上载数据][hdinsight-upload-data]。

> [AZURE.NOTE]
此过程使用你在上一个过程中创建的“联系人”HBase 表。
> 
> 

1. 在 RDP 会话中，单击桌面上的“Hadoop 命令行”快捷方式。
2. 更改目录：
   
        cd %HBASE_HOME%\bin
3. 运行以下命令，将数据文件转换成 StoreFiles 并将其存储在 Dimporttsv.bulk.output 指定的相对路径：
   
        hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns="HBASE_ROW_KEY,Personal:Name, Personal:Phone, Office:Phone, Office:Address" -Dimporttsv.bulk.output="/example/data/storeDataFileOutput" Contacts wasbs://hbasecontacts@hditutorialdata.blob.core.windows.net/contacts.txt
4. 运行以下命令，将数据从 /example/data/storeDataFileOutput 上载到 HBase 表：
   
        hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /example/data/storeDataFileOutput Contacts
5. 可以打开 HBase Shell，并使用扫描命令来列出表内容。

## 使用 Hive 查询 HBase 表
可以使用 Hive 查询存储在 HBase 中的数据。本部分将创建要映射到 HBase 表的 Hive 表，并使用该表来查询 HBase 表中的数据。

**打开群集仪表板**

1. 浏览到 **https://\<HDInsight 群集名称>.azurehdinsight.cn/**。
2. 输入 Hadoop 用户帐户的用户名和密码。默认用户名是 **admin**，而密码是在创建过程中输入的密码。此时将打开新浏览器标签。
3. 单击该页顶部的“Hive 编辑器”。Hive 编辑器的外观如下：
   
    ![HDInsight 群集仪表板。][img-hdinsight-hbase-hive-editor]

**运行 Hive 查询**

1. 在 Hive 编辑器中输入以下 HiveQL 脚本，然后单击“提交”，以创建映射到 HBase 表的 Hive 表。确保已创建本教程中前面引用的示例表，方法是在运行此语句前使用 HBase shell。
   
        CREATE EXTERNAL TABLE hbasecontacts(rowkey STRING, name STRING, homephone STRING, officephone STRING, officeaddress STRING)
        STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
        WITH SERDEPROPERTIES ('hbase.columns.mapping' = ':key,Personal:Name,Personal:Phone,Office:Phone,Office:Address')
        TBLPROPERTIES ('hbase.table.name' = 'Contacts');
   
    等到“状态”更新为“已完成”。
2. 在 Hive 编辑器中输入以下 HiveQL 脚本，然后单击“提交”。Hive 查询会在 HBase 表中查询数据：
   
         SELECT count(*) FROM hbasecontacts;
3. 要检索 Hive 查询的结果，请在作业运行完成时单击“作业会话”窗口中的“查看详细信息”链接。由于你在 HBase 表中放入了一条记录，因此将只有一个作业输出文件。

**浏览输出文件**

1. 在查询控制台中，单击“文件浏览器”。
2. 单击用作 HBase 群集的默认文件系统的 Azure 存储帐户。
3. 单击 HBase 群集名称。默认 Azure 存储帐户容器将使用群集名称。
4. 单击“用户”，然后单击“Admin”。（这是 Hadoop 用户名。）
5. 单击“上次修改时间”与运行 SELECT Hive 查询的时间匹配的作业名称。
6. 单击“stdout”。保存文件，并使用记事本打开该文件。将会有一个输出文件。
   
    ![HDInsight HBase Hive 编辑器文件浏览器][img-hdinsight-hbase-file-browser]

## 使用 .NET HBase REST API 客户端库
必须从 GitHub 下载适用于 .NET 的 HBase REST API 客户端库并构建项目，以便于使用 HBase .NET SDK。以下过程包括有关此任务的说明。

1. 创建新 C# Visual Studio Windows 桌面控制台应用程序。
2. 通过单击“工具”>“NuGet 包管理器”>“包管理器控制台”，打开 NuGet 包管理器控制台。
3. 在控制台中运行以下 NuGet 命令：
   
        Install-Package Microsoft.HBase.Client
4. 在文件的顶部添加以下 **using** 语句：
   
        using Microsoft.HBase.Client;
        using org.apache.hadoop.hbase.rest.protobuf.generated;
5. 将 **Main** 函数替换为以下内容：
   
        static void Main(string[] args)
        {
            string clusterURL = "https://<yourHBaseClusterName>.azurehdinsight.cn";
            string hadoopUsername= "<yourHadoopUsername>";
            string hadoopUserPassword = "<yourHadoopUserPassword>";
   
            string hbaseTableName = "sampleHbaseTable";
   
            // Create a new instance of an HBase client.
            ClusterCredentials creds = new ClusterCredentials(new Uri(clusterURL), hadoopUsername, hadoopUserPassword);
            HBaseClient hbaseClient = new HBaseClient(creds);
   
            // Retrieve the cluster version.
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
6. 在 **Main** 函数中设置前三个变量。
7. 按 **F5** 运行应用程序。

## 检查群集状态
HDInsight 中的 HBase 随附了一个 Web UI 用于监视群集。使用该 Web UI 可以请求有关区域的统计或信息。

要打开该 Web UI，必须通过 RDP 连接到群集，然后在桌面上单击“HMaster Info Web UI”快捷方式，或者在 Web 浏览器中使用以下 URL：

    http://zookeeper[0-2]: master-status

在高可用性群集中，会找到要托管 WebUI 的当前活动 HBase 主节点的链接。

## 删除群集
为了避免不一致，建议你在删除群集之前先禁用 HBase 表。[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## 后续步骤
在针对 HDInsight 的本 HBase 教程中，已学习了如何创建 HBase 群集、如何创建表以及如何从 HBase shell 查看这些表中的数据。还学习了如何对 HBase 表中的数据使用 Hive 查询，以及如何使用 HBase C# REST API 创建 HBase 表并从该表中检索数据。

有关详细信息，请参阅：

* [HDInsight HBase 概述][hdinsight-hbase-overview]。HBase 是构建于 Hadoop 上的 Apache 开源 NoSQL 数据库，用于为大量非结构化和半结构化数据提供随机访问和高度一致性。
* [在 Azure 虚拟网络上创建 HBase 群集][hdinsight-hbase-provision-vnet]。通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。
* [在 HDInsight 中配置 HBase 复制](/documentation/articles/hdinsight-hbase-replication/)。了解如何跨两个 Azure 数据中心配置 HBase 复制。

[hdinsight-manage-portal]: /documentation/articles/hdinsight-administer-use-management-portal/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hbase-reference]: http://hbase.apache.org/book.html#importtsv
[hbase-schema]: http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf
[hbase-quick-start]: http://hbase.apache.org/book.html#quickstart

[hdinsight-hbase-overview]: /documentation/articles/hdinsight-hbase-overview/
[hdinsight-hbase-provision-vnet]: /documentation/articles/hdinsight-hbase-provision-vnet/
[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/
[azure-management-portal]: https://portal.azure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

[img-hdinsight-hbase-cluster-quick-create]: ./media/hdinsight-hbase-tutorial-get-started/hdinsight-hbase-quick-create.png
[img-hdinsight-hbase-hive-editor]: ./media/hdinsight-hbase-tutorial-get-started/hdinsight-hbase-hive-editor.png
[img-hdinsight-hbase-file-browser]: ./media/hdinsight-hbase-tutorial-get-started/hdinsight-hbase-file-browser.png
[img-hbase-shell]: ./media/hdinsight-hbase-tutorial-get-started/hdinsight-hbase-shell.png
[img-hbase-sample-data-tabular]: ./media/hdinsight-hbase-tutorial-get-started/hdinsight-hbase-contacts-tabular.png
[img-hbase-sample-data-bigtable]: ./media/hdinsight-hbase-tutorial-get-started/hdinsight-hbase-contacts-bigtable.png

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->