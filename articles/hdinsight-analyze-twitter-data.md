<properties linkid="manage-services-hdinsight-howto-social-data" urlDisplayName="Analyze Twitter data with Hive" pageTitle="Analyze Twitter data with HDinsight | Azure" metaKeywords="" description="Learn how to use Hive to analyze Twitter data on HDInsight to find usage frequency of a particular word." metaCanonical="" services="HDInsight" documentationCenter="" title="Analyze Twitter data with Hive" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 使用 HDInsight 分析 Twitter 数据

了解如何使用 Hive 和 HDInsight 来分析 Twitter 数据。

社交网站是采用大数据的主要推动力之一。Twitter 等网站所提供的公共 API 是一类用于分析和了解流行趋势的有用数据源。在本教程中，你将使用 Twitter 流式传输 API 连接到 Twitter Web 服务以获取一些推文，然后使用 Hive 获取发送包含某个特定词的推文数量最多的 Twitter 用户列表。

估计完成时间：30 分钟

## 本文内容

-   [先决条件][]
-   [获取 Twitter 源][]
-   [创建 HiveQL 脚本][]
-   [使用 Hive 处理数据][]
-   [教程结束][]
-   [后续步骤][]

## 先决条件

在开始阅读本教程前，你必须具有：

-   已安装并已配置 Azure PowerShell 的**工作站**。有关说明，请参阅[安装和配置 Azure PowerShell][]。若要执行 PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为“RemoteSigned”**。请参阅[运行 Windows PowerShell 脚本][]。

    在运行 PowerShell 脚本之前，请确保你已使用以下 cmdlet 连接到 Azure 订阅：

        Add-AzureAccount

    如果你有多个 Azure 订阅，请使用以下 cmdlet 来设置当前订阅：

        Select-AzureSubscription <AzureSubscriptionName>

-   **一个 Azure HDInsight 群集**。有关群集设置的说明，请参阅[开始使用 HDInsight][] 或[设置 HDInsight 群集][]。你将需要以下数据才能完成本教程：

<table border="1">
<tr><th>群集属性</th><th>PowerShell 变量名</th><th>值</th><th>说明</th></tr>
<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>这是你的 HDInsight 群集名称。</td></tr>
<tr><td>Azure 存储帐户名称</td><td>$storageAccountName</td><td></td><td>可用于 HDInsight 群集的 Azure 存储帐户。在本教程中，使用在群集设置过程中指定的默认存储帐户。</td></tr>
<tr><td>Azure Blob 容器名称</td><td>$containerName</td><td></td><td>在此示例中，使用用于默认 HDInsight 群集文件系统的 Azure Blob 存储容器。默认情况下，该容器与 HDInsight 群集同名。</td></tr>
</table>

**了解 HDInsight 存储**

HDInsight 将 Azure Blob 存储用于数据存储。它称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

设置 HDInsight 群集时，请将 Blob 存储容器指定为默认文件系统，就像在 HDFS 中一样。除了此容器外，你还可以在设置过程中从同一 Azure 存储帐户或不同 Azure 存储帐户添加其他容器。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][]。

为了简化本教程中使用的 PowerShell 脚本，所有文件都存储在默认文件系统容器（位于 */tutorials/twitter*）中。默认情况下，此容器与 HDInsight 群集同名。

WASB 语法为：

    wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<路径>/<文件名>

> [WACOM.NOTE] HDInsight 群集 3.0 版只支持 *wasb://* 语法。较早的 *asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持，以后的版本将不会支持该语法。

> WASB 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

对于存储在默认文件系统容器中的文件，可以使用以下任一 URI 从 HDInsight 进行访问（以 tweets.txt 为例）：

    wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/twitter/tweets.txt
    wasb:///tutorials/twitter/tweets.txt
    /tutorials/twitter/tweets.txt

如果要从存储帐户直接访问该文件，则请注意，该文件的 Blob 名称是：

    tutorials/twitter/tweets.txt

下表列出了本教程中使用的文件：

<table border="1">
<tr><th>文件</th><th>说明</th></tr>
<tr><td>/tutorials/twitter/data/tweets.txt</td><td>Hive 作业的源数据。</td></tr>
<tr><td>/tutorials/twitter/output</td><td>Hive 作业的输出文件夹。默认 Hive 作业输出文件名是 <strong>000000_0</strong>。 </td></tr>
<tr><td>tutorials/twitter/twitter.hql</td><td>HiveQL 脚本文件。</td></tr>
<tr><td>/tutorials/twitter/jobstatus</td><td>Hadoop 作业状态。</td></tr>
</table>

## 获取 Twitter 源

在本教程中，你将使用 [Twitter 流式传输 API][]。你将使用的特定 Twitter 流式传输 API 是 [statuses/filter][]。

[推文数据][]以包含复杂的嵌套结构的 JSon 格式存储。你可以将此嵌套结构转换为 Hive 表（而不是使用传统的编程语言编写多行代码），使其能够通过类似 SQL 的语言（称作 HiveQL）进行查询。

Twitter 使用 OAuth 提供对其 API 的授权访问。OAuth 是一种身份验证协议，允许用户批准应用程序代表其执行操作，而不用共享其密码。更多信息可以在 [oauth.net][] 上或 Hueniverse 提供的出色的 [OAuth 初学者指南][]中找到。

使用 OAuth 的第一步是在 Twitter 开发人员网站上创建新的应用程序。

**创建 Twitter 应用程序**

1.  登录到 [][]<https://apps.twitter.com/>。单击“立即注册” 链接（如果你没有 Twitter 帐户）。
2.  单击“新建应用程序” 。
3.  输入“名称” 、“说明” 、“网站” 。你可以为“网站”字段补充 URL。下表显示了一些要使用的示例值：

<table border="1">
<tr><th>字段</th><th>值</th></tr>
<tr><td>名称</td><td>MyHDInsightApp</td></tr>
<tr><td>说明</td><td>MyHDInsightApp</td></tr>
<tr><td>网站</td><td>http://www.myhdinsightapp.com</td></tr>
</table>

4.  选中“是，我同意” ，然后单击“创建 Twitter 应用程序” 。
5.  单击“权限” 选项卡。默认权限为“只读” 。这对于本教程来说已足够。
6.  单击“API 密钥” 选项卡。
7.  单击“创建我的访问令牌” 。
8.  在页面右上角单击“测试 OAuth” 。
9.  记下“使用者密钥” 、“使用者机密” 、“访问令牌” 和“访问令牌机密” 。本教程后面的步骤中将会用到这些值。

在本教程中，你将使用 PowerShell 调用 Web 服务。其他常用于调用 Web 服务的工具是 [*Curl*][]。Curl 可从[此处][]下载。

> [WACOM.NOTE] 在 Windows 上使用 curl 命令时，请使用双引号（而不是单引号）括起选项值。

**获取推文**

1.  打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell\_ISE**，然后单击 **Windows PowerShell ISE**。请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][]）。

2.  将以下脚本复制到脚本窗格中：

        Write-Host "Set variables ..."-ForegroundColor Green
        $storageAccountName = "<AzureStorageAccountName>"
        $containerName = "<BlobContainerName>"

        $destBlobName = "tutorials/twitter/data/tweets.txt"

        $trackString = "Azure, Cloud, HDInsight"
        $track = [System.Uri]::EscapeDataString($trackString);
        $lineMax = 100  #每 100 行约 3 分钟

        $oauth_consumer_key = "<TwitterAppConsumerKey>";
        $oauth_consumer_secret = "<TwitterAppConsumerSecret>";
        $oauth_token = "<TwitterAppAccessToken>";
        $oauth_token_secret = "<TwitterAppAccessTokenSecret>";

        Write-Host "Define the Azure storage connection string ..."-ForegroundColor Green
        $storageAccountKey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$storageAccountName;AccountKey=$storageAccountKey"

        Write-Host "Create block blob object ..."-ForegroundColor Green
        $storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
        $storageClient = $storageAccount.CreateCloudBlobClient();
        $storageContainer = $storageClient.GetContainerReference($containerName)
        $destBlob = $storageContainer.GetBlockBlobReference($destBlobName)

        Write-Host "Define a MemoryStream and a StreamWriter for writing ..."-ForegroundColor Green
        $memStream = New-Object System.IO.MemoryStream
        $writeStream = New-Object System.IO.StreamWriter $memStream

        Write-Host "Format oauth strings ..."-ForegroundColor Green
        $oauth_nonce = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes([System.DateTime]::Now.Ticks.ToString()));
        $ts = [System.DateTime]::UtcNow - [System.DateTime]::ParseExact("01/01/1970", "dd/MM/yyyy", $null)
        $oauth_timestamp = [System.Convert]::ToInt64($ts.TotalSeconds).ToString();

        $signature = "POST&";
        $signature += [System.Uri]::EscapeDataString("https://stream.twitter.com/1.1/statuses/filter.json") + "&";
        $signature += [System.Uri]::EscapeDataString("oauth_consumer_key=" + $oauth_consumer_key + "&");
        $signature += [System.Uri]::EscapeDataString("oauth_nonce=" + $oauth_nonce + "&"); 
        $signature += [System.Uri]::EscapeDataString("oauth_signature_method=HMAC-SHA1&");
        $signature += [System.Uri]::EscapeDataString("oauth_timestamp=" + $oauth_timestamp + "&");
        $signature += [System.Uri]::EscapeDataString("oauth_token=" + $oauth_token + "&");
        $signature += [System.Uri]::EscapeDataString("oauth_version=1.0&");
        $signature += [System.Uri]::EscapeDataString("track=" + $track);

        $signature_key = [System.Uri]::EscapeDataString($oauth_consumer_secret) + "&" + [System.Uri]::EscapeDataString($oauth_token_secret);

        $hmacsha1 = new-object System.Security.Cryptography.HMACSHA1;
        $hmacsha1.Key = [System.Text.Encoding]::ASCII.GetBytes($signature_key);
        $oauth_signature = [System.Convert]::ToBase64String($hmacsha1.ComputeHash([System.Text.Encoding]::ASCII.GetBytes($signature)));

        $oauth_authorization = 'OAuth ';
        $oauth_authorization += 'oauth_consumer_key="' + [System.Uri]::EscapeDataString($oauth_consumer_key) + '",';
        $oauth_authorization += 'oauth_nonce="' + [System.Uri]::EscapeDataString($oauth_nonce) + '",';
        $oauth_authorization += 'oauth_signature="' + [System.Uri]::EscapeDataString($oauth_signature) + '",';
        $oauth_authorization += 'oauth_signature_method="HMAC-SHA1",'
        $oauth_authorization += 'oauth_timestamp="' + [System.Uri]::EscapeDataString($oauth_timestamp) + '",'
        $oauth_authorization += 'oauth_token="' + [System.Uri]::EscapeDataString($oauth_token) + '",';
        $oauth_authorization += 'oauth_version="1.0"';

        $post_body = [System.Text.Encoding]::ASCII.GetBytes("track=" + $track); 

        Write-Host "Create HTTP web request ..."-ForegroundColor Green
        [System.Net.HttpWebRequest] $request = [System.Net.WebRequest]::Create("https://stream.twitter.com/1.1/statuses/filter.json");
        $request.Method = "POST";
        $request.Headers.Add("Authorization", $oauth_authorization);
        $request.ContentType = "application/x-www-form-urlencoded";
        $body = $request.GetRequestStream();

        $body.write($post_body, 0, $post_body.length);
        $body.flush();
        $body.close();
        $response = $request.GetResponse() ;

        Write-Host "Start stream reading ..."-ForegroundColor Green

        $sReader = New-Object System.IO.StreamReader($response.GetResponseStream())

        $inrec = $sReader.ReadLine()
        $count = 0
        while (($inrec -ne $null) -and ($count -le $lineMax))
        {
        if ($inrec -ne "")
            {
        Write-Host $count.ToString() ", " -NoNewline -ForegroundColor Green
        if ($count%10 -eq 0){
        write-host " --- " -NoNewline
        Get-Date -Format hh:mm:sstt
                }

        $writeStream.WriteLine($inrec)
        $count ++
            }

        $inrec=$sReader.ReadLine()
        }

        Write-Host "Write to the destination blob ..."-ForegroundColor Green
        $writeStream.Flush()
        $memStream.Seek(0, "Begin")
        $destBlob.UploadFromStream($memStream) 

        $sReader.close()
        Write-Host "Complete!"-ForegroundColor Green

3.  设置此脚本中的前九个变量：

<table border="1">
<tr><th>变量</th><th>说明</th></tr>
<tr><td>$storageAccountName</td><td>用于默认 HDInsight 群集文件系统的 Azure 存储帐户。它是在设置时指定的存储帐户。请参阅<a href="#prerequisites">先决条件</a>。</td></tr>
<tr><td>$containerName</td><td>用于默认 HDInsight 群集文件系统的 Blob 容器。它是在设置时指定的容器。请参阅<a href="#prerequisites">先决条件</a>。</td></tr>
<tr><td>$destBlobName</td><td>这是输出 Blob 名称。默认值是 <strong>tutorials/twitter/data/tweets.txt</strong>。如果更改默认值，则需要相应地更新 PowerShell 脚本。</td></tr>
<tr><td>$trackString</td><td>Web 服务将返回与这些关键字相关的推文。默认值为&ldquo;Azure, 云, HDInsight&rdquo;<strong></strong>。如果更改默认值，则需要相应地更新 PowerShell 脚本。</td></tr>
<tr><td>$lineMax</td><td>此值确定脚本将读取多少篇推文。读取 100 篇推文需要大约三分钟。你可以设置更大的数，但需要更长时间来下载。</td></tr>
<tr><td>$oauth_consumer_key</td><td>这是你先前在创建 Twitter 应用程序时记下的 Twitter 应用程序<strong>使用者密钥</strong>。</td></tr>
<tr><td>$oauth_consumer_secret</td><td>这是你先前记下的 Twitter 应用程序<strong>使用者机密</strong>。</td></tr>
<tr><td>$oauth_token</td><td>这是你先前记下的 Twitter 应用程序<strong>访问令牌</strong>。</td></tr>
<tr><td>$oauth_token_secret</td><td>这是你先前记下的 Twitter 应用程序<strong>访问令牌机密</strong>。</td></tr>
</table>

4.  按 **F5** 运行脚本。如果遇到问题，一种解决方法是选择所有行，然后按 **F8**。
5.  你会在输出的末尾看到“Complete!”。任何错误消息将显示为红色。

在验证过程中，你可以使用 Azure 存储资源管理器或 Azure PowerShell 查看 Azure Blob 存储中的输出文件 **/tutorials/twitter/data/tweets.txt**。有关用于列出文件的示例 PowerShell 脚本，请参阅[将 Blob 存储与 HDInsight 配合使用][]。

## 创建 HiveQL 脚本

使用 Azure PowerShell，你可以一次运行多个 HiveQL 语句，或者将 HiveQL 语句打包到一个脚本文件中。在本教程中，你将创建 HiveQL 脚本。必须将脚本文件上载到 WASB。在下一节中，你将使用 Azure PowerShell 运行脚本文件。

HiveQL 脚本将执行以下操作：

1.  **删除 tweets\_raw 表**（如果该表已存在）。
2.  **创建 tweets\_raw Hive 表**。此临时 Hive 结构化表保存用于进一步 ETL 处理的数据。有关分区的信息，请参阅 [Hive 教程][]。
3.  从源文件夹 (/tutorials/twitter/data) **加载数据**。采用嵌套 JSon 格式的大型 Tweets 数据集现已转换为临时 Hive 表结构。
4.  **删除 tweets 表**（如果该表已存在）；
5.  **创建 tweets 表**。你需要先运行其他 ETL 过程，然后才能使用 Hive 对 Tweets 数据集进行查询。此 ETL 过程将为已在“twitter\_raw”表中存储的数据定义更详细的表架构。
6.  **插入覆盖表**。此复杂的 Hive 脚本将通过 Hadoop 群集启动一组较长的 Map Reduce 作业。这可能需要约 10 分钟的时间，具体取决于你的数据集和群集大小。
7.  **插入覆盖目录**。运行查询并将数据集输出到文件。此查询将返回发送最多包含单词“Azure”的推文的 Twitter 用户列表。

**创建 Hive 脚本并将它上载到 Azure**

1.  打开 Windows PowerShell ISE。
2.  将以下脚本复制到脚本窗格中：

        Write-Host "Define variables ..."-ForegroundColor Green
        $storageAccountName = "<AzureStorageAccountName>"
        $containerName = "<BlobContainerName>"

        $sourceDataPath = "/tutorials/twitter/data"
        $outputPath = "/tutorials/twitter/output"

        $hqlScriptFile = "tutorials/twitter/twitter.hql"

        $hqlStatements = @"
        set hive.exec.dynamic.partition = true;
        set hive.exec.dynamic.partition.mode = nonstrict;

        DROP TABLE tweets_raw;
        CREATE TABLE tweets_raw (
        json_response STRING
        ) 
        PARTITIONED BY (filesequence INT);

        LOAD DATA INPATH '$sourceDataPath'
        INTO TABLE tweets_raw
        PARTITION (filesequence = 1);

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
        )
        PARTITIONED BY (filesequence INT);

        FROM tweets_raw
        INSERT OVERWRITE TABLE tweets
        PARTITION (filesequence)
        SELECT
        cast(get_json_object(json_response, '$.id_str') as BIGINT),
        get_json_object(json_response, '$.created_at'),
        concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
        substr (get_json_object(json_response, '$.created_at'),27,4)),
        substr (get_json_object(json_response, '$.created_at'),27,4),
        case substr (get_json_object(json_response, '$.created_at'),5,3)
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
        substr (get_json_object(json_response, '$.created_at')0.90.2),
        substr (get_json_object(json_response, '$.created_at')0.120.8),
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
        json_response,
        filesequence
        WHERE (length(json_response) > 500);

        INSERT OVERWRITE DIRECTORY '$outputPath'
        SELECT name, screen_name, count(1) as cc 
        FROM tweets 
        WHERE text like "%Azure%" 
        GROUP BY name,screen_name 
        ORDER BY cc DESC LIMIT 10;
        "@

        Write-Host "Define the connection string ..."-ForegroundColor Green
        $storageAccountKey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$storageAccountName;AccountKey=$storageAccountKey"

        Write-Host "Create block blob objects referencing the hql script file" -ForegroundColor Green
        $storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
        $storageClient = $storageAccount.CreateCloudBlobClient();
        $storageContainer = $storageClient.GetContainerReference($containerName)
        $hqlScriptBlob = $storageContainer.GetBlockBlobReference($hqlScriptFile)

        Write-Host "Define a MemoryStream and a StreamWriter for writing ..." -ForegroundColor Green
        $memStream = New-Object System.IO.MemoryStream
        $writeStream = New-Object System.IO.StreamWriter $memStream
        $writeStream.Writeline($hqlStatements)

        Write-Host "Write to the destination blob ..." -ForegroundColor Green
        $writeStream.Flush()
        $memStream.Seek(0, "Begin")
        $hqlScriptBlob.UploadFromStream($memStream)

        Write-Host "Complete!"-ForegroundColor Green

3.  设置此脚本中的前两个变量：

<table border="1">
<tr><th>变量</th><th>说明</th></tr>
<tr><td>$storageAccountName</td><td>用于默认 HDInsight 群集文件系统的 Azure 存储帐户。它是在设置时指定的存储帐户。请参阅<a href="#prerequisites">先决条件</a>。</td></tr>
<tr><td>$containerName</td><td>用于默认 HDInsight 群集文件系统的 Blob 容器。它是在设置时指定的容器。请参阅<a href="#prerequisites">先决条件</a>。</td></tr>
<tr><td>$sourceDataPath</td><td>Hive 查询将从中读取数据的 WASB 位置。你无需更改此变量。</td></tr>
<tr><td>$outputPath</td><td>Hive 查询将结果输出到的 WASB 位置。你无需更改此变量。</td></tr>
<tr><td>$hqlScriptFile</td><td>HiveQL 脚本文件的位置和文件名。</td></tr>
</table>

4.  按 **F5** 运行脚本。如果遇到问题，一种解决方法是选择所有行，然后按 **F8**。
5.  你会在输出的末尾看到“Complete!”。任何错误消息将显示为红色。

在验证过程中，你可以使用 Azure 存储资源管理器或 Azure PowerShell 查看 Azure Blob 存储中的输出文件 **/tutorials/twitter/twitter.hql**。有关用于列出文件的示例 PowerShell 脚本，请参阅[将 Blob 存储与 HDInsight 配合使用][]。

## 使用 Hive 处理 Twitter 数据

你已完成所有准备工作。现在，你可以调用 Hive 脚本，并检查结果。

### 提交 Hive 作业

使用以下 PowerShell 脚本运行 Hive 脚本。你将需要设置第一个变量。

    Write-Host "Set the variables ..." -ForegroundColor Green
    $clusterName = "<HDInsightClusterName>"

    $hqlScriptFile = "/tutorials/twitter/twitter.hql"
    $statusFolder = "/tutorials/twitter/jobstatus"

    Write-Host "Invoke Hive ..." -ForegroundColor Green
    Use-AzureHDInsightCluster $clusterName
    $response = Invoke-Hive -file $hqlScriptFile -StatusFolder $statusFolder -OutVariable $outVariable

    Write-Host "Display the standard error log ..." -ForegroundColor Green
    $jobID = ($response | Select-String job_ | Select-Object -First 1) -replace ‘\s*$’ -replace ‘.*\s’
    Get-AzureHDInsightJobOutput -cluster $clusterName -JobId $jobID -StandardError 

### 检查结果

使用以下 PowerShell 脚本检查 Hive 作业输出。你将需要设置前两个变量。

    Write-Host "Set the variables ..." -ForegroundColor Green
    $storageAccountName = "<AzureStorageAccountName>"   # 用于在设置时指定的默认文件系统的存储帐户。
    $containerName = "<BlobStorageContainerName>"  # 默认文件系统容器与群集同名。
    $blob = "tutorials/twitter/output/000000_0" # 要下载的 Blob 的名称。

    Write-Host "Create a context object ..." -ForegroundColor Green
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    Write-Host "Download the blob ..."-ForegroundColor Green
    Get-AzureStorageBlobContent -Container $ContainerName -Blob $blob -Context $storageContext -Force

    Write-Host "Display the output ..."-ForegroundColor Green
    cat "./$blob"

> [WACOM.NOTE] Hive 表使用 \\001 作为字段分隔符。该分隔符在输出中不可见。

在将分析结果放到 WASB 上后，可以将数据导出到 Azure SQL Database/SQL Server，使用 Power Query 将数据导出到 Excel，或者使用 Hive ODBC 驱动程序将应用程序连接到数据。有关详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][]、[使用 HDInsight 分析航班延误数据][]、[利用 Power Query 将 Excel 连接到 HDInsight][]，以及[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][]。

## 教程结束

若要重新运行本教程，你将需要：

-   **重新创建推文数据文件**。源推文数据文件已由 Hive 作业删除。你将需要生成新数据文件。文件名是 **tutorials/twitter/data/tweets.txt**。请参阅[获取 Twitter 源][]。

## 后续步骤

在本教程中，我们已了解如何将非结构化 Json 数据集转换为结构化 Hive 表，以便在 Azure 上使用 HDInsight 查询、探究和分析来自 Twitter 的数据。若要了解更多信息，请参阅以下文章：

-   [HDInsight 入门][开始使用 HDInsight]
-   [使用 HDInsight 分析航班延误数据][]
-   [利用 Power Query 将 Excel 连接到 HDInsight][]
-   [使用 Microsoft Hive ODBC Driver 将 Excel 连接到 HDInsight][使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]
-   [将 Sqoop 与 HDInsight 配合使用][]

  [先决条件]: #prerequisites
  [获取 Twitter 源]: #feed
  [创建 HiveQL 脚本]: #script
  [使用 Hive 处理数据]: #process
  [教程结束]: #cleanup
  [后续步骤]: #nextsteps
  [安装和配置 Azure PowerShell]: ../install-configure-powershell
  [运行 Windows PowerShell 脚本]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx
  [开始使用 HDInsight]: ../hdinsight-get-started/
  [设置 HDInsight 群集]: ../hdinsight-provision-clusters/
  [将 Azure Blob 存储与 HDInsight 配合使用]: ../hdinsight-use-blob-storage/
  [Twitter 流式传输 API]: https://dev.twitter.com/docs/streaming-apis
  [statuses/filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter
  [推文数据]: https://dev.twitter.com/docs/platform-objects/tweets
  [oauth.net]: http://oauth.net/
  [OAuth 初学者指南]: http://hueniverse.com/oauth/
  []: https://apps.twitter.com/
  [*Curl*]: http://curl.haxx.se
  [此处]: http://curl.haxx.se/download.html
  [在 Windows 8 和 Windows 上启动 Windows PowerShell]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
  [将 Blob 存储与 HDInsight 配合使用]: ../hdinsight-use-blob-storage/#powershell
  [Hive 教程]: https://cwiki.apache.org/confluence/display/Hive/Tutorial
  [将 Sqoop 与 HDInsight 配合使用]: ../hdinsight-use-sqoop/
  [使用 HDInsight 分析航班延误数据]: ../hdinsight-analyze-flight-delay-data/
  [利用 Power Query 将 Excel 连接到 HDInsight]: ../hdinsight-connect-excel-power-query/
  [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]: ../hdinsight-connect-excel-hive-ODBC-driver/
