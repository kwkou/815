<properties linkid="manage-services-hdinsight-recommendation-engine-using-mahout" urlDisplayName="Hadoop Recommendation Engine" pageTitle="Hadoop recommendation engine (.NET) | Azure" metaKeywords="Azure Apache Mahout, Azure recommendation example, Azure recommendation tutorial, Azure recommendation engine" description="A tutorial that teaches how to use the Apache Mahout recommendation engine with Azure to create song suggestions based on listening habits." disqusComments="1" umbracoNaviHide="1" title="Simple recommendation engine using Apache Mahout" authors="jgao" manager="paulettm" editor="cgronlun" />

# 使用 Apache Mahout 的简单推荐引擎

Apache Mahout™ 是一个专用于可缩放的计算机学习应用程序的计算机学习库。推荐引擎是目前使用的最流行的计算机学习应用程序类型之一，包含许多很明显是市场营销应用程序的应用程序。

Apache Mahout 提供基于项目的协作筛选的内置实现。此方法广泛用于执行建议数据挖掘。基于项目的协作筛选由 Amazon.com 开发。这里的理念是，用于展示项目喜好之间关联的用户首选项中的数据可用于从类似组推断未来用户的偏好。

在本教程中，你使用 [Million Song Dataset][] 网站，并下载[数据集][]来根据用户过去的听歌习惯为其创建歌曲推荐。

你将了解到以下内容：

-   如何使用推荐引擎

本教程包含以下部分：

1.  [安装和配置][]
2.  [检查数据并设置数据的格式][]
3.  [安装 Mahout][]
4.  [运行 Mahout 作业][]

## 安装和配置

本教程假定你已安装了 Azure 和 HDinsight 预览版，并且你已创建了一个可在其上运行示例的 HDInsight 群集。如果你尚未执行这些操作，请查阅 [Azure HDInsight 入门][]教程，找到有关如何满足这些先决条件的说明。

## 检查数据并设置数据的格式

此示例处理用户表达对特定歌曲的喜好的方式。假设用户听一首歌的次数提供了用户对那首歌的喜好的度量。在喜好数据中检测到的模式可用于根据用户表达的某些音乐喜好预测未来的用户首选项。你可以在 [Echo Nest 偏好配置文件子集][Million Song Dataset]网页的“描述” 部分中，查看此数据集的示例：

![Echo Nest 偏好配置文件子集][]

### 来自 Million Song Dataset 的示例数据

若要将此数据集用于 Mahout，需要执行以下两项操作：

1.  将歌曲和用户的 ID 转换为整数值。
2.  将新值及其排名保存到一个以逗号分隔的文件。

如果你未安装 Visual Studio 2010，请跳过此步骤并转到“运行 Mahout 作业”一节，以获取预生成的版本。

首先启动 Visual Studio 2010。在 Visual Studio 中，选择“文件”-\>“新建”-\>“项目” 。在“已安装的模板” 窗格中的 **Visual C\#** 节点内，选择“窗口” 类别，然后从列表中选择“控制台应用程序” 。将项目命名为“ConvertToMahoutInput”，然后单击“确定” 按钮。

![创建控制台应用程序][]

### 创建控制台应用程序

1.  创建应用程序后，打开 **Program.cs** 文件并将以下静态成员添加到 **Program** 类：

        const char tab = '\u0009';
        static Dictionary<string, int> usersMapping = new Dictionary<string, int>();
        static Dictionary<string, int> songMapping = new Dictionary<string, int>(); 

2.  接下来，添加 `using System.IO;` 语句，并使用以下代码填充 **Main** 方法：

        var inputStream = File.Open(args[0], FileMode.Open);
        var reader = new StreamReader(inputStream);

        var outStream = File.Open("mInput.txt", FileMode.OpenOrCreate);
        var writer = new StreamWriter(outStream);

        var i = 1;

        var line = reader.ReadLine();
        while (!string.IsNullOrWhiteSpace(line))
        {
        i++;
        if (i > 5000)
        break;
        var outLine = line.Split(tab);

        int user = GetUser(outLine[0]);
        int song = GetSong(outLine[1]);

        writer.Write(user);
        writer.Write(',');
        writer.Write(song);
        writer.Write(',');
        writer.WriteLine(outLine[2]);

        line = reader.ReadLine();
        }

        Console.WriteLine("saved {0} lines to {1}", i, args[0]);

        reader.Close();
        writer.Close();

        SaveMapping(usersMapping, "users.csv");
        SaveMapping(songMapping, "mInput.csv");

        Console.WriteLine("Mapping saved");
        Console.ReadKey();

3.  现在，创建 **GetUser** 和 **GetSong** 函数，这两个函数将 ID 转换为整数：

        static int GetUser(string user)
        {
        if (!usersMapping.ContainsKey(user))
        usersMapping.Add(user, usersMapping.Count + 1);

        return usersMapping[user];
        }

        static int GetSong(string song)
        {
        if (!songMapping.ContainsKey(song))
        songMapping.Add(song, songMapping.Count + 1);

        return songMapping[song];
        }

4.  最后，创建一个实现 SaveMapping 方法的实用程序，该方法将这两个映射字典保存为 .csv 文件。

        static void SaveMapping(Dictionary<string, int> mapping, string fileName)
        {
        var stream = File.Open(fileName, FileMode.Create);
        var writer = new StreamWriter(stream);

        foreach (var key in mapping.Keys)
            {
        writer.Write(key);
        writer.Write(',');
        writer.WriteLine(mapping[key]);
            }

        writer.Close();
        }

5.  从[此链接][数据集]下载示例数据。下载完成后，打开 **train\_triplets.txt.zip** 并提取 **train\_triplets.txt**。

    运行此实用程序时，在 **train\_triplets.txt** 的位置附带一个命令行参数。为此，请右键单击**解决方案资源管理器**中的 **ConvertToMahoutInput** 项目节点，然后选择“属性” 。在项目属性页上，选择左侧的“调试” 选项卡，并将 \<localpath\>train\_triplets.txt 的路径添加到“命令行参数” 文本框中：

    ![设置命令行参数][]

### 设置命令行参数

-   按 **F5** 运行程序。完成后，从此项目的保存位置打开 **bin\\Debug** 文件夹并查看该实用程序的输出。你会找到 users.txt 和 mInput.txt

## 安装 Mahout

-   打开 HDInsight 群集门户，然后单击“远程桌面” 图标。

    ![“管理群集”图标][]

### “远程桌面”图标

默认情况下，HDInsight 未提供 Mahout。不过，由于它是 Hadoop 生态系统的一部分，因此可以从 [Mahout][] 网站下载它。最新版本是 0.7，但此指令集兼容版本 0.5 或 0.7。

1.  首先，将 [Mahout 0.7 版][]下载到你的本地计算机上。

2.  然后，将它复制到群集上，方法是：选择本地 zip 文件，按 control-v 进行复制，然后将它粘贴到你的 Hadoop 群集。

    ![上载 Mahout][]

### 将 Mahout 复制到头节点

1.  最后，在复制过程完成后右键单击 zip 文件，将 Mahout 分发提取到 C:\\apps\\dist。现在你已将 Mahout 安装在 C:\\apps\\dist\\mahout-distriution-0.7 中。

2.  为简单起见，将该文件夹重命名为 c:\\apps\\dist\\mahout-0.7。

### 运行 Mahout 作业

1.  将 **bin\\Debug** 文件夹中的 **mInput.txt** 文件复制到远程群集上的 **c:\\**。复制该文件后，将其解压缩。按上一节所述，将文件复制到远程 RDP 会话，方法是：在本地计算机上选择文件后按 control-C，然后在“RDP 会话”窗口上按 control-v。

2.  创建一个文件，其中包含你将为其生成推荐的用户的 ID。为此，只需在 **c:\\** 中创建一个名为 **users.txt** 的文本文件，其中包含单个用户的 ID。

**说明**

你可以通过将每个用户的 ID 放在单独的行上来为更多的用户生成推荐。如果你在生成 mInput.txt 和 users.txt 时遇到问题，可以在此 github [存储库][]中下载预生成的版本。 将所有内容下载为一个 [zip 文件][]是最方便的。找到 users.txt 和 mInput.txt，并将它们复制到远程群集上的文件夹 c:\\ 中

此时，你应打开 Hadoop 终端窗口，并导航到包含 users.txt 和 mInput.txt 的文件夹。

![Mahout 命令窗口][]

### Hadoop 命令窗口

1.  接下来，将 **mInput.txt** 和 **users.txt** 复制到 HDFS 中。为此，请打开 **Hadoop 命令外壳**并运行以下命令：

        hadoop dfs -copyFromLocal c:\mInput.txt input\mInput.txt
        hadoop dfs -copyFromLocal c:\users.txt input\users.txt

2.  验证文件是否已复制到 HDFS：

        hadoop fs -ls input/

    这时应显示：

        Found 2 items
        -rwxrwxrwx   1 writer supergroup      53322 2013-03-08 20:32 /user/writer/input/mInput.txt
        -rwxrwxrwx   1 writer supergroup        353 2013-03-08 20:33 /user/writer/input/users.txt

3.  现在，可以使用以下命令运行 Mahout 作业：

        c:\apps\dist\mahout-0.7\bin>hadoop jar c:\Apps\dist\mahout-0.7\mahout-core-0.7-job.jar org.apache.mahout.cf.taste.hadoop.item.RecommenderJob -s SIMILARITY_COOCCURRENCE --input=input/mInput.txt --output=output --usersFile=input/users.txt

    还有许多其他推荐引擎可用来比较不同用户特征因素的“距离”函数，你可以进行实验并将 Similarity 类更改为 SIMILARITY\_COOCCURRENCE、SIMILARITY\_LOGLIKELIHOOD、SIMILARITY\_TANIMOTO\_COEFFICIENT、SIMILARITY\_CITY\_BLOCK、SIMILARITY\_COSINE、SIMILARITY\_PEARSON\_CORRELATION、SIMILARITY\_EUCLIDEAN\_DISTANCE。就本教程来说，我们将不会详细探究 Mahout 的数据科学性。

4.  Mahout 作业会运行几分钟，之后将创建一个输出文件。运行以下命令以创建输出文件的本地副本：

        hadoop fs -copyToLocal output/part-r-00000 c:\output.txt

5.  从 **c:\\** 根目录文件夹打开 **output.txt** 文件并检查其内容。文件的结构如下所示：

        user    [song:rating,song:rating, ...]

6.  如果你要在群集上使用 Mahout 的其他部分，则应在 Mahout 分发的 bin 目录中保存 Mahout.cmd 的副本。

## 总结

推荐引擎为许多现代社交、在线购物、流媒体网站以及其他 Internet 站点提供重要功能。Mahout 提供易用的现成推荐引擎，其中包含许多有用的功能，并可在 Hadoop 上缩放。

## 后续步骤

虽然本文演示的是对 Hadoop 命令行的使用，但你也可以使用 HDInsight 交互式控制台执行任务。有关详细信息，请参阅 [指南：HDInsight 交互式 JavaScript 和 Hive 控制台][交互式控制台]。

  [Million Song Dataset]: http://labrosa.ee.columbia.edu/millionsong/tasteprofile
  [数据集]: http://labrosa.ee.columbia.edu/millionsong/sites/default/files/challenge/train_triplets.txt.zip
  [安装和配置]: #setup
  [检查数据并设置数据的格式]: #segment1
  [安装 Mahout]: #Segment2
  [运行 Mahout 作业]: #segment2
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [Echo Nest 偏好配置文件子集]: ./media/hdinsight-hadoop-recommendation-engine/the-echo-nest-taste-profile-subset.png
  [创建控制台应用程序]: ./media/hdinsight-hadoop-recommendation-engine/creating-a-console-application.png
  [设置命令行参数]: ./media/hdinsight-hadoop-recommendation-engine/setting-command-line-arguments.png
  [“管理群集”图标]: ./media/hdinsight-hadoop-recommendation-engine/the-manage-cluster-icon.png
  [Mahout]: http://mahout.apache.org/
  [Mahout 0.7 版]: http://www.apache.org/dyn/closer.cgi/mahout/
  [上载 Mahout]: ./media/hdinsight-hadoop-recommendation-engine/uploading-mahout.PNG
  [存储库]: https://github.com/wenming/BigDataSamples/tree/master/mahout
  [zip 文件]: https://github.com/wenming/BigDataSamples/archive/master.zip
  [Mahout 命令窗口]: ./media/hdinsight-hadoop-recommendation-engine/mahout-commandwindow.PNG
