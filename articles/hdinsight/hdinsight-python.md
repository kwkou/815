<properties
	pageTitle="在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用 | Azure"
	description="了解如何在 HDInsight（Azure 上的 Hadoop 技术堆栈）中通过 Hive 和 Pig 使用 Python 用户定义的函数 (UDF)。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="jhubbard"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="python"
	ms.topic="article"
	ms.date="09/07/2016"
	wacn.date="12/12/2016" 
	ms.author="larryfr"/>

#在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用

Hive 和 Pig 非常适用于在 HDInsight 中处理数据，但有时需要使用更通用的语言。Hive 和 Pig 都允许使用多种编程语言创建用户定义的功能 (UDF)。在本文中，你将了解如何通过 Hive 和 Pig 使用 Python UDF。

##要求

* HDInsight 群集（基于 Windows）

* 文本编辑器
 
##<a name="python"></a>HDInsight 上的 Python

默认情况下，Python2.7 安装在 HDInsight 3.0 和更高版本的群集上。可以将 Hive 与此版本的 Python 配合使用，以进行流式处理（使用 STDOUT/STDIN 在 Hive 和 Python 之间传递数据）。

HDInsight 还包含 Jython，后者是用 Java 编写的 Python 实现。Pig 无需采用流式处理即可知道如何与 Jython 通信，因此，在使用 Pig 时，Jython 是首选。但是，也可以将普通 Python (C Python) 与 Pig 配合使用。

##<a name="hivepython"></a>Hive 和 Python

可以通过 HiveQL **TRANSFORM** 语句将 Python 用作 Hive 中的 UDF。例如，以下 HiveQL 将调用 **streaming.py** 文件中存储的 Python 脚本。

**基于 Windows 的 HDInsight**

	add file wasbs:///streaming.py;

	SELECT TRANSFORM (clientid, devicemake, devicemodel)
	  USING 'D:\Python27\python.exe streaming.py' AS
	  (clientid string, phoneLable string, phoneHash string)
	FROM hivesampletable
	ORDER BY clientid LIMIT 50;

> [AZURE.NOTE] 在基于 Windows 的 HDInsight 群集上，**USING** 子句必须指定 python.exe 的完整路径。该路径始终为 `D:\Python27\python.exe`。

本示例执行以下操作：

1. 文件开头的 **add file** 语句将 **streaming.py** 文件添加到分布式缓存，使群集中的所有节点都可访问该文件。

2. **SELECT TRANSFORM ...USING** 语句从 **hivesampletable** 中选择数据，并将 clientid、devicemake 和 devicemodel 传递到 **streaming.py** 脚本。

3. **AS** 子句描述从 **streaming.py** 返回的字段

<a name="streamingpy"></a>下面是该 HiveQL 示例使用的 **streaming.py** 文件。

	#!/usr/bin/env python

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

由于使用流式处理，因此此脚本必须执行以下操作：

1. 从 STDIN 中读取数据。在本示例中，使用 `sys.stdin.readline()` 完成此操作。

2. 已使用 `string.strip(line, "\n ")` 删除尾部的换行符，因为我们只需要文本数据，而不需要行尾指示符。

2. 执行流式处理时，一个行就包含了所有值，每两个值之间有一个制表符。因此，`string.split(line, "\t")` 可用于在每个制表符处拆分输入，并只返回字段。

3. 在处理完成后，必须将输出以单行形式写入到 STDOUT，并在每两个字段之间提供一个制表符。可使用 `print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])` 来实现此操作。

4. 这全都发生在 `while` 循环中，该循环将不断重复，直到不再读到任何 `line` 为止，此时 `break` 退出循环且脚本终止。

除此之外，脚本只会连接 `devicemake` 和 `devicemodel` 的输入值，并计算连接值的哈希。此示例很简单，但它说明了从 Hive 调用的任何 Python 脚本工作原理的基本知识：循环，读取输入直到没有更多的输入，用制表符拆分每个输入行，处理，写入以制表符分隔的单行输出。

有关如何在 HDInsight 群集上运行此示例的信息，请参阅[运行示例](#running)。

##<a name="pigpython"></a>Pig 和 Python

在整个 **GENERATE** 语句中，Python 脚本可用作 Pig 中的 UDF。可通过两种方法实现此目的：使用 Jython（Java 虚拟机中实现的 Python）和 C Python（普通 Python）。

这两种脚本的主要差别在于，Jython 在 JVM 上运行，并且原本就能从 Pig（也在 JVM 上运行）调用。 C Python 是外部进程（用 C 语言编写）。 因此，JVM 上的 Pig 中的数据将发送到 Python 进程中运行的脚本，然后，脚本的输出将发回到 Pig。

若要确定 Pig 是使用 Jython 还是 C Python 来运行脚本，请在从 Pig Latin 引用 Python 脚本时使用 __register__。这样就会告知 Pig 要使用哪个解释器，以及要为脚本创建哪个别名。以下示例将脚本作为 __myfuncs__ 注册到 Pig：

* __使用 Jython__：`register '/path/to/pig_python.py' using jython as myfuncs;`
* __使用 C Python__：`register '/path/to/pig_python.py' using streaming_python as myfuncs;`

> [AZURE.IMPORTANT] 使用 Jython 时，pig\_jython 文件的路径可以是本地路径或 WASB:// 路径。但是，使用 C Python 时，必须引用用于提交 Pig 作业的节点的本地文件系统上的文件。

通过注册后，此示例的 Pig Latin 对于两个脚本是相同的：

    LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
    DUMP DETAILS;

本示例执行以下操作：

1. 第一行代码将示例数据文件 **sample.log** 加载到 **LOGS** 中。由于此日志文件不包含一致的架构，因此它还将每条记录（在本例中为 **LINE**）定义为 **chararray**。简单而言，Chararray 就是一个字符串。

2. 第二行代码筛选出所有 null 值，并将操作结果存储在 **LOG** 中。

3. 接下来，它将循环访问 **LOG** 中的记录，并使用 **GENERATE** 来调用作为 **myfuncs** 加载的 Python/Jython 脚本中包含的 **create\_structure** 方法。**LINE** 用于将当前记录传递给函数。

4. 最后，使用 **DUMP** 命令将输出转储到 STDOUT。这只是为了在完成操作后立即显示结果；在实际脚本中，你通常会使用 **STORE** 将数据存储到新文件中。

实际的 Python 脚本文件在 C Python 与 Jython 之间也很类似，唯一的差别在于，使用 C Python 时必须从 __pig\_util__ 导入。下面是 __pig\_python.py__ 脚本：

# <a name="streamingpy"></a>如果使用 C Python，请取消注释以下代码
    #from pig_util import outputSchema

    @outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
    def create_structure(input):
    if (input.startswith('java.lang.Exception')):
        input = input[21:len(input)] + ' - java.lang.Exception'
    date, time, classname, level, detail = input.split(' ', 4)
    return date, time, classname, level, detail

> [AZURE.NOTE] 安装时不必要考虑“pig\_util”，脚本会自动使用该选项。

还记得吗？我们前面只是将 **LINE** 输入定义为 chararray，因为输入没有一致的架构。 Python 脚本的任务是将数据转换成用于输出的一致架构。其工作方式如下所述：

1. **@outputSchema** 语句定义要返回到 Pig 的数据的格式。在本例中，该格式为**数据袋**，这是一种 Pig 数据类型。该数据袋包含以下字段，所有这些字段都是 chararray（字符串）：

	* date - 创建日志条目的日期
	* time - 创建日志条目的时间
	* classname - 为其创建该条目的类名
	* level - 日志级别
	* detail - 日志条目的详细信息

2. 接下来，**def create\_structure(input)** 将定义 Pig 要将行项传递到的函数。

3. 示例数据 **sample.log** 基本上符合我们要返回的日期、时间、类名、级别和详细信息架构。但是，它还包含了一些以字符串“*java.lang.Exception*”开头的行，我们需要修改这些行，使之与架构匹配。**if** 语句将检查这些行，然后调整输入数据以将“*java.lang.Exception*”字符串移到末尾，使数据与预期的输出架构相一致。

4. 接下来，使用 **split** 命令在前四个空格字符处拆分数据。这将生成五个值，分别分配给 **date**、**time**、**classname**、**level** 和 **detail**。

5. 最后，将值返回到 Pig。

当数据返回到 Pig 时，其架构将与 **@outputSchema** 语句中的定义一致。

##<a name="running"></a>运行示例

如果使用的是基于 Windows 的 HDInsight 群集和 Windows 客户端，请使用 **PowerShell** 步骤。

####Hive

1. 使用 `hive` 命令来启动 Hive Shell。加载 Shell 后，应可看到 `hive>` 提示符。

2. 在 `hive>` 提示符下输入以下命令。

		add file wasbs:///streaming.py;
		SELECT TRANSFORM (clientid, devicemake, devicemodel)
		  USING 'python streaming.py' AS
		  (clientid string, phoneLabel string, phoneHash string)
		FROM hivesampletable
		ORDER BY clientid LIMIT 50;

3. 输入最后一行后，应会启动作业。最终，它将返回类似于以下内容的输出。

		100041	RIM 9650	d476f3687700442549a83fac4560c51c
		100041	RIM 9650	d476f3687700442549a83fac4560c51c
		100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
		100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
		100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9

####Pig

1. 使用 `pig` 命令来启动该 shell。加载 Shell 后，应可看到 `grunt>` 提示符。

2. 在 `grunt>` 提示符下输入以下语句，使用 Jython 解释器运行 Python 脚本。

		Register wasbs:///pig_python.py using jython as myfuncs;
	    LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
	    LOG = FILTER LOGS by LINE is not null;
	    DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
	    DUMP DETAILS;

3. 输入以下行后，应会启动作业。最终，它将返回类似于以下内容的输出。

		((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
		((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
		((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
		((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
		((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

4. 使用 `quit` 退出 Grunt Shell，然后在本地文件系统上使用以下命令编辑 pig\_python.py 文件：

    nano pig\_python.py

5. 进入编辑器后，删除行开头的 `#` 字符以取消注释以下行：

        #from pig_util import outputSchema

    完成更改后，使用 Ctrl+X 退出编辑器。选择“Y”，然后按 Enter 保存更改。

6. 使用 `pig` 命令再次启动 shell。在 `grunt>` 提示符下，使用以下命令运行带有 Jython 解释器的 Python 脚本。

        Register 'pig_python.py' using streaming_python as myfuncs;
	    LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
	    LOG = FILTER LOGS by LINE is not null;
	    DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
	    DUMP DETAILS;

    完成此作业后，看到的输出应该与之前使用 Jython 运行脚本后的输出相同。

###PowerShell

这些步骤使用 Azure PowerShell。如果尚未在开发计算机上安装并配置 Azure PowerShell，请在使用以下步骤之前，参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

1. 使用 Python 示例 [streaming.py](#streamingpy) 和 pig_python.py 在开发计算机上创建文件的本地副本。

2. 使用以下 PowerShell 脚本将 **streaming.py** 和 **pig\_python.py** 文件上传到服务器。在脚本的前三行中，替换 Azure HDInsight 群集的名称，以及 **streaming.py** 和 **pig\_python.py** 文件的路径。

		$clusterName = YourHDIClusterName
		$pathToStreamingFile = "C:\path\to\streaming.py"
		$pathToJythonFile = "C:\path\to\pig_python.py"

		$hdiStore = get-azurehdinsightcluster -name $clusterName
		$storageAccountName = $hdiStore.DefaultStorageAccount.StorageAccountName.Split(".",2)[0]
		$storageAccountKey = $hdiStore.defaultstorageaccount.storageaccountkey
		$defaultContainer = $hdiStore.DefaultStorageAccount.StorageContainerName

		$destContext = new-azurestoragecontext -storageaccountname $storageAccountName -storageaccountkey $storageAccountKey
		set-azurestorageblobcontent -file $pathToStreamingFile -Container $defaultContainer -Blob "streaming.py" -context $destContext
		set-azurestorageblobcontent -file $pathToJythonFile -Container $defaultContainer -Blob "jython.py" -context $destContext

	此脚本将检索 HDInsight 群集的信息，然后提取默认存储帐户的名称和密钥，并将文件上传到容器的根目录。

	> [AZURE.NOTE] [在 HDInsight 中上传 Hadoop 作业的数据](/documentation/articles/hdinsight-upload-data/)文档中介绍了上传脚本的其他方法。

上传文件后，使用以下 PowerShell 脚本启动作业。完成作业时，会将输出写入到 PowerShell 控制台。

####Hive

    # Replace 'YourHDIClusterName' with the name of your cluster
	$clusterName = YourHDIClusterName
    # If using a Windows-based HDInsight cluster, change the USING statement to:
    # "USING 'D:\Python27\python.exe streaming.py' AS " +
	$HiveQuery = "add file wasbs:///streaming.py;" +
	             "SELECT TRANSFORM (clientid, devicemake, devicemodel) " +
	               "USING 'python streaming.py' AS " +
	               "(clientid string, phoneLabel string, phoneHash string) " +
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

**Hive** 作业的输出应该如下所示：

	100041	RIM 9650	d476f3687700442549a83fac4560c51c
	100041	RIM 9650	d476f3687700442549a83fac4560c51c
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9

####Pig (Jython)
> [AZURE.NOTE] 使用 PowerShell 远程提交作业时，无法使用 C Python 作为解释器。

	# Replace 'YourHDIClusterName' with the name of your cluster
	$clusterName = YourHDIClusterName
	$PigQuery = "Register wasbs:///jython.py using jython as myfuncs;" +
	            "LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);" +
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

**Pig** 作业的输出应该如下所示：

	((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
	((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
	((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
	((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
	((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

##<a name="troubleshooting"></a>故障排除

###运行作业时出现错误

运行 hive 作业时，可能遇到如下错误：

    Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.
    
此问题可能是由 streaming.py 文件中的行尾结束符号导致的。许多 Windows 编辑器默认为使用 CRLF 作为行尾结束符号，但 Linux 应用程序通常应使用 LF。

如果所用编辑器无法创建 LF 行尾结束符号，或者不确定要使用什么行尾结束符号，在将文件上传到 HDInsight 之前，请使用以下 PowerShell 语句删除 CR 字符：

    $original_file ='c:\path\to\streaming.py'
    $text = [IO.File]::ReadAllText($original_file) -replace "`r`n", "`n"
    [IO.File]::WriteAllText($original_file, $text)

###PowerShell 脚本

用于运行示例的两个示例 PowerShell 脚本都包含一个带注释的行，该行将显示作业的错误输出。如果你未看到作业的预期输出，请取消注释以下行，并查看错误信息中是否指明了问题。

	# Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName

错误信息 (STDERR) 和作业的结果 (STDOUT) 还会记录到群集默认 Blob 容器中的以下位置。

对于此作业...|在 Blob 容器中查看这些文件
---|---
Hive|/HivePython/stderr<p>/HivePython/stdout
Pig|/PigPython/stderr<p>/PigPython/stdout

##<a name="next"></a>后续步骤

如果需要加载默认情况下未提供的 Python 模块，请参阅[如何将模块部署到 Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/03/03/how-to-deploy-a-python-module-to-windows-azure-hdinsight.aspx)，以获取有关如何执行此操作的示例。

若要了解使用 Pig、Hive 的其他方式以及如何使用 MapReduce，请参阅以下内容。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_Quality_Review_1118_2016-->