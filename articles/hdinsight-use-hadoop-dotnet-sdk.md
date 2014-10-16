<properties linkid="manage-services-hdinsight-howto-sdk" urlDisplayName="HDInsight SDK" pageTitle="How to use the HDInsight .NET libraries | Azure" metaKeywords="" description="Learn how to get the HDInsight NuGet packages and use them from your .NET application." metaCanonical="" services="hdinsight" documentationCenter="" title="Use the Hadoop .NET SDK with HDInsight" authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

# 将 Hadoop .NET SDK 与 HDInsight 配合使用

Hadoop .NET SDK 提供 .NET 客户端库，可简化从 .NET 中使用 Hadoop 的操作。在本教程中，你将学习如何获取 Hadoop .NET SDK 并使用它来生成一个简单的基于 .NET 的应用程序，该程序使用 Azure HDInsight 服务运行 Hive 查询。给定一个 actors.txt 文件，你将编写应用程序来查找获奖最多的男女演员。

若要启用 HDInsight，请单击[此处][]。

## 本文内容

-   [下载和安装 Hadoop .NET SDK][]
-   [为教程做准备][]
-   [创建应用程序][]
-   [运行应用程序][]
-   [后续步骤][]

## 下载和安装 Hadoop .NET SDK

可以从 [NuGet][] 安装该 SDK 的最新发行版。该 SDK 包括以下组件：

-   **MapReduce 库** - 使用 Hadoop 流接口简化用 .NET 语言编写 MapReduce 作业的过程。

-   **LINQ to Hive 客户端库** - 将 C\# 或 F\# LINQ 查询转换为 HiveQL 查询，并在 Hadoop 群集上执行这些查询。此库还可从 .NET 应用程序执行任意 HiveQL 查询。

-   **WebClient 库** - 包含用于 *WebHDFS* 和 *WebHCat* 的客户端库。

    -   **WebHDFS 客户端库** - 在 HDFS 和 Azure Blog 存储中处理文件。

    -   **WebHCat 客户端库** - 管理 HDInsight 群集中作业的计划和执行。

用于安装库的 NuGet 语法：

    install-package Microsoft.Hadoop.MapReduce
    install-package Microsoft.Hadoop.Hive 
    install-package Microsoft.Hadoop.WebClient 
            

这些命令将库以及引用添加到当前 Visual Studio 项目中。

## 为教程做准备

继续操作之前，你必须先有 [Azure 订阅][]和 [Azure 存储帐户][]。你还必须知道你的 Azure 存储帐户名称和帐户密钥。有关如何获取此信息的说明，请参阅[如何管理存储空间帐户][]中的*如何：查看、复制和重新生成存储访问密钥*一节。

你还必须下载本教程中使用的 Actors.txt 文件。执行以下步骤将此文件下载到你的开发环境中：

1.  在本地计算机上创建 C:\\Tutorials 文件夹。

2.  下载 [Actors.txt][]，并将该文件保存到 C:\\Tutorials 文件夹。

## 创建应用程序

在本节中，你将了解如何以编程方式将文件上载到 Hadoop 群集，以及如何使用 LINQ to Hive 执行 Hive 作业。

1.  打开 Visual Studio 2012。

2.  在“文件”菜单中，单击“新建” ，然后单击“项目” 。

3.  从“新建项目”中，键入或选择以下值：

	<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
	<tr>
	<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">属性</th>
	<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">值</th></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">类别</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">模板/Visual C#/Windows</td></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">模板</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">控制台应用程序</td></tr>
	<tr>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">名称</td>
	<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">SimpleHiveJob</td></tr>
	</table>

4.  单击“确定” 以创建该项目。

5.  从“工具”菜单中 ，单击“库程序包管理器” ，然后单击“程序包管理器控制台” 。

6.  在控制台中运行下列命令以安装程序包。

        install-package Microsoft.Hadoop.Hive 
        install-package Microsoft.Hadoop.WebClient 

    这些命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

7.  从解决方案资源管理器中，双击 **Program.cs** 将其打开。

8.  将下列 using 语句添加到文件顶部：

        using Microsoft.Hadoop.WebHDFS.Adapters;
        using Microsoft.Hadoop.WebHDFS;
        using Microsoft.Hadoop.Hive;

9.  在 Main() 函数中，复制并粘贴以下代码：

        // 将 actors.txt 上载到 Blob 存储
        var asvAccount = [Storage-account-name.blob.core.chinacloudapi.cn];
        var asvKey = [Storage account key];
        var asvContainer = [Container name];
        var localFile = "C:/Tutorials/Actors.txt";
        var hadoopUser = [Hadoop user name]; // The HDInsight cluster user
        var hadoopUserPassword = [Hadoop user password]; // HDInsight 群集用户密码
        var clusterURI = [HDInsight cluster URL]; //"https://HDInsightCluster Name.azurehdinsight.net:563";

        var storageAdapter = new BlobStorageAdapter(asvAccount, asvKey, asvContainer, true);
        var HDFSClient = new WebHDFSClient(hadoopUser, storageAdapter);

        Console.WriteLine("Creating MovieData directory and uploading actors.txt...");
        HDFSClient.CreateDirectory("/user/hadoop/MovieData").Wait();
        HDFSClient.CreateFile(localFile, "/user/hadoop/MovieData/Actors.txt").Wait();

        // 创建 Hive 连接
        var hiveConnection = new HiveConnection(
        new System.Uri(clusterURI),
        hadoopUser, 
        hadoopUserPassword,
        asvAccount, 
        asvKey);

        // 删除任何名为 Actors 的现有表
        // 只在你希望重新运行此应用程序
        // 并删除以前的 Actors 表时需要。
        //hiveConnection.GetTable<HiveRow>("Actors").Drop();

        Console.WriteLine("Creating a Hive table named 'Actors'...");
        string command =
        @"CREATE TABLE Actors(
        MovieId string, 
        ActorId string,
        Name string, 
        AwardsCount int) 
        row format delimited fields 
        terminated by ',';";
        hiveConnection.ExecuteHiveQuery(command).Wait();

        Console.WriteLine("Load data from Actors.txt into the 'Actors' table in Hive...");
        command =
        @"LOAD DATA INPATH 
        '/user/hadoop/MovieData/Actors.txt'
        OVERWRITE INTO TABLE Actors;";
        hiveConnection.ExecuteHiveQuery(command).Wait();

        Console.WriteLine("Performing Hive query...");
        var result = hiveConnection.ExecuteQuery(@"select name, awardscount
        from actors sort by awardscount desc
        limit 1");
        result.Wait();

        Console.WriteLine("The results are:{0}", result.Result.ReadToEnd());
        Console.WriteLine("\nPress any key to continue.");
        Console.ReadKey();

10. 更新应用程序中的常量。Azure HDInsight 服务使用 Azure Blob 存储作为默认文件系统。在 HDInsight 设置过程中，必须将一个 Blob 指定为默认文件系统。你可以选择使用默认文件系统容器，也可以选择使用其他 Blob 存储中的容器。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

    如果你选择使用默认文件系统容器，可通过远程连接到该群集从 *c:\\apps\\dist\\hadoop-1.1.0-SNAPSHOT\\conf\>core-site.xml* 配置文件中找到存储帐户名、存储密钥和容器名。通过搜索 *fs.default.name* 可找到用作默认文件系统的容器；通过搜索 *fs.azure.account.key* 可找到存储帐户名和帐户密钥。

## 运行应用程序

该应用程序在 Visual Studio 中打开时，按 **F5** 键以运行该应用程序。此时将打开一个控制台窗口，其中显示在上载数据、将数据存储到 Hive 表中以及最终查询时，应用程序执行的步骤。该应用程序一运行完，查询结果就已经返回，按任意键可终止该应用程序。

![HDI.HadoopSDKOutput][]

**说明**

该应用程序执行的每一步可能都需要数秒乃至数分钟才能完成。上载 Actors.txt 文件的时间因通过 Internet 连接到 Azure 数据中心的速度而异，其他步骤所需时间取决于群集大小。

**说明**

LOAD DATA INPATH 操作是一个移动操作，它将 Actors.txt 数据移入由 Hive 控制的文件系统命名空间。这可以高效地从上载位置移除 Actors.txt 文件。因此，每次运行此应用程序时都必须上载 Actors.txt 文件。

有关将数据载入 Hive 的详细信息，请参阅 [Hive 入门][]。

## 后续步骤

现在你了解了如何使用 Hadoop .NET SDK 创建 .NET 应用程序。若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][]
-   [Pig 与 HDInsight 配合使用][]
-   [MapReduce 与 HDInsight 配合使用][]
-   [Hive 与 HDInsight 配合使用][]

  [此处]: https://account.windowsazure.com/PreviewFeatures
  [下载和安装 Hadoop .NET SDK]: #install
  [为教程做准备]: #prepare
  [创建应用程序]: #create
  [运行应用程序]: #run
  [后续步骤]: #nextsteps
  [NuGet]: http://nuget.codeplex.com/wikipage?title=Getting%20Started
  [Azure 订阅]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [Azure 存储帐户]: http://www.windowsazure.cn/zh-cn/manage/services/storage/how-to-create-a-storage-account/
  [如何管理存储空间帐户]: /en-us/manage/services/storage/how-to-manage-a-storage-account/
  [Actors.txt]: http://www.microsoft.com/en-us/download/details.aspx?id=37003
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store/
  [HDI.HadoopSDKOutput]: ./media/hdinsight-use-hadoop-dotnet-sdk/HDI.HadoopSDKOutput.PNG "控制台应用程序"
  [Hive 入门]: https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DMLOperations
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [MapReduce 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-mapreduce-with-hdinsight/
  [Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
