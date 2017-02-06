<!-- not suitable for Mooncake -->

<properties 
	pageTitle="使用 HBase 分析实时 Twitter 观点 | Azure" 
	description="了解如何通过在 HDInsight (Hadoop) 群集中使用 HBase 对 Twitter 中的大数据执行实时观点分析。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="mumian" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/09/2016" 
	wacn.date="02/06/2017" 
	ms.author="jgao"/>

# 在 HDInsight 中使用 HBase 分析实时 Twitter 观点

了解如何通过在 HDInsight (Hadoop) 群集中使用 HBase 对 Twitter 中的大数据执行实时[观点分析](http://en.wikipedia.org/wiki/Sentiment_analysis)。


社交网站是采用大数据的主要推动力之一。Twitter 等网站所提供的公共 API 是一类用于分析和了解流行趋势的有用数据源。在本教程中，你将开发一个控制台流式处理服务应用程序和一个 ASP.NET Web 应用程序用于执行以下操作：

![HDInsight HBase 分析 Twitter 观点][img-app-arch]

- 流式处理应用程序
	- 使用 Twitter 流 API 实时获取标记地理位置的推文
	- 评估这些推文的观点
	- 使用 Microsoft HBase SDK 将观点信息存储在 HBase 中
- Azure 网站应用程序
	- 使用 ASP.NET Web 应用程序将实时统计结果绘制在必应地图上。推文的视觉效果如下所示：

	![hdinsight.hbase.twitter.sentiment.bing.map][img-bing-map]
	
	你可以利用某些关键字来查询推文，以此感知推文中表达的观点是肯定的、否定的还是中立的。

有关完整的 Visual Studio 解决方案示例，请访问 GitHub：[实时社交观点分析应用](https://github.com/maxluk/tweet-sentiment)。































### 先决条件
在开始阅读本教程前，你必须具有：

- **HDInsight 中的 HBase 群集**。有关创建群集的说明，请参阅 [Get started using HBase with Hadoop in HDInsight][hbase-get-started]（开始在 HDInsight 中将 HBase 与 Hadoop 配合使用）。你将需要以下数据才能完成本教程：


	<table border="1">
	<tr><th>群集属性</th><th>说明</th></tr>
	<tr><td>HBase 群集名称</td><td>你的 HDInsight HBase 群集名称。例如：https://myhbase.azurehdinsight.cn/</td></tr>
	<tr><td>群集用户名</td><td>Hadoop 用户帐户名称。默认的 Hadoop 用户名为 <strong>admin</strong>。</td></tr>
	<tr><td>群集用户密码</td><td>Hadoop 群集用户密码。</td></tr>
	</table>

- 安装有 Visual Studio 2013 的**工作站**。有关说明，请参阅[安装 Visual Studio](http://msdn.microsoft.com/zh-cn/library/e2h7fzkw.aspx)。





## 创建 Twitter 应用程序 ID 和机密

Twitter 流 API 使用 [OAuth](http://oauth.net/) 对请求授权。使用 OAuth 的第一步是在 Twitter 开发人员网站上创建新的应用程序。

**创建 Twitter 应用程序 ID 和机密**

1. 登录到 [Twitter 应用](https://apps.twitter.com/)。单击“立即注册”链接（如果你没有 Twitter 帐户）。
2. 单击“创建新应用”。
3. 输入“名称”、“说明”和“网站”。Twitter 应用程序名称必须是唯一的名称。我们实际上不会用到“网站”字段。因此不必输入有效的 URL。
4. 选中“是，我同意”，然后单击“创建 Twitter 应用程序”。
5. 单击“权限”选项卡。默认权限为“只读”。这对于本教程来说已足够。
6. 单击“密钥和访问令牌”选项卡。
7. 单击“创建我的访问令牌”。
8. 在页面右上角单击“测试 OAuth”。
9. 复制“使用者密钥”、“使用者机密”、“访问令牌”和“访问令牌机密”值。本教程后面的步骤中将会用到这些值。

	![hdi.hbase.twitter.sentiment.twitter.app][img-twitter-app]






























## 创建 Twitter 流式处理服务

需要创建一个应用程序来获取推文、计算推文观点分数，以及将处理过的推文文字发送到 HBase。

**创建流式处理应用程序**

1. 打开 **Visual Studio**，创建名为 **TweetSentimentStreaming** 的 C# 控制台应用程序。
2. 在“包管理器控制台”中运行以下命令：

		Install-Package Microsoft.HBase.Client -version 0.4.2.0
		Install-Package TweetinviAPI -version 1.0.0.0

	这些命令将安装 [HBase .NET SDK](https://www.nuget.org/packages/Microsoft.HBase.Client/) 包（用于访问 HBase 群集的客户端库）和 [Tweetinvi API](https://www.nuget.org/packages/TweetinviAPI/) 包（用于访问 Twitter API）。

	> [AZURE.NOTE] 本文中使用的示例已使用上面指定的版本进行测试。可以删除 -version 开关来安装最新版本。

3. 从“解决方案资源管理器”中，将 **System.Configuration** 添加到引用中。
4. 将新的类文件添加到名为 **HBaseWriter.cs** 的项目中，然后将代码替换为以下内容：

		using System;
		using System.Collections.Generic;
		using System.IO;
		using System.Linq;
		using System.Text;
		using System.Threading;
		using Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling;
		using org.apache.hadoop.hbase.rest.protobuf.generated;
		using Microsoft.HBase.Client;
		using Tweetinvi.Models;

        namespace TweetSentimentStreaming
        {
            class HBaseWriter
            {
                // HDinsight HBase cluster and HBase table information
                const string CLUSTERNAME = "https://<Enter Your Cluster Name>.azurehdinsight.cn/";
                const string HADOOPUSERNAME = "admin"; //the default name is "admin"
                const string HADOOPUSERPASSWORD = "<Enter the Hadoop User Password>";

                const string HBASETABLENAME = "tweets_by_words";
				const string COUNT_ROW_KEY = "~ROWCOUNT";
				const string COUNT_COLUMN_NAME = "d:COUNT";
        		
				long rowCount = 0;

                // Sentiment dictionary file and the punctuation characters
                const string DICTIONARYFILENAME = @"..\..\dictionary.tsv";
                private static char[] _punctuationChars = new[] {
            ' ', '!', '"', '#', '$', '%', '&', '\'', '(', ')', '*', '+', ',', '-', '.', '/',   //ascii 23--47
            ':', ';', '<', '=', '>', '?', '@', '[', ']', '^', '_', '`', '{', '|', '}', '~' };   //ascii 58--64 + misc.

                // For writting to HBase
                HBaseClient client;

                // a sentiment dictionary for estimate sentiment. It is loaded from a physical file.
                Dictionary<string, DictionaryItem> dictionary;

                // use multithread write
                Thread writerThread;
                Queue<ITweet> queue = new Queue<ITweet>();
                bool threadRunning = true;

                // This function connects to HBase, loads the sentiment dictionary, and starts the thread for writting.
                public HBaseWriter()
                {
                    ClusterCredentials credentials = new ClusterCredentials(new Uri(CLUSTERNAME), HADOOPUSERNAME, HADOOPUSERPASSWORD);
                    client = new HBaseClient(credentials);

                    // create the HBase table if it doesn't exist
                    if (!client.ListTablesAsync().Result.name.Contains(HBASETABLENAME))
                    {
                        TableSchema tableSchema = new TableSchema();
                        tableSchema.name = HBASETABLENAME;
                        tableSchema.columns.Add(new ColumnSchema { name = "d" });
						client.CreateTableAsync(tableSchema).Wait();
                        Console.WriteLine("Table "{0}" is created.", HBASETABLENAME);
                    }

					// Read current row count cell
            		rowCount = GetRowCount();

                    // Load sentiment dictionary from a file
                    LoadDictionary();

                    // Start a thread for writting to HBase
                    writerThread = new Thread(new ThreadStart(WriterThreadFunction));
                    writerThread.Start();
                }

                ~HBaseWriter()
                {
                    threadRunning = false;
                }

				private long GetRowCount()
				{
					try
					{
						RequestOptions options = RequestOptions.GetDefaultOptions();
						options.RetryPolicy = RetryPolicy.NoRetry;
						var cellSet = client.GetCellsAsync(HBASETABLENAME, COUNT_ROW_KEY, null, null, options).Result;
						if (cellSet.rows.Count != 0)
						{
							var countCol = cellSet.rows[0].values.Find(cell => Encoding.UTF8.GetString(cell.column) == COUNT_COLUMN_NAME);
							if (countCol != null)
							{
								return Convert.ToInt64(Encoding.UTF8.GetString(countCol.data));
							}
						}
					}
					catch(Exception ex)
					{
						if (ex.InnerException.Message.Equals("The remote server returned an error: (404) Not Found.", StringComparison.OrdinalIgnoreCase))
						{
							return 0;
						}
						else
						{
							throw ex;
						}
						
					}

					return 0;
				}

                // Enqueue the Tweets received
                public void WriteTweet(ITweet tweet)
                {
                    lock (queue)
                    {
                        queue.Enqueue(tweet);
                    }
                }

                // Load sentiment dictionary from a file
                private void LoadDictionary()
                {
                    List<string> lines = File.ReadAllLines(DICTIONARYFILENAME).ToList();
                    var items = lines.Select(line =>
                    {
                        var fields = line.Split('\t');
                        var pos = 0;
                        return new DictionaryItem
                        {
                            Type = fields[pos++],
                            Length = Convert.ToInt32(fields[pos++]),
                            Word = fields[pos++],
                            Pos = fields[pos++],
                            Stemmed = fields[pos++],
                            Polarity = fields[pos++]
                        };
                    });

                    dictionary = new Dictionary<string, DictionaryItem>();
                    foreach (var item in items)
                    {
                        if (!dictionary.Keys.Contains(item.Word))
                        {
                            dictionary.Add(item.Word, item);
                        }
                    }
                }

                // Calculate sentiment score
                private int CalcSentimentScore(string[] words)
                {
                    Int32 total = 0;
                    foreach (string word in words)
                    {
                        if (dictionary.Keys.Contains(word))
                        {
                            switch (dictionary[word].Polarity)
                            {
                                case "negative": total -= 1; break;
                                case "positive": total += 1; break;
                            }
                        }
                    }
                    if (total > 0)
                    {
                        return 1;
                    }
                    else if (total < 0)
                    {
                        return -1;
                    }
                    else
                    {
                        return 0;
                    }
                }

                // Popular a CellSet object to be written into HBase
                private void CreateTweetByWordsCells(CellSet set, ITweet tweet)
                {
                    // Split the Tweet into words
                    string[] words = tweet.Text.ToLower().Split(_punctuationChars);

                    // Calculate sentiment score base on the words
                    int sentimentScore = CalcSentimentScore(words);
                    var word_pairs = words.Take(words.Length - 1)
                                        .Select((word, idx) => string.Format("{0} {1}", word, words[idx + 1]));
                    var all_words = words.Concat(word_pairs).ToList();

                    // For each word in the Tweet add a row to the HBase table
                    foreach (string word in all_words)
                    {
                        string time_index = (ulong.MaxValue - (ulong)tweet.CreatedAt.ToBinary()).ToString().PadLeft(20) + tweet.IdStr;
                        string key = word + "_" + time_index;

                        // Create a row
                        var row = new CellSet.Row { key = Encoding.UTF8.GetBytes(key) };

                        // Add columns to the row, including Tweet identifier, language, coordinator(if available), and sentiment 
                        var value = new Cell { column = Encoding.UTF8.GetBytes("d:id_str"), data = Encoding.UTF8.GetBytes(tweet.IdStr) };
                        row.values.Add(value);

                        value = new Cell { column = Encoding.UTF8.GetBytes("d:lang"), data = Encoding.UTF8.GetBytes(tweet.Language.ToString()) };
                        row.values.Add(value);

                        if (tweet.Coordinates != null)
                        {
                            var str = tweet.Coordinates.Longitude.ToString() + "," + tweet.Coordinates.Latitude.ToString();
                            value = new Cell { column = Encoding.UTF8.GetBytes("d:coor"), data = Encoding.UTF8.GetBytes(str) };
                            row.values.Add(value);
                        }

                        value = new Cell { column = Encoding.UTF8.GetBytes("d:sentiment"), data = Encoding.UTF8.GetBytes(sentimentScore.ToString()) };
                        row.values.Add(value);

                        set.rows.Add(row);
                    }
                }

                // Write a Tweet (CellSet) to HBase
                public void WriterThreadFunction()
                {
                    try
                    {
                        while (threadRunning)
                        {
                            if (queue.Count > 0)
                            {
                                CellSet set = new CellSet();
                                lock (queue)
                                {
                                    do
                                    {
                                        ITweet tweet = queue.Dequeue();
                                        CreateTweetByWordsCells(set, tweet);
                                    } while (queue.Count > 0);
                                }

                                // Write the Tweet by words cell set to the HBase table
								client.StoreCellsAsync(HBASETABLENAME, set).Wait();
                                Console.WriteLine("\tRows written: {0}", set.rows.Count);
                            }
                            Thread.Sleep(100);
                        }
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine("Exception: " + ex.Message);
                    }
                }
            }
            public class DictionaryItem
            {
                public string Type { get; set; }
                public int Length { get; set; }
                public string Word { get; set; }
                public string Pos { get; set; }
                public string Stemmed { get; set; }
                public string Polarity { get; set; }
            }
        }

6. 设置上述代码中的常量，包括 **CLUSTERNAME**、**HADOOPUSERNAME**、**HADOOPUSERPASSWORD** 和 DICTIONARYFILENAME。DICTIONARYFILENAME 是 direction.tsv 的文件名和位置。可以从 **https://hditutorialdata.blob.core.windows.net/twittersentiment/dictionary.tsv** 下载该文件。如果你想要更改 HBase 表名称，则必须相应地更改 Web 应用程序中的表名称。

7. 打开 **Program.cs**，将代码替换为以下内容：

        using System;
        using System.Diagnostics;
        using Tweetinvi;
        using Tweetinvi.Models;

        namespace TweetSentimentStreaming
        {
            class Program
            {
                const string TWITTERAPPACCESSTOKEN = "<Enter Twitter App Access Token>";
                const string TWITTERAPPACCESSTOKENSECRET = "<Enter Twitter Access Token Secret>";
                const string TWITTERAPPAPIKEY = "<Enter Twitter App API Key>";
                const string TWITTERAPPAPISECRET = "<Enter Twitter App API Secret>";

                static void Main(string[] args)
                {
                    Auth.SetUserCredentials(TWITTERAPPAPIKEY, TWITTERAPPAPISECRET, TWITTERAPPACCESSTOKEN, TWITTERAPPACCESSTOKENSECRET);

                    Stream_FilteredStreamExample();
                }

                private static void Stream_FilteredStreamExample()
                {
                    for (;;)
                    {
                        try
                        {
                            HBaseWriter hbase = new HBaseWriter();
                            var stream = Stream.CreateFilteredStream();
                            stream.AddLocation(new Coordinates(-180, -90), new Coordinates(180, 90)); 

                            var tweetCount = 0;
                            var timer = Stopwatch.StartNew();

                            stream.MatchingTweetReceived += (sender, args) =>
                            {
                                tweetCount++;
                                var tweet = args.Tweet;

                                // Write Tweets to HBase
                                hbase.WriteTweet(tweet);

                                if (timer.ElapsedMilliseconds > 1000)
                                {
                                    if (tweet.Coordinates != null)
                                    {
                                        Console.ForegroundColor = ConsoleColor.Green;
                                        Console.WriteLine("\n{0}: {1} {2}", tweet.Id, tweet.Language.ToString(), tweet.Text);
                                        Console.ForegroundColor = ConsoleColor.White;
                                        Console.WriteLine("\tLocation: {0}, {1}", tweet.Coordinates.Longitude, tweet.Coordinates.Latitude);
                                    }

                                    timer.Restart();
                                    Console.WriteLine("\tTweets/sec: {0}", tweetCount);
                                    tweetCount = 0;
                                }
                            };

                            stream.StartStreamMatchingAllConditions();
                        }
                        catch (Exception ex)
                        {
                            Console.WriteLine("Exception: {0}", ex.Message);
                        }
                    }
                }

            }
        }

8. 设置常量，包括 **TWITTERAPPACCESSTOKEN**、**TWITTERAPPACCESSTOKENSECRET**、**TWITTERAPPAPIKEY** 和 **TWITTERAPPAPISECRET**。

若要运行流式处理服务，请按 **F5**。下面是该控制台应用程序的屏幕截图：

![hdinsight.hbase.twitter.sentiment.streaming.service][img-streaming-service]
    
开发 Web 应用程序时，请保持流式处理控制台应用程序的运行状态，以便有更多的数据可用。若要检查数据是否已插入表中，可以使用 HBase Shell。请参阅 [Get started with HBase in HDInsight](/documentation/articles/hdinsight-hbase-tutorial-get-started/#create-tables-and-insert-data)（HDInsight 中的 HBase 入门）。


## 可视化实时观点

在本部分中，你将创建一个 ASP.NET MVC Web 应用程序，以便从 HBase 读取实时观点数据，并将数据绘制在必应地图上。

**创建 ASP.NET MVC Web 应用程序**

1. 打开 Visual Studio。
2. 依次单击“文件”、“新建”和“项目”。
3. 输入以下信息：

	- 模板类别：**Visual C#/Web**
	- 模板：**ASP.NET Web 应用程序**
	- 名称：**TweetSentimentWeb**
	- 位置：**C:\\Tutorials**
4. 单击**“确定”**。
5. 在“选择模板”中单击“MVC”。
6. 在“Microsoft Azure”中，单击“管理订阅”。
7. 在“管理 Microsoft Azure 订阅”中单击“登录”。
8. 输入你的 Azure 凭据。Azure 订阅信息将显示在“帐户”选项卡中。
9. 单击“关闭”，关闭“管理 Microsoft Azure 订阅”窗口。
10. 在“新建 ASP.NET 项目 - TweetSentimentWeb”中单击“确定”。
11. 在“配置 Microsoft Azure 站点设置”中，选择与你最靠近的“区域”。你不需要指定数据库服务器。
12. 单击**“确定”**。

**安装 NuGet 包**

1. 在“工具”菜单中，单击“Nuget Package Manager”，然后单击“Package Manager Console”。控制台面板将在页面底部打开。
2. 使用以下命令安装 [HBase .NET SDK](https://www.nuget.org/packages/Microsoft.HBase.Client/) 包（访问 HBase 群集的客户端库）：

		Install-Package Microsoft.HBase.Client 

**添加 HBaseReader 类**

1. 在“解决方案资源管理器”中展开“TweetSentiment”。
2. 右键单击“模型”，然后依次单击“添加”和“类”。
3. 在“名称”字段中键入 **HBaseReader.cs**，然后单击“添加”。
4. 将此代码替换为以下代码：

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Web;
		
		using System.Configuration;
		using System.Threading.Tasks;
		using System.Text;
		using Microsoft.HBase.Client;
		using org.apache.hadoop.hbase.rest.protobuf.generated;
		
		namespace TweetSentimentWeb.Models
		{
		    public class HBaseReader
		    {
		        // For reading Tweet sentiment data from HDInsight HBase
		        HBaseClient client;
		
		        // HDinsight HBase cluster and HBase table information
		        const string CLUSTERNAME = "<HBaseClusterName>";
		        const string HADOOPUSERNAME = "<HBaseClusterHadoopUserName>"
		        const string HADOOPUSERPASSWORD = "<HBaseCluserUserPassword>";
		        const string HBASETABLENAME = "tweets_by_words";
		
		        // The constructor
		        public HBaseReader()
		        {
		            ClusterCredentials creds = new ClusterCredentials(
		                            new Uri(CLUSTERNAME),
		                            HADOOPUSERNAME,
		                            HADOOPUSERPASSWORD);
		            client = new HBaseClient(creds);
		        }
		
		        // Query Tweets sentiment data from the HBase table asynchronously 
		        public async Task<IEnumerable<Tweet>> QueryTweetsByKeywordAsync(string keyword)
		        {
		            List<Tweet> list = new List<Tweet>();
		
		            // Demonstrate Filtering the data from the past 6 hours the row key
		            string timeIndex = (ulong.MaxValue -
		                (ulong)DateTime.UtcNow.Subtract(new TimeSpan(6, 0, 0)).ToBinary()).ToString().PadLeft(20);
		            string startRow = keyword + "_" + timeIndex;
		            string endRow = keyword + "|";
		            Scanner scanSettings = new Scanner
		            {
		                batch = 100000,
		                startRow = Encoding.UTF8.GetBytes(startRow),
		                endRow = Encoding.UTF8.GetBytes(endRow)
		            };
		
		            // Make async scan call
		            ScannerInformation scannerInfo =
		                await client.CreateScannerAsync(HBASETABLENAME, scanSettings);
		
		            CellSet next;
		
		            while ((next = await client.ScannerGetNextAsync(scannerInfo)) != null)
		            {
		                foreach (CellSet.Row row in next.rows)
		                {
		                    // find the cell with string pattern "d:coor" 
		                    var coordinates =
		                        row.values.Find(c => Encoding.UTF8.GetString(c.column) == "d:coor");
		
		                    if (coordinates != null)
		                    {
		                        string[] lonlat = Encoding.UTF8.GetString(coordinates.data).Split(',');
		
		                        var sentimentField =
		                            row.values.Find(c => Encoding.UTF8.GetString(c.column) == "d:sentiment");
		                        Int32 sentiment = 0;
		                        if (sentimentField != null)
		                        {
		                            sentiment = Convert.ToInt32(Encoding.UTF8.GetString(sentimentField.data));
		                        }
		
		                        list.Add(new Tweet
		                        {
		                            Longtitude = Convert.ToDouble(lonlat[0]),
		                            Latitude = Convert.ToDouble(lonlat[1]),
		                            Sentiment = sentiment
		                        });
		                    }
		
		                    if (coordinates != null)
		                    {
		                        string[] lonlat = Encoding.UTF8.GetString(coordinates.data).Split(',');
		                    }
		                }
		            }
		
		            return list;
		        }
		    }
		
		    public class Tweet
		    {
		        public string IdStr { get; set; }
		        public string Text { get; set; }
		        public string Lang { get; set; }
		        public double Longtitude { get; set; }
		        public double Latitude { get; set; }
		        public int Sentiment { get; set; }
		    }
		}

4. 在 **HBaseReader** 类中更改常量值，如下所示：

	- **CLUSTERNAME**：HBase 群集名称，例如 *https://<HBaseClusterName>.azurehdinsight.cn/* 。
    - **HADOOPUSERNAME**：HBase 群集 Hadoop 用户的用户名。默认名称为 *admin* 。
    - **HADOOPUSERPASSWORD**：HBase 群集 Hadoop 用户密码。
    - **HBASETABLENAME** = "tweets\_by\_words";

	The HBase 表名称为 **"tweets\_by\_words";**。这些值必须与在流式处理服务中发送的值匹配，以便 Web 应用程序能够从相同的 HBase 表中读取数据。




**添加 TweetsController 控制器**

1. 在“解决方案资源管理器”中展开“TweetSentimentWeb”。
2. 右键单击“控制器”，单击“添加”，然后单击“控制器”。
3. 单击“Web API 2 Controller - Empty”，然后单击“添加”。
4. 在“控制器名称”字段中键入 **TweetsController**，然后单击“添加”。
5. 在“解决方案资源管理器”中，双击 TweetsController.cs 打开该文件。
5. 修改该文件，使其类似于：

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Net;
		using System.Net.Http;
		using System.Web.Http;
		
		using System.Threading.Tasks;
		using TweetSentimentWeb.Models;
		
		namespace TweetSentimentWeb.Controllers
		{
		    public class TweetsController : ApiController
		    {
		        HBaseReader hbase = new HBaseReader();
		
		        public async Task<IEnumerable<Tweet>> GetTweetsByQuery(string query)
		        {
		            return await hbase.QueryTweetsByKeywordAsync(query);
		        }
		    }
		}

**添加 heatmap.js**

1. 在“解决方案资源管理器”中展开“TweetSentimentWeb”。
2. 右键单击“脚本”，然后依次单击“添加”和“JavaScript 文件”。
3. 在“项名称”字段中键入 **heatmap.js**。
4. 将以下代码粘贴到文件中。该代码由 Alastair Aitchison 编写。有关详细信息，请参阅[必应地图 AJAX v7 HeatMap 库](http://alastaira.wordpress.com/2011/04/15/bing-maps-ajax-v7-heatmap-library/)。

		/*******************************************************************************
		* Author: Alastair Aitchison
		* Website: http://alastaira.wordpress.com
		* Date: 15th April 2011
		* 
		* Description: 
		* This JavaScript file provides an algorithm that can be used to add a heatmap
		* overlay on a Bing Maps v7 control. The intensity and temperature palette
		* of the heatmap are designed to be easily customisable.
		*
		* Requirements:
		* The heatmap layer itself is created dynamically on the client-side using
		* the HTML5 <canvas> element, and therefore requires a browser that supports
		* this element. It has been tested on IE9, Firefox 3.6/4 and 
		* Chrome 10 browsers. If you can confirm whether it works on other browsers or
		* not, I'd love to hear from you!
		
		* Usage:
		* The HeatMapLayer constructor requires:
		* - A reference to a map object
		* - An array or Microsoft.Maps.Location items
		* - Optional parameters to customise the appearance of the layer
		*  (Radius,, Unit, Intensity, and ColourGradient), and a callback function
		*
		*/
		
		var HeatMapLayer = function (map, locations, options) {
		
		    /* Private Properties */
		    var _map = map,
		      _canvas,
		      _temperaturemap,
		      _locations = [],
		      _viewchangestarthandler,
		      _viewchangeendhandler;
		
		    // Set default options
		    var _options = {
		        // Opacity at the centre of each heat point
		        intensity: 0.5,
		
		        // Affected radius of each heat point
		        radius: 1000,
		
		        // Whether the radius is an absolute pixel value or meters
		        unit: 'meters',
		
		        // Colour temperature gradient of the map
		        colourgradient: {
		            "0.00": 'rgba(255,0,255,20)',  // Magenta
		            "0.25": 'rgba(0,0,255,40)',    // Blue
		            "0.50": 'rgba(0,255,0,80)',    // Green
		            "0.75": 'rgba(255,255,0,120)', // Yellow
		            "1.00": 'rgba(255,0,0,150)'    // Red
		        },
		
		        // Callback function to be fired after heatmap layer has been redrawn 
		        callback: null
		    };
		
		    /* Private Methods */
		    function _init() {
		        var _mapDiv = _map.getRootElement();
		
		        if (_mapDiv.childNodes.length >= 3 && _mapDiv.childNodes[2].childNodes.length >= 2) {
		            // Create the canvas element
		            _canvas = document.createElement('canvas');
		            _canvas.style.position = 'relative';
		
		            var container = document.createElement('div');
		            container.style.position = 'absolute';
		            container.style.left = '0px';
		            container.style.top = '0px';
		            container.appendChild(_canvas);
		
		            _mapDiv.childNodes[2].childNodes[1].appendChild(container);
		
		            // Override defaults with any options passed in the constructor
		            _setOptions(options);
		
		            // Load array of location data
		            _setPoints(locations);
		
		            // Create a colour gradient from the suppied colourstops
		            _temperaturemap = _createColourGradient(_options.colourgradient);
		
		            // Wire up the event handler to redraw heatmap canvas
		            _viewchangestarthandler = Microsoft.Maps.Events.addHandler(_map, 'viewchangestart', _clearHeatMap);
		            _viewchangeendhandler = Microsoft.Maps.Events.addHandler(_map, 'viewchangeend', _createHeatMap);
		
		            _createHeatMap();
		
		            delete _init;
		        } else {
		            setTimeout(_init, 100);
		        }
		    }
		
		    // Resets the heat map
		    function _clearHeatMap() {
		        var ctx = _canvas.getContext("2d");
		        ctx.clearRect(0, 0, _canvas.width, _canvas.height);
		    }
		
		    // Creates a colour gradient from supplied colour stops on initialisation
		    function _createColourGradient(colourstops) {
		        var ctx = document.createElement('canvas').getContext('2d');
		        var grd = ctx.createLinearGradient(0, 0, 256, 0);
		        for (var c in colourstops) {
		            grd.addColorStop(c, colourstops[c]);
		        }
		        ctx.fillStyle = grd;
		        ctx.fillRect(0, 0, 256, 1);
		        return ctx.getImageData(0, 0, 256, 1).data;
		    }
		
		    // Applies a colour gradient to the intensity map
		    function _colouriseHeatMap() {
		        var ctx = _canvas.getContext("2d");
		        var dat = ctx.getImageData(0, 0, _canvas.width, _canvas.height);
		        var pix = dat.data; // pix is a CanvasPixelArray containing height x width x 4 bytes of data (RGBA)
		        for (var p = 0, len = pix.length; p < len;) {
		            var a = pix[p + 3] * 4; // get the alpha of this pixel
		            if (a != 0) { // If there is any data to plot
		                pix[p] = _temperaturemap[a]; // set the red value of the gradient that corresponds to this alpha
		                pix[p + 1] = _temperaturemap[a + 1]; //set the green value based on alpha
		                pix[p + 2] = _temperaturemap[a + 2]; //set the blue value based on alpha
		            }
		            p += 4; // Move on to the next pixel
		        }
		        ctx.putImageData(dat, 0, 0);
		    }
		
		    // Sets any options passed in
		    function _setOptions(options) {
		        for (attrname in options) {
		            _options[attrname] = options[attrname];
		        }
		    }
		
		    // Sets the heatmap points from an array of Microsoft.Maps.Locations  
		    function _setPoints(locations) {
		        _locations = locations;
		    }
		
		    // Main method to draw the heatmap
		    function _createHeatMap() {
		        // Ensure the canvas matches the current dimensions of the map
		        // This also has the effect of resetting the canvas
		        _canvas.height = _map.getHeight();
		        _canvas.width = _map.getWidth();
		
		        _canvas.style.top = -_canvas.height / 2 + 'px';
		        _canvas.style.left = -_canvas.width / 2 + 'px';
		
		        // Calculate the pixel radius of each heatpoint at the current map zoom
		        if (_options.unit == "pixels") {
		            radiusInPixel = _options.radius;
		        } else {
		            radiusInPixel = _options.radius / _map.getMetersPerPixel();
		        }
		
		        var ctx = _canvas.getContext("2d");
		
		        // Convert lat/long to pixel location
		        var pixlocs = _map.tryLocationToPixel(_locations, Microsoft.Maps.PixelReference.control);
		        var shadow = 'rgba(0, 0, 0, ' + _options.intensity + ')';
		        var mapWidth = 256 * Math.pow(2, _map.getZoom());
		
		        // Create the Intensity Map by looping through each location
		        for (var i = 0, len = pixlocs.length; i < len; i++) {
		            var x = pixlocs[i].x;
		            var y = pixlocs[i].y;
		
		            if (x < 0) {
		                x += mapWidth * Math.ceil(Math.abs(x / mapWidth));
		            }
		
		            // Create radial gradient centred on this point
		            var grd = ctx.createRadialGradient(x, y, 0, x, y, radiusInPixel);
		            grd.addColorStop(0.0, shadow);
		            grd.addColorStop(1.0, 'transparent');
		
		            // Draw the heatpoint onto the canvas
		            ctx.fillStyle = grd;
		            ctx.fillRect(x - radiusInPixel, y - radiusInPixel, 2 * radiusInPixel, 2 * radiusInPixel);
		        }
		
		        // Apply the specified colour gradient to the intensity map
		        _colouriseHeatMap();
		
		        // Call the callback function, if specified
		        if (_options.callback) {
		            _options.callback();
		        }
		    }
		
		    /* Public Methods */
		
		    this.Show = function () {
		        if (_canvas) {
		            _canvas.style.display = '';
		        }
		    };
		
		    this.Hide = function () {
		        if (_canvas) {
		            _canvas.style.display = 'none';
		        }
		    };
		
		    // Sets options for intensity, radius, colourgradient etc.
		    this.SetOptions = function (options) {
		        _setOptions(options);
		    }
		
		    // Sets an array of Microsoft.Maps.Locations from which the heatmap is created
		    this.SetPoints = function (locations) {
		        // Reset the existing heatmap layer
		        _clearHeatMap();
		        // Pass in the new set of locations
		        _setPoints(locations);
		        // Recreate the layer
		        _createHeatMap();
		    }
		
		    // Removes the heatmap layer from the DOM
		    this.Remove = function () {
		        _canvas.parentNode.parentNode.removeChild(_canvas.parentNode);
		
		        if (_viewchangestarthandler) { Microsoft.Maps.Events.removeHandler(_viewchangestarthandler); }
		        if (_viewchangeendhandler) { Microsoft.Maps.Events.removeHandler(_viewchangeendhandler); }
		
		        _locations = null;
		        _temperaturemap = null;
		        _canvas = null;
		        _options = null;
		        _viewchangestarthandler = null;
		        _viewchangeendhandler = null;
		    }
		
		    // Call the initialisation routine
		    _init();
		};
		
		// Call the Module Loaded method
		Microsoft.Maps.moduleLoaded('HeatMapModule');


**添加 twitterStream.js**

1. 在“解决方案资源管理器”中展开“TweetSentimentWeb”。
2. 右键单击“脚本”，然后依次单击“添加”和“JavaScript 文件”。
3. 在“项名称”字段中键入 **twitterStream.js**。
4. 将以下代码复制并粘贴到文件中：

		var liveTweetsPos = [];
		var liveTweets = [];
		var liveTweetsNeg = [];
		var map;
		var heatmap;
		var heatmapNeg;
		var heatmapPos;
		
		function initialize() {
		    // Initialize the map
		    var options = {
		        credentials: "AvFJTZPZv8l3gF8VC3Y7BPBd0r7LKo8dqKG02EAlqg9WAi0M7la6zSIT-HwkMQbx",
		        center: new Microsoft.Maps.Location(23.0, 8.0),
		        mapTypeId: Microsoft.Maps.MapTypeId.ordnanceSurvey,
		        labelOverlay: Microsoft.Maps.LabelOverlay.hidden,
		        zoom: 2.5
		    };
		    var map = new Microsoft.Maps.Map(document.getElementById('map_canvas'), options);
		
		    // Heatmap options for positive, neutral and negative layers
		
		    var heatmapOptions = {
		        // Opacity at the centre of each heat point
		        intensity: 0.5,
		
		        // Affected radius of each heat point
		        radius: 15,
		
		        // Whether the radius is an absolute pixel value or meters
		        unit: 'pixels'
		    };
		
		    var heatmapPosOptions = {
		        // Opacity at the centre of each heat point
		        intensity: 0.5,
		
		        // Affected radius of each heat point
		        radius: 15,
		
		        // Whether the radius is an absolute pixel value or meters
		        unit: 'pixels',
		
		        colourgradient: {
		            0.0: 'rgba(0, 255, 255, 0)',
		            0.1: 'rgba(0, 255, 255, 1)',
		            0.2: 'rgba(0, 255, 191, 1)',
		            0.3: 'rgba(0, 255, 127, 1)',
		            0.4: 'rgba(0, 255, 63, 1)',
		            0.5: 'rgba(0, 127, 0, 1)',
		            0.7: 'rgba(0, 159, 0, 1)',
		            0.8: 'rgba(0, 191, 0, 1)',
		            0.9: 'rgba(0, 223, 0, 1)',
		            1.0: 'rgba(0, 255, 0, 1)'
		        }
		    };
		
		    var heatmapNegOptions = {
		        // Opacity at the centre of each heat point
		        intensity: 0.5,
		
		        // Affected radius of each heat point
		        radius: 15,
		
		        // Whether the radius is an absolute pixel value or meters
		        unit: 'pixels',
		
		        colourgradient: {
		            0.0: 'rgba(0, 255, 255, 0)',
		            0.1: 'rgba(0, 255, 255, 1)',
		            0.2: 'rgba(0, 191, 255, 1)',
		            0.3: 'rgba(0, 127, 255, 1)',
		            0.4: 'rgba(0, 63, 255, 1)',
		            0.5: 'rgba(0, 0, 127, 1)',
		            0.7: 'rgba(0, 0, 159, 1)',
		            0.8: 'rgba(0, 0, 191, 1)',
		            0.9: 'rgba(0, 0, 223, 1)',
		            1.0: 'rgba(0, 0, 255, 1)'
		        }
		    };
		
		    // Register and load the Client Side HeatMap Module
		    Microsoft.Maps.registerModule("HeatMapModule", "scripts/heatmap.js");
		    Microsoft.Maps.loadModule("HeatMapModule", {
		        callback: function () {
		            // Create heatmap layers for positive, neutral and negative tweets
		            heatmapPos = new HeatMapLayer(map, liveTweetsPos, heatmapPosOptions);
		            heatmap = new HeatMapLayer(map, liveTweets, heatmapOptions);
		            heatmapNeg = new HeatMapLayer(map, liveTweetsNeg, heatmapNegOptions);
		        }
		    });
		
		    $("#searchbox").val("xbox");
		    $("#searchBtn").click(onsearch);
		    $("#positiveBtn").click(onPositiveBtn);
		    $("#negativeBtn").click(onNegativeBtn);
		    $("#neutralBtn").click(onNeutralBtn);
		    $("#neutralBtn").button("toggle");
		}
		
		function onsearch() {
		    var uri = 'api/tweets?query=';
		    var query = $('#searchbox').val();
		    $.getJSON(uri + query)
		        .done(function (data) {
		            liveTweetsPos = [];
		            liveTweets = [];
		            liveTweetsNeg = [];
		
		            // On success, 'data' contains a list of tweets.
		            $.each(data, function (key, item) {
		                addTweet(item);
		            });
		
		            if (!$("#neutralBtn").hasClass('active')) {
		                $("#neutralBtn").button("toggle");
		            }
		            onNeutralBtn();
		        })
		        .fail(function (jqXHR, textStatus, err) {
		            $('#statustext').text('Error: ' + err);
		        });
		}
		
		function addTweet(item) {
		    //Add tweet to the heat map arrays.
		    var tweetLocation = new Microsoft.Maps.Location(item.Latitude, item.Longtitude);
		    if (item.Sentiment > 0) {
		        liveTweetsPos.push(tweetLocation);
		    } else if (item.Sentiment < 0) {
		        liveTweetsNeg.push(tweetLocation);
		    } else {
		        liveTweets.push(tweetLocation);
		    }
		}
		
		function onPositiveBtn() {
		    if ($("#neutralBtn").hasClass('active')) {
		        $("#neutralBtn").button("toggle");
		    }
		    if ($("#negativeBtn").hasClass('active')) {
		        $("#negativeBtn").button("toggle");
		    }
		
		    heatmapPos.SetPoints(liveTweetsPos);
		    heatmapPos.Show();
		    heatmapNeg.Hide();
		    heatmap.Hide();
		
		    $('#statustext').text('Tweets: ' + liveTweetsPos.length + "   " + getPosNegRatio());
		}
		
		function onNeutralBtn() {
		    if ($("#positiveBtn").hasClass('active')) {
		        $("#positiveBtn").button("toggle");
		    }
		    if ($("#negativeBtn").hasClass('active')) {
		        $("#negativeBtn").button("toggle");
		    }
		
		    heatmap.SetPoints(liveTweets);
		    heatmap.Show();
		    heatmapNeg.Hide();
		    heatmapPos.Hide();
		
		    $('#statustext').text('Tweets: ' + liveTweets.length + "   " + getPosNegRatio());
		}
		
		function onNegativeBtn() {
		    if ($("#positiveBtn").hasClass('active')) {
		        $("#positiveBtn").button("toggle");
		    }
		    if ($("#neutralBtn").hasClass('active')) {
		        $("#neutralBtn").button("toggle");
		    }
		
		    heatmapNeg.SetPoints(liveTweetsNeg);
		    heatmapNeg.Show();
		    heatmap.Hide();;
		    heatmapPos.Hide();;
		
		    $('#statustext').text('Tweets: ' + liveTweetsNeg.length + "\t" + getPosNegRatio());
		}
		
		function getPosNegRatio() {
		    if (liveTweetsNeg.length == 0) {
		        return "";
		    }
		    else {
		        var ratio = liveTweetsPos.length / liveTweetsNeg.length;
		        var str = parseFloat(Math.round(ratio * 10) / 10).toFixed(1);
		        return "Positive/Negative Ratio: " + str;
		    }
		}


**修改 layout.cshtml**

1. 在“解决方案资源管理器”中，依次展开“TweetSentimentWeb”、“视图”和“共享”，然后双击“\_Layout.cshtml”。
2. 将文件中的内容替换为以下内容：

		<!DOCTYPE html>
		<html>
		<head>
		    <meta charset="utf-8" />
		    <meta name="viewport" content="width=device-width, initial-scale=1.0">
		    <title>@ViewBag.Title</title>
		    @Styles.Render("~/Content/css")
		    @Scripts.Render("~/bundles/modernizr")
		    <!-- Bing Maps -->
		    <script type="text/javascript" src="http://ecn.dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=7.0&mkt=en-gb"></script>
		    <!-- Spatial Dashboard JavaScript -->
		    <script src="~/Scripts/twitterStream.js" type="text/javascript"></script>
		</head>
		<body onload="initialize()">
		    <div class="navbar navbar-inverse navbar-fixed-top">
		        <div class="container">
		            <div class="navbar-header">
		                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
		                    <span class="icon-bar"></span>
		                    <span class="icon-bar"></span>
		                    <span class="icon-bar"></span>
		                </button>
		            </div>
		            <div class="navbar-collapse collapse">
		                <div class="row">
		                    <ul class="nav navbar-nav col-lg-5">
		                        <li class="col-lg-12">
		                            <div class="navbar-form">
		                                <input id="searchbox" type="search" class="form-control">
		                                <button type="button" id="searchBtn" class="btn btn-primary">Go</button>
		                            </div>
		                        </li>
		                    </ul>
		                    <ul class="nav navbar-nav col-lg-7">
		                        <li>
		                            <div class="navbar-form">
		                                <div class="btn-group" data-toggle="buttons-radio">
		                                    <button type="button" id="positiveBtn" class="btn btn-primary">Positive</button>
		                                    <button type="button" id="neutralBtn" class="btn btn-primary">Neutral</button>
		                                    <button type="button" id="negativeBtn" class="btn btn-primary">Negative</button>
		                                </div>
		                            </div>
		                        </li>
		                        <li><span id="statustext" class="navbar-text"></span></li>
		                    </ul>
		                </div>
		            </div>
		        </div>
		    </div>
		    <div class="map_container">
		        @RenderBody()
		    </div>
		    @Scripts.Render("~/bundles/jquery")
		    @Scripts.Render("~/bundles/bootstrap")
		    @RenderSection("scripts", required: false)
		</body>
		</html>



**修改 Index.cshtml**

1. 在“解决方案资源管理器”中，依次展开“TweetSentimentWeb”、“视图”和“主页”，然后双击“Index.cshtml”。
2. 将文件中的内容替换为以下内容：

		@{
		    ViewBag.Title = "Tweet Sentiment";
		}
		
		<div class="map_container">
		    <div id="map_canvas"/>
		</div>

**修改 site.css 文件**

1. 在“解决方案资源管理器”中，依次展开“TweetSentimentWeb”和“内容”，然后双击“Site.css”。
2. 将以下代码附加到文件中：
		
		/* make container, and thus map, 100% width */
		.map_container {
			width: 100%;
			height: 100%;
		}
		
		#map_canvas{
		  height:100%;
		}
		
		#tweets{
		  position: absolute;
		  top: 60px;
		  left: 75px;
		  z-index:1000;
		  font-size: 30px;
		}

**修改 global.asax 文件**

1. 在“解决方案资源管理器”中，展开“TweetSentimentWeb”，然后双击“Global.asax”。
2. 添加以下 **using** 语句：

		using System.Web.Http;

2. 将以下行添加到 **Application\_Start()** 函数中：

		// Register API routes
		GlobalConfiguration.Configure(WebApiConfig.Register);
  
	修改 API 路由的注册，使 Web API 控制器能在 MVC 应用程序中运行。

**运行 Web 应用程序**

1. 验证流式处理服务控制台应用程序是否仍在运行，以便可以查看实时更改。
2. 按 **F5** 运行 Web 应用程序：

	![hdinsight.hbase.twitter.sentiment.bing.map][img-bing-map]
2. 在文本框中输入一个关键字，然后单击“搜索”。根据在 HBase 表中收集的数据，可能找不到某些关键字。请尝试搜索一些常见的关键字，例如“love”、“xbox”和“playstation”。
3. 在“肯定”、“否定”和“中立”之间切换，以比较用户对主题的观点。
4. 让流式处理服务再多运行一小时，然后搜索相同的关键字并比较结果。

 
你也可以选择性地将应用程序部署到 Azure 网站。有关说明，请参阅 [Azure 网站和 ASP.NET 入门][website-get-started]。
 
## 后续步骤

在本教程中，你已学习如何获取推文、分析推文的观点、将观点数据保存到 HBase，以及在必应地图上呈现实时的 Twitter 观点数据。若要了解更多信息，请参阅以下文章：

- [开始使用 HDInsight][hdinsight-get-started]
- [在 HDInsight 中配置 HBase 复制](/documentation/articles/hdinsight-hbase-geo-replication/)
- [使用 HDInsight 中的 Hadoop 分析 Twitter 数据][hdinsight-analyze-twitter-data]
- [使用 HDInsight 分析航班延误数据][hdinsight-analyze-flight-delay-data]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]


[hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started/
[website-get-started]: /documentation/articles/web-sites-dotnet-get-started/



[img-app-arch]: ./media/hdinsight-hbase-analyze-twitter-sentiment/AppArchitecture.png
[img-twitter-app]: ./media/hdinsight-hbase-analyze-twitter-sentiment/TwitterApp.png
[img-streaming-service]: ./media/hdinsight-hbase-analyze-twitter-sentiment/StreamingService.png
[img-bing-map]: ./media/hdinsight-hbase-analyze-twitter-sentiment/TwitterSentimentBingMap.png



[hdinsight-develop-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/
[hdinsight-analyze-twitter-data]: /documentation/articles/hdinsight-analyze-twitter-data/




[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[apache-hive-tutorial]: https://cwiki.apache.org/confluence/display/Hive/Tutorial

[twitter-streaming-api]: https://dev.twitter.com/docs/streaming-apis
[twitter-statuses-filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-install]: /documentation/articles/powershell-install-configure/
[powershell-script]: https://technet.microsoft.com/zh-cn/library/dn425048.aspx

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows/
[hdinsight-storage-powershell]: /documentation/articles/hdinsight-hadoop-use-blob-storage/#powershell
[hdinsight-analyze-flight-delay-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/
[hdinsight-hive-odbc]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/
 

<!---HONumber=Mooncake_0926_2016-->