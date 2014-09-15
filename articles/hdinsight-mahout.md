<properties title="Generate movie recommendations using Mahout" pageTitle="Generate movie recommendations using Mahout with Microsoft Azure HDInsight (Hadoop)" description="Learn how to use the Apache Mahout machine learning library to generate movie recommendations with HDInsight (Hadoop)" metaKeywords="Azure hdinsight mahout, Azure hdinsight machine learning, azure hadoop mahout, azure hadoop machine learning" services="hdinsight" solutions="" documentationCenter="big-data" authors="larryfr" videoId="" scriptId="" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="larryfr"></tags>

# 将 Apache Mahout 与 HDInsight (Hadoop) 配合使用以生成电影推荐

了解如何在 Microsoft Azure HDInsight (Hadoop) 上使用 [Apache Mahout](http://mahout.apache.org) 计算机学习库生成电影推荐。

> [WACOM.NOTE] 若要使用本文中的信息，必须先创建一个 HDInsight 群集。有关创建 HDInsight 群集的信息，请参阅[开始使用 HDInsight 中的 Hadoop][getstarted]。

> [WACOM.NOTE] HDInsight 3.1 群集随附了 Mahout。如果你使用的是早期版本的 HDInsight，请在继续操作之前参阅[安装 Mahout](#install)。

## <a name="learn"></a>你将会学到的知识

Mahout 是适用于 Apache Hadoop 的[计算机学习][ml]库。Mahout 包含用于处理数据的算法，例如筛选、分类和群集。在本文中，你将要使用一个推荐引擎根据你的好友观看的电影生成电影推荐。你还将了解如何使用决策林执行分类。本文将为你传授以下知识。

- 如何从 PowerShell 运行 Mahout 作业

- 如何从 Hadoop 命令行运行 Mahout 作业

- 如何在 HDInsight 2.0 和 3.0 群集上安装 Mahout

## 本文内容

- [使用 PowerShell 生成推荐](#recommendations)
- [使用 Hadoop 命令行为数据分类](#classify)
- [故障排除](#troubleshooting)

## <a name="recommendations"></a>使用 PowerShell 生成推荐

> [WACOM.NOTE] 尽管本部分中使用的作业适用于 PowerShell，但 Mahout 随附的许多类目前并不适用于 PowerShell，而必须使用 Hadoop 命令行运行。有关不适用于 PowerShell 的类的列表，请参阅[故障排除](#troubleshooting)部分。
>
> 有关使用 Hadoop 命令行运行 Mahout 作业的示例，请参阅[使用 Hadoop 命令行为数据分类](#classify)。

推荐引擎是 Mahout 提供的功能之一。此引擎接受`userID`, `itemId`, `prefValue` （用户对该项的偏好程度）格式的数据。然后，Mahout 将执行共现分析，以确定*偏好某个项的用户也偏好其他类似项*。Mahout 将会确定具有类似偏好的用户，并凭此做出推荐。

下面是一个极其简单的电影推荐示例：

- **共现** - Joe、Alice 和 Bob 都喜欢电影*星球大战*、*帝国反击战*和*绝地大反击*。Mahout 将会据此判断喜欢其中任何一部电影的用户也喜欢其他两部。

- **共现** - Bob 和 Alice 还喜欢电影*幽灵的威胁*、*克隆人的进攻*和*西斯的复仇*。Mahout 将会据此判断喜欢前三部电影的用户也喜欢这三部电影

- **类似性推荐** - 由于 Joe 喜欢前三部电影，Mahout 将查找其他具有类似偏好的用户喜欢的，但 Joe 尚未观看（称赞/评分）过的电影。在此情况下，Mahout 将会推荐*幽灵的威胁*、*克隆人的进攻*和*西斯的复仇*。

### 加载数据

为了提供方便，GroupLens Research 提供了 Mahout 兼容格式的[电影评分数据][movielens]。

1. 下载 [MovieLens 100k][100k] 存档，其中包含 1000 名用户对 1700 部电影给出的 100,000 项评分。

2. 提取该存档。其中应该包含一个 **ml-100k** 目录，该目录包含许多前缀为 **u.** 的数据文件。Mahout 要分析的文件为 **u.data**。此文件的数据结构为`userID`, `movieID`, `userRating` 和`timestamp`. 下面是数据的示例。

        196 242 3   881250949
        186 302 3   891717742
        22  377 1   878887116
        244 51  2   880606923
        166 346 1   886397596

3. 将 **u.data** 文件上载到 HDInsight 群集上的 **example/data/u.data**。如果你安装了 [Azure PowerShell][aps]，则可以使用 [HDInsight-Tools][tools] PowerShell 模块上载该文件。有关上载文件的其他方法，请参阅[在 HDInsight 中上载 Hadoop 作业的数据][upload]。下面演示了如何使用 `Add-HDInsightFile` 上载文件

        PS C:\> Add-HDInsightFile -LocalPath "path\to\u.data" -DestinationPath "example/data/u.data" -ClusterName "your cluster name"

    这样就会将 **u.data** 文件上载到群集的默认存储中的 **example/data/u.data**。然后，我们可以使用 **wasb:///example/data/u.data** URI 从 HDInsight 作业访问此数据。

### 运行作业

使用以下 PowerShell 脚本，结合前面上载的 **u.data** 文件运行使用 Mahout 推荐引擎的作业。

    # The HDInsight cluster name.
    $clusterName = "the cluster name"

    # The location of the Mahout jar file.
    $jarFile = "C:\apps\dist\mahout-0.9.0.2.1.3.0-1887\examples\target\mahout-examples-0.9.0.2.1.3.0-1887-job.jar"
    # NOTE: The version number portion of the file path
    # may change in future versions of HDInsight.
    # Use the following to find the location and name
    # of the Mahout jar on HDInsight 3.1, and modify
    # the $jarFile= line to point to it.
    #
    # Use-AzureHDInsightCluster -Name $clusterName
    # $jarFile = Invoke-Hive -Query '!${env:COMSPEC} /c dir /b /s ${env:MAHOUT_HOME}\examples\target\*-job.jar'
    #
    # If you are using an earlier version of HDInsight,
    # set $jarFile to the jar file you
    # uploaded.
    # For example,
    # $jarFile = "wasb:///example/jars/mahout-core-0.9-job.jar"

    # The arguments for this job
    # * input - the path to the data uploaded to HDInsight
    # * output - the path to store output data
    # * tempDir - the directory for temp files
    $jobArguments = "-s", "SIMILARITY_COOCCURRENCE",
                    "--input", "wasb:///example/data/u.data",
                    "--output", "wasb:///example/out",
                    "--tempDir", "wasb:///temp/mahout"

    # Create the job definition
    $jobDefinition = New-AzureHDInsightMapReduceJobDefinition `
      -JarFile $jarFile `
      -ClassName "org.apache.mahout.cf.taste.hadoop.item.RecommenderJob" `
      -Arguments $jobArguments

    # Start the job
    $job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition

    # Wait on the job to complete
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzureHDInsightJob -Job $job

    # Write out any error information
    Write-Host "STDERR"
    Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardError

> [WACOM.NOTE] Mahout 作业不会删除处理该作业时创建的临时数据。正因如此，我们在示例作业中指定了`--tempDir` 参数 - 为了将临时文件隔离到特定的路径以方便删除。
>
> 若要删除这些文件，你可以使用[在 HDInsight 中上载 Hadoop 作业的数据][upload]中所述的实用工具之一。或者使用`Remove-HDInsightFile` 函数（在 [HDInsight-Tools][tools] PowerShell 脚本中）。
>
> 如果你不删除临时文件或输出文件，则再次运行该作业时会收到错误。

Mahout 作业不会向 STDOUT 返回输出，而是将输出作为 **part-r-00000** 存储在指定的输出目录中。若要下载并查看该文件，请使用`Get-HDInsightFile` 函数（在 [HDInsight-Tools][tools] PowerShell 模块中）。

下面是该文件的内容示例：

    1   [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
    2   [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
    3   [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
    4   [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]

第一列为`userID`. “[”和“]”中包含的值为`movieId`:`recommendationScore`.

### 查看输出

尽管生成的输出也许能够正常地在应用程序中使用，但它的用户可读性并不太好。前面已提取到 **ml-100k** 文件夹中的其他某些文件可用于将`movieId` 解析为电影名称。**ml-100k** 文件夹中有一个 Python 脚本 (**show_recommendations.py**) 将执行此解析，不过，你也可以使用以下 PowerShell 脚本。

    <#
    .SYNOPSIS
        Displays recommendations for movies.
    .DESCRIPTION
        Displays recommendations generated by Mahout
        with HDInsight example in a human readable format.
    .EXAMPLE
        .\Show-Recommendation -userId 4
            -userDataFile "u.data"
            -movieFile "u.item"
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

若要使用此脚本，你必须事先提取 **ml-100k** 文件夹，以及 Mahout 作业生成的 **part-r-00000** 输出文件的本地副本。以下示例演示了如何运行该脚本。

    PS C:\> show-recommendation.ps1 -userId 4 -userDataFile .\ml-100k\u.data -movieFile .\ml-100k\u.item -recommendationFile .\output.txt

> [WACOM.NOTE] 示例 Python 脚本 **show_recommendations.py** 使用相同的参数。

输出应如下所示。

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

## <a name="classify"></a>使用 Hadoop 命令行为数据分类

Mahout 提供的分类方法之一是生成[随机林][forest]。这是一个多步骤过程，涉及到使用训练数据来生成决策树，然后使用决策树为数据分类。此过程使用 Mahout 提供的 **org.apache.mahout.classifier.df.tools.Describe** 类，目前必须使用 Hadoop 命令行来运行。

### 加载数据

当前的 Mahout 实现与加州大学欧文分校 (UCI) 的存储库格式兼容 [为什么这很重要？此格式是什么格式？]

1. 从 [http://nsl.cs.unb.ca/NSL-KDD/](http://nsl.cs.unb.ca/NSL-KDD/) 下载以下文件。
 - [KDDTrain+.ARFF](http://nsl.cs.unb.ca/NSL-KDD/KDDTrain+.arff) - 训练文件
 - [KDDTest+.ARFF](http://nsl.cs.unb.ca/NSL-KDD/KDDTest+.arff) - 测试数据


2. 打开每个文件，并删除顶部以“@”开头的行，然后保存文件。如果不删除这些行，则在 Mahout 中使用这些数据时会收到错误。

3.  将文件上载到 **example/data**。你可以使用 `Add-HDInsightFile` 函数（在 [HDInsight-Tools][tools] PowerShell 模块中）执行此操作。

### 运行作业

1. 由于需要从 Hadoop 命令行运行此作业，因此，你必须先通过 [Azure 管理门户][management]启用远程桌面。在门户中选择你的 HDInsight 群集，然后在“配置”页的底部选择“启用远程”。

	![启用远程][enableremote]

    出现提示时，输入用于启动远程会话的用户名和密码。

2. 启用远程访问后，选择“连接”以开始建立连接。随后将会下载可用于启动远程桌面会话的 **.rdp** 文件。

	![连接][connect]

3. 建立连接后，使用“Hadoop 命令行”图标打开 Hadoop 命令行。

	![hadoop cli][hadoopcli]

4. 在 Mahout 中使用以下命令生成文件描述符 (**KDDTrain+.info**)。

        hadoop jar "c:/apps/dist/mahout-0.9.0.2.1.3.0-1887/examples/target/mahout-examples-0.9.0.2.1.3.0-1887-job.jar" org.apache.mahout.classifier.df.tools.Describe -p "wasb:///example/data/KDDTrain+.arff" -f "wasb:///example/data/KDDTrain+.info" -d N 3 C 2 N C 4 N C 8 N 2 C 19 N L

    其中，`N 3 C 2 N C 4 N C 8 N 2 C 19 N L` 描述文件中数据的属性。例如，1 表示数字属性，2 表示类别属性，L 表示标签。

5. 使用以下命令生成决策树的林。

		hadoop jar c:/apps/dist/mahout-0.9.0.2.1.3.0-1887/examples/target/mahout-examples-0.9.0.2.1.3.0-1887-job.jar org.apache.mahout.classifier.df.mapreduce.BuildForest -Dmapred.max.split.size=1874231 -d wasb:///example/data/KDDTrain+.arff -ds wasb:///example/data/KDDTrain+.info -sl 5 -p -t 100 -o nsl-forest

    此操作的输出将存储在 **nsl-forest** 目录中，该目录位于 HDInsight 群集存储中的 \_\_wasb://user/\<username\>/nsl-forest/nsl-forest.seq 下。\<username\> 表示用于远程桌面会话的用户名。用户不可读取此文件。

6. 通过使用以下命令为 **KDDTest+.arff** 数据集分类来测试该林。

		hadoop jar c:/apps/dist/mahout-0.9.0.2.1.3.0-1887/examples/target/mahout-examples-0.9.0.2.1.3.0-1887-job.jar org.apache.mahout.classifier.df.mapreduce.TestForest -i wasb:///example/data/KDDTest+.arff -ds wasb:///example/data/KDDTrain+.info -m nsl-forest -a -mr -o wasb:///example/data/predictions

    该命令将返回类似于下面的有关分类过程的摘要信息。

        14/07/02 14:29:28 INFO mapreduce.TestForest:

        =======================================================
        Summary
        -------------------------------------------------------
        Correctly Classified Instances          :      17560       77.8921%
        Incorrectly Classified Instances        :       4984       22.1079%
        Total Classified Instances              :      22544

        =======================================================
        Confusion Matrix
        -------------------------------------------------------
        a       b       <--Classified as
        9437    274      |  9711        a     = normal
        4710    8123     |  12833       b     = anomaly

        =======================================================
        Statistics
        -------------------------------------------------------
        Kappa                                       0.5728
        Accuracy                                   77.8921%
        Reliability                                53.4921%
        Reliability (standard deviation)            0.4933

此作业还将在 **wasb:///example/data/predictions/KDDTest+.arff.out** 中生成一个文件，但是，用户无法读取此文件。

> [WACOM.NOTE] Mahout 作业不会覆盖文件。如果要再次运行这些作业，则必须删除以前的作业创建的文件。

## <a name="troubleshooting"></a>故障排除

### <a name="install"></a>安装 Mahout

Mahout 已安装在 HDInsight 3.1 群集上，你可以使用以下步骤手动将其安装在 3.0 或 2.1 群集上。

1. 要使用的 Mahout 版本取决于群集的 HDInsight 版本。可以在 [Azure PowerShell][aps] 中使用以下命令找到群集版本：

        PS C:\> Get-AzureHDInsightCluster -Name YourClusterName | Select version

- **对于 HDInsight 2.1**，可以下载包含 [Mahout 0.9](http://repo2.maven.org/maven2/org/apache/mahout/mahout-core/0.9/mahout-core-0.9-job.jar) 的 jar 文件。

- **对于 HDInsight 3.0**，必须[从源生成 Mahout][build] 并指定 HDInsight 提供的 Hadoop 版本。安装生成页上列出的必备组件，下载源，然后使用以下命令创建 Mahout jar 文件。

            mvn -Dhadoop2.version=2.2.0 -DskipTests clean package

        Once the build completes, the jar file will be created at __mahout\mrlegacy\target\mahout-mrlegacy-1.0-SNAPSHOT-job.jar__.

        > [WACOM.NOTE] Once Mahout 1.0 is released, you should be able to use the pre-built packages with HDInsight 3.0.

1. 将该 jar 文件上载到群集默认存储的 **example/jars** 中。以下示例使用 [send-hdinsight][sendhdinsight] 脚本上载该文件。

        PS C:\> .\Send-HDInsight -LocalPath "path\to\mahout-core-0.9-job.jar" -DestinationPath "example/jars/mahout-core-0.9-job.jar" -ClusterName "your cluster name"

### 无法覆盖文件

Mahout 作业不会清理处理期间创建的临时文件。此外，这些作业不会覆盖现有的输出文件。

若要避免运行 Mahout 作业时出错，请在每次运行作业之前删除临时文件和输出文件，或者使用唯一的临时目录名称和输出目录名称。

### 找不到 jar 文件

尽管 HDInsight 3.1 包含了 Mahout，但路径和文件名包含的是群集上安装的 Mahout 的版本号。本教程中的示例 PowerShell 脚本使用的路径的有效截止期为 2014 年 7 月，但是，将来对 HDInsight 做出更新后，版本号将发生更改。若要确定群集的 Mahout jar 文件的当前路径，请使用以下 PowerShell 命令，然后修改脚本以引用返回的文件路径。

    Use-AzureHDInsightCluster -Name $clusterName
    $jarFile = Invoke-Hive -Query '!${env:COMSPEC} /c dir /b /s ${env:MAHOUT_HOME}\examples\target\*-job.jar'

### <a name="nopowershell"></a>不适用于 PowerShell 的类

如果从 PowerShell 使用以下类，使用这些类的 Mahout 作业将返回各种错误。

- org.apache.mahout.utils.clustering.ClusterDumper
- org.apache.mahout.utils.SequenceFileDumper
- org.apache.mahout.utils.vectors.lucene.Driver
- org.apache.mahout.utils.vectors.arff.Driver
- org.apache.mahout.text.WikipediaToSequenceFile
- org.apache.mahout.clustering.streaming.tools.ResplitSequenceFiles
- org.apache.mahout.clustering.streaming.tools.ClusterQualitySummarizer
- org.apache.mahout.classifier.sgd.TrainLogistic
- org.apache.mahout.classifier.sgd.RunLogistic
- org.apache.mahout.classifier.sgd.TrainAdaptiveLogistic
- org.apache.mahout.classifier.sgd.ValidateAdaptiveLogistic
- org.apache.mahout.classifier.sgd.RunAdaptiveLogistic
- org.apache.mahout.classifier.sequencelearning.hmm.BaumWelchTrainer
- org.apache.mahout.classifier.sequencelearning.hmm.ViterbiEvaluator
- org.apache.mahout.classifier.sequencelearning.hmm.RandomSequenceGenerator
- org.apache.mahout.classifier.df.tools.Describe

若要运行使用这些类的作业，请连接到 HDInsight 群集，然后使用 Hadoop 命令行运行这些作业。有关示例，请参阅[使用 Hadoop 命令行为数据分类](#classify)。

[build]: http://mahout.apache.org/developers/buildingmahout.html
[aps]: http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-get-started/
[upload]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-upload-data/
[ml]: http://en.wikipedia.org/wiki/Machine_learning
[forest]: http://en.wikipedia.org/wiki/Random_forest
[management]: https://manage.windowsazure.cn/
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools