<properties
   pageTitle="在 HDInsight 中使用 Hadoop Pig | Azure"
   description="了解如何通过远程桌面将 Pig 与 HDInsight 上的 Hadoop 配合使用。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="04/03/2015"
    wacn.date="04/15/2015"
    />



# 使用 Pig 命令运行 Pig 作业（远程桌面）

[AZURE.INCLUDE [pig-selector](../includes/hdinsight-selector-use-pig.md)]

本文档将会演练使用 Pig 命令，以交互方式或批处理作业形式在基于 Linux 的 HDInsight 上的 Hadoop 群集中运行 Pig Latin 语句。Pig Latin 可让你通过描述数据转换来创建 MapReduce 应用程序，而不是创建映射和化简函数。

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Windows 的 HDInsight（HDInsight 上的 Hadoop）群集

* Windows 7、8 或 10 客户端

## <a id="connect"></a>使用远程桌面进行连接

为 HDInsight 群集启用远程桌面，然后根据<a href="/documentation/articles/hdinsight-administer-use-management-portal/#rdp" target="_blank">使用 RDP 连接到 HDInsight 群集</a>中的说明与该群集建立连接：

## <a id="pig"></a>使用 Pig 命令

2. 连接后，使用桌面上的图标启动"Hadoop 命令行"。

2. 使用以下命令来启动 Pig 命令行。

		%pig_home%\bin\pig

	你会看到 `grunt>` 提示符。 

3. 输入以下语句。

		LOGS = LOAD 'wasb:///example/data/sample.log';

	此命令会将 sample.log 文件的内容载入到 LOGS。可以使用以下方法查看该文件的内容。

		DUMP LOGS;

4. 接下来，应用正则表达式以使用以下命令来只提取每条记录的日志记录级别，从而转换数据。

		LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

	转换后，你可以使用 **DUMP** 来查看数据。在本例中为 `DUMP LEVELS;`。

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
	<td>FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;</td><td>创建一组新的数据，其中包含每个唯一日志级别值和其发生次数。此数据将存储到 FREQUENCIES</td>
	</tr>
	<tr>
	<td>RESULT = order FREQUENCIES by COUNT desc;</td><td>按计数为日志级别排序（降序），并存储到 RESULT</td>
	</tr>
	</table>

6. 你也可以使用 `STORE` 语句保存转换结果。例如，以下语句会将 `RESULT` 保存到群集的默认存储容器上的 **/example/data/pigout** 目录。

		STORE RESULT into 'wasb:///example/data/pigout'

	> [AZURE.NOTE] 数据将存储到指定目录中名为 **part-nnnnn** 的文件中。如果该目录已存在，则你会收到错误。

7. 若要退出 grunt 提示符，请输入以下语句。

		QUIT;

### Pig Latin 批处理文件

你也可以使用 Pig 命令运行文件中包含的 Pig Latin。

3. 退出 grunt 提示符之后，请打开"记事本"，并在 **%PIG_HOME%** 目录中创建名为 **pigbatch.pig** 的新文件。

4. 键入以下行，或将其粘贴到 **pigbatch.pig** 文件中，然后在完成时保存。

		LOGS = LOAD 'wasb:///example/data/sample.log';
		LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
		FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
		GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
		FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
		RESULT = order FREQUENCIES by COUNT desc;
		DUMP RESULT;

5. 使用以下命令，以使用 pig 命令运行 **pigbatch.pig** 文件。

		pig %PIG_HOME%\pigbatch.pig

	批处理作业完成后，你应该会看到以下输出，而输出应该会与先前步骤中使用 `DUMP RESULT;` 时相同。

		(TRACE,816)
		(DEBUG,434)
		(INFO,96)
		(WARN,11)
		(ERROR,6)
		(FATAL,2)

## <a id="summary"></a>摘要

如你所见，Pig 命令可让你使用 Pig Latin 以交互方式运行 MapReduce 作业，以及运行批处理文件中存储的语句。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Pig 的一般信息。

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce)

<!--HONumber=50-->