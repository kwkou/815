<properties urlDisplayName="Python with HDInsight" pageTitle="在 Azure HDInsight 中将 Python 与 Hive 和 Pig 配合使用" metaKeywords="" description="Learn how to use Python User Defined Functions (UDF) from Hive and Pig in Azure HDInsight." metaCanonical="" services="hdinsight" documentationCenter="" title="Use Python with Hive and Pig in HDInsight" authors="larryfr" solutions="" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="python" ms.topic="article" ms.date="09/17/2014" ms.author="larryfr" />

# 在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用

Hive 和 Pig 非常适合用于处理 HDInsight 中的数据，但有时你需要一种更通用的语言。Hive 和 Pig 都可让你使用各种编程语言创建用户定义的功能 (UDF)。在本文中，你将了解如何从 Hive 和 Pig 使用 Python UDF。

> [WACOM.NOTE] 本文中的步骤适用于 HDInsight 群集版本 3.0 和 3.1。

## 目录

* [HDInsight 上的 Python](#python)
*   [Hive 和 Python](#hivepython)
*   [Pig 和 Python](#pigpython)
* [运行示例](#running)
* [故障排除](#troubleshooting)
* [后续步骤](#next)

## <a name="python"></a>HDInsight 上的 Python

默认情况下，Python2.7 已安装在 HDInsight 3.0 群集上。可以在 D:\Python 文件夹中找到该安装。可以将 Hive 与此版本的 Python 一起使用以进行流式处理（使用 STDOUT/STDIN 在 Hive 和 Python 之间传递数据）。

HDInsight 还包含 Jython，它是使用 Java 编写的 Python 实现。Pig 无需采用流式处理就知道如何与 Jython 通信，因此，使用 Pig 时，Jython 是首选。

### <a name="hivepython"></a>Hive 和 Python

可以通过 HiveQL **TRANSFORM** 语句将 Python 用作 Hive 中的 UDF。例如，以下 HiveQL 将调用 **streaming.py** 文件中存储的 Python 脚本。

	add file wasb:///streaming.py;
	
	SELECT TRANSFORM (clientid, devicemake, devicemodel)
	  USING 'D:\Python27\python.exe streaming.py' AS
	  (clientid string, phoneLable string, phoneHash string)
	FROM hivesampletable
	ORDER BY clientid LIMIT 50;

以下是此示例执行的操作：

1. 文件开头的 **add file** 语句将 **streaming.py** 文件添加到分布式缓存，使群集中的所有节点都可访问该文件。

2. **SELECT TRANSFORM ...USING 'D:\Python27\python.exe streaming.py'** 语句从 **hivesampletable** 中选择数据，然后将 clientid、devicemake 和 devicemodel 传递给 **streaming.py** 脚本。

	> [WACOM.NOTE]**USING** 子句指定 python.exe 的完整路径，因为路径中未包含该文件。

3. **AS** 子句描述从 **streaming.py** 返回的字段


<a name="streamingpy"></a>
以下是该 HiveQL 示例使用的 **streaming.py** 文件。

	import sys
	import string
	import hashlib
	
	while True:
	  line = sys.stdin.readline()
	  if not line:
	    break
	
	  line = string.strip(line, "\n ")
	  clientid, devicemake, devicemodel = string.split(line, "\t")
	  phone_label = devicemake + ' ' + devicemodel
	  print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])

由于我们要使用流式处理，因此此脚本必须执行以下操作：

1. 从 STDIN 中读取数据。这可以通过使用本示例中的以下函数来实现：sys.stdin.readline()。

2. 已使用"string.strip(line, "\n ")"删除尾部的换行符，因为我们只需要文本数据，而不需要行尾指示符。

2. 执行流式处理时，一个行就包含了所有值，每两个值之间有一个制表符。因此，"string.split(line, "\t")"可用于在每个制表符处拆分输入，并只返回字段。

3. 处理完成后，必须将输出以单行形式写入到 STDOUT，并在每两个字段之间提供一个制表符。这可以通过使用以下函数来实现：print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])。

4. 这全都发生在"while"循环中，该循环将不断重复，直到不再读到任何"line"为止，此时"break"退出循环且脚本终止。

除此之外，脚本只会连接"devicemake"和"devicemodel"的输入值，并计算连接值的哈希。这段内容看上去相当简单，不过，它也基本上描述了从 Hive 调用的任何 Python 脚本的工作原理：循环、读取输入直到没有更多的输入、用制表符拆分每个输入行、处理、写入以制表符分隔的单行输出。

有关如何在 HDInsight 群集上运行此示例的信息，请参阅[运行示例](#running)。

### <a name="pigpython"></a>Pig 和 Python

可以通过 **GENERATE** 语句将 Python 脚本用作 Pig 中的 UDF。例如，以下示例使用 **jython.py** 文件中存储的 Python 脚本。

	Register 'wasb:///jython.py' using jython as myfuncs;
    LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
    DUMP DETAILS;

以下是此示例的工作原理：

1. 使用 **Jython** 注册包含 Python 脚本 (**jython.py**) 的文件，并将该脚本作为 **myfuncs** 公开给 Pig。Jython 是以 Java 编写的 Python 实现，可在 Pig 所在的同一台 Java 虚拟机中运行。这样，我们便可以将 Python 脚本视为传统的函数调用，而不是对 Hive 使用的流式处理方法。

2. 下一代码行将示例数据文件 **sample.log** 载入 **LOGS**。由于此日志文件不包含一致的架构，因此它还将每条记录（在本例中为 **LINE**）定义为 **chararray**。简单而言，Chararray 就是一个字符串。

3. 第三个代码行筛选出所有 null 值，并将操作结果存储在 **LOG** 中。

4. 接下来，它将循环访问 **LOG** 中的记录，并使用 **GENERATE** 来调用 **jython.py** 脚本（已作为 **myfuncs** 加载）中包含的 **create_structure** 方法。**LINE** 用于将当前记录传递给函数。

5. 最后，使用 **DUMP** 命令将输出转储到 STDOUT。这只是为了在完成操作后立即显示结果；在实际脚本中，你通常会使用 **STORE** 将数据存储到新文件中。

<a name="jythonpy"></a>
以下是该 Pig 示例使用的 **jython.py** 文件：

	@outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
	def create_structure(input):
	  if (input.startswith('java.lang.Exception')):
	    input = input[21:len(input)] + ' - java.lang.Exception'
	  date, time, classname, level, detail = input.split(' ', 4)
	  return date, time, classname, level, detail

还记得吗？我们前面只是将 **LINE** 输入定义为 chararray，因为输入没有一致的架构。**jython.py** 的任务是将数据转换成用于输出的一致架构。其工作方式如下所述：

1. **@outputSchema** 语句定义要返回到 Pig 的数据的格式。在本例中，该格式为**数据袋**，一种 Pig 数据类型。该数据袋包含以下字段，所有这些字段都是 chararray（字符串）：

	* date - 创建日志条目的日期
	* time - 创建日志条目的时间
	* classname - 为其创建该条目的类名
	* level - 日志级别
	* detail - 日志条目的详细信息

2. 接下来，**def create_structure(input)** 将定义 Pig 要将行项传递到的函数。

3. 示例数据 **sample.log** 基本上符合我们要返回的日期、时间、类名、级别和详细信息架构。但是，它还包含了一些以字符串"*java.lang.Exception*"开头的行，我们需要修改这些行，使之与架构匹配。**if** 语句将检查这些行，然后调整输入数据以将"*java.lang.Exception*"字符串移到末尾，使数据与预期的输出架构相一致。

4. 接下来，使用 **split** 命令在前四个空白字符处拆分数据。这样就会生成五个值，这些值将分配到 **date**、**time**、**classname**、**level** 和 **detail**。

5. 最后，将值返回到 Pig。

当数据返回到 Pig 时，其架构与 **@outputSchema** 语句中的定义一致。

有关如何在 HDInsight 群集上运行此示例的信息，请参阅[运行示例](#running)。

## <a name="running"></a>运行示例

下面的步骤使用了 Windows Azure PowerShell。如果尚未在开发计算机上安装并配置 Azure PowerShell，请在使用以下步骤之前参阅[如何安装和配置 Azure PowerShell](http://www.windowsazure.cn/zh-cn/documentation/articles/install-configure-powershell/)。


1. 使用 Python 示例 [streaming.py](#streamingpy) 和 [jython.py](#jythonpy) 创建开发计算机上的文件的本地副本。

2. 使用以下 PowerShell 脚本将 **streaming.py** 和 **jython.py** 文件上载到服务器。更改 Azure HDInsight 群集的名称，并在脚本的前三行中更改 **streaming.py** 和 **jython.py** 文件的路径。

		$clusterName = YourHDIClusterName
		$pathToStreamingFile = "C:\path\to\streaming.py"
		$pathToJythonFile = "C:\path\to\jython.py"

		$hdiStore = get-azurehdinsightcluster -name $clusterName
		$storageAccountName = $hdiStore.DefaultStorageAccount.StorageAccountName.Split(".",2)[0]
		$storageAccountKey = $hdiStore.defaultstorageaccount.storageaccountkey
		$defaultContainer = $hdiStore.DefaultStorageAccount.StorageContainerName
		
		$destContext = new-azurestoragecontext -storageaccountname $storageAccountName -storageaccountkey $storageAccountKey
		set-azurestorageblobcontent -file $pathToStreamingFile -Container $defaultContainer -Blob "streaming.py" -context $destContext
		set-azurestorageblobcontent -file $pathToJythonFile -Container $defaultContainer -Blob "jython.py" -context $destContext

	此脚本将检索 HDInsight 群集的信息，然后提取默认存储帐户的名称和密钥，并将文件上载到容器的根目录。

	> [WACOM.NOTE][在 HDInsight 中上载 Hadoop 作业的数据](/zh-cn/documentation/articles/hdinsight-upload-data/)文档中介绍了上载脚本的其他方法。

###使用 Hive 仪表板（仅提供 Hive 示例）

1. 上载文件后，请打开一个浏览器并导航到 https://YourClusterName.azurehdinsight.cn/。当系统提示输入凭据时，请输入群集的管理员用户名和密码。

	> [WACOM.NOTE] 也可以在 Azure 管理门户中使用 HDInsight **仪表板**底部的**管理群集**链接来启动 Hive 仪表板。

2. 使用 **Hive 编辑器**，将"select * from hivesampletable"行替换为以下 HiveQL。

		add file wasb:///streaming.py;
		SELECT TRANSFORM (clientid, devicemake, devicemodel)
		  USING 'D:\Python27\python.exe streaming.py' AS
		  (clientid string, phoneLable string, phoneHash string)
		FROM hivesampletable
		ORDER BY clientid LIMIT 50;

3. 单击**提交**按钮以提交作业。根据使用的 HDInsight 群集版本，系统可能会将你重定向到"作业详细信息"页。如果没有，请在页底部的**作业会话**区域中选择**查看详细信息**。

4. 在作业完成之前，**作业详细信息**页将会刷新。完成作业后，将显示有关作业的信息以及输出。

### 使用 PowerShell（Hive 和 Pig 示例）

上载文件后，使用以下 PowerShell 脚本启动作业。完成作业时，会将输出写入到 PowerShell 控制台。

**运行 Hive 作业**
    
    # Replace 'YourHDIClusterName' with the name of your cluster
	$clusterName = YourHDIClusterName

	$HiveQuery = "add file wasb:///streaming.py;" +
	             "SELECT TRANSFORM (clientid, devicemake, devicemodel) " +
	               "USING 'D:\Python27\python.exe streaming.py' AS " +
	               "(clientid string, phoneLable string, phoneHash string) " +
	             "FROM hivesampletable " +
	             "ORDER BY clientid LIMIT 50;"
	
	$jobDefinition = New-AzureHDInsightHiveJobDefinition -Query $HiveQuery -StatusFolder '/hivepython'
	
	$job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition
	Write-Host "Wait for the Hive job to complete ..." -ForegroundColor Green
	Wait-AzureHDInsightJob -Job $job
    # Uncomment the following to see stderr output
    # Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardOutput

**Hive** 作业的输出应该类似于以下内容：

	100041	RIM 9650	d476f3687700442549a83fac4560c51c
	100041	RIM 9650	d476f3687700442549a83fac4560c51c
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9

**运行 Pig 作业**

	# Replace 'YourHDIClusterName' with the name of your cluster
	$clusterName = YourHDIClusterName

	$PigQuery = "Register wasb:///jython.py using jython as myfuncs;" +
	            "LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);" +
	            "LOG = FILTER LOGS by LINE is not null;" +
	            "DETAILS = foreach LOG generate myfuncs.create_structure(LINE);" +
	            "DUMP DETAILS;"
	
	$jobDefinition = New-AzureHDInsightPigJobDefinition -Query $PigQuery -StatusFolder '/pigpython'
	
	$job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition
	Write-Host "Wait for the Pig job to complete ..." -ForegroundColor Green
	Wait-AzureHDInsightJob -Job $job
    # Uncomment the following to see stderr output
    # Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardOutput

**Pig** 作业的输出应该类似于以下内容：

	((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
	((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
	((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
	((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
	((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

## <a name="troubleshooting"></a>故障排除

用于运行示例的两个示例 PowerShell 脚本都包含一个带注释的行，该行将显示作业的错误输出。如果你未看到作业的预期输出，请取消注释以下行，并查看错误信息中是否指明了问题。

	# Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName

错误信息 (STDERR) 和作业的结果 (STDOUT) 还会记录到群集默认 Blob 容器中的以下位置。

<table>
<tr>
<td>对于此作业</td><td>在 Blob 容器中查看这些文件</td>
</tr>
<td>Hive</td><td>/HivePython/stderr</br>/HivePython/stdout</td>
</tr>
<td>Pig</td><td>/PigPython/stderr</br>/PigPython/stdout</td>
</tr>
</table>

## <a name="next"></a>后续步骤

如果需要加载未按默认提供的 Python 模块，请参阅[如何将模块部署到 Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/03/03/how-to-deploy-a-python-module-to-windows-azure-hdinsight.aspx)，以获取有关此操作的示例。

如果你要在 HDInsight 上远程运行作业且不使用 PowerShell，请参阅[如何从 Linux 使用 Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/02/18/how-to-use-hdinsight-from-linux.aspx)，以获取有关使用 Python 通过 WebHCat REST API 运行作业的示例。

