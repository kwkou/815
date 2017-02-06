<!-- not suitable for Mooncake -->

<properties
    pageTitle="使用 HDInsight 上的 Apache Hive 分析 Twitter 数据 | Azure"
    description="了解如何使用 Python 存储包含特定关键字的推文，然后使用 HDInsight 上的 Hive 和 Hadoop 将原始 TWitter 数据转换为可搜索的 Hive 表。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="e1e249ed-5f57-40d6-b3bc-a1b4d9a871d3"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 使用 HDInsight 中的 Hive 分析 Twitter 数据

在此文档中，将通过使用 Twitter 流式处理 API 获取推文，然后在 HDInsight 群集上使用 Apache Hive 处理 JSON 格式数据。结果将是发送最多包含某个特定词的推文的 Twitter 用户列表。

> [AZURE.IMPORTANT]
本文档中的步骤已在基于 Linux 的 HDInsight 群集上进行测试。
>
> Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

### 先决条件
在开始阅读本教程前，你必须具有：

* **基于 Linux 的 Azure HDInsight 群集**。有关创建群集的信息，请参阅[基于 Linux 的 HDInsight 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)了解有关创建群集的步骤。
* **SSH 客户端**。有关将 SSH 与基于 Linux 的 HDInsight 配合使用的详细信息，请参阅以下文章：
  
  * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
  * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
* **Python** 和 [pip](https://pypi.python.org/pypi/pip)

## 获取 Twitter 源
Twitter 允许你通过 REST API 检索[每个推文的数据](https://dev.twitter.com/docs/platform-objects/tweets)作为 JavaScript 对象表示法 (JSON) 文档。要对 API 进行身份验证，需要 [OAuth](http://oauth.net)。你还必须创建包含用于访问 API 的设置的 *Twitter 应用程序*。

### 创建 Twitter 应用程序
1. 从 Web 浏览器登录到 [https://apps.twitter.com/](https://apps.twitter.com/)。单击“立即注册”链接（如果你没有 Twitter 帐户）。
2. 单击“新建应用”。
3. 输入“名称”、“说明”、“网站”。你可以为“网站”字段补充 URL。下表显示了一些要使用的示例值：
   
    | 字段 | 值 |
    |:--- |:--- |
    | 名称 |MyHDInsightApp |
    | 说明 |MyHDInsightApp |
    | 网站 |http://www.myhdinsightapp.com |
4. 选中“是，我同意”，然后单击“创建 Twitter 应用程序”。
5. 单击“权限”选项卡。默认权限为“只读”。这对于本教程来说已足够。
6. 单击“密钥和访问令牌”选项卡。
7. 单击“创建我的访问令牌”。
8. 在页面右上角单击“测试 OAuth”。
9. 记下“使用者密钥”、“使用者机密”、“访问令牌”和“访问令牌机密”。在后面的步骤中将会用到这些值。

> [AZURE.NOTE]
在 Windows 中使用 curl 命令时，请使用双引号（而不是单引号）括起选项值。

### 下载推文
以下 Python 代码将从 Twitter 下载 10,000 篇推文并将其保存到一个名为 **tweets.txt** 的文件中。

> [AZURE.NOTE]
由于已安装了 Python，请在 HDInsight 群集上执行以下步骤。
> 
> 

1. 使用 SSH 连接到 HDInsight 群集：
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn
   
    如果你使用了密码来保护 SSH 帐户，系统会提示你输入密码。如果你使用了公钥，则可能需要使用 `-i` 参数来指定匹配的私钥。例如，`ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn`。
   
    有关将 SSH 与基于 Linux 的 HDInsight 配合使用的详细信息，请参阅以下文章：
   
    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
2. 默认情况下，HDInsight 头节点上未安装 **pip** 实用程序。使用以下方法来安装，然后更新此实用程序：
   
        sudo apt-get install python-pip
        sudo pip install --upgrade pip
3. 用于下载推文的代码依赖于 [Tweepy](http://www.tweepy.org/) 和 [Progressbar](https://pypi.python.org/pypi/progressbar/2.2)。若要安装这些项，请使用以下命令：
   
        sudo apt-get install python-dev libffi-dev libssl-dev
        sudo apt-get remove python-openssl
        sudo pip install tweepy progressbar pyOpenSSL requests[security]
   
    > [AZURE.NOTE]
    有关删除 python-openssl，安装 python-dev、libffi-dev、libssl-dev、pyOpenSSL 和 requests[security] 的信息是为了避免从 Python 通过 SSL 连接到 Twitter 时出现 InsecurePlatform 警告。
    > 
    > Tweepy v3.2.0 用于避免在在处理推文时可能发生的[错误](https://github.com/tweepy/tweepy/issues/576)。
    > 
    > 
4. 使用以下命令创建一个名为 **gettweets.py** 的新文件：
   
        nano gettweets.py
5. 使用以下内容作为 **gettweets.py** 文件的内容。将“consumer\_secret”、“consumer\_key”、“access/\_token”和“access\_token\_secret”的占位符信息替换为 Twitter 应用程序的信息。
   
        #!/usr/bin/python
   
        from tweepy import Stream, OAuthHandler
        from tweepy.streaming import StreamListener
        from progressbar import ProgressBar, Percentage, Bar
        import json
        import sys
   
        #Twitter app information
        consumer_secret='Your consumer secret'
        consumer_key='Your consumer key'
        access_token='Your access token'
        access_token_secret='Your access token secret'
   
        #The number of tweets we want to get
        max_tweets=10000
   
        #Create the listener class that will receive and save tweets
        class listener(StreamListener):
            #On init, set the counter to zero and create a progress bar
            def __init__(self, api=None):
                self.num_tweets = 0
                self.pbar = ProgressBar(widgets=[Percentage(), Bar()], maxval=max_tweets).start()
   
            #When data is received, do this
            def on_data(self, data):
                #Append the tweet to the 'tweets.txt' file
                with open('tweets.txt', 'a') as tweet_file:
                    tweet_file.write(data)
                    #Increment the number of tweets
                    self.num_tweets += 1
                    #Check to see if we have hit max_tweets and exit if so
                    if self.num_tweets >= max_tweets:
                        self.pbar.finish()
                        sys.exit(0)
                    else:
                        #increment the progress bar
                        self.pbar.update(self.num_tweets)
                return True
   
            #Handle any errors that may occur
            def on_error(self, status):
                print status
   
        #Get the OAuth token
        auth = OAuthHandler(consumer_key, consumer_secret)
        auth.set_access_token(access_token, access_token_secret)
        #Use the listener class for stream processing
        twitterStream = Stream(auth, listener())
        #Filter for these topics
        twitterStream.filter(track=["azure","cloud","hdinsight"])
6. 使用 **Ctrl + X**，然后单击 **Y** 以保存该文件。
7. 使用以下命令运行该文件并下载推文：
   
        python gettweets.py
   
    进度指示器将出现，并在推文已下载并保存到文件时计数达到 100%。
   
    > [AZURE.NOTE]
    如果进度条前移耗时过长，则应更改筛选器来跟踪趋势主题；当存在大量与你现在筛选的主题相关的推文时，可快速获取所需的 10000 条推文。
    > 
    > 

### 上载数据
若要将数据上载到 WASB（由 HDInsight 使用的分布式文件系统），请使用以下命令：

    hdfs dfs -mkdir -p /tutorials/twitter/data
    hdfs dfs -put tweets.txt /tutorials/twitter/data/tweets.txt

这将在群集中的所有节点都可以访问的位置中存储数据。

## 运行 HiveQL 作业
1. 使用以下命令来创建包含 HiveQL 语句的文件：
   
        nano twitter.hql
   
    将以下内容用作该文件的内容：
   
        set hive.exec.dynamic.partition = true;
        set hive.exec.dynamic.partition.mode = nonstrict;
        -- Drop table, if it exists
        DROP TABLE tweets_raw;
        -- Create it, pointing toward the tweets logged from Twitter
        CREATE EXTERNAL TABLE tweets_raw (
            json_response STRING
        )
        STORED AS TEXTFILE LOCATION '/tutorials/twitter/data';
        -- Drop and recreate the destination table
        DROP TABLE tweets;
        CREATE TABLE tweets
        (
            id BIGINT,
            created_at STRING,
            created_at_date STRING,
            created_at_year STRING,
            created_at_month STRING,
            created_at_day STRING,
            created_at_time STRING,
            in_reply_to_user_id_str STRING,
            text STRING,
            contributors STRING,
            retweeted STRING,
            truncated STRING,
            coordinates STRING,
            source STRING,
            retweet_count INT,
            url STRING,
            hashtags array<STRING>,
            user_mentions array<STRING>,
            first_hashtag STRING,
            first_user_mention STRING,
            screen_name STRING,
            name STRING,
            followers_count INT,
            listed_count INT,
            friends_count INT,
            lang STRING,
            user_location STRING,
            time_zone STRING,
            profile_image_url STRING,
            json_response STRING
        );
        -- Select tweets from the imported data, parse the JSON,
        -- and insert into the tweets table
        FROM tweets_raw
        INSERT OVERWRITE TABLE tweets
        SELECT
            cast(get_json_object(json_response, '$.id_str') as BIGINT),
            get_json_object(json_response, '$.created_at'),
            concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
            substr (get_json_object(json_response, '$.created_at'),27,4)),
            substr (get_json_object(json_response, '$.created_at'),27,4),
            case substr (get_json_object(json_response,    '$.created_at'),5,3)
                when "Jan" then "01"
                when "Feb" then "02"
                when "Mar" then "03"
                when "Apr" then "04"
                when "May" then "05"
                when "Jun" then "06"
                when "Jul" then "07"
                when "Aug" then "08"
                when "Sep" then "09"
                when "Oct" then "10"
                when "Nov" then "11"
                when "Dec" then "12" end,
            substr (get_json_object(json_response, '$.created_at'),9,2),
            substr (get_json_object(json_response, '$.created_at'),12,8),
            get_json_object(json_response, '$.in_reply_to_user_id_str'),
            get_json_object(json_response, '$.text'),
            get_json_object(json_response, '$.contributors'),
            get_json_object(json_response, '$.retweeted'),
            get_json_object(json_response, '$.truncated'),
            get_json_object(json_response, '$.coordinates'),
            get_json_object(json_response, '$.source'),
            cast (get_json_object(json_response, '$.retweet_count') as INT),
            get_json_object(json_response, '$.entities.display_url'),
            array(
                trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[1].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[2].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[3].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[4].text')))),
            array(
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[1].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[2].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[3].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[4].screen_name')))),
            trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
            get_json_object(json_response, '$.user.screen_name'),
            get_json_object(json_response, '$.user.name'),
            cast (get_json_object(json_response, '$.user.followers_count') as INT),
            cast (get_json_object(json_response, '$.user.listed_count') as INT),
            cast (get_json_object(json_response, '$.user.friends_count') as INT),
            get_json_object(json_response, '$.user.lang'),
            get_json_object(json_response, '$.user.location'),
            get_json_object(json_response, '$.user.time_zone'),
            get_json_object(json_response, '$.user.profile_image_url'),
            json_response
        WHERE (length(json_response) > 500);
2. 按 **Ctrl + X**，然后按 **Y** 以保存该文件。
3. 使用以下命令运行该文件中包含的 HiveQL：
   
        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin -i twitter.hql
   
    这将加载 Hive shell，运行 **twitter.hql** 文件中的 HiveQL，并最终返回 `jdbc:hive2//localhost:10001/>` 提示符。
4. 在 Beeline 提示符下，使用以下项来验证是否可从 “twitter.hql”文件中的 HiveQL 创建的“tweets”表中选择数据：
   
        SELECT name, screen_name, count(1) as cc
            FROM tweets
            WHERE text like "%Azure%"
            GROUP BY name,screen_name
            ORDER BY cc DESC LIMIT 10;
   
    这将在消息文本中返回最多 10 篇包含 **Azure** 一词的推文。

## 后续步骤
在本教程中，我们已了解如何将非结构化 JSON 数据集转换为结构化 Hive 表，以便在 Azure 上使用 HDInsight 查询、探究和分析来自 Twitter 的数据。若要了解详细信息，请参阅以下文章：

* [开始使用 HDInsight](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)
* [使用 HDInsight 分析航班延误数据](/documentation/articles/hdinsight-analyze-flight-delay-data-linux/)

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[apache-hive-tutorial]: https://cwiki.apache.org/confluence/display/Hive/Tutorial

[twitter-streaming-api]: https://dev.twitter.com/docs/streaming-apis
[twitter-statuses-filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter

<!---HONumber=Mooncake_0120_2017-->