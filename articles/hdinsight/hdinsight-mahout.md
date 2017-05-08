<properties
    pageTitle="通过 PowerShell 使用 Mahout HDInsight 生成推荐 | Azure"
    description="了解如何使用 Apache Mahout 机器学习库通过客户端上运行的 PowerShell 脚本中的 HDInsight (Hadoop) 生成电影推荐。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="07b57208-32aa-4e59-900a-6c934fa1b7a7"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/14/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="3407df096fa6c63c165efb64821a3a73460df68d"
    ms.lasthandoff="04/28/2017" />

# <a name="generate-movie-recommendations-by-using-apache-mahout-with-hadoop-in-hdinsight-powershell"></a>将 Apache Mahout 与 HDInsight (PowerShell) 中的 Hadoop 配合使用生成电影推荐

[AZURE.INCLUDE [mahout-selector](../../includes/hdinsight-selector-mahout.md)]

了解如何使用 [Apache Mahout](http://mahout.apache.org) 机器学习库通过 Azure HDInsight 生成电影推荐。 本文档中的示例使用 Azure PowerShell 运行 Mahout 作业。

## <a name="prerequisites"></a>先决条件

* 基于 Linux 的 HDInsight 群集。 有关创建该群集的信息，请参阅 [开始在 HDInsight 中使用基于 Linux 的 Hadoop][getstarted]。

    [AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [AZURE.IMPORTANT]
    > Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* [Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azure/overview)

## <a name="recommendations"></a>使用 Azure PowerShell 生成推荐

> [AZURE.WARNING]
> 本节中的作业通过使用 Azure PowerShell 运行。 许多通过 Mahout 提供的类当前无法与 Azure PowerShell 配合使用。 有关不适用于 Azure PowerShell 的类的列表，请参阅[故障排除](#troubleshooting)部分。
> <p>
> 有关使用 SSH 连接到 HDInsight 和直接在群集上运行 Mahout 示例的示例，请参阅[使用 Mahout 和 HDInsight (SSH) 生成电影推荐](/documentation/articles/hdinsight-hadoop-mahout-linux-mac/)。

由 Mahout 提供的功能之一是推荐引擎。 此引擎接受 `userID`、`itemId` 和 `prefValue` 格式（项的用户首选项）的数据。 Mahout 使用该数据确定拥有类似项首选项的用户，这些首选项可用于提供建议。

以下示例是对于建议流程的工作原理的简化演练：

* **共现**：Joe、Alice 和 Bob 都喜欢电影《星球大战》、《帝国反击战》和《绝地归来》。 Mahout 可确定喜欢以上电影之一的用户也喜欢其他两部。

* **共现**：Bob 和 Alice 还喜欢电影《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。 Mahout 确定喜欢前面三部电影的用户也喜欢这些电影。

* **类似性推荐**：由于 Joe 喜欢前三部电影，Mahout 会查看具有类似首选项的其他人喜欢的电影，但是 Joe 还未观看过（喜欢/评价）。 在这种情况下，Mahout 推荐《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。

### <a name="understanding-the-data"></a>了解数据

[GroupLens 研究][movielens]以兼容 Mahout 的格式提供电影的评价数据。 此数据可用于群集默认存储的 `/HdiSamples//HdiSamples/MahoutMovieData` 中。

包含以下两个文件：`moviedb.txt`（有关电影的信息）和 `user-ratings.txt`。 `user-ratings.txt` 文件在分析期间使用。 `moviedb.txt` 文件用于在显示分析结果时，提供便于用户阅读的文本。

user-ratings.txt 中包含的数据具有 `userID`、`movieID`、`userRating` 和 `timestamp` 结构，它将告诉我们每个用户对电影评级的情况。 下面是数据的示例：

    196    242    3    881250949
    186    302    3    891717742
    22    377    1    878887116
    244    51    2    880606923
    166    346    1    886397596

### <a name="run-the-job"></a>运行作业

使用以下 Windows PowerShell 脚本来运行作业，以将 Mahout 推荐引擎用于电影数据：

> [AZURE.NOTE]
> 此文件将提示你输入用于连接到 HDInsight 群集和运行作业的信息。 完成作业和下载 output.txt 文件可能需要几分钟时间。

    # Login to your Azure subscription
    # Is there an active Azure subscription?
    $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    }

    # Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $creds=Get-Credential -UserName "admin" -Message "Enter the login for the cluster"

    #Get the cluster info so we can get the resource group, storage, etc.
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroup = $clusterInfo.ResourceGroup
    $storageAccountName = $clusterInfo.DefaultStorageAccount.split('.')[0]
    $container = $clusterInfo.DefaultStorageContainer
    $storageAccountKey = (Get-AzureRmStorageAccountKey `
        -Name $storageAccountName `
    -ResourceGroupName $resourceGroup)[0].Value

    #Create a storage context and upload the file
    $context = New-AzureStorageContext `
        -StorageAccountName $storageAccountName `
        -StorageAccountKey $storageAccountKey

    #Use Hive to figure out the path to the mahout examples
    #Because the file name/path has a version number in it that changes
    $queryString = "!ls /usr/hdp/current/mahout-client"
    $hiveJobDefinition = New-AzureRmHDInsightHiveJobDefinition -Query $queryString
    $hiveJob=Start-AzureRmHDInsightJob -ClusterName $clusterName -JobDefinition $hiveJobDefinition -HttpCredential $creds
    wait-azurermhdinsightjob -ClusterName $clusterName -JobId $hiveJob.JobId -HttpCredential $creds > $null
    #Get the files returned from Hive
    $files=get-azurermhdinsightjoboutput -clustername $clusterName -JobId $hiveJob.JobId -DefaultContainer $container -DefaultStorageAccountName $storageAccountName -DefaultStorageAccountKey $storageAccountKey -HttpCredential $creds
    #Find the file that starts with mahout-examples and ends in job.jar
    $jarFile = $files | select-string "mahout-examples.+job\.jar" | % {$_.Matches.Value}
    #Add the full path
    $jarFile = "file:///usr/hdp/current/mahout-client/$jarFile"

    # The arguments for the mahout job
    # * input - the path to the data uploaded to HDInsight
    # * output - the path to store output data
    # * tempDir - the directory for temp files
    $jobArguments = "-s", "SIMILARITY_COOCCURRENCE", `
                    "--input", "/HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt",
                    "--output", "/example/out",
                    "--tempDir", "/example/temp"

    # Create the job definition
    $jobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
        -JarFile $jarFile `
        -ClassName "org.apache.mahout.cf.taste.hadoop.item.RecommenderJob" `
        -Arguments $jobArguments

    # Start the job
    $job = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds

    # Wait on the job to complete
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds

    # Write out any error information
    Write-Host "STDERR"
    Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

    # Download the output
    Get-AzureStorageBlobContent `
            -Blob example/out/part-r-00000 `
            -Container $container `
            -Destination output.txt `
            -Context $context
    #Download movie and user files for use in displaying results
    Get-AzureStorageBlobContent -blob "HdiSamples/HdiSamples/MahoutMovieData/moviedb.txt" `
            -Container $container `
            -Destination moviedb.txt `
            -Context $context
    Get-AzureStorageBlobContent -blob "HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt" `
            -Container $container `
            -Destination user-ratings.txt `
            -Context $context

> [AZURE.NOTE]
> Mahout 作业不删除在处理作业时创建的临时数据。 在示例作业中指定 `--tempDir` 参数，以将临时文件隔离到特定目录中。

Mahout 作业不会将输出返回到 STDOUT。 而是会将其作为 **part-r-00000**存储在指定的输出目录中。 该脚本将此文件下载到你工作站上的当前目录中的 **output.txt** 中。

以下文本是此文件内容的示例：

    1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
    2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
    3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
    4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]

第一列是 `userID`。 “[”和“]”中包含的值为 `movieId`:`recommendationScore`。

该脚本还会下载 `moviedb.txt` 和 `user-ratings.txt` 文件，需要这些文件格式化输出使其更具可读性。

### <a name="view-the-output"></a>查看输出

虽然生成的输出也许可用于应用程序中，但不便于用户阅读。 可以使用服务器中的 `moviedb.txt` 将 `movieId` 解析为电影名称。 使用以下 PowerShell 脚本显示包含影片名称的推荐：

    <#
    .SYNOPSIS
        Displays recommendations for movies.
    .DESCRIPTION
        Displays recommendations generated by Mahout
        with HDInsight example in a human readable format.
    .EXAMPLE
        .\Show-Recommendation -userId 4
            -userDataFile "user-ratings.txt"
            -movieFile "moviedb.txt"
            -recommendationFile "output.txt"
    #>

    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
        #The user ID
        [Parameter(Mandatory = $true)]
        [String]$userId,

        [Parameter(Mandatory = $true)]
        [String]$userDataFile,

        [Parameter(Mandatory = $true)]
        [String]$movieFile,

        [Parameter(Mandatory = $true)]
        [String]$recommendationFile
    )
    # Read movie ID & description into hash table
    Write-Host "Reading movies descriptions" -ForegroundColor Green
    $movieById = @{}
    foreach($line in Get-Content $movieFile)
    {
        $tokens = $line.Split("|")
        $movieById[$tokens[0]] = $tokens[1]
    }
    # Load movies user has already seen (rated)
    # into a hash table
    Write-Host "Reading rated movies" -ForegroundColor Green
    $ratedMovieIds = @{}
    foreach($line in Get-Content $userDataFile)
    {
        $tokens = $line.Split("`t")
        if($tokens[0] -eq $userId)
        {
            # Resolve the ID to the movie name
            $ratedMovieIds[$movieById[$tokens[1]]] = $tokens[2]
        }
    }
    # Read recommendations generated by Mahout
    Write-Host "Reading recommendations" -ForegroundColor Green
    $recommendations = @{}
    foreach($line in get-content $recommendationFile)
    {
        $tokens = $line.Split("`t")
        if($tokens[0] -eq $userId)
        {
            #Trim leading/treailing [] and split at ,
            $movieIdAndScores = $tokens[1].TrimStart("[").TrimEnd("]").Split(",")
            foreach($movieIdAndScore in $movieIdAndScores)
            {
                #Split at : and store title and score in a hash table
                $idAndScore = $movieIdAndScore.Split(":")
                $recommendations[$movieById[$idAndScore[0]]] = $idAndScore[1]
            }
            break
        }
    }

    Write-Host "Rated movies" -ForegroundColor Green
    Write-Host "---------------------------" -ForegroundColor Green
    $ratedFormat = @{Expression={$_.Name};Label="Movie";Width=40}, `
                    @{Expression={$_.Value};Label="Rating"}
    $ratedMovieIds | format-table $ratedFormat
    Write-Host "---------------------------" -ForegroundColor Green

    write-host "Recommended movies" -ForegroundColor Green
    Write-Host "---------------------------" -ForegroundColor Green
    $recommendationFormat = @{Expression={$_.Name};Label="Movie";Width=40}, `
                            @{Expression={$_.Value};Label="Score"}
    $recommendations | format-table $recommendationFormat

使用以下命令，以便于用户阅读的格式显示建议： 

    .\show-recommendation.ps1 -userId 4 -userDataFile .\user-ratings.txt -movieFile .\moviedb.txt -recommendationFile .\output.txt

输出与以下文本类似：

    Reading movies descriptions
    Reading rated movies
    Reading recommendations
    Rated movies
    ---------------------------
    Movie                                    Rating
    -----                                    ------
    Devil's Own, The (1997)                  1
    Alien: Resurrection (1997)               3
    187 (1997)                               2
    (lines ommitted)

    ---------------------------
    Recommended movies
    ---------------------------

    Movie                                    Score
    -----                                    -----
    Good Will Hunting (1997)                 4.6504064
    Swingers (1996)                          4.6862745
    Wings of the Dove, The (1997)            4.6666665
    People vs. Larry Flynt, The (1996)       4.834559
    Everyone Says I Love You (1996)          4.707071
    Secrets & Lies (1996)                    4.818182
    That Thing You Do! (1996)                4.75
    Grosse Pointe Blank (1997)               4.8235292
    Donnie Brasco (1997)                     4.6792455
    Lone Star (1996)                         4.7099237

## <a name="troubleshooting"></a>故障排除

### <a name="cannot-overwrite-files"></a>无法覆盖文件

Mahout 作业不清理在处理期间创建的临时文件。 此外，作业不会覆盖现有的输出文件。

若要避免运行 Mahout 作业时出错，请在每次运行作业之前删除临时文件和输出文件。 若要删除由本文档前面的脚本创建的文件，请使用以下 PowerShell 脚本：

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

    #Get the cluster info so we can get the resource group, storage, etc.
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroup = $clusterInfo.ResourceGroup
    $storageAccountName = $clusterInfo.DefaultStorageAccount.split('.')[0]
    $container = $clusterInfo.DefaultStorageContainer
    $storageAccountKey = (Get-AzureRmStorageAccountKey `
        -Name $storageAccountName `
    -ResourceGroupName $resourceGroup)[0].Value

    #Create a storage context and upload the file
    $context = New-AzureStorageContext `
        -StorageAccountName $storageAccountName `
        -StorageAccountKey $storageAccountKey

    #Azure PowerShell can't delete blobs using wildcard,
    #so have to get a list and delete one at a time
    # Start with the output
    $blobs = Get-AzureStorageBlob -Container $container -Context $context -Prefix "example/out"
    foreach($blob in $blobs)
    {
        Remove-AzureStorageBlob -Blob $blob.Name -Container $container -context $context
    }
    # Next the temp files
    $blobs = Get-AzureStorageBlob -Container $container -Context $context -Prefix "example/temp"
    foreach($blob in $blobs)
    {
        Remove-AzureStorageBlob -Blob $blob.Name -Container $container -context $context
    }

### <a name="nopowershell"></a>不适用于 Azure PowerShell 的类

在 Windows PowerShell 中使用时，使用以下类的 Mahout 作业将返回各种错误消息：

* org.apache.mahout.utils.clustering.ClusterDumper
* org.apache.mahout.utils.SequenceFileDumper
* org.apache.mahout.utils.vectors.lucene.Driver
* org.apache.mahout.utils.vectors.arff.Driver
* org.apache.mahout.text.WikipediaToSequenceFile
* org.apache.mahout.clustering.streaming.tools.ResplitSequenceFiles
* org.apache.mahout.clustering.streaming.tools.ClusterQualitySummarizer
* org.apache.mahout.classifier.sgd.TrainLogistic
* org.apache.mahout.classifier.sgd.RunLogistic
* org.apache.mahout.classifier.sgd.TrainAdaptiveLogistic
* org.apache.mahout.classifier.sgd.ValidateAdaptiveLogistic
* org.apache.mahout.classifier.sgd.RunAdaptiveLogistic
* org.apache.mahout.classifier.sequencelearning.hmm.BaumWelchTrainer
* org.apache.mahout.classifier.sequencelearning.hmm.ViterbiEvaluator
* org.apache.mahout.classifier.sequencelearning.hmm.RandomSequenceGenerator
* org.apache.mahout.classifier.df.tools.Describe

若要运行使用这些类的作业，请使用 SSH 连接到 HDInsight 群集，然后从命令行运行这些作业。 有关使用 SSH 运行 Mahout 作业的示例，请参阅[使用 Mahout 和 HDInsight (SSH) 生成电影推荐](/documentation/articles/hdinsight-hadoop-mahout-linux-mac/)。

## <a name="next-steps"></a>后续步骤

既已学习如何使用 Mahout，可探索在 HDInsight 上处理数据的其他方式：

* [Hive 和 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [Pig 和 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [MapReduce 和 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

[build]: http://mahout.apache.org/developers/buildingmahout.html
[aps]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[upload]: /documentation/articles/hdinsight-upload-data/
[ml]: http://en.wikipedia.org/wiki/Machine_learning
[forest]: http://en.wikipedia.org/wiki/Random_forest
[management]: https://manage.windowsazure.cn/
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools