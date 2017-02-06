<!-- not suitable for Mooncake -->

<properties
    pageTitle="使用 HDInsight 中的 Hadoop 分析 Twitter 数据 | Azure"
    description="了解如何在 HDInsight 中的 Hadoop 上使用 Hive 分析 Twitter 数据以找到特定单词的使用频率。"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="78e4ea33-9714-424d-ac07-3d60ecaebf2e"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="02/06/2017"
    ms.author="jgao" />

# 使用 HDInsight 中的 Hive 分析 Twitter 数据
社交网站是采用大数据的主要推动力之一。Twitter 等网站所提供的公共 API 是一类用于分析和了解流行趋势的有用数据源。在本教程中，你将通过使用 Twitter 流式传输 API 获取推文，然后使用 Azure HDInsight 上的 Apache Hive 获取发送最多包含某个特定词的推文的 Twitter 用户列表。

> [AZURE.NOTE]
本文档中的步骤要求使用基于 Windows 的 HDInsight 群集。有关特定于基于 Linux 的群集的步骤，请参阅[使用 HDInsight 中的 Hive 分析 Twitter 数据](/documentation/articles/hdinsight-analyze-twitter-data-linux/)。

## 先决条件
在开始本教程前，你必须具有以下各项：

* 已安装并已配置 Azure PowerShell 的**工作站**。

    若要执行 Windows PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为 *RemoteSigned*。请参阅[运行 Windows PowerShell 脚本][powershell-script]。

    运行 Windows PowerShell 脚本之前，确保使用以下 cmdlet 连接到 Azure 订阅：

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

如果有多个 Azure 订阅，请使用以下 cmdlet 设置当前订阅：

        Select-AzureRmSubscription -SubscriptionID <Azure Subscription ID>

> [AZURE.IMPORTANT]
    > Azure PowerShell 对于使用 Azure Service Manager 管理 HDInsight 资源的支持已**弃用**，将于 2017 年 1 月 1 日删除。本文档中的步骤使用的是与 Azure Resource Manager 兼容的新 HDInsight cmdlet。
    >
    > 请按照[安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 中的步骤安装最新版本的 Azure PowerShell。如果你的脚本需要修改才能使用与 Azure Resource Manager 兼容的新 cmdlet，请参阅[迁移到适用于 HDInsight 群集的基于 Azure Resource Manager 的开发工具](/documentation/articles/hdinsight-hadoop-development-using-azure-resource-manager/)，了解详细信息。

* **一个 Azure HDInsight 群集**。有关群集预配的说明，请参阅[开始使用 HDInsight][hdinsight-get-started] 或[预配 HDInsight 群集][hdinsight-provision]。本教程后面的步骤中将会用到此群集名称。

下表列出了本教程中使用的文件：

| 文件 | 说明 |
| --- | --- |
| /tutorials/twitter/data/tweets.txt |Hive 作业的源数据。 |
| /tutorials/twitter/output |Hive 作业的输出文件夹。默认 Hive 作业输出文件名是 **000000\_0**。 |
| tutorials/twitter/twitter.hql |HiveQL 脚本文件。 |
| /tutorials/twitter/jobstatus |Hadoop 作业状态。 |

## 获取 Twitter 源
在本教程中，你将使用 [Twitter 流式传输 API][twitter-streaming-api]。你将使用的特定 Twitter 流式传输 API 是 [statuses/filter][twitter-statuses-filter]。

> [AZURE.NOTE]
已将包含 10,000 个推文的文件和 Hive 脚本文件（在下一节中介绍）上传到公共 Blob 容器中。如果要使用已上传的文件，可以跳过本节。

[推文数据](https://dev.twitter.com/docs/platform-objects/tweets)以包含复杂的嵌套结构的 JavaScript 对象表示法 (JSON) 格式存储。你可以将此嵌套结构转换为 Hive 表（而不是使用传统的编程语言编写多行代码），使其能够通过类似结构化查询语言 (SQL) 的语言（称作 HiveQL）进行查询。

Twitter 使用 OAuth 提供对其 API 的授权访问。OAuth 是一种身份验证协议，允许用户批准应用程序代表其执行操作，而不用共享其密码。更多信息可以在 [oauth.net](http://oauth.net/) 上或 Hueniverse 提供的出色的 [OAuth 初学者指南](http://hueniverse.com/oauth/)中找到。

使用 OAuth 的第一步是在 Twitter 开发人员网站上新建应用程序。

**创建 Twitter 应用程序**

1. 登录到 [https://apps.twitter.com/](https://apps.twitter.com/)。单击“立即注册”链接（如果你没有 Twitter 帐户）。
2. 单击“新建应用”。
3. 输入“名称”、“说明”、“网站”。你可以为“网站”字段补充 URL。下表显示了一些要使用的示例值：

    | 字段 | 值 |
    | --- | --- |
    | 名称 |MyHDInsightApp |
    | 说明 |MyHDInsightApp |
    | 网站 |http://www.myhdinsightapp.com |
4. 选中“是，我同意”，然后单击“创建 Twitter 应用程序”。
5. 单击“权限”选项卡。默认权限为“只读”。这对于本教程来说已足够。
6. 单击“密钥和访问令牌”选项卡。
7. 单击“创建我的访问令牌”。
8. 在页面右上角单击“测试 OAuth”。
9. 记下“使用者密钥”、“使用者机密”、“访问令牌”和“访问令牌机密”。本教程后面的步骤中将会用到这些值。

在本教程中，你将使用 Windows PowerShell 调用 Web 服务。有关 .NET C# 示例，请参阅[在 HDInsight 中使用 HBase 分析实时 Twitter 观点][hdinsight-hbase-twitter-sentiment]。其他常用于调用 Web 服务的工具是 [*Curl*][curl]。Curl 可从[此处][curl-download]下载。

> [AZURE.NOTE]
在 Windows 中使用 curl 命令时，请使用双引号（而不是单引号）括起选项值。

**获取推文**

1. 打开 Windows PowerShell 集成脚本环境 (ISE)。（在 Windows 8“开始”屏幕上，键入 **PowerShell\_ISE**，然后单击“Windows PowerShell ISE”。请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]）。
2. 将以下脚本复制到脚本窗格中：

        #region - variables and constants
        $clusterName = "<HDInsightClusterName>" # Enter the HDInsight cluster name

        # Enter the OAuth information for your Twitter application
        $oauth_consumer_key = "<TwitterAppConsumerKey>";
        $oauth_consumer_secret = "<TwitterAppConsumerSecret>";
        $oauth_token = "<TwitterAppAccessToken>";
        $oauth_token_secret = "<TwitterAppAccessTokenSecret>";

        $destBlobName = "tutorials/twitter/data/tweets.txt" # This script saves the tweets into this blob.

        $trackString = "Azure, Cloud, HDInsight" # This script gets the tweets containing these keywords.
        $track = [System.Uri]::EscapeDataString($trackString);
        $lineMax = 10000  # The script will get this number of tweets. It is about 3 minutes every 100 lines.
        #endregion

        #region - Connect to Azure subscription
        Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        #endregion

        #region - Create a block blob object for writing tweets into Blob storage
        Write-Host "Get the default storage account name and Blob container name using the cluster name ..." -ForegroundColor Green
        $myCluster = Get-AzureRmHDInsightCluster -Name $clusterName
        $resourceGroupName = $myCluster.ResourceGroup
        $storageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.chinacloudapi.cn", "")
        $containerName = $myCluster.DefaultStorageContainer
        Write-Host "`tThe storage account name is $storageAccountName." -ForegroundColor Yellow
        Write-Host "`tThe blob container name is $containerName." -ForegroundColor Yellow

        Write-Host "Define the Azure storage connection string ..." -ForegroundColor Green
        $storageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName)[0].Value
        $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$storageAccountName;AccountKey=$storageAccountKey"
        Write-Host "`tThe connection string is $storageConnectionString." -ForegroundColor Yellow

        Write-Host "Create block blob object ..." -ForegroundColor Green
        $storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
        $storageClient = $storageAccount.CreateCloudBlobClient();
        $storageContainer = $storageClient.GetContainerReference($containerName)
        $destBlob = $storageContainer.GetBlockBlobReference($destBlobName)
        #end region

        # region - Format OAuth strings
        Write-Host "Format oauth strings ..." -ForegroundColor Green
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
        #endregion

        #region - Read tweets
        Write-Host "Create HTTP web request ..." -ForegroundColor Green
        [System.Net.HttpWebRequest] $request = [System.Net.WebRequest]::Create("https://stream.twitter.com/1.1/statuses/filter.json");
        $request.Method = "POST";
        $request.Headers.Add("Authorization", $oauth_authorization);
        $request.ContentType = "application/x-www-form-urlencoded";
        $body = $request.GetRequestStream();

        $body.write($post_body, 0, $post_body.length);
        $body.flush();
        $body.close();
        $response = $request.GetResponse() ;

        Write-Host "Start stream reading ..." -ForegroundColor Green

        Write-Host "Define a MemoryStream and a StreamWriter for writing ..." -ForegroundColor Green
        $memStream = New-Object System.IO.MemoryStream
        $writeStream = New-Object System.IO.StreamWriter $memStream

        $sReader = New-Object System.IO.StreamReader($response.GetResponseStream())

        $inrec = $sReader.ReadLine()
        $count = 0
        while (($inrec -ne $null) -and ($count -le $lineMax))
        {
            if ($inrec -ne "")
            {
                Write-Host "`n`t $count tweets received." -ForegroundColor Yellow

                $writeStream.WriteLine($inrec)
                $count ++
            }

            $inrec=$sReader.ReadLine()
        }
        #endregion

        #region - Write tweets to Blob storage
        Write-Host "Write to the destination blob ..." -ForegroundColor Green
        $writeStream.Flush()
        $memStream.Seek(0, "Begin")
        $destBlob.UploadFromStream($memStream)

        $sReader.close()
        #endregion

        Write-Host "Completed!" -ForegroundColor Green

3. 设置此脚本中的前五个到前八个变量：

    变量|说明
    ---|---
    $clusterName|这是要在其运行该应用程序的 HDInsight 群集的名称。
    $oauth\_consumer\_key|这是你先前在创建 Twitter 应用程序时记下的 Twitter 应用程序**使用者密钥**。
    $oauth\_consumer\_secret|这是你先前记下的 Twitter 应用程序**使用者机密**。
    $oauth\_token|这是你先前记下的 Twitter 应用程序**访问令牌**。
    $oauth\_token\_secret|这是你先前记下的 Twitter 应用程序**访问令牌机密**。
    $destBlobName|这是输出 Blob 名称。默认值是 **tutorials/twitter/data/tweets.txt**。如果更改默认值，则需要相应地更新 Windows PowerShell 脚本。
    $trackString|Web 服务将返回与这些关键字相关的推文。默认值为“Azure, 云, HDInsight”。如果更改默认值，则需要相应地更新 Windows PowerShell 脚本。
    $lineMax|此值确定脚本将读取多少篇推文。读取 100 篇推文需要大约三分钟。你可以设置更大的数，但需要更长时间来下载。

1. 按 **F5** 运行脚本。如果遇到问题，一种解决方法是选择所有行，然后按 **F8**。
2. 你会在输出的末尾看到“Complete!”。任何错误消息都将显示为红色。

在验证过程中，你可以使用 Azure 存储资源管理器或 Azure PowerShell 查看 Azure Blob 存储中的输出文件 **/tutorials/twitter/data/tweets.txt**。有关用于列出文件的示例 Windows PowerShell 脚本，请参阅[将 Blob 存储与 HDInsight 配合使用][hdinsight-storage-powershell]。

## 创建 HiveQL 脚本
使用 Azure PowerShell，你可以一次运行多个 HiveQL 语句，或者将 HiveQL 语句打包到一个脚本文件中。在本教程中，你将创建 HiveQL 脚本。脚本文件必须上传到 Azure Blob 存储中。在下一节中，你将使用 Azure PowerShell 运行脚本文件。

> [AZURE.NOTE]
已将 Hive 脚本文件和包含 10,000 篇推文的文件上传到公共 Blob 容器中。如果要使用已上传的文件，可以跳过本节。

HiveQL 脚本将执行以下操作：

1. **删除 tweets\_raw 表**（如果该表已存在）。
2. **创建 tweets\_raw Hive 表**。此临时 Hive 结构化表保存要进一步进行提取、转换和加载 (ETL) 处理的数据。有关分区的信息，请参阅 [Hive 教程][apache-hive-tutorial]。
3. 从源文件夹 (/tutorials/twitter/data) **加载数据**。采用嵌套 JSON 格式的大型推文数据集现已转换为临时 Hive 表结构。
4. **删除 tweets 表**（如果该表已存在）；
5. **创建 tweets 表**。你需要先运行其他 ETL 过程，然后才能使用 Hive 对推文数据集进行查询。此 ETL 过程将为已存储在“twitter\_raw”表中的数据定义更详细的表架构。
6. **插入覆盖表**。此复杂的 Hive 脚本将通过 Hadoop 群集启动一组较长的 Map Reduce 作业。这可能需要约 10 分钟的时间，具体取决于你的数据集和群集大小。
7. **插入覆盖目录**。运行查询并将数据集输出到文件。此查询将返回发送最多包含单词“Azure”的推文的 Twitter 用户列表。

**创建 Hive 脚本并将它上传到 Azure**

1. 打开 Windows PowerShell ISE。
2. 将以下脚本复制到脚本窗格中：

        #region - variables and constants
        $clusterName = "<Existing HDInsight Cluster Name>" # Enter your HDInsight cluster name
        $subscriptionID = "<Azure Subscription ID>"

        $sourceDataPath = "/tutorials/twitter/data"
        $outputPath = "/tutorials/twitter/output"
        $hqlScriptFile = "tutorials/twitter/twitter.hql"

        $hqlStatements = @"
        set hive.exec.dynamic.partition = true;
        set hive.exec.dynamic.partition.mode = nonstrict;

        DROP TABLE tweets_raw;
        CREATE EXTERNAL TABLE tweets_raw (
            json_response STRING
        )
        STORED AS TEXTFILE LOCATION '$sourceDataPath';

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

        FROM tweets_raw
        INSERT OVERWRITE TABLE tweets
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

        INSERT OVERWRITE DIRECTORY '$outputPath'
        SELECT name, screen_name, count(1) as cc
            FROM tweets
            WHERE text like "%Azure%"
            GROUP BY name,screen_name
            ORDER BY cc DESC LIMIT 10;
        "@
        #endregion

        #region - Connect to Azure subscription
        Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green

        Try{
            Get-AzureRmSubscription
        }
        Catch{
            Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        Select-AzureRmSubscription -SubscriptionId $subscriptionID

        #endregion

        #region - Create a block blob object for writing the Hive script file
        Write-Host "Get the default storage account name and container name based on the cluster name ..." -ForegroundColor Green
        $myCluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroupName = $myCluster.ResourceGroup
        $defaultStorageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.chinacloudapi.cn", "")
        $defaultBlobContainerName = $myCluster.DefaultStorageContainer
        Write-Host "`tThe storage account name is $defaultStorageAccountName." -ForegroundColor Yellow
        Write-Host "`tThe blob container name is $defaultBlobContainerName." -ForegroundColor Yellow

        Write-Host "Define the connection string ..." -ForegroundColor Green
        $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
        $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$defaultStorageAccountName;AccountKey=$defaultStorageAccountKey"

        Write-Host "Create block blob objects referencing the hql script file" -ForegroundColor Green
        $storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
        $storageClient = $storageAccount.CreateCloudBlobClient();
        $storageContainer = $storageClient.GetContainerReference($defaultBlobContainerName)
        $hqlScriptBlob = $storageContainer.GetBlockBlobReference($hqlScriptFile)

        Write-Host "Define a MemoryStream and a StreamWriter for writing ... " -ForegroundColor Green
        $memStream = New-Object System.IO.MemoryStream
        $writeStream = New-Object System.IO.StreamWriter $memStream
        $writeStream.Writeline($hqlStatements)
        #endregion

        #region - Write the Hive script file to Blob storage
        Write-Host "Write to the destination blob ... " -ForegroundColor Green
        $writeStream.Flush()
        $memStream.Seek(0, "Begin")
        $hqlScriptBlob.UploadFromStream($memStream)
        #endregion

        Write-Host "Completed!" -ForegroundColor Green

3. 设置此脚本中的前两个变量：

    | 变量 | 说明 |
    | --- | --- |
    | $clusterName |输入要在其中运行该应用程序的 HDInsight 群集名称。 |
    | $subscriptionID |输入 Azure 订阅 ID。 |
    | $sourceDataPath |Hive 查询将从中读取数据的 Azure Blob 存储位置。无需更改此变量。 |
    | $outputPath |Hive 查询将在其中输出结果的 Azure Blob 存储位置。无需更改此变量。 |
    | $hqlScriptFile |HiveQL 脚本文件的位置和文件名。无需更改此变量。 |
4. 按 **F5** 运行脚本。如果遇到问题，一种解决方法是选择所有行，然后按 **F8**。
5. 你会在输出的末尾看到“Complete!”。任何错误消息都将显示为红色。

在验证过程中，你可以使用 Azure 存储资源管理器或 Azure PowerShell 查看 Azure Blob 存储中的输出文件 **/tutorials/twitter/twitter.hql**。有关用于列出文件的示例 Windows PowerShell 脚本，请参阅[将 Blob 存储与 HDInsight 配合使用][hdinsight-storage-powershell]。

## 使用 Hive 处理 Twitter 数据
你已完成所有准备工作。现在，你可以调用 Hive 脚本，并检查结果。

### 提交 Hive 作业
使用以下 Windows PowerShell 脚本运行 Hive 脚本。你将需要设置第一个变量。

> [AZURE.NOTE]
若要使用最后两部分中上传的推文和 HiveQL 脚本，请将 $hqlScriptFile 设置为“/tutorials/twitter/twitter.hql”。若要使用已为你上传到公共 blob 的推文和 HiveQL 脚本，请将 $hqlScriptFile 设置为“wasbs://twittertrend@hditutorialdata.blob.core.windows.net/twitter.hql”。

    #region variables and constants
    $clusterName = "<Existing Azure HDInsight Cluster Name>"
    $httpUserName = "admin"
    $httpUserPassword = "<HDInsight Cluster HTTP User Password>"

    #use one of the following
    $hqlScriptFile = "wasbs://twittertrend@hditutorialdata.blob.core.windows.net/twitter.hql"
    $hqlScriptFile = "/tutorials/twitter/twitter.hql"

    $statusFolder = "/tutorials/twitter/jobstatus"
    #endregion

    $myCluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $myCluster.ResourceGroup
    $defaultStorageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.chinacloudapi.cn", "")
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value

    $defaultBlobContainerName = $myCluster.DefaultStorageContainer

    #region - Invoke Hive
    Write-Host "Invoke Hive ... " -ForegroundColor Green

    # Create the HDInsight cluster
    $pw = ConvertTo-SecureString -String $httpUserPassword -AsPlainText -Force
    $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

    Use-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName -HttpCredential $httpCredential
    $response = Invoke-AzureRmHDInsightHiveJob -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -file $hqlScriptFile -StatusFolder $statusFolder #-OutVariable $outVariable

    Write-Host "Display the standard error log ... " -ForegroundColor Green
    $jobID = ($response | Select-String job_ | Select-Object -First 1) -replace '\s*$' -replace '.*\s'
    Get-AzureRmHDInsightJobOutput -ClusterName $clusterName -JobId $jobID -DefaultContainer $defaultBlobContainerName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -HttpCredential $httpCredential
    #endregion

### 检查结果
使用以下 Windows PowerShell 脚本检查 Hive 作业输出。你将需要设置前两个变量。

    #region variables and constants
    $clusterName = "<Existing Azure HDInsight Cluster Name>"

    $blob = "tutorials/twitter/output/000000_0" # The name of the blob to be downloaded.
    #endregion

    #region - Create an Azure storage context object
    Write-Host "Get the default storage account name and container name based on the cluster name ..." -ForegroundColor Green
    $myCluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $myCluster.ResourceGroup
    $defaultStorageAccountName = $myCluster.DefaultStorageAccount.Replace(".blob.core.chinacloudapi.cn", "")
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $defaultBlobContainerName = $myCluster.DefaultStorageContainer

    Write-Host "`tThe storage account name is $defaultStorageAccountName." -ForegroundColor Yellow
    Write-Host "`tThe blob container name is $defaultBlobContainerName." -ForegroundColor Yellow

    Write-Host "Create a context object ... " -ForegroundColor Green
    $storageContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey
    #endregion

    #region - Download blob and display blob
    Write-Host "Download the blob ..." -ForegroundColor Green
    cd $HOME
    Get-AzureStorageBlobContent -Container $defaultBlobContainerName -Blob $blob -Context $storageContext -Force

    Write-Host "Display the output ..." -ForegroundColor Green
    Write-Host "==================================" -ForegroundColor Green
    cat "./$blob"
    Write-Host "==================================" -ForegroundColor Green
    #end region

> [AZURE.NOTE]
Hive 表使用 \\001 作为字段分隔符。该分隔符在输出中不可见。

在将分析结果放到 Azure Blob 存储中后，可以将数据导出到 Azure SQL 数据库/SQL Server、使用 Power Query 将数据导出到 Excel，或者使用 Hive ODBC 驱动程序将应用程序连接到数据。有关详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]、[使用 HDInsight 分析外部测试版延迟数据][hdinsight-analyze-flight-delay-data]、[利用 Power Query 将 Excel 连接到 HDInsight][hdinsight-power-query]，以及[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][hdinsight-hive-odbc]。

## 后续步骤
在本教程中，我们已了解如何将非结构化 JSON 数据集转换为结构化 Hive 表，以便在 Azure 上使用 HDInsight 查询、探究和分析来自 Twitter 的数据。若要了解详细信息，请参阅以下文章：

* [HDInsight 入门][hdinsight-get-started]
* [在 HDInsight 中使用 HBase 分析实时 Twitter 观点][hdinsight-hbase-twitter-sentiment]
* [使用 HDInsight 分析外部测试版延迟数据][hdinsight-analyze-flight-delay-data]
* [利用 Power Query 将 Excel 连接到 HDInsight][hdinsight-power-query]
* [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][hdinsight-hive-odbc]
* [将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[apache-hive-tutorial]: https://cwiki.apache.org/confluence/display/Hive/Tutorial

[twitter-streaming-api]: https://dev.twitter.com/docs/streaming-apis
[twitter-statuses-filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-install]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[powershell-script]: http://technet.microsoft.com/zh-cn/library/ee176961.aspx

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows/
[hdinsight-storage-powershell]: /documentation/articles/hdinsight-hadoop-use-blob-storage/#access-blobs-using-azure-powershell
[hdinsight-analyze-flight-delay-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/
[hdinsight-hive-odbc]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/
[hdinsight-hbase-twitter-sentiment]: /documentation/articles/hdinsight-hbase-analyze-twitter-sentiment/

<!---HONumber=Mooncake_0103_2017-->