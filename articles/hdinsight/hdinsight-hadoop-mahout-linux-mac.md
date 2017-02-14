<!-- not suitable for Mooncake -->

<properties
    pageTitle="使用 Mahout 和 HDInsight (SSH) 生成推荐 | Azure"
    description="了解如何使用 Apache Mahout 机器学习库通过 HDInsight (Hadoop) 生成电影推荐。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="c78ec37c-9a8c-4bb6-9e38-0bdb9e89fbd7"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 通过 HDInsight (SSH) 中基于 Linux 的 Hadoop 使用 Apache Mahout 生成电影推荐

[AZURE.INCLUDE [mahout-selector](../../includes/hdinsight-selector-mahout.md)]

了解如何使用 [Apache Mahout](http://mahout.apache.org) 机器学习库通过 Azure HDInsight 生成电影推荐。

Mahout 是适用于 Apache Hadoop 的[计算机学习][ml]库。Mahout 包含用于处理数据的算法，例如筛选、分类和群集。在本文中，用户使用推荐引擎根据好友看过的电影生成电影推荐。

## 先决条件

* 基于 Linux 的 HDInsight 群集。有关创建该群集的信息，请参阅[开始在 HDInsight 中使用基于 Linux 的 Hadoop][getstarted]。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## Mahout 版本控制

有关 HDInsight 中 Mahout 版本的详细信息，请参阅 [HDInsight 版本和 Hadoop 组件](/documentation/articles/hdinsight-component-versioning/)。

## <a name="recommendations"></a>了解建议

Mahout 提供的功能之一是推荐引擎。此引擎接受 `userID`、`itemId` 和 `prefValue` 格式（此项的用户偏好）的数据。然后，Mahout 将执行共同匹配项分析，以确定：*偏好某个项的用户也偏好其他类似项*。随后，Mahout 确定拥有类似项偏好的用户，这些偏好可用于推荐。

下面的工作流是使用电影数据的极其简单的示例：

* **共同匹配项**：Joe、Alice 和 Bob 都喜欢电影《星球大战》、《帝国反击战》和《绝地归来》。Mahout 可确定喜欢以上电影之一的用户也喜欢其他两部。

* **共同匹配项**：Bob 和 Alice 还喜欢电影《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。Mahout 可确定喜欢前面三部电影的用户也喜欢这三部电影。

* **类似性推荐**：由于 Joe 喜欢前三部电影，Mahout 会查看具有类似偏好的其他人喜欢、但 Joe 还未观看过（喜欢/评价）的电影。在本例中，Mahout 推荐《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。

### 了解数据

为方便起见，[GroupLens 研究][movielens]以兼容 Mahout 的格式提供电影的评价数据。此数据在 `/HdiSamples/HdiSamples/MahoutMovieData` 中你的群集的默认存储中可用。

存在两个文件，`moviedb.txt`（有关影片的信息）和 `user-ratings.txt`。user-ratings.txt 文件用于分析过程中，而 moviedb.txt 用于在显示分析结果时，提供用户友好的文本信息。

user-ratings.txt 中包含的数据具有 `userID`、`movieID`、`userRating` 和 `timestamp` 结构，它将告诉我们每个用户对电影评级的情况。下面是数据的示例：

    196    242    3    881250949
    186    302    3    891717742
    22    377    1    878887116
    244    51    2    880606923
    166    346    1    886397596

## 运行分析

通过 SSH 到群集的连接，使用以下命令运行推荐作业：

    mahout recommenditembased -s SIMILARITY_COOCCURRENCE -i /HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt -o /example/data/mahoutout --tempDir /temp/mahouttemp

> [AZURE.NOTE]
该作业可能需要几分钟才能完成，并可能运行多个 MapReduce 作业。

## 查看输出

1. 作业完成后，使用以下命令查看生成的输出：

        hdfs dfs -text /example/data/mahoutout/part-r-00000

    输出将如下所示：
   
        1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
        2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
        3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
        4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]
   
    第一列是 `userID`。“[”和“]”中包含的值为 `movieId`:`recommendationScore`。

2. 可以使用输出和 moviedb.txt 显示更多适用于用户的信息。首先需使用以下命令按本地方式复制文件：

        hdfs dfs -get /example/data/mahoutout/part-r-00000 recommendations.txt
        hdfs dfs -get /HdiSamples/HdiSamples/MahoutMovieData/* .

    这会将输出数据以及电影数据文件复制到当前目录中名为 **recommendations.txt** 的文件。

3. 使用以下命令创建新的 Python 脚本，该脚本查找电影名称中是否存在建议输出中的数据：

        nano show_recommendations.py

    编辑器打开后，使用以下文本作为该文件的内容：

        #!/usr/bin/env python

        import sys

        if len(sys.argv) != 5:
                print "Arguments: userId userDataFilename movieFilename recommendationFilename"
                sys.exit(1)

        userId, userDataFilename, movieFilename, recommendationFilename = sys.argv[1:]

        print "Reading Movies Descriptions"
        movieFile = open(movieFilename)
        movieById = {}
        for line in movieFile:
                tokens = line.split("|")
                movieById[tokens[0]] = tokens[1:]
        movieFile.close()

        print "Reading Rated Movies"
        userDataFile = open(userDataFilename)
        ratedMovieIds = []
        for line in userDataFile:
                tokens = line.split("\t")
                if tokens[0] == userId:
                        ratedMovieIds.append((tokens[1],tokens[2]))
        userDataFile.close()

        print "Reading Recommendations"
        recommendationFile = open(recommendationFilename)
        recommendations = []
        for line in recommendationFile:
                tokens = line.split("\t")
                if tokens[0] == userId:
                        movieIdAndScores = tokens[1].strip("[]\n").split(",")
                        recommendations = [ movieIdAndScore.split(":") for movieIdAndScore in movieIdAndScores ]
                        break
        recommendationFile.close()

        print "Rated Movies"
        print "------------------------"
        for movieId, rating in ratedMovieIds:
                print "%s, rating=%s" % (movieById[movieId][0], rating)
        print "------------------------"

        print "Recommended Movies"
        print "------------------------"
        for movieId, score in recommendations:
                print "%s, score=%s" % (movieById[movieId][0], score)
        print "------------------------"

    按 **Ctrl-X**、**Y**，最后按 **Enter** 来保存数据。
    
4. 使用以下命令以使该文件成为可执行文件：

        chmod +x show_recommendations.py

5. 运行 Python 脚本。以下命令假设用户处于内含所有已下载文件的目录中：

        ./show_recommendations.py 4 user-ratings.txt moviedb.txt recommendations.txt

    此命令查看为用户 ID 4 生成的建议。
   
    * **user-ratings.txt** 文件用于检索用户评价过的电影。

    * **moviedb.txt** 文件用于检索电影的名称。

    * **recommendations.txt** 用于检索此用户的电影建议。
     
    此命令的输出类似于以下文本：

           Reading Movies Descriptions
           Reading Rated Movies
           Reading Recommendations
     
         ##   Rated Movies
           Mimic (1997), rating=3
           Ulee's Gold (1997), rating=5
           Incognito (1997), rating=5
           One Flew Over the Cuckoo's Nest (1975), rating=4
           Event Horizon (1997), rating=4
           Client, The (1994), rating=3
           Liar Liar (1997), rating=5
           Scream (1996), rating=4
           Star Wars (1977), rating=5
           Wedding Singer, The (1998), rating=5
           Starship Troopers (1997), rating=4
           Air Force One (1997), rating=5
           Conspiracy Theory (1997), rating=3
           Contact (1997), rating=5
           Indiana Jones and the Last Crusade (1989), rating=3
           Desperate Measures (1998), rating=5
           Seven (Se7en) (1995), rating=4
           Cop Land (1997), rating=5
           Lost Highway (1997), rating=5
           Assignment, The (1997), rating=5
           Blues Brothers 2000 (1998), rating=5
           Spawn (1997), rating=2
           Wonderland (1997), rating=5
     
         ##   In & Out (1997), rating=5
         ##   Recommended Movies
           Seven Years in Tibet (1997), score=5.0
           Indiana Jones and the Last Crusade (1989), score=5.0
           Jaws (1975), score=5.0
           Sense and Sensibility (1995), score=5.0
           Independence Day (ID4) (1996), score=5.0
           My Best Friend's Wedding (1997), score=5.0
           Jerry Maguire (1996), score=5.0
           Scream 2 (1997), score=5.0
           Time to Kill, A (1996), score=5.0
     
         ##   Rock, The (1996), score=5.0

## 删除临时数据

Mahout 作业不删除在处理作业时创建的临时数据。在示例作业中指定 `--tempDir` 参数，以将临时文件隔离到特定路径中轻松删除。若要删除临时文件，请使用以下命令：

    hdfs dfs -rm -f -r /temp/mahouttemp

> [AZURE.WARNING]
如需再次运行此命令，则还必须删除输出目录。使用以下命令删除此目录：
> <p>
> ```hdfs dfs -rm -f -r /example/data/mahoutout```  


## 后续步骤

既已学习如何使用 Mahout，可探索在 HDInsight 上处理数据的其他方式：

* [Hive 和 HDInsight](/documentation/articles/hdinsight-use-hive/)
* [Pig 和 HDInsight](/documentation/articles/hdinsight-use-pig/)
* [MapReduce 和 HDInsight](/documentation/articles/hdinsight-use-mapreduce/)

[build]: http://mahout.apache.org/developers/buildingmahout.html
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

<!---HONumber=Mooncake_0120_2017-->