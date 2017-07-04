<properties
    pageTitle="在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用 | Azure"
    description="了解如何在 HDInsight（Azure 上的 Hadoop 技术堆栈）中通过 Hive 和 Pig 使用 Python 用户定义的函数 (UDF)。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="c44d6606-28cd-429b-b535-235e8f34a664"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="02/27/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.custom="H1Hack27Feb2017,hdinsightactive"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="9b0881301e378c8aafcc443168a39e8a18fe6636"
    ms.lasthandoff="04/28/2017" />

# <a name="use-python-user-defined-functions-udf-with-hive-and-pig-in-hdinsight"></a>在 HDInsight 中通过 Hive 和 Pig 使用 Python 用户定义的函数 (UDF)

Hive 和 Pig 非常适用于在 HDInsight 中处理数据，但有时需要使用更通用的语言。 Hive 和 Pig 都允许使用多种编程语言创建用户定义的功能 (UDF)。 本文介绍如何通过 Hive 和 Pig 使用 Python UDF。

## <a name="requirements"></a>要求

* HDInsight 群集

    [AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [AZURE.IMPORTANT]
    > Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* 文本编辑器

## <a name="python"></a>HDInsight 上的 Python

默认情况下，Python2.7 安装在 HDInsight 3.0 和更高版本的群集上。 可以将 Hive 与此版本的 Python 配合使用，以进行流式处理（使用 STDOUT/STDIN 在 Hive 和 Python 之间传递数据）。

HDInsight 还包含 Jython，后者是用 Java 编写的 Python 实现。 Pig 无需采用流式处理即可知道如何与 Jython 通信，因此，在使用 Pig 时，Jython 是首选。 也可以将普通 Python (C Python) 与 Pig 配合使用。

## <a name="hivepython"></a>Hive 和 Python

可以通过 HiveQL **TRANSFORM** 语句将 Python 用作 Hive 中的 UDF。 例如，以下 HiveQL 将调用 **streaming.py** 文件中存储的 Python 脚本。

**基于 Linux 的 HDInsight**

    add file wasbs:///streaming.py;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
        USING 'python streaming.py' AS
        (clientid string, phoneLable string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;

**基于 Windows 的 HDInsight**

    add file wasbs:///streaming.py;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
        USING 'D:\Python27\python.exe streaming.py' AS
        (clientid string, phoneLable string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;

> [AZURE.NOTE]
> 在基于 Windows 的 HDInsight 群集上， **USING** 子句必须指定 python.exe 的完整路径。

下面是本示例执行的操作：

1. 文件开头的 **add file** 语句将 **streaming.py** 文件添加到分布式缓存，使群集中的所有节点都可访问该文件。
2. **SELECT TRANSFORM ...USING** 语句从 **hivesampletable** 中选择数据。 它还将 clientid、devicemake 和 devicemodel 值传递到 **streaming.py** 脚本。
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

此脚本可执行以下操作：

1. 从 STDIN 读取一行数据。
2. 尾随的换行符使用 `string.strip(line, "\n ")`删除。
3. 执行流式处理时，一个行就包含了所有值，每两个值之间有一个制表符。 因此， `string.split(line, "\t")` 可用于在每个制表符处拆分输入，并只返回字段。
4. 在处理完成后，必须将输出以单行形式写入到 STDOUT，并在每两个字段之间提供一个制表符。 可使用 `print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])` 来实现此操作。
5. `while` 循环一直重复到没有 `line` 可以读取。

脚本输出是 `devicemake` 和 `devicemodel` 的输入值的连接，并且是连接值的哈希。

有关如何在 HDInsight 群集上运行此示例的信息，请参阅[运行示例](#running)。

## <a name="pigpython"></a>Pig 和 Python

在整个 **GENERATE** 语句中，Python 脚本可用作 Pig 中的 UDF。 可以使用 Jython 或 C Python 运行此脚本。

这两种脚本的差别在于，Jython 在 JVM 上运行，并且原本就能从 Pig 调用。 C Python 是外部进程，因此 JVM 上的 Pig 中的数据将发送到 Python 进程中运行的脚本。 Python 脚本的输出将发回到 Pig。

若要确定 Pig 是使用 Jython 还是 C Python 来运行脚本，请在从 Pig Latin 引用 Python 脚本时使用 **register** 。 以下示例将脚本作为 **myfuncs** 注册到 Pig：

* **使用 Jython**：`register '/path/to/pig_python.py' using jython as myfuncs;`
* **使用 C Python**：`register '/path/to/pig_python.py' using streaming_python as myfuncs;`

> [AZURE.IMPORTANT]
> 使用 Jython 时，pig_jython 文件的路径可以是本地路径或 WASB:// 路径。 但是，使用 C Python 时，必须引用用于提交 Pig 作业的节点的本地文件系统上的文件。

通过注册后，此示例的 Pig Latin 对于两个脚本是相同的：

    LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
    DUMP DETAILS;

下面是本示例执行的操作：

1. 第一行代码将示例数据文件 **sample.log** 加载到 **LOGS** 中。 它还将每个记录定义为 **chararray**。
2. 第二行代码筛选出所有 null 值，并将操作结果存储在 **LOG** 中。
3. 接下来，它将循环访问 **LOG** 中的记录，并使用 **GENERATE** 来调用作为 **myfuncs** 加载的 Python/Jython 脚本中包含的 **create_structure** 方法。  **LINE** 用于将当前记录传递给函数。
4. 最后，使用 **DUMP** 命令将输出转储到 STDOUT。 在操作完成后，此时会显示结果。

Python 脚本文件在 C Python 与 Jython 之间很类似，唯一的差别在于，使用 C Python 时必须从 **pig\_util** 导入。 下面是 **pig\_python.py** 脚本：

<a name="streamingpy"></a>

    # Uncomment the following if using C Python
    #from pig_util import outputSchema

    @outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
    def create_structure(input):
        if (input.startswith('java.lang.Exception')):
            input = input[21:len(input)] + ' - java.lang.Exception'
        date, time, classname, level, detail = input.split(' ', 4)
        return date, time, classname, level, detail

> [AZURE.NOTE]
> 安装时不必要考虑“pig_util”，脚本会自动使用该选项。

你应该记得，我们前面将 **LINE** 输入定义为 chararray，因为输入没有一致的架构。 Python 脚本将数据转换成用于输出的一致架构。

1. **@outputSchema** 语句定义返回到 Pig 的数据的格式。 在本例中，该格式为**数据袋**，这是一种 Pig 数据类型。 该数据袋包含以下字段，所有这些字段都是 chararray（字符串）：

    * date - 创建日志条目的日期
    * time - 创建日志条目的时间
    * classname - 为其创建该条目的类名
    * level - 日志级别
    * detail - 日志条目的详细信息

2. 接下来，**def create_structure(input)** 将定义一个函数，以便 Pig 将行项传递到其中。

3. 示例数据 **sample.log** 基本上符合我们要返回的日期、时间、类名、级别和详细信息架构。 但是，它还包含了一些以字符串“*java.lang.Exception*”开头的行，我们需要修改这些行，使之与架构匹配。 **if** 语句将检查这些行，然后调整输入数据以将“*java.lang.Exception*”字符串移到末尾，使数据与预期的输出架构相一致。

4. 接下来，使用 **split** 命令在前四个空格字符处拆分数据。 输出分配到**日期**、**时间**、**classname**、**级别**和**详细信息**。

5. 最后，将值返回到 Pig。

当数据返回到 Pig 时，其架构与 **@outputSchema** 语句将 Python 用作 Hive 中的 UDF。

## <a name="running"></a>运行示例
如果使用的是基于 Linux 的 HDInsight 群集，请使用 **SSH** 步骤。 如果使用的是基于 Windows 的 HDInsight 群集和 Windows 客户端，请使用 **PowerShell** 步骤。

### <a name="ssh"></a>SSH

有关使用 SSH 的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

1. 使用 Python 示例 [streaming.py](#streamingpy) 和 [pig_python.py](#jythonpy) 在开发计算机上创建文件的本地副本。

2. 使用 `scp` 将文件复制到你的 HDInsight 群集。 例如，以下命令将文件复制到名为 **mycluster**的群集。

        scp streaming.py pig_python.py myuser@mycluster-ssh.azurehdinsight.cn:

3. 使用 SSH 连接到群集。 例如，下面会以用户 **myuser** 的身份连接到名为 **mycluster** 的群集。

        ssh myuser@mycluster-ssh.azurehdinsight.cn
4. 从 SSH 会话将前面上载的 python 文件添加到群集的 WASB 存储中。

        hdfs dfs -put streaming.py /streaming.py
        hdfs dfs -put pig_python.py /pig_python.py

在上载文件后，使用以下步骤来运行 Hive 和 Pig 作业。

#### <a name="hive"></a>Hive

1. 使用 `hive` 命令来启动 Hive Shell。 加载 Shell 后，应可看到 `hive>` 提示符。
2. 在 `hive>` 提示符下输入以下命令。

        add file wasbs:///streaming.py;
        SELECT TRANSFORM (clientid, devicemake, devicemodel)
            USING 'python streaming.py' AS
            (clientid string, phoneLabel string, phoneHash string)
        FROM hivesampletable
        ORDER BY clientid LIMIT 50;

3. 输入最后一行后，该作业应该启动。 在作业完成后，会返回类似于以下示例的输出：

        100041    RIM 9650    d476f3687700442549a83fac4560c51c
        100041    RIM 9650    d476f3687700442549a83fac4560c51c
        100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
        100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
        100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9

#### <a name="pig"></a>Pig

1. 使用 `pig` 命令来启动该 shell。 加载 Shell 后，应可看到 `grunt>` 提示符。

2. 在 `grunt>` 提示符下输入以下语句：

        Register wasbs:///pig_python.py using jython as myfuncs;
        LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
        LOG = FILTER LOGS by LINE is not null;
        DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
        DUMP DETAILS;

3. 输入以下行后，应会启动作业。 在作业完成后，会返回类似于以下内容的输出。

        ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
        ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
        ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
        ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
        ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

4. 使用 `quit` 退出 Grunt Shell，然后在本地文件系统上使用以下命令编辑 pig_python.py 文件：

    nano pig_python.py

5. 进入编辑器后，删除行开头的 `#` 字符以取消注释以下行：

        #from pig_util import outputSchema

    完成更改后，使用 Ctrl+X 退出编辑器。 选择“Y”，然后按 Enter 保存更改。

6. 使用 `pig` 命令再次启动 shell。 在 `grunt>` 提示符下，使用以下命令运行带有 Jython 解释器的 Python 脚本。

        Register 'pig_python.py' using streaming_python as myfuncs;
        LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
        LOG = FILTER LOGS by LINE is not null;
        DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
        DUMP DETAILS;

    完成此作业后，看到的输出应该与之前使用 Jython 运行脚本后的输出相同。

### <a name="powershell"></a>PowerShell

这些步骤使用 Azure PowerShell。 有关如何使用 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

1. 使用 Python 示例 [streaming.py](#streamingpy) 和 [pig_python.py](#jythonpy) 在开发计算机上创建文件的本地副本。
2. 使用以下 PowerShell 脚本将 **streaming.py** 和 **pig\_python.py**  文件上传到服务器。 在脚本的前三行中，替换 Azure HDInsight 群集的名称，以及 **streaming.py** 和 **pig\_python.py** 文件的路径。

        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        # Get cluster info
        $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
        $pathToStreamingFile = "C:\path\to\streaming.py"
        $pathToJythonFile = "C:\path\to\pig_python.py"

        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroup = $clusterInfo.ResourceGroup
        $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
        $container=$clusterInfo.DefaultStorageContainer
        $storageAccountKey=(Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value

        #Create a storage content and upload the file
        $context = New-AzureStorageContext `
            -StorageAccountName $storageAccountName `
            -StorageAccountKey $storageAccountKey

        Set-AzureStorageBlobContent `
            -File $pathToStreamingFile `
            -Blob "streaming.py" `
            -Container $container `
            -Context $context

        Set-AzureStorageBlobContent `
            -File $pathToJythonFile `
            -Blob "pig_python.py" `
            -Container $container `
            -Context $context

    此脚本将检索 HDInsight 群集的信息，然后提取默认存储帐户的名称和密钥，并将文件上载到容器的根目录。

    > [AZURE.NOTE]
    > [在 HDInsight 中上传 Hadoop 作业的数据](/documentation/articles/hdinsight-upload-data/)文档中介绍了上传脚本的其他方法。

上传文件后，使用以下 PowerShell 脚本启动作业。 完成作业时，会将输出写入到 PowerShell 控制台。

#### <a name="hive"></a>Hive

以下脚本运行 **streaming.py** 脚本。 在运行前，它会提示你输入 HDInsight 群集的 HTTPs/Admin 帐户信息。

    # Login to your Azure subscription
    # Is there an active Azure subscription?
    $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    }

    # Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $creds=Get-Credential -Message "Enter the login for the cluster"

    # If using a Windows-based HDInsight cluster, change the USING statement to:
    # "USING 'D:\Python27\python.exe streaming.py' AS " +
    $HiveQuery = "add file wasbs:///streaming.py;" +
                    "SELECT TRANSFORM (clientid, devicemake, devicemodel) " +
                    "USING 'python streaming.py' AS " +
                    "(clientid string, phoneLabel string, phoneHash string) " +
                    "FROM hivesampletable " +
                    "ORDER BY clientid LIMIT 50;"

    $jobDefinition = New-AzureRmHDInsightHiveJobDefinition `
        -Query $HiveQuery

    $job = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the Hive job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
        -JobId $job.JobId `
        -ClusterName $clusterName `
        -HttpCredential $creds
    # Uncomment the following to see stderr output
    # Get-AzureRmHDInsightJobOutput `
    #   -Clustername $clusterName `
    #   -JobId $job.JobId `
    #   -HttpCredential $creds `
    #   -DisplayOutputType StandardError
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds

**Hive** 作业的输出应该如以下示例所示：

    100041    RIM 9650    d476f3687700442549a83fac4560c51c
    100041    RIM 9650    d476f3687700442549a83fac4560c51c
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9

#### <a name="jythonpy"></a> Pig (Jython)

以下脚本通过 Jython 解释程序使用 **pig_python.py** 脚本。 在运行前，它会提示用户输入 HDInsight 群集的 HTTPs/Admin 信息。

> [AZURE.NOTE]
> 使用 PowerShell 远程提交作业时，无法使用 C Python 作为解释器。

    # Login to your Azure subscription
    # Is there an active Azure subscription?
    $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    }

    # Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $creds=Get-Credential -Message "Enter the login for the cluster"

    $PigQuery = "Register wasbs:///pig_python.py using jython as myfuncs;" +
                "LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);" +
                "LOG = FILTER LOGS by LINE is not null;" +
                "DETAILS = foreach LOG generate myfuncs.create_structure(LINE);" +
                "DUMP DETAILS;"

    $jobDefinition = New-AzureRmHDInsightPigJobDefinition -Query $PigQuery

    $job = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds

    Write-Host "Wait for the Pig job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
        -Job $job.JobId `
        -ClusterName $clusterName `
        -HttpCredential $creds
    # Uncomment the following to see stderr output
    # Get-AzureRmHDInsightJobOutput `
    #    -Clustername $clusterName `
    #    -JobId $job.JobId `
    #    -HttpCredential $creds `
    #    -DisplayOutputType StandardError
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds

**Pig** 作业的输出应该如下所示：

    ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
    ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
    ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
    ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
    ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

## <a name="troubleshooting"></a>故障排除

### <a name="errors-when-running-jobs"></a>运行作业时出现错误

运行 hive 作业时，可能遇到如下错误：

    Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.

此问题可能是由 streaming.py 文件中的行尾结束符号导致的。 许多 Windows 编辑器默认为使用 CRLF 作为行尾结束符号，但 Linux 应用程序通常应使用 LF。

可以使用以下 PowerShell 语句删除 CR 字符，然后再将文件上载到 HDInsight：

    $original_file ='c:\path\to\streaming.py'
    $text = [IO.File]::ReadAllText($original_file) -replace "`r`n", "`n"
    [IO.File]::WriteAllText($original_file, $text)

### <a name="powershell-scripts"></a>PowerShell 脚本

用于运行示例的两个示例 PowerShell 脚本都包含一个带注释的行，该行显示作业的错误输出。 如果你未看到作业的预期输出，请取消注释以下行，并查看错误信息中是否指明了问题。

    # Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

错误信息 (STDERR,) 和作业的结果 (STDOUT,) 也记录到 HDInsight 存储中。

| 对于此作业... | 在 Blob 容器中查看这些文件 |
| --- | --- |
| Hive |/HivePython/stderr<p>/HivePython/stdout |
| Pig |/PigPython/stderr<p>/PigPython/stdout |

## <a name="next"></a>后续步骤

如果需要加载默认情况下未提供的 Python 模块，请参阅 [How to deploy a module to Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/03/03/how-to-deploy-a-python-module-to-windows-azure-hdinsight.aspx)（如何将模块部署到 Azure HDInsight）。

若要了解使用 Pig 和 Hive 的其他方式以及如何使用 MapReduce，请参阅以下文档：

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!--Update_Description: wording update-->