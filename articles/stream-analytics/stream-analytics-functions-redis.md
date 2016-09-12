<properties
	pageTitle="使用 Azure Functions 从 Azure 流分析输出到 Azure Redis 缓存 | Azure"
	description="了解如何使用已连接到服务总线队列的 Azure 函数在 Azure Redis 缓存中填充流分析作业的输出。"
	keywords="数据流, redis 缓存, 服务总线队列"
	services="stream-analytics"
	authors="ryancrawcour"
	manager="paulettm"
	documentationCenter=""
	/>

<tags
	ms.service="stream-analytics"
	ms.date="08/09/2016"
	wacn.date="09/12/2016"
/>

# 如何使用 Azure Functions 在 Azure Redis 缓存中存储来自 Azure 流分析的数据

Azure 流分析可帮助快速开发和部署低成本分析解决方案，以便从设备、传感器、基础结构和应用程序或任何数据流实时获取深入了解。它可以实现不同的使用方案，例如，实时管理与监视、命令与控制、欺诈检测、车联网等等。在其中的许多情况下，可能需要将 Azure 流分析输出的数据存储到分布式数据存储，例如 Azure Redis 缓存。

假设某个用户是某一家电信公司的员工。该用户要尝试检测 SIM 卡欺诈，查出在同一时间使用同一身份，但位于不同地理位置的多个通话。任务是将所有潜在的欺诈电话存储在 Azure Redis 缓存中。在本博客中，我们将提供有关如何轻松完成此任务的指导。

## 先决条件
完成 ASA 的 [Real-time Fraud Detection][fraud-detection]（实时欺诈检测）演练

## 体系结构概述
![体系结构屏幕截图](./media/stream-analytics-functions-redis/architecture-overview.png)  


如上图所示，流分析允许查询流输入数据并将其发送到输出。根据输出，Azure Functions 可以触发某种类型的事件。

在本博客中，我们着重于此管道中 Azure Functions 的部分，更具体地说，则触发将欺诈数据存储到缓存的事件。完成 [Real-time Fraud Detection][fraud-detection]（实时欺诈检测）教程之后，已配置并运行输入（事件中心）、查询和输出（Blob 存储）。在本博客中，我们将输出改为使用服务总线队列。接下来，我们将一个 Azure 函数连接到此队列。

## 创建并连接服务总线队列输出
若要创建服务总线队列，请遵循 [Get Started with Service Bus Queues][servicebus-getstarted]（服务总线队列入门）的“.NET”部分中的步骤 1 和 2。
现在，让我们将队列连接到以前在欺诈检测演练中所创建的流分析作业。



1. 在 Azure 门户中，转到作业的“输出”边栏选项卡，然后选择页面顶部的“添加”。

	![添加输出](./media/stream-analytics-functions-redis/adding-outputs.png)  


2. 选择“服务总线队列”作为“接收器”，然后遵循屏幕上的说明操作。请务必选择在 [Get Started with Service Bus Queues][servicebus-getstarted]（服务总线队列入门）中创建的服务总线队列命名空间。完成后，单击“向右箭头”按钮。
3. 指定以下值：
	- **事件序列化程序格式**：JSON
	- **编码**：UTF8
	- **格式**：行分隔

4. 单击“创建”按钮添加此源，并验证流分析是否可以成功连接到存储帐户。

5. 在“查询”选项卡上，将当前查询替换为以下内容。将 *[YOUR SERVICE BUS NAME]* 替换为步骤 3 中创建的输出名称。

    ```    

	    SELECT 
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1, 
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2

        INTO [YOUR SERVICE BUS NAME]
    
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
	    JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
    
        WHERE CS1.SwitchNum != CS2.SwitchNum
    
    ```

## 创建 Azure Redis 缓存
遵循 [How to Use Azure Redis Cache][use-rediscache]（如何使用 Azure Redis 缓存）中从“.NET”部分到 ***Configure the cache clients***（配置缓存客户端）部分的内容，创建 Azure Redis 缓存。
完成后，将获得新的 Redis 缓存。在“所有设置”下面，选择“访问密钥”并记下“主要连接字符串”。

![体系结构屏幕截图](./media/stream-analytics-functions-redis/redis-cache-keys.png)  


## 创建 Azure 函数
遵循 [Create your first Azure Function][functions-getstarted]（创建第一个 Azure 函数）教程开始使用 Azure Functions。如果已有想要使用的 Azure 函数，请直接跳到[写入 Redis 缓存](#Writing-to-Redis-Cache)

1. 在门户中，从左侧导航栏中选择“应用程序服务”，然后单击 Azure 函数应用名称，获取函数的应用网站。
![应用程序服务函数列表屏幕截图](./media/stream-analytics-functions-redis/app-services-function-list.png)

2. 单击“添加函数”>“ServiceBusQueueTrigger – C#”。对于以下字段，请遵循以下说明：
	- **队列名称**：与在 [Get Started with Service Bus Queues][servicebus-getstarted]（服务总线队列入门）中创建队列时所输入的名称相同（不是服务总线的名称）。请确保使用已连接到流分析输出的队列。
	- **服务总线连接**：选择“添加连接字符串”。若要查找连接字符串，请转到经典门户，然后依次选择“服务总线”、创建的服务总线，以及屏幕底部的“连接信息”。请确保在此页的主屏幕上操作。复制并粘贴连接字符串。可以输入任意连接名称。
	
		![服务总线连接屏幕截图](./media/stream-analytics-functions-redis/servicebus-connection.png)  

	- **AccessRights**：选择“管理”


3. 单击“创建”

## 写入 Redis 缓存
我们现已创建一个可从服务总线队列进行读取的 Azure 函数。余下的工作就是使用函数将此数据写入 Redis 缓存。

1. 选择刚刚创建的 **ServiceBusQueueTrigger**，然后单击右上角的“函数应用设置”。选择“转到应用服务设置”>“设置”>“应用程序设置”

2. 在“连接字符串”部分中的“名称”部分中创建一个名称。在“值”部分中，粘贴在“创建 Redis 缓存”步骤中找到的主要连接字符串。选择“SQL 数据库”旁边的“自定义”。

3. 单击顶部的“保存”。

	![服务总线连接屏幕截图](./media/stream-analytics-functions-redis/function-connection-string.png)

4. 现在，请返回到“应用服务设置”，然后选择“工具”>“应用服务编辑器(预览版)”>“打开”>“转到”。

	![服务总线连接屏幕截图](./media/stream-analytics-functions-redis/app-service-editor.png)

5. 在所选的编辑器中，创建包含以下内容的名为 **project.json** 的 JSON 文件，并将它保存到本地磁盘。

        {
            "frameworks": {
		        "net46": {
		            "dependencies": {
				        "StackExchange.Redis":"1.1.603",
				        "Newtonsoft.Json": "9.0.1"
			        }
		        }
	        }
        }

6. 将此文件上载到函数的根目录（不是 WWWROOT）。应会看到名为 **project.lock.json** 的文件自动显示，确认 Nuget 包“StackExchange.Redis”和“Newtonsoft.Json”已导入。

7. 在 **run.csx** 文件中，将预先生成的代码替换为以下代码。在 lazyConnection 函数中，将“CONN NAME”替换为**将数据存储到 Redis 缓存**的步骤 2 中所创建的名称。

````

	using System;
    using System.Threading.Tasks;
    using StackExchange.Redis;
    using Newtonsoft.Json;
    using System.Configuration;
    
    public static void Run(string myQueueItem, TraceWriter log)
    {
        log.Info($"Function processed message: {myQueueItem}");

        // Connection refers to a property that returns a ConnectionMultiplexer
        IDatabase db = Connection.GetDatabase();
        log.Info($"Created database {db}");
    
        // Parse JSON and extract the time
        var message = JsonConvert.DeserializeObject<dynamic>(myQueueItem);
        string time = message.time;
        string callingnum1 = message.callingnum1;

        // Perform cache operations using the cache object...
        // Simple put of integral data types into the cache
        string key = time + " - " + callingnum1;
        db.StringSet(key, myQueueItem);
        log.Info($"Object put in database. Key is {key} and value is {myQueueItem}");

        // Simple get of data types from the cache
        string value = db.StringGet(key);
        log.Info($"Database got: {value}"); 
    }

    // Connect to the Service Bus
    private static Lazy<ConnectionMultiplexer> lazyConnection = 
		new Lazy<ConnectionMultiplexer>(() =>
    		{
				var cnn = ConfigurationManager.ConnectionStrings["CONN NAME"].ConnectionString
        		return ConnectionMultiplexer.Connect();
    		});
    
    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }

````

## 启动流分析作业

1. 启动 telcodatagen.exe 应用程序。用法如下：````telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]````

2. 在门户上的“流分析作业”边栏选项卡中，单击页面顶部的“启动”。

	![启动作业屏幕截图](./media/stream-analytics-functions-redis/starting-job.png)

3. 在出现的“启动作业”边栏选项卡中，选择“立即”，然后单击屏幕底部的“启动”按钮。作业状态将更改为“正在启动”，片刻之后会更改为“正在运行”。
 
	![启动作业时间选择屏幕截图](./media/stream-analytics-functions-redis/start-job-time.png)  


## 运行解决方案并检查结果
返回到“ServiceBusQueueTrigger”页，现在应会看到日志语句。这些日志显示从服务总线队列获取了某些内容、将这些内容放入了数据库，并使用时间作为键提取了这些内容！

若要验证数据是否在 Redis 缓存中，请转到新门户中的 Redis 缓存页（如前面的[创建 Azure Redis 缓存](#Create-an-Azure-Redis-Cache)步骤中所示），然后选择“控制台”。

现在可以编写 Redis 命令来确认数据是否确实位于缓存中。

![Redis 控制台屏幕截图](./media/stream-analytics-functions-redis/redis-console.png)

## 后续步骤
我们对于 Azure Functions 及流分析可以配合工作的新功能感到兴奋，并希望为带来突破。

如果是 Azure 的新用户，我们邀请通过注册一个 [Azure 试用帐户](/pricing/1rmb-trial/)，以尝试这项服务。如果不熟悉流分析，我们邀请[创建第一个流分析作业](/documentation/articles/stream-analytics-create-a-job/)。

如需任何帮助或有任何疑问，请在 [MSDN](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStreamAnalytics) 或 [Stackoverflow](http://stackoverflow.com/questions/tagged/azure-stream-analytics) 论坛中发布。

也可以参阅以下资源：

- [Azure Functions developer reference](/documentation/articles/functions-reference/)（Azure Functions 开发人员参考）
- [Azure Functions C# developer reference](/documentation/articles/functions-reference-csharp/)（Azure Functions C# 开发人员参考）
- [Azure Functions NodeJS developer reference](/documentation/articles/functions-reference/)（Azure Functions NodeJS 开发人员参考）
- [Azure Functions triggers and bindings](/documentation/articles/functions-triggers-bindings/)（Azure Functions 触发器和绑定）
- [如何监视 Azure Redis 缓存](/documentation/articles/cache-how-to-monitor/)


[fraud-detection]: /documentation/articles/stream-analytics-real-time-fraud-detection/
[servicebus-getstarted]: /documentation/articles/service-bus-dotnet-get-started-with-queues/
[use-rediscache]: /documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/
[functions-getstarted]: /documentation/articles/functions-create-first-azure-function/

<!---HONumber=Mooncake_0905_2016-->