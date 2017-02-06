<properties
   pageTitle="在 HDInsight 群集上配合使用 Hadoop Pig 和 SSH | Azure"
   description="了解如何使用 SSH 连接到基于 Linux 的 Hadoop 群集，然后使用 Pig 命令以交互方式或以批处理作业形式运行 Pig Latin 语句。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight" 
   ms.date="07/06/2015"
   wacn.date="02/06/2017"/>
 
#使用 Pig 命令 (SSH) 在基于 Linux 的群集上运行 Pig 作业

[AZURE.INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

在本文档中，你将要演练使用安全外壳 (SSH) 连接到基于 Linux 的 Azure HDInsight 群集，然后使用 Pig 命令以交互方式或以批处理作业形式运行 Pig Latin 语句的过程。

可以使用 Pig Latin 编程语言来描述应用到输入数据以生成所需输出的转换。

> [AZURE.NOTE]如果你已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅[基于 Linux 的 HDInsight 提示](/documentation/articles/hdinsight-hadoop-linux-information/)。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Linux 的 HDInsight（HDInsight 上的 Hadoop）群集。

* SSH 客户端。SSH 客户端上应该装有 Linux、Unix 和 Mac OS。Windows 用户必须下载 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 之类的客户端。

##<a id="ssh"></a>使用 SSH 进行连接

使用 SSH 命令连接到 HDInsight 群集的完全限定域名 (FQDN)。FQDN 是你为群集指定的名称后接 **.azurehdinsight.cn**。例如，以下命令将连接到名为 **myhdinsight** 的群集：

	ssh admin@myhdinsight-ssh.azurehdinsight.cn

如果你在创建 HDInsight 群集时**提供了 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置。

	ssh admin@myhdinsight-ssh.azurehdinsight.cn -i ~/mykey.key

如果你在创建 HDInsight 群集时**提供了 SSH 身份验证的密码**，则需要根据提示提供该密码。

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[在 Linux、OS X 和 Unix 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

###PuTTY（基于 Windows 的客户端）

Windows 未提供内置的 SSH 客户端。建议使用可从 [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 下载的 **PuTTY**。

有关使用 PuTTY 的详细信息，请参阅[从 Windows 在 HDInsight 上配合使用 SSH 与基于 Linux 的 Hadoop](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)。

##<a id="pig"></a>使用 Pig 命令

2. 连接后，请使用以下命令启动 Pig 命令行界面 (CLI)。

        pig

	稍后，你应该会看到 `grunt>` 提示符。

3. 输入以下语句。

		LOGS = LOAD 'wasbs:///example/data/sample.log';

	此命令会将 sample.log 文件的内容载入到 LOGS。可以使用以下方法查看该文件的内容。

		DUMP LOGS;

4. 接下来，应用正则表达式以使用以下命令来只提取每条记录的日志记录级别，从而转换数据。

		LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

	转换后，你可以使用 **DUMP** 来查看数据。在这种情况下，使用 `DUMP LEVELS;`。

5. 使用以下语句继续应用转换。使用 `DUMP` 查看每个步骤后的转换结果。

	<table>
<tr>
<th>语句</th><th>作用</th>
</tr>
<tr>
<td>FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;</td><td>删除包含日志级别 Null 值的行，并将结果存储到 FILTEREDLEVELS。</td>
</tr>
<tr>
<td>GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;</td><td>按日志级别对行进行分组，并将结果存储到 GROUPEDLEVELS。</td>
</tr>
<tr>
<td>FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;</td><td>创建一组新的数据，其中包含每个唯一日志级别值和其发生次数。此数据将存储到 FREQUENCIES。</td>
</tr>
<tr>
<td>RESULT = order FREQUENCIES by COUNT desc;</td><td>按计数为日志级别排序（降序），并存储到 RESULT。</td>
</tr>
</table>

6. 你也可以使用 `STORE` 语句保存转换结果。例如，以下语句将 `RESULT` 保存到群集的默认存储容器上的 **/example/data/pigout** 目录。

		STORE RESULT into 'wasbs:///example/data/pigout'

	> [AZURE.NOTE]数据将存储到指定目录中名为 **part-nnnnn** 的文件中。如果该目录已存在，则你会收到错误。

7. 若要退出 grunt 提示符，请输入以下语句。

		QUIT;

###Pig Latin 批处理文件

你也可以使用 Pig 命令运行文件中包含的 Pig Latin。

3. 退出 grunt 提示符之后，请使用以下命令将 STDIN 输送到名为 **pigbatch.pig** 的文件中。将会在用于登录以建立 SSH 会话的帐户的主目录中创建此文件。

		cat > ~/pigbatch.pig

4. 输入或粘贴以下行，然后在完成后按 Ctrl+D。

		LOGS = LOAD 'wasbs:///example/data/sample.log';
		LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
		FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
		GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
		FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
		RESULT = order FREQUENCIES by COUNT desc;
		DUMP RESULT;

5. 使用以下命令，以使用 Pig 命令运行 **pigbatch.pig** 文件。

		pig ~/pigbatch.pig

	批处理作业完成后，你应该会看到以下输出，而输出应该会与先前步骤中使用 `DUMP RESULT;` 时相同。

		(TRACE,816)
		(DEBUG,434)
		(INFO,96)
		(WARN,11)
		(ERROR,6)
		(FATAL,2)

##<a id="summary"></a>摘要

如你所见，Pig 命令可让你使用 Pig Latin 以交互方式运行 MapReduce 作业，以及运行批处理文件中存储的语句。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Pig 的一般信息。

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=71-->