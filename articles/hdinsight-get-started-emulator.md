<properties linkid="manage-services-hdinsight-get-started-hdinsight" urlDisplayName="Get Started" pageTitle="Get started with the HDInsight Emulator | Azure" metaKeywords="hdinsight, Azure hdinsight, hdinsight azure, get started hdinsight, emulator, hdinsight emulator" description="Learn how to use HDInsight Emulator for Azure." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" title="Get started with the HDInsight Emulator" authors="jgao" />

# HDInsight Emulator 入门

本指南指导你开始使用 Microsoft HDInsight Emulator for Azure（以前称作 HDInsight Server 开发者预览版）。HDInsight Emulator 附带来自 Hadoop 生态系统的与 Azure HDInsight 相同的组件。有关详细信息（包括与部署的版本有关的信息），请参阅 [Azure HDInsight 包含哪个版本的 Hadoop？][]。

HDInsight Emulator 提供了针对 Azure HDInsight 的本地开发环境。如果你对 Hadoop 比较熟悉，则可以开始通过 HDFS 使用 HDInsight Emulator。但在 HDInsight 中，默认文件系统是 Azure Blob 存储（WASB，即 Azure 存储空间 - Blob），因此，最终你将需要使用 WASB 开发你的作业。你可以通过使用 Azure 存储模拟器针对 WASB 开始开发 – 可能只需要使用你的少量数据（在 HDInsight Emulator 中无需更改配置，只是存储帐户名称不同）。然后，再次针对 Windows Azure 存储空间在本地测试你的作业 – 还是只使用你的少量数据（要求在 HDInsight Emulator 中进行配置更改）。最后，你准备好将你的作业的计算部分移到 HDInsight 并且针对生产数据运行作业。

> [WACOM.NOTE] HDInsight Emulator 只能使用单节点部署。

有关使用 HDInsight 的教程，请参阅[开始使用 Azure HDInsight][]。

**先决条件**
在开始本教程之前，你必须具备以下先决条件：

-   HDInsight Emulator 需要 64 位版本的 Windows。必须满足以下要求之一：

    -   Windows 7 Service Pack 1
    -   Windows Server 2008 R2 Service Pack1
    -   Windows 8
    -   Windows Server 2012。
-   安装和配置 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]。

## 在本教程中

-   [安装 HDInsight Emulator][]
-   [运行单词计数示例][]
-   [运行入门示例][]
-   [连接到 Azure Blob 存储][]
-   [运行 HDInsight PowerShell][]
-   [后续步骤][]

## 安装 HDInsight Emulator

Microsoft HDInsight Emulator 可通过 Microsoft Web 平台安装程序进行安装。

> [WACOM.NOTE] HDInsight Emulator 目前只支持英语 OS。

> [WACOM.NOTE] 如果你已经安装了 Microsoft HDInsight 开发者预览版，则必须首先从“控制面板/程序和功能”中卸载以下两个组件。
>
> -   HDInsight 开发者预览版
> -   Hortonworks Data Platform 开发者预览版

**安装 HDInsight Emulator**

1.  打开 Internet Explorer，然后浏览到 [Microsoft HDInsight Emulator for Azure 安装页][]。
2.  单击“立即安装” 。
3.  在页面底部提示安装 HDINSIGHT.exe 时单击“运行” 。
4.  在弹出的“用户帐户控制” 窗口中单击“是” 按钮，以便完成安装。你应该看到 Web 平台安装程序 4.6 窗口。
5.  单击页面底部的“安装” 。
6.  单击“我接受” 同意许可条款。
7.  确认 Web 平台安装程序显示“已成功安装以下产品” ，然后单击“完成” 。
8.  单击“退出” 以关闭 Web 平台安装程序 4.6 窗口。

    该安装应该已经在你的桌面上安装了三个图标。这三个图标按如下所示进行链接：

    -   **Hadoop 命令行**：在 HDInsight Emulator 中从其运行 MapReduce、Pig 和 Hive 作业的 Hadoop 命令提示符。

    -   **Hadoop 名称节点状态**：NameNode 对于 HDFS 中的所有文件维持基于树的目录。它还跟踪 Hadoop 群集中所有文件的数据位置。客户端与 NameNode 进行通信，以便确定将所有文件的数据节点存储于何处。

    -   **Hadoop MapReduce 状态**：将 MapReduce 任务分配给群集中的节点的作业跟踪器。

    该安装应该还安装了若干本地服务。下面是“服务”窗口的屏幕快照：

    ![HDI.Emulator.Services][]

    有关安装和运行 HDInsight Server 的已知问题，请参阅 [HDInsight Emulator 发行说明][]。安装日志位于 **C:\\HadoopFeaturePackSetup\\HadoopFeaturePackSetupTools\\gettingStarted.winpkg.install.log**。

## 运行单词计数 MapReduce 作业

现在你已在工作站上配置了 HDInsight Emulator。你可以运行 MapReduce 作业以便测试安装。你将首先将某些文本文件上载到 HDFS，然后运行单词计数 MapReduce 作业以便计算该单词在这些文件中出现的频率。

单词计数 MapReduce 程序已打包到 *hadoop-examples.jar* 中。jar 文件位于 *C:\\Hadoop\\hadoop-1.1.0-SNAPSHOT* 文件夹中。

jar 命令的语法是：

    hadoop jar <jar> [mainClass] args...

你还将使用一些 fs 命令。有关 Hadoop 命令的详细信息，请参阅 [Hadoop 命令参考（可能为英文页面）][]。

单词计数 MapReduce 作业采用两个参数：输入文件夹和输出文件夹。你将使用 *hdfs://localhost/user/HDIUser* 作为输入文件夹，使用 *hdfs://localhost/user/HDIUser/WordCount\_Output* 作为输出目录。输出文件夹不能是现有文件夹，否则 MapReduce 作业将失败。如果要第二次运行 MapReduce 作业，则必须指定一个不同的输出文件夹，或删除现有的输出文件夹。

**运行单词计数 MapReduce 作业**

1.  从桌面双击“Hadoop 命令行” 以打开 Hadoop 命令行窗口。当前文件夹应为：

        c:\Hadoop\hadoop-1.1.0-SNAPSHOT>

    如果不是，请运行以下命令：

        cd %hadoop_home%

2.  运行以下 Hadoop 命令以使 HDFS 文件夹对输入和输出文件进行排序：

        hadoop fs -mkdir /user/HDIUser

3.  运行以下 Hadoop 命令以便将某些本地文件复制到 HDFS：

        hadoop fs -copyFromLocal *.txt /user/HDIUser/

4.  运行以下命令以便列出 /user/HDIUser 文件夹中的文件：

        hadoop fs -ls /user/HDIUser

    你应该看到以下文件：

        c:\Hadoop\hadoop-1.1.0-SNAPSHOT>hadoop fs -ls /user/HDIUser
        找到 8 项
        -rw-r--r--   1 username supergroup      16372 2013-10-30 12:07 /user/HDIUser/CHANGES.branch-1-win.txt
        -rw-r--r--   1 username supergroup     463978 2013-10-30 12:07 /user/HDIUser/CHANGES.txt
        -rw-r--r--   1 username supergroup       6631 2013-10-30 12:07 /user/HDIUser/Jira-Analysis.txt
        -rw-r--r--   1 username supergroup      13610 2013-10-30 12:07 /user/HDIUser/LICENSE.txt
        -rw-r--r--   1 username supergroup       1663 2013-10-30 12:07 /user/HDIUser/Monarch-CHANGES.txt
        -rw-r--r--   1 username supergroup        103 2013-10-30 12:07 /user/HDIUser/NOTICE.txt
        -rw-r--r--   1 username supergroup       2295 2013-10-30 12:07 /user/HDIUser/README.Monarch.txt
        -rw-r--r--   1 username supergroup       1397 2013-10-30 12:07 /user/HDIUser/README.txt

5.  运行以下命令来运行单词计数 MapReduce 作业：

        hadoop jar hadoop-examples.jar wordcount /user/HDIUser/*.txt /user/HDIUser/WordCount_Output

6.  运行以下命令以便从输出文件中列出其中含有“windows”的单词：

        hadoop fs -cat /user/HDIUser/WordCount_Output/part-r-00000 | findstr "windows"

    输出应该是：

        c:\Hadoop\hadoop-1.1.0-SNAPSHOT>hadoop fs -cat /user/HDIUser/WordCount_Output/pa
        rt-r-00000 | findstr "windows"
        windows 12
        windows+java6.  1
        windows.        3

## 运行入门示例

HDInsight Emulator 安装提供了一些示例，以便新用户能够快速地在 Windows 上开始学习基于 Apache Hadoop 的服务。这些示例涉及在处理大型数据集时通常需要的一些任务。通过执行这些示例，你可以熟悉与 MapReduce 编程模型及其生态系统相关联的概念。

这些示例是围绕处理 IIS W3C 日志数据方案进行组织的。提供数据生成工具以便创建不同大小的数据集并将这些数据集导入到 HDFS 或 WASB（Azure Blob 存储）中。有关详细信息，请参阅[将 Azure Blob 存储用于 HDInsight][]。然后，可以在 PowerShell 脚本生成的数据页上运行 MapReduce、Pig 或 Hive 作业。请注意，使用的 Pig 和 Hive 脚本都编译到 MapReduce 程序。用户可以运行一系列作业，以便亲自观察使用这些不同技术的影响以及数据大小对执行这些处理任务的影响。

### 本节内容

-   [IIS w3c 日志数据方案][]
-   [加载示例 w3c 日志数据][]
-   [运行 Java MapReduce 作业][]
-   [运行 Hive 作业][]
-   [运行 Pig 作业][]
-   [重新生成示例][]

### IIS w3c 日志数据方案

w3c 方案生成以下三种大小的 IIS W3C 日志数据并且将这些数据导入到 HDFS 或 WASB 中：1MB、500MB 和 2GB。它提供三种作业类型并且分别在 C\#、Java、Pig 和 Hive 中实现它们。

-   **totalhits**：计算针对某一给定页的请求总数
-   **avgtime**：计算每页某一请求所用的平均时间（单位为秒）
-   **errors**：计算对于其状态为 404 或 500 的请求，每页、每小时的错误数

这些示例及其文档并未提供针对关键 Hadoop 技术的深入研究或完整实现。使用的群集只具有单个节点，因此对于此版本，无法观察添加多个节点的影响。

### 加载示例 W3c 日志数据

使用 PowerShell 脚本 importdata.ps1 实现生成数据并且将数据导入到 HDFS。

**导入示例 w3c 日志数据：**

1.  从桌面打开 Hadoop 命令行。
2.  运行以下命令以便将目录更改为 **C:\\Hadoop\\GettingStarted**：

        cd \Hadoop\GettingStarted

3.  运行以下命令以生成数据并且将数据导入到 HDFS：

        powershell -File importdata.ps1 w3c -ExecutionPolicy unrestricted 

    如果要改为将数据导入到 WASB，请参阅[连接到 Azure Blob 存储][]。

4.  从 Hadoop 命令行运行以下命令以便列出 HDFS 上导入的文件：

        hadoop fs -lsr /w3c

    输出应如下所示：

        c:\Hadoop\GettingStarted\w3c>hadoop fs -lsr /w3c
        drwxr-xr-x   - username supergroup          0 2013-10-30 13:29 /w3c/input
        drwxr-xr-x   - username supergroup          0 2013-10-30 13:29 /w3c/input/large
        -rw-r--r--   1 username supergroup  543692369 2013-10-30 13:29 /w3c/input/large/data_w3c_large.txt
        drwxr-xr-x   - username supergroup          0 2013-10-30 13:28 /w3c/input/medium
        -rw-r--r--   1 username supergroup  272394671 2013-10-30 13:28 /w3c/input/medium/data_w3c_medium.txt
        drwxr-xr-x   - username supergroup          0 2013-10-30 13:28 /w3c/input/small
        -rw-r--r--   1 username supergroup    1058328 2013-10-30 13:28 /w3c/input/small/data_w3c_small.txt

5.  运行下列命令以便将数据文件之一显示到控制台窗口：

        hadoop fs -cat /w3c/input/small/data_w3c_small.txt

现在你已创建数据文件并已将其导入到 HDFS。你可以运行其他 Hadoop 作业。

### 运行 Java MapReduce 作业

MapReduce 是针对 Hadoop 的基本计算引擎。默认情况下，它是在 Java 中实现的，但也有利用采用 C\# 的 .NET 和 Hadoop Streaming 的示例。运行 MapReduce 作业的语法是：

    hadoop jar <jarFileName>.jar <className> <inputFiles> <outputFolder>

jar 文件和源文件位于 C:\\Hadoop\\GettingStarted\\Java 文件夹中。

**运行 MapReduce 作业以便计算网页点击数**

1.  打开 Hadoop 命令行。
2.  运行以下命令以便将目录更改为 **C:\\Hadoop\\GettingStarted**：

        cd \Hadoop\GettingStarted

3.  运行以下命令以便在该文件夹存在时删除输出目录。如果输出文件夹已存在，该 MapReduce 作业将失败。

        hadoop fs -rmr /w3c/output

4.  运行以下命令：

        hadoop jar .\Java\w3c_scenarios.jar "microsoft.hadoop.w3c.TotalHitsForPage" "/w3c/input/small/data_w3c_small.txt" "/w3c/output"

    下表介绍了该命令的元素：

    参数

    说明

    w3c\_scenarios.jar

    jar 文件位于 C:\\Hadoop\\GettingStarted\\Java 文件夹中。

    microsoft.hadoop.w3c.TotalHitsForPage

    可使用以下项之一替代该类型：

    -   microsoft.hadoop.w3c.AverageTimeTaken
    -   microsoft.hadoop.w3c.ErrorsByPage

    /w3c/input/small/data\_w3c\_small.txt

    可使用以下项之一替代该输入文件：

    -   /w3c/input/medium/data\_w3c\_medium.txt
    -   /w3c/input/large/data\_w3c\_large.txt

    /w3c/output

    这是输出文件夹名称。

5.  运行以下命令以显示输出文件：

        hadoop fs -cat /w3c/output/part-00000

    输出应如下所示：

        c:\Hadoop\GettingStarted\Java>hadoop fs -cat /w3c/output/part-00000
        /Default.aspx   3409
        /Info.aspx      1115
        /UserService    1130

    因此，Default.aspx 页得到了 3409 次点击，依此类推。

### 运行 Hive 作业

精通 SQL 的分析人士对于 Hive 查询引擎将会感到很熟悉。它提供与 SQL 类似的界面以及用于 HDFS 的关系数据模型。Hive 使用一种称作 HiveQL（或 HQL）的语言，这是一种 SQL 行话。

**运行 Hive 作业**

1.  打开 Hadoop 命令行。
2.  将目录更改到 **C:\\Hadoop\\GettingStarted** 文件夹
3.  运行以下命令以便在 **/w3c/hive/input** 文件夹存在时删除该文件夹。如果该文件夹存在，则 hive 作业将失败。

        hadoop fs -rmr /w3c/hive/input

4.  运行以下命令以便创建 **/w3c/hive/input** 文件夹，并且将数据文件从工作站复制到 HDFS：

        hadoop fs -mkdir /w3c/hive/input
        hadoop fs -cp /w3c/input/small/data_w3c_small.txt /w3c/hive/input

5.  运行以下命令以执行 **w3ccreate.hql** 脚本文件。该脚本将创建一个 Hive 表，并且将数据导入到该 Hive 表中：

        C:\Hadoop\hive-0.9.0\bin\hive.cmd -f ./Hive/w3c/w3ccreate.hql -hiveconf "input=/w3c/hive/input/data_w3c_small.txt"

    该 HiveQL 脚本是：

        DROP TABLE w3c;

        CREATE TABLE w3c(
        logdate string,
        logtime string,
        c_ip string,
        cs_username string,
        s_ip string,
        s_port string,
        cs_method string,
        cs_uri_stem string,
        cs_uri_query string,
        sc_status int,
        sc_bytes int,
        cs_bytes int,
        time_taken int,
        cs_agent string, 
        cs_Referrer string)
        ROW FORMAT delimited
        FIELDS TERMINATED BY ' ';

        LOAD DATA INPATH '${hiveconf:input}' OVERWRITE INTO TABLE w3c;

    输出将如下所示：

        c:\Hadoop\GettingStarted>C:\Hadoop\hive-0.9.0\bin\hive.cmd -f ./Hive/w3c/w3ccrea    te.hql -hiveconf "input=/w3c/hive/input/data_w3c_small.txt"
        Hive history file=c:\hadoop\hive-0.9.0\logs\history/hive_job_log_username_201310311452_1053491002.txt
        Logging initialized using configuration in file:/C:/Hadoop/hive-0.9.0/conf/hive-log4j.properties
        OK
        Time taken:0.616 seconds
        OK
        Time taken:0.139 seconds
        Loading data to table default.w3c
        Moved to trash:hdfs://localhost:8020/apps/hive/warehouse/w3c
        OK
        Time taken:0.573 seconds

6.  运行以下命令以便运行 **w3ctotalhitsbypate.hql** HiveQL 脚本文件。

        C:\Hadoop\hive-0.9.0\bin\hive.cmd -f ./Hive/w3c/w3ctotalhitsbypage.hql

    下表介绍了该命令的元素：

	<table border="1"><br /><tr><td>文件</td><td>说明</td></tr><br /><tr><td>C:\Hadoop\hive-0.9.0\bin\hive.cmd</td><td>Hive 命令脚本。</td></tr><br /><tr><td>C:\Hadoop\GettingStarted\Hive\w3c\w3ctotalhitsbypage.hql</td><td> 你可以使用以下文件之一替代该 Hive 脚本文件：<br /><ul><br /><li>C:\Hadoop\GettingStarted\Hive\w3c\w3caveragetimetaken.hql</li><br /><li>C:\Hadoop\GettingStarted\Hive\w3c\w3cerrorsbypage.hql</li><br /></ul><br /></td></tr></p>
	</table>

    w3ctotalhitsbypage.hql HiveQL 脚本是：

        SELECT filtered.cs_uri_stem,COUNT(*) 
        FROM (
        SELECT logdate,cs_uri_stem from w3c WHERE logdate NOT RLIKE '.*#.*'
        ) filtered
        GROUP BY (filtered.cs_uri_stem);

    输出的末尾将如下所示：

        MapReduce Total cumulative CPU time:3 seconds 47 msec
        Ended Job = job_201310291309_0006
        MapReduce Jobs Launched:
        Job 0:Map:1  Reduce:1   Cumulative CPU:3.047 sec   HDFS Read:1058546 HDFS W
        rite:53 SUCCESS
        Total MapReduce CPU Time Spent:3 seconds 47 msec
        OK
        /Default.aspx   3409
        /Info.aspx      1115
        /UserService    1130
        Time taken:34.68 seconds

请注意，作为每个作业的第一步，将创建一个表并将之前创建的文件中的数据加载到该表中。你可以通过使用以下命令查看 HDFS 中的 /Hive 节点，浏览已创建的文件：

    hadoop fs -lsr /apps/hive/

### 运行 Pig 作业

Pig 处理使用称作 *Pig Latin* 的数据流语言。Pig Latin 抽象提供了比 MapReduce 更丰富的数据结构，并为 Hadoop 执行 SQL 对 RDBMS 系统执行的操作。

**运行 pig 作业：**

1.  打开 Hadoop 命令行。
2.  将目录更改到 C:\\Hadoop\\GettingStarted 文件夹。
3.  运行以下命令来提交 Pig 作业：

        C:\Hadoop\pig-0.9.3-SNAPSHOT\bin\pig.cmd -f ".\Pig\w3c\TotalHitsForPage.pig" -p "input=/w3c/input/small/data_w3c_small.txt"

    下表显示了该命令的元素：

	<table border="1"><br /><tr><td>文件</td><td>说明</td></tr><br /><tr><td>C:\Hadoop\pig-0.9.3-SNAPSHOT\bin\pig.cmd</td><td>Pig 命令脚本。</td></tr><br /><tr><td>C:\Hadoop\GettingStarted\Pig\w3c\TotalHitsForPage.pig</td><td> 你可以使用以下文件之一替代该 Pig Latin 脚本文件：<br /><ul><br /><li>C:\Hadoop\GettingStarted\Pig\w3c\AverageTimeTaken.pig</li><br /><li>C:\Hadoop\GettingStarted\Pig\w3c\ErrorsByPage.pig</li><br /></ul><br /></td></tr><br /><tr><td>/w3c/input/small/data_w3c_small.txt</td><td> 你可以使用更大的文件替换该参数：</p>
	<ul>
	<li>/w3c/input/medium/data_w3c_medium.txt</li>
	<li>/w3c/input/large/data_w3c_large.txt</li>
	</ul>
	
	
	</td>
	</tr>
	
	</table>

    输出应如下所示：

        (/Info.aspx,1115)
        (/UserService,1130)
        (/Default.aspx,3409)

请注意，因为 Pig 脚本编译到 MapReduce 作业，并且可能会编译到多个此类作业，所以，用户在处理 Pig 作业的过程中可能会看到多个 MapReduce 作业正在执行。

### 重新生成示例

这些示例目前包含所有必需的二进制文件，因此无需生成。如果你想要对 Java 或 .NET 示例进行更改，则可以使用 msbuild 或随附的 PowerShell 脚本重新生成它们。

**重新生成示例**

1.  打开 Hadoop 命令行。
2.  运行以下命令：

        powershell -F buildsamples.ps1

## 连接到 Azure Blob 存储

Azure HDInsight 将 Azure Blob 存储用作默认文件系统。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][将 Azure Blob 存储用于 HDInsight]。

可以在 HDInsight Emulator 中配置本地群集，以便使用 Azure Blob 存储而不是本地存储。本节涉及：

-   连接到存储模拟器
-   连接到 Azure Blob 存储
-   将 Azure Blob 存储配置为针对 HDInsight Emulator 的默认文件系统

### 连接到存储模拟器

[Azure SDK for .NET][] 随附 Azure 存储模拟器。存储模拟器不会自动启动。你必须手动启动它。应用程序名称为“Azure 存储模拟器”**。若要启动/停止模拟器，请在 Windows 系统任务栏中右键单击蓝色的 Azure 图标，然后单击“显示存储模拟器 UI”。

> [WACOM.NOTE] 在启动存储模拟器时，你可能会看到以下错误消息：

>     进程无法访问该文件，因为它正在被另一个进程使用。

> 这是因为 Hadoop Hive 服务之一也使用端口 10000。若要解决该问题，请使用以下过程：

> 1.  使用 services.msc 停止两个 Hadoop Hive 服务：Apache Hadoop hiveserver 和 Apache Hadoop Hiveserver2。
> 2.  启动 Blob 存储模拟器。
> 3.  重新启动两个 Hadoop Hive 服务。

用于访问存储模拟器的语法是：

    wasb://<ContainerName>@storageemulator

例如：

    hadoop fs -ls wasb://myContainer@storageemulator

> [WACOM.NOTE] 如果你看到以下错误消息：

>     ls:No FileSystem for scheme:wasb

> 这是因为你仍在使用开发者预览版。请按照本文中“安装 HDInsight Emulator”一节中的说明卸载该开发者预览版，然后重新安装应用程序。

### 连接到 Azure Blob 存储

有关创建存储帐户的说明，请参阅[如何创建存储帐户][]。

**创建容器**

1.  登录到[管理门户][]。
2.  单击左侧的“存储” 。你应该在你的订阅下看到存储帐户的列表。
3.  单击要从该列表创建容器的存储帐户。
4.  单击页面顶部的“容器” 。
5.  单击页面底部的“添加” 。
6.  输入**名称**并选择“访问” 。你可以使用三种访问级别中的任何一种。默认级别为“私有” 。
7.  单击“确定” 以保存更改。你将看到在门户上列出的新容器。

必须先将帐户名称和帐户密钥添加到配置文件，然后才能访问 Azure 存储帐户。

**配置与 Azure 存储帐户的连接**

1.  在记事本中打开 **C:\\Hadoop\\hadoop-1.1.0-SNAPSHOT\\conf\\core-site.xml**。
2.  将以下 \<property\> 标记添加到其他 \<property\> 标记旁：

        <property>
        <name>fs.azure.account.key.<StorageAccountName>.blob.core.chinacloudapi.cn</name>
        <value><StorageAccountKey></value>
        </property>

    你必须使用与你的存储帐户信息匹配的值替代 \<StorageAccountName\> 和 \<StorageAccountKey\>。

3.  保存更改。你无需重新启动 Hadoop 服务。

使用以下语法可访问该存储帐户：

    wasb://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/

例如：

    hadoop fs -ls wasb://myContainer@myStorage.blob.core.chinacloudapi.cn/

### 使用 Azure Blob 存储容器作为默认文件系统

还可以使用 Azure Blob 存储容器作为默认文件系统，就像在 Azure HDInsight 中一样。

**使用 Azure Blob 存储容器配置默认文件系统**

1.  在记事本中打开 **C:\\Hadoop\\hadoop-1.1.0-SNAPSHOT\\conf\\core-site.xml**。
2.  找到以下 \<property\> 标记：

        <property>
        <name>fs.default.name</name>
        <!-- 群集变量 -->
        <value>hdfs://localhost:8020</value>
        <description>默认文件系统的名称。文字字符串“local”或采用“主机:端口”形式（用于 NDFS）。</description>
        <final>true</final>
        </property>

3.  用以下两个 \<property\> 标记替换它：

        <property>
        <name>fs.default.name</name>
        <!-- 群集变量 -->
        <!--<value>hdfs://localhost:8020</value>-->
        <value>wasb://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn</value>
        <description>默认文件系统的名称。文字字符串“local”或采用“主机:端口”形式（用于 NDFS）。</description>
        <final>true</final>
        </property>

        <property>
        <name>dfs.namenode.rpc-address</name>
        <value>hdfs://localhost:8020</value>
        <description>其他临时目录的基址。</description>
        </property>

    你必须使用与你的存储帐户信息匹配的值替代 \<StorageAccountName\> 和 \<StorageAccountKey\>。

4.  保存更改。
5.  在你的桌面上以提升的模式打开 Hadoop 命令行（以管理员身份运行）
6.  运行以下命令以便重新启动 Hadoop 服务：

        C:\Hadoop\stop-onebox.cmd
        C:\Hadoop\start-onebox.cmd

7.  运行以下命令以便测试与默认文件系统的连接:

        hadoop fs -ls /

    以下命令将列出相同文件夹中的内容：

        hadoop fs -ls wasb:///
        hadoop fs -ls wasb://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/
        hadoop fs -ls wasbs://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/

    若要访问 HDFS，请使用以下命令：

        hadoop fs -ls hdfs://localhost:8020/

## 运行 HDInsight PowerShell

HDInsight Emulator 支持某些 HDInsight PowerShell cmdlet。这些 cmdlet 包括：

-   HDInsight 作业定义 cmdlet

    -   New-AzureHDInsightSqoopJobDefinition
    -   New-AzureHDInsightStreamingMapReduceJobDefinition
    -   New-AzureHDInsightPigJobDefinition
    -   New-AzureHDInsightHiveJobDefinition
    -   New-AzureHDInsightMapReduceJobDefinition
-   Start-AzureHDInsightJob
-   Get-AzureHDInsightJob
-   Wait-AzureHDInsightJob

下面是用于提交 Hadoop 作业的示例：

    $creds = Get-Credential (hadoop as username, password can be anything)
    $hdinsightJob = <JobDefinition>
    Start-AzureHDInsightJob -Cluster http://localhost:50111 -Credential $creds -JobDefinition $hdinsightJob

在调用 Get-Credential 时系统将会向你显示一个提示。你必须将 **hadoop** 用作用户名。密码可以是任意字符串。群集名称始终是 **<http://localhost:50111>**。

有关提交 Hadoop 作业的详细信息，请参阅[以编程方式提交 Hadoop 作业][]。有关 HDInsight PowerShell cmdlet 的详细信息，请参阅 [HDInsight cmdlet 参考][]。

## 后续步骤

在本教程中，你安装了 HDInsight Emulator，并且运行了一些 Hadoop 作业。若要了解更多信息，请参阅下列文章：

-   [开始使用 Azure HDInsight][]
-   [为 HDInsight 开发 Java MapReduce 程序][]
-   [为 HDInsight 开发 C\# Hadoop 流 MapReduce 程序][]
-   [HDInsight Emulator 发行说明][]
-   [用于讨论 HDInsight 的 MSDN 论坛][]

  [Azure HDInsight 包含哪个版本的 Hadoop？]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-component-versioning/ "HDInsight 组件和版本"
  [开始使用 Azure HDInsight]: ../hdinsight-get-started/
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  [安装 HDInsight Emulator]: #install
  [运行单词计数示例]: #runwordcount
  [运行入门示例]: #rungetstartedsamples
  [连接到 Azure Blob 存储]: #blobstorage
  [运行 HDInsight PowerShell]: #powershell
  [后续步骤]: #nextsteps
  [Microsoft HDInsight Emulator for Azure 安装页]: http://www.microsoft.com/web/gallery/install.aspx?appid=HDINSIGHT
  [HDI.Emulator.Services]: ./media/hdinsight-get-started-emulator/HDI.Emulator.Services.png
  [HDInsight Emulator 发行说明]: ../hdinsight-emulator-release-notes/
  [Hadoop 命令参考（可能为英文页面）]: http://hadoop.apache.org/docs/r1.1.1/commands_manual.html
  [将 Azure Blob 存储用于 HDInsight]: ../howto-blob-store/
  [IIS w3c 日志数据方案]: #scenarios
  [加载示例 w3c 日志数据]: #loaddata
  [运行 Java MapReduce 作业]: #javamapreduce
  [运行 Hive 作业]: #hive
  [运行 Pig 作业]: #pig
  [重新生成示例]: #rebuild
  [Azure SDK for .NET]: http://www.windowsazure.cn/zh-cn/downloads/
  [如何创建存储帐户]: ../storage-create-storage-account/
  [管理门户]: https://manage.windowsazure.cn/
  [以编程方式提交 Hadoop 作业]: ../hdinsight-submit-hadoop-jobs-programmatically/
  [HDInsight cmdlet 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dn479228.aspx
  [为 HDInsight 开发 Java MapReduce 程序]: ../hdinsight-develop-deploy-java-mapreduce/
  [为 HDInsight 开发 C\# Hadoop 流 MapReduce 程序]: ../hdinsight-hadoop-develop-deploy-streaming-jobs/
  [用于讨论 HDInsight 的 MSDN 论坛]: http://social.msdn.microsoft.com/Forums/windowsazure/zh-cn/home?forum=windowsazurezhchs&filter=alltypes&brandIgnore=True&sort=relevancedesc&filter=alltypes&searchTerm=hdinsight
