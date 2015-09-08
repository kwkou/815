<properties
	pageTitle="使用 Mahout 和 Hadoop 生成建议 |Windows Azure"
	description="了解如何使用 Apache Mahout 机器学习库通过基于 Linux 的 HDInsight (Hadoop) 生成电影推荐。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor=""/>

<tags
	ms.service="hdinsight"
	ms.date="06/16/2015"
	wacn.date="08/29/2015"/>

#通过 HDInsight 中基于 Linux 的 Hadoop（预览版）使用 Apache Mahout 生成电影推荐

[AZURE.INCLUDE [mahout-selector](../includes/hdinsight-selector-mahout.md)]

了解如何使用 [Apache Mahout](http://mahout.apache.org) 机器学习库通过 Azure HDInsight 生成电影推荐。

Mahout 是适用于 Apache Hadoop 的[计算机学习][ml]库。Mahout 包含用于处理数据的算法，例如筛选、分类和群集。在本文中，你将使用推荐引擎生成基于你的朋友看过的电影的电影推荐。

> [AZURE.NOTE]本文档中的步骤要求在 HDInsight 群集（预览版）上存在基于 Linux 的 Hadoop。有关如何将 Mahout 用于基于 Windows 的群集的信息，请参阅[通过 HDInsight 中基于 Windows 的 Hadoop（预览版）使用 Apache Mahout 生成电影推荐](/documentation/articles/hdinsight-mahout)

##先决条件

* 基于 Linux 的 HDInsight 上的 Hadoop 群集。有关创建该群集的信息，请参阅[开始在 HDInsight 中使用基于 Linux 的 Hadoop][getstarted]

##<a name="recommendations"></a>了解建议

由 Mahout 提供的功能之一是推荐引擎。此引擎接受 `userID`、`itemId` 和 `prefValue` 格式（项的用户首选项）的数据。然后，Mahout 将执行共现分析，以确定：_偏好某个项的用户也偏好其他类似项_。Mahout 然后确定拥有类似项首选项的用户，这些首选项可用于做出推荐。

下面是使用电影的极其简单的示例：

* __共现__：Joe、Alice 和 Bob 都喜欢电影_星球大战_、_帝国反击战_和_绝地大反击_。Mahout 可确定喜欢以上电影之一的用户也喜欢其他两部。

* __共现__：Bob 和 Alice 还喜欢电影_幽灵的威胁_、_克隆人的进攻_和_西斯的复仇_。Mahout 可确定喜欢前面三部电影的用户也喜欢这三部电影。

* __类似性推荐__：由于 Joe 喜欢前三部电影，Mahout 会查看具有类似首选项的其他人喜欢的电影，但是 Joe 还未观看过（喜欢/评价）。在这种情况下，Mahout 推荐《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。

##加载数据

为方便起见，[GroupLens 研究][movielens]以兼容 Mahout 的格式提供电影的评价数据。使用以下步骤来下载数据，然后将其加载到群集的默认存储：

1. 使用 SSH 连接到基于 Linux 的 HDInsight 群集。连接时要使用的地址为 `CLUSTERNAME-ssh.azurehdinsight.cn`，端口为 `22`。

	有关如何使用 SSH 连接到 HDInsight 的详细信息，请参阅以下文档：

    * **Linux、Unix 或 OS X 客户端**：请参阅[从 Linux、OS X 或 Unix 连接到基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix#connect-to-a-linux-based-hdinsight-cluster)

    * **Windows 客户端**：请参阅[从 Windows 连接到基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows#connect-to-a-linux-based-hdinsight-cluster)

2. 下载 MovieLens 100k 存档，其中包含 1000 名用户对 1700 部电影的 100,000 条评价信息。

        curl -O http://files.grouplens.org/datasets/movielens/ml-100k.zip

3. 使用以下命令解压存档文件：

        unzip ml-100k.zip

    此时会将内容提取到一个名为 **ml-100 k** 的新文件夹。

4. 使用以下命令将数据复制到 HDInsight 存储：

        cd ml-100k
        hadoop fs -copyFromLocal ml-100k/u.data /example/data/u.data

    此文件中包含的数据具有 `userID`、`movieID`、`userRating` 和 `timestamp` 结构，它将告诉我们每个用户对电影评级的情况。下面是数据的示例：


		196	242	3	881250949
		186	302	3	891717742
		22	377	1	878887116
		244	51	2	880606923
		166	346	1	886397596

##运行作业

使用以下命令来运行推荐作业：

	mahout recommenditembased -s SIMILARITY_COOCCURRENCE --input /example/data/u.data --output /example/data/mahoutout  --tempDir /temp/mahouttemp

> [AZURE.NOTE]该作业可能需要几分钟才能完成，并可能运行多个 MapReduce 作业。

##查看输出

1. 作业完成后，使用以下命令查看生成的输出：

		hadoop fs -text /example/data/mahoutout/part-r-00000

	输出将如下所示：

		1	[234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
		2	[282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
		3	[284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
		4	[690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]

	第一列是 `userID`。“[”和“]”中包含的值为 `movieId`:`recommendationScore`。

2. 可以利用 **ml-100k** 目录中包含的部分其他数据，从而使该数据在用户看来更亲切。首先，使用以下命令下载数据：

		hadoop fs -copyToLocal /example/data/mahoutout/part-r-00000 recommendations.txt

	这会将输出数据复制到当前目录中名为 **recommendations.txt** 的文件。

3. 使用以下命令来创建新的 Python 脚本，该脚本将查找电影名称中是否存在建议输出中的数据：

		nano show_recommendations.py

	编辑器打开后，使用以下内容作为该文件的内容：

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

3. 使用以下命令以使该文件成为可执行文件：

		chmod +x show_recommendations.py

4. 运行 Python 脚本：

		./show_recommendations.py 4 ml-100k/u.data ml-100k/u.item recommendations.txt

	这将查看为用户 ID 4 生成的建议。

	* **u.data** 文件用于检索用户评价过的电影
	* **u.item** 文件用于检索电影的名称
	* **recommendations.txt** 用于检索此用户的电影建议

	此命令的输出将类似于以下内容：

		Reading Movies Descriptions
		Reading Rated Movies
		Reading Recommendations
		Rated Movies
		------------------------
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
		In & Out (1997), rating=5
		------------------------
		Recommended Movies
		------------------------
		Seven Years in Tibet (1997), score=5.0
		Indiana Jones and the Last Crusade (1989), score=5.0
		Jaws (1975), score=5.0
		Sense and Sensibility (1995), score=5.0
		Independence Day (ID4) (1996), score=5.0
		My Best Friend's Wedding (1997), score=5.0
		Jerry Maguire (1996), score=5.0
		Scream 2 (1997), score=5.0
		Time to Kill, A (1996), score=5.0
		Rock, The (1996), score=5.0
		------------------------

##删除临时数据

Mahout 作业不删除在处理作业时创建的临时数据。在示例作业中指定 `--tempDir` 参数，以将临时文件隔离到特定路径中轻松删除。若要删除临时文件，请使用以下命令：

	hadoop fs -rm -f -r wasb:///temp/mahouttemp

> [AZURE.WARNING]如果你未删除临时文件或输出文件，则在使用相同的 `--tempDir` 路径重新运行作业时将会收到“无法覆盖文件”错误消息。

##后续步骤

现在，你已经学习了如何使用 Mahout，因此可以探索通过其他方式来使用 HDInsight 上的数据：

* [Hive 和 HDInsight](/documentation/articles/hdinsight-use-hive)
* [Pig 和 HDInsight](/documentation/articles/hdinsight-use-pig)
* [MapReduce 和 HDInsight](/documentation/articles/hdinsight-use-mapreduce)

[build]: http://mahout.apache.org/developers/buildingmahout.html
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started
[upload]: /documentation/articles/hdinsight-upload-data
[ml]: http://zh.wikipedia.org/wiki/Machine_learning
[forest]: http://zh.wikipedia.org/wiki/Random_forest
[management]: https://manage.windowsazure.cn/
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools
 

<!---HONumber=67-->