<properties
    pageTitle="使用 HDInsight 开发 Python MapReduce 作业 | Azure"
    description="了解如何在基于 Linux 的 HDInsight 群集上创建和运行 Python MapReduce 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="7631d8d9-98ae-42ec-b9ec-ee3cf7e57fb3"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/06/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="ad4ab1c3082d640f386bab81d41016a88d46c44f"
    ms.lasthandoff="04/28/2017" />

# <a name="develop-python-streaming-programs-for-hdinsight"></a>开发适用于 HDInsight 的 Python 流式处理程序

Hadoop 为 MapReduce 提供了一个流式处理 API，这样，除 Java 外，你还能使用其他语言编写映射和化简函数。 本文介绍如何使用 Python 执行 MapReduce 操作。

本文的内容基于 Michael Noll 在 [Writing an Hadoop MapReduce Program in Python](http://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/)（以 Python 编写 Hadoop MapReduce 程序）中发布的信息和示例。

## <a name="prerequisites"></a>先决条件

要完成本文中的步骤，需要：

* 基于 Linux 的 HDInsight 上的 Hadoop 群集

    > [AZURE.IMPORTANT]
    > 本文档中的步骤需要使用 Linux 的 HDInsight 群集。 Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* 文本编辑器

    > [AZURE.IMPORTANT]
    > 文本编辑器必须使用 LF 作为行尾。 如果使用 CRLF，则在基于 Linux 的 HDInsight 群集上运行 MapReduce 作业时会出错。 如果不确定它使用哪种行尾，请使用 [运行 MapReduce](#run-mapreduce-ssh) 部分中的可选步骤，将所有 CRLF 转换为 LF。

* **熟悉 SSH 和 SCP**。 有关详细信息，请参阅 [对 HDInsight 使用 SSH](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a name="word-count"></a>字数统计

在本示例中，将通过使用映射器和化简器实现基本字数统计。 映射器会将句子分解成不同的单词，而化简器会汇总单词并统计字数以生成输出。

下图演示了在映射和化简阶段发生的情况。

![映射化简示意图](./media/hdinsight-hadoop-streaming-python/HDI.WordCountDiagram.png)

## <a name="why-python"></a>为何要使用 Python？

Python 是一种通用高级编程语言，可让你以少于其他许多语言的代码行快速表达概念。 数据科学家最近很喜欢将它用作原型设计语言，因为它的自释性、动态类型化和简洁语法非常适合用于快速开发应用程序。

Python 已安装在所有 HDInsight 群集上。

## <a name="streaming-mapreduce"></a>流式处理 MapReduce

Hadoop 允许你指定包含作业所用映射和化简逻辑的文件。 映射和化简逻辑的具体要求如下：

* **输入**：映射和化简组件必须从 STDIN 读取输入数据。
* **输出**：映射和化简组件必须将输出数据写入到 STDOUT。
* **数据格式**：使用和生成的数据必须是键/值对，并以制表符分隔。

Python 可以使用 **sys** 模块从 STDIN 读取数据，并使用 **print** 输出到 STDOUT，从而可以轻松应对这些要求。 余下的任务就是在键和值之间以制表符 (`\t`) 设置数据的格式。

## <a name="create-the-mapper-and-reducer"></a>创建映射器和化简器

映射器和化简器是文本文件，在本示例中为 **mapper.py** 和 **reducer.py**（以区分各自的作用）。 你可以通过使用自选的编辑器创建这些文件。

### <a name="mapperpy"></a>Mapper.py

创建名为 **mapper.py** 的新文件并使用以下代码作为内容：

    #!/usr/bin/env python

    # Use the sys module
    import sys

    # 'file' in this case is STDIN
    def read_input(file):
        # Split each line into words
        for line in file:
            yield line.split()

    def main(separator='\t'):
        # Read the data using read_input
        data = read_input(sys.stdin)
        # Process each words returned from read_input
        for words in data:
            # Process each word
            for word in words:
                # Write to STDOUT
                print '%s%s%d' % (word, separator, 1)

    if __name__ == "__main__":
        main()

花些时间通读代码，以便你可以了解它的功能。

### <a name="reducerpy"></a>reducer.py

创建名为 **reducer.py** 的新文件并使用以下代码作为内容：

    #!/usr/bin/env python

    # import modules
    from itertools import groupby
    from operator import itemgetter
    import sys

    # 'file' in this case is STDIN
    def read_mapper_output(file, separator='\t'):
        # Go through each line
        for line in file:
            # Strip out the separator character
            yield line.rstrip().split(separator, 1)

    def main(separator='\t'):
        # Read the data using read_mapper_output
        data = read_mapper_output(sys.stdin, separator=separator)
        # Group words and counts into 'group'
        #   Since MapReduce is a distributed process, each word
        #   may have multiple counts. 'group' will have all counts
        #   which can be retrieved using the word as the key.
        for current_word, group in groupby(data, itemgetter(0)):
            try:
                # For each word, pull the count(s) for the word
                #   from 'group' and create a total count
                total_count = sum(int(count) for current_word, count in group)
                # Write to stdout
                print "%s%s%d" % (current_word, separator, total_count)
            except ValueError:
                # Count was not a number, so do nothing
                pass

    if __name__ == "__main__":
        main()

## <a name="upload-the-files"></a>上载文件

**mapper.py** 和 **reducer.py** 都必须位于群集的头节点上才能运行。 本部分提供了一个示例 `scp` 命令和一个 Azure PowerShell 脚本，可用于将文件上传到群集。

> [AZURE.IMPORTANT]
> 使用 `scp` 与使用 PowerShell 上传文件有很大区别：
><p>
><p> * 使用 `scp` 是将文件放置在群集的主头节点上。 这假定你以后将连接到头节点并从 SSH 会话运行作业。
><p> * 使用 PowerShell 脚本是将文件放置在群集的默认存储中。 这假定你以后将使用 PowerShell 脚本从远程客户端运行作业。

### <a name="upload-using-scp"></a>使用 scp 上传

在开发环境中，在与 **mapper.py** 和 **reducer.py** 相同的目录中，使用以下命令。 将 **username** 替换为群集的 SSH 用户名，并将 **clustername** 替换为群集的名称。

`scp mapper.py reducer.py username@clustername-ssh.azurehdinsight.cn:`

这样就会将两个文件从本地系统复制到头节点。

> [AZURE.NOTE]
> 如果使用了密码来保护 SSH 帐户，系统会提示你输入密码。 如果使用了 SSH 密钥，则可能需要使用 `-i` 参数和私钥路径，例如 `scp -i /path/to/private/key mapper.py reducer.py username@clustername-ssh.azurehdinsight.cn:`。

### <a name="upload-using-powershell"></a>使用 PowerShell 上载

1. 从开发环境创建一个名为 `Put-FilesInHDInsight.ps1` 的新文件，并使用以下代码作为该文件的内容：

        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName,
            [Parameter(Mandatory = $true)]
            [String]$source,
            [Parameter(Mandatory = $true)]
            [String]$destination
        )
        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        # Get the cluster info
        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $storageInfo = $clusterInfo.DefaultStorageAccount.split('.')

        # What type of default storage are we using?
        switch ($storageInfo[1])
        {
            "blob" {
                # Get the blob storage information for the cluster
                $resourceGroup = $clusterInfo.ResourceGroup
                $storageAccountName=$storageInfo[0]
                $storageContainer=$clusterInfo.DefaultStorageContainer
                $storageAccountKey=(Get-AzureRmStorageAccountKey `
                    -Name $storageAccountName `
                    -ResourceGroupName $resourceGroup)[0].Value
                # Create a storage context and upload the file
                $context = New-AzureStorageContext `
                    -StorageAccountName $storageAccountName `
                    -StorageAccountKey $storageAccountKey
                # Upload the file
                Set-AzureStorageBlobContent `
                    -File $source `
                    -Blob $destination `
                    -Container $storageContainer `
                    -Context $context
            }
            default {
                Throw "Unknown storage type: $storageInfo[1]"
            }
        }

2. 若要使用此脚本上载文件，请从 Azure PowerShell 提示符使用以下命令：

    `.\Put-FilesInHDInsight.ps1 -clusterName <your HDInsight cluster name> -source mapper.py -destination mapper.py`

    出现提示时，请输入群集的 HTTPS 登录凭据。

3. 重复执行该命令，并将 `mapper.py` 替换为 `reducer.py`。 这会同时将 mapper 和 reducer 文件上传到群集的默认存储。

## <a name="run-mapreduce-ssh"></a>运行 MapReduce (SSH)

使用以下步骤连接到群集并从 SSH 会话运行流式处理 MapReduce 作业。

1. 通过使用 SSH 连接到群集：

    `ssh username@clustername-ssh.azurehdinsight.cn`

    > [AZURE.NOTE]
    > 如果使用了密码来保护 SSH 帐户，系统会提示你输入密码。 如果使用了 SSH 密钥，则可能需要使用 `-i` 参数和私钥路径，例如 `ssh -i /path/to/private/key username@clustername-ssh.azurehdinsight.cn`。

2. （可选）在创建 mapper.py 和 reducer.py 文件时，如果使用的文本编辑器使用 CRLF 作为行尾，或者不知道该编辑器使用哪种行尾，请使用以下命令将 mapper.py 和 reducer.py 中出现的 CRLF 转换为 LF。

    `perl -pi -e 's/\r\n/\n/g' mappery.py`
    `perl -pi -e 's/\r\n/\n/g' reducer.py`

3. 使用以下命令启动 MapReduce 作业。

    `yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar -files mapper.py,reducer.py -mapper mapper.py -reducer reducer.py -input /example/data/gutenberg/davinci.txt -output /example/wordcountout`

    此命令包括以下几个部分：

    * **hadoop-streaming.jar**：运行流式处理 MapReduce 操作时使用。 它可以将 Hadoop 和你提供的外部 MapReduce 代码连接起来。

    * **-files**：告知 Hadoop 此 MapReduce 作业需要指定的文件，而且应将这些文件复制到所有辅助节点。

    * **-mapper**：告诉 Hadoop 要用作映射器的文件。

    * **-reducer**：告诉 Hadoop 要用作化简器的文件。

    * **-input**：要从中统计字数的输入文件。

    * **-output**：要将输出写入到的目录。

        > [AZURE.NOTE]
        > 该作业会创建此目录。

作业启动后，会看到一堆 **INFO** 语句，最后会看到以百分比显示的**映射**和**化简**操作。

    15/02/05 19:01:04 INFO mapreduce.Job:  map 0% reduce 0%
    15/02/05 19:01:16 INFO mapreduce.Job:  map 100% reduce 0%
    15/02/05 19:01:27 INFO mapreduce.Job:  map 100% reduce 100%

你将在作业完成时收到有关作业的状态信息。

## <a name="run-mapreduce-powershell"></a>运行 MapReduce (PowerShell)

使用以下步骤从开发环境中的 PowerShell 运行流式处理 MapReduce。 PowerShell 脚本通过使用 WebHCat 连接到 HDInsight 群集运行作业。

1. 创建一个名为 `Start-HDInsightStreamingPythonJob` 的新文件，并使用以下代码作为该文件的内容：

        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName,
            [Parameter(Mandatory = $true)]
            [String]$inputPath,
            [Parameter(Mandatory = $true)]
            [String]$outputPath
        )
        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        # Get the login (HTTPS) credentials for the cluster
        $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

        # Create the streaming job definition
        # Note: This assumes that the mapper.py and reducer.py
        #       are in the root of default storage. If you put them in a
        #       subdirectory, change the -Files parameter to the correct path.
        $jobDefinition = New-AzureRmHDInsightStreamingMapReduceJobDefinition `
            -Files "/mapper.py", "/reducer.py" `
            -Mapper "mapper.py" `
            -Reducer "reducer.py" `
            -InputPath $inputPath `
            -OutputPath $outputPath

        # Start the job
        $job = Start-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobDefinition $jobDefinition `
            -HttpCredential $creds

        # Wait for the job to complete
        Wait-AzureRmHDInsightJob `
            -JobId $job.JobId `
            -ClusterName $clusterName `
            -HttpCredential $creds

2. 在 Azure PowerShell 提示符中使用以下命令运行作业：

    `.\Start-HDInsightStreamingPythonJob.ps1 -clusterName <your HDInsight cluster name> -inputPath "/example/data/gutenberg/davinci.txt" -outputPath "/example/wordcountout"`

    这将使用先前上传到群集的 `mapper.py` 和 `reducer.py` 文件对 `davinci.txt` 文件中的文本统计字数。 输出会存储在群集默认存储上的 `/example/wordcount` 文件夹中。

    作业完成时，将显示如下信息：

        Cluster         : myhdicluster
        HttpEndpoint    : myhdicluster.azurehdinsight.cn
        State           : SUCCEEDED
        JobId           : job_1486046226168_0004
        ParentId        :
        PercentComplete : map 100% reduce 100%
        ExitValue       : 0
        User            : admin
        Callback        :
        Completed       : done
        StatusFolder    : 2017-02-06T19-14-28-a3dda3ca-f489-4287-afc9-b5e16e2e7c7a

## <a name="view-the-output"></a>查看输出

作业的输出存储在 `/example/wordcountout` 目录中。 可以从 SSH 会话查看此输出，也可以下载到本地，从 PowerShell 查看它。

若要查看群集的 SSH 会话的数据，请使用以下命令：

`hdfs dfs -text /example/wordcountout/part-00000`

这会显示单词及单词出现次数的列表。 下面是输出数据的示例：

    wrenching       1
    wretched        6
    wriggling       1
    wrinkled,       1
    wrinkles        2
    wrinkling       2

若要使用 PowerShell 查看开发环境的输出，请使用以下步骤：

1. 创建一个名为 `Get-FilesInHDInsight.ps1` 的新文件，并使用以下代码作为文件内容：

        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName,
            [Parameter(Mandatory = $true)]
            [String]$source
        )
        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        # Get the cluster info
        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $storageInfo = $clusterInfo.DefaultStorageAccount.split('.')

        # What type of default storage are we using?
        switch ($storageInfo[1])
        {
            "blob" {
                # Get the blob storage information for the cluster
                $resourceGroup = $clusterInfo.ResourceGroup
                $storageAccountName=$storageInfo[0]
                $storageContainer=$clusterInfo.DefaultStorageContainer
                $storageAccountKey=(Get-AzureRmStorageAccountKey `
                    -Name $storageAccountName `
                    -ResourceGroupName $resourceGroup)[0].Value
                # Create a storage context and download the file
                $context = New-AzureStorageContext `
                    -StorageAccountName $storageAccountName `
                    -StorageAccountKey $storageAccountKey
                # Download the file
                Get-AzureStorageBlobContent `
                    -Container $storageContainer `
                    -Blob $source `
                    -Context $context `
                    -Destination "./output.txt"
                # Display the output
                Get-Content "./output.txt"
            }
            default {
                Throw "Unknown storage type: $storageInfo[1]"
            }
        }

2. 在 Azure PowerShell 提示符下使用以下命令运行脚本，并查看输出：

    `Get-FilesInHDInsight.ps1 -clusterName <your HDInsight cluster name> -source "example/wordcountout/part-00000"`

    这会显示单词及单词出现次数的列表。 下面是输出数据的示例：

        wrenching       1
        wretched        6
        wriggling       1
        wrinkled,       1
        wrinkles        2
        wrinkling       2

## <a name="next-steps"></a>后续步骤

了解如何将流式处理 MapRedcue 作业用于 HDInsight 后，就可以使用以下链接来学习 Azure HDInsight 的其他用法。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)