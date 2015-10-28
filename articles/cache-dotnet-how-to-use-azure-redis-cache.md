<properties 
	pageTitle="如何使用 Azure Redis Cache" 
	description="了解如何使用 Azure Redis 缓存提高 Azure 应用程序的性能" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.date="05/20/2015" 
	wacn.date="06/26/2015"/>

# 如何使用 Azure Redis Cache

本指南说明如何开始使用 **Azure Redis 缓存（预览版）**。示例是用 C# 代码编写的，并使用了 [StackExchange.Redis][] 客户端。涉及的应用场景包括**创建和配置缓存**、**配置缓存客户端**、**从缓存添加和删除对象**，以及**在缓存中存储 ASP.NET 会话状态**。有关使用 Azure Redis 缓存的详细信息，请参阅[后续步骤][]部分。

<a name="what-is"></a>
## 什么是 Azure Redis Cache？

Windows Azure Redis 缓存目前以预览版提供，基于流行的开放源代码 Redis 缓存。它让您访问 Microsoft 管理的安全专用的 Redis 缓存。使用 Azure Redis 缓存创建的缓存可从 Windows Azure 内的任何应用程序进行访问。

Windows Azure Redis 缓存现已推出两种层级：

-	**基本** - 单个节点。多种大小，最大 53 GB。
-	**标准** – 双节点主/副本配置。多种大小，最大 53 GB。99.9% SLA。

每个级别在功能和定价方面存在差异。在本指南的后面将介绍这些功能，而有关定价的详细信息，则请参阅[缓存定价详细信息][]。

本指南提供了 Azure Redis Cache 的入门概述。有关超出本入门指南范围的那些功能的更多详细信息，请参阅 [Azure Redis 缓存概述][]。

<a name="getting-started-cache-service"></a>
## 开始使用 Azure Redis Cache

Azure Redis Cache 非常容易上手。若要开始使用，需要首先设置和配置缓存。接下来，配置缓存客户端，以便它们可以访问缓存。在配置了缓存客户端后，就可以开始使用它们。

-	[创建缓存][]
-	[配置缓存客户端][]

<a name="create-cache"></a>
## 创建缓存

若要创建缓存，首先请登录到 [Windows Azure 门户][]，然后单击“新建”、“数据 + 存储”、“Redis 缓存”。

![新建缓存][NewCacheMenu]

>[AZURE.NOTE]如果你没有 Azure 帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用][]。

在“新建 Redis 缓存”边栏选项卡中，指定所需的缓存配置。

![创建缓存][CacheCreate]

在“DNS 名称”中，输入要用于缓存终结点的域名。该终结点必须是长度在 6 到 20 个字符之间的字符串，仅包含小写的数字和字母，并且必须以字母开头。

使用“定价层”选择所需的缓存大小和功能。“基本”缓存提供单个节点和多种大小（最大为 53 GB）。“标准”缓存提供双节点主/辅助配置，、99.9% SLA 和多种大小（最大为 53 GB）。

在“资源组”中，为缓存选择或创建资源组。

对于“订阅”，请选择要用于缓存的 Azure 订阅。如果你的帐户仅具有一个订阅，将自动选择该订阅并且将不显示“订阅”下拉菜单。

使用“位置”指定在其中托管你的缓存的地理位置。Microsoft 强烈推荐的最佳做法，是在与缓存客户端应用程序相同的区域中创建缓存。

在配置了新的缓存选项后，单击“创建”。创建缓存可能耗时几分钟。要检查的状态，可以监视开始板上的进度。创建缓存后，新缓存的状态为“正在运行”并可供用于默认设置。

![创建的缓存][CacheCreated]

创建缓存后，可以从“浏览”边栏选项卡访问它。

![浏览边栏选项卡][BrowseCaches]

单击“Redis 缓存”查看你的缓存。

![缓存][Caches]

<a name="NuGet"></a>
## 配置缓存客户端

使用 Azure Redis 缓存创建的缓存可从任何应用程序访问。在 Visual Studio 中开发的 .NET 应用程序可以使用 **StackExchange.Redis** 缓存客户端，可使用 NuGet 包进行配置，以简化缓存客户端应用程序的配置。

>[AZURE.NOTE]有关详细信息，请参阅 [StackExchange.Redis][] github 页面和 [StackExchange.Redis 缓存客户端文档][]。

若要使用 StackExchange.Redis NuGet 包配置客户端应用程序，请在“解决方案资源管理器”中右键单击项目，然后选择“管理 NuGet 包”。

![管理 NuGet 包][NuGetMenu]

在“联机搜索”文本框中键入 **StackExchange.Redis** 或 **StackExchange.Redis.StrongName**，从结果选择它，然后单击“安装”。

>[AZURE.NOTE]如果你希望使用 **StackExchange.Redis** 客户端库强名称版本，请选择 **StackExchange.Redis.StrongName**；否则选择 **StackExchange.Redis**。

![StackExchange.Redis NuGet 程序包][StackExchangeNuget]

NuGet 程序包会给客户端应用程序下载并添加所需的程序集引用，以访问带 StackExchange.Redis 缓存客户端的 Azure Redis Cache。

在配置了你的客户端项目的缓存后，你可以使用以下各节中介绍的方法来使用你的缓存。

<a name="working-with-caches"></a>
## 使用缓存

本节中的步骤介绍如何使用缓存执行常见任务。

-	[连接到缓存][]
-   [添加和从缓存检索对象][]
-   [在缓存中存储 ASP.NET 会话状态][]

<a name="connect-to-cache"></a>
## 连接到缓存

若要以编程方式使用缓存，你需要引用该缓存。以下代码添加到您想使用 StackExchange.Redis 客户端的任何文件的顶部，以访问 Azure Redis Cache：

    using StackExchange.Redis;

>[AZURE.NOTE]StackExchange.Redis 客户端需要.NET Framework 4 或更高版本。

到 Azure Redis 缓存的连接由 `ConnectionMultiplexer` 类管理。此类旨在共享并在客户端应用程序中重复使用，不需要在每次执行操作的基础上创建。

要连接到 Azure Redis 缓存并返回连接的 `ConnectionMultiplexer` 的实例，请调用静态 `Connect` 方法并传递到缓存端点和密钥中，如下例所示。使用从门户生成的 Azure 密钥作为 password 参数。

	ConnectionMultiplexer connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.chinacloudapi.cn,ssl=true,password=...");

>[AZURE.IMPORTANT]警告：切勿将凭据存储在源代码中。为了使本示例简单明了，我将以源代码来呈现凭据内容。有关如何存储凭据的详细信息，请参阅[应用程序字符串和连接字符串的工作原理][]。

如果你不想要使用 SSL，请设置 `ssl=false` 或只传入终结点和密钥。

>[AZURE.NOTE]默认情况下，将为新缓存禁用非 SSL 端口。<!--有关启用非 SSL 端口的说明，请参阅[在 Azure Redis 缓存中配置缓存][]主题中的“访问端口”部分。-->

	connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.chinacloudapi.cn,password=...");

有关高级连接配置选项的详细信息，请参阅 [StackExchange.Redis 配置模型][]。

可以从缓存实例的“Redis 缓存”边栏选项卡中获取缓存终结点和密钥。

![缓存属性][CacheProperties]

![管理密钥][ManageKeys]

建立连接后，通过调用 `ConnectionMultiplexer.GetDatabase` 方法返回对 Redis 缓存数据库的引用。

	// connection referes to a previously configured ConnectionMultiplexer
	IDatabase cache = connection.GetDatabase();

>[AZURE.NOTE]从 `GetDatabase` 方法返回的对象是一个轻型直通对象，不需要存储。

	ConnectionMultiplexer connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.chinacloudapi.cn,ssl=true,password=...");

	IDatabase cache = connection.GetDatabase();

	// Perform cache operations using the cache object...
	// Simple put of integral data types into the cache
	cache.StringSet("key1", "value");
	cache.StringSet("key2", 25);

	// Simple get of data types from the cache
	string key1 = cache.StringGet("key1");
	int key2 = (int)cache.StringGet("key2");

现在您知道如何连接到 Azure Redis Cache 实例并将引用返回缓存数据库，让我们看看如何使用缓存。



<a name="add-object"></a>
## 添加和从缓存检索对象

可以使用 `StringSet` 和 `StringGet` 方法在缓存中存储和检索项。

	// If key1 exists, it is overwritten.
	cache.StringSet("key1", "value1");

	string value = cache.StringGet("key1");

>[AZURE.NOTE]Redis 将大多数数据存储为 Redis 字符串，但这些字符串可能包含许多类型的数据，包括序列化的二进制数据，可在缓存中存储 .NET 对象时使用。

调用 `StringGet` 时，如果该对象存在，则返回它，如果该对象不存在，则返回 null。在这种情况可以从所需的数据源检索值，并将其存储在缓存中供后续使用。这称为缓存端模式。

    string value = cache.StringGet("key1");
    if (value == null)
    {
        // The item keyed by "key1" is not in the cache. Obtain
        // it from the desired data source and add it to the cache.
        value = GetValueFromDataSource();

        cache.StringSet("key1", value);
    }

>[AZURE.NOTE]Azure Redis 缓存可以缓存 .NET 对象以及基元数据类型，但在缓存 .NET 对象之前，必须对其进行序列化。这是应用程序开发人员的责任，同时赋与开发人员选择序列化程序的弹性。有关详细信息，请参阅[处理缓存中的 .NET 对象][]。

<a name="specify-expiration"></a>
## 在缓存中指定项目的有效期

要在缓存中指定项的过期时间，请使用 `StringSet` 的 `TimeSpan` 参数。

	cache.StringSet("key1", "value1", TimeSpan.FromMinutes(90));


<a name="store-session"></a>
## 在缓存中存储 ASP.NET 会话状态

Azure Redis Cache 提供一个会话状态提供程序，可用于在缓存而不是内存或 SQL Server 数据库中存储您的会话状态。若要使用缓存会话状态提供程序，首先配置你的缓存，然后使用 Redis 缓存会话状态 NuGet 包为缓存配置 ASP.NET 应用程序。

若要使用 Redis 缓存会话状态 NuGet 包配置客户端应用程序，请在“解决方案资源管理器”中右键单击项目，然后选择“管理 NuGet 包”。

![管理 NuGet 包][NuGetMenu]

在“联机搜索”文本框中键入 **RedisSessionStateProvider**，从结果选择它，然后单击“安装”。

![Redis Cache 会话状态 NuGet 程序包][SessionStateNuGet]

NuGet 程序包会下载并添加所需的程序集引用，并将以下部分添加到您的 web.config 文件，包含 ASP.NET 应用程序所需的配置，以使用 Redis Cache 会话状态提供程序。

	<sessionState mode="Custom" customProvider="MySessionStateStore">
      <providers>
        <!--
          <add name="MySessionStateStore" 
            host = "127.0.0.1" [String]
            port = "" [number]
            accessKey = "" [String]
            ssl = "false" [true|false]
            throwOnError = "true" [true|false]
            retryTimeoutInMilliseconds = "0" [number]
            databaseId = "0" [number]
            applicationName = "" [String]
            connectionTimeoutInMilliseconds = "5000" [number]
            operationTimeoutInMilliseconds = "5000" [number]
          />
        -->
        <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="127.0.0.1" accessKey="" ssl="false" />
      </providers>
    </sessionState>

注释部分为它们提供属性和示例设置的一个示例。

使用来自门户的缓存边栏选项卡的值配置属性，并根据需要配置其他值。

	<sessionState mode="Custom" customProvider="MySessionStateStore">
      <providers>
        <!--
          <add name="MySessionStateStore" 
            host = "127.0.0.1" [String]
            port = "" [number]
            accessKey = "" [String]
            ssl = "false" [true|false]
            throwOnError = "true" [true|false]
            retryTimeoutInMilliseconds = "0" [number]
            databaseId = "0" [number]
            applicationName = "" [String]
            connectionTimeoutInMilliseconds = "5000" [number]
            operationTimeoutInMilliseconds = "5000" [number]
          />
        -->
        <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="contoso5.redis.cache.chinacloudapi.cn" 
		accessKey="..." ssl="true" />
      </providers>
    </sessionState>

请务必注释掉标准 **InProc** 会话状态提供程序。

    <!-- <sessionState mode="InProc" customProvider="DefaultSessionProvider">
      <providers>
        <add name="DefaultSessionProvider" type="System.Web.Providers.DefaultSessionStateProvider, System.Web.Providers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" connectionStringName="DefaultConnection" />
      </providers>
    </sessionState> -->

有关配置这些设置和使用 Azure Redis 会话状态提供程序的详细信息，请参阅 [Azure Redis 会话状态提供程序][]。

<a name="next-steps"></a>
## 后续步骤

现在，你已了解 Azure Redis 缓存的基础知识，请单击下面的链接了解如何执行更复杂的缓存任务。

<!---	[启用缓存诊断](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx#EnableDiagnostics)，以便可以[监视](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx)缓存的运行状况。可以在门户中查看度量值，也可以使用所选的工具[下载和查看](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)这些度量值。-->
-	查看 [StackExchange.Redis 缓存客户端文档][]。
	-	可以从许多 Redis 客户端和开发语言访问 azure Redis 缓存。有关详细信息，请参阅 [http://redis.io/clients][] 和[以其他语言开发 Azure Redis 缓存][]。
	-	Azure Redis 缓存还可用于 Redsmin 等服务。有关详细信息，请参阅[如何检索 Azure Redis 连接字符串并将其用于 Redsmin][]。
-	请参阅 [redis][] 文档并阅读 [redis 数据类型][]和 [Redis 数据类型的十五分钟介绍][]。
-   请参阅有关 [Azure Redis 缓存][]的 MSDN 参考内容。 


<!-- INTRA-TOPIC LINKS -->
[后续步骤]: #next-steps
[What is Azure Redis Cache?]: #what-is
[Create an Azure Cache]: #create-cache
[Which type of caching is right for me?]: #choosing-cache
[Prepare Your Visual Studio Project to Use Azure Caching]: #prepare-vs
[Configure Your Application to Use Caching]: #configure-app
[Get Started with Azure Redis Cache]: #getting-started-cache-service
[创建缓存]: #create-cache
[Configure the cache]: #enable-caching
[配置缓存客户端]: #NuGet
[Working with Caches]: #working-with-caches
[连接到缓存]: #connect-to-cache
[添加和从缓存检索对象]: #add-object
[Specify the expiration of an object in the cache]: #specify-expiration
[在缓存中存储 ASP.NET 会话状态]: #store-session

  
<!-- IMAGES -->
[NewCacheMenu]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-new-cache-menu.png

[CacheCreate]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-cache-create.png

[StackExchangeNuget]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-stackexchange-redis.png

[NuGetMenu]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-nuget-menu.png

[CacheProperties]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-properties.png

[ManageKeys]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-keys.png

[SessionStateNuGet]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-session-state-provider.png

[BrowseCaches]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-browse-caches.png

[Caches]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-caches.png

[CacheCreated]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-cache-created.png




   
<!-- LINKS -->
[Windows Azure 门户]: https://manage.windowsazure.cn
[http://redis.io/clients]: http://redis.io/clients
[以其他语言开发 Azure Redis 缓存]: http://msdn.microsoft.com/zh-cn/library/azure/dn690470.aspx
[如何检索 Azure Redis 连接字符串并将其用于 Redsmin]: https://redsmin.uservoice.com/knowledgebase/articles/485711-how-to-connect-redsmin-to-azure-redis-cache
[Azure Redis 会话状态提供程序]: https://msdn.microsoft.com/zh-cn/library/dn690522.aspx
[How to: Configure a Cache Client Programmatically]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg618003.aspx
[How to: Configure a Cache Client Programmatically]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg618003.aspx
[Session State Provider for Azure Cache]: http://go.microsoft.com/fwlink/?LinkId=320835
[Azure AppFabric Cache: Caching Session State]: http://www.microsoft.com/showcase/details.aspx?uuid=87c833e9-97a9-42b2-8bb1-7601f9b5ca20
[Output Cache Provider for Azure Cache]: http://go.microsoft.com/fwlink/?LinkId=320837
[Azure Shared Caching]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg278356.aspx
[Team Blog]: http://blogs.msdn.com/b/windowsazure/
[Azure Caching]: http://www.microsoft.com/showcase/Search.aspx?phrase=azure+caching
[How to Configure Virtual Machine Sizes]: http://go.microsoft.com/fwlink/?LinkId=164387
[Azure Caching Capacity Planning Considerations]: http://go.microsoft.com/fwlink/?LinkId=320167
[Azure Caching]: http://go.microsoft.com/fwlink/?LinkId=252658
[How to: Set the Cacheability of an ASP.NET Page Declaratively]: http://msdn.microsoft.com/zh-cn/library/zd1ysf1y.aspx
[How to: Set a Page's Cacheability Programmatically]: http://msdn.microsoft.com/zh-cn/library/z852zf6b.aspx
[在 Azure Redis 缓存中配置缓存]: http://msdn.microsoft.com/zh-cn/library/azure/dn793612.aspx

[StackExchange.Redis 配置模型]: http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md

[处理缓存中的 .NET 对象]: http://msdn.microsoft.com/zh-cn/library/dn690521.aspx#Objects


[NuGet Package Manager Installation]: http://go.microsoft.com/fwlink/?LinkId=240311
[缓存定价详细信息]: /home/features/redis-cache/#price
[Management Portal]: https://manage.windowsazure.cn/

[Azure Redis 缓存概述]: https://msdn.microsoft.com/zh-cn/library/azure/dn690523.aspx
[Azure Redis 缓存]: https://msdn.microsoft.com/zh-cn/library/azure/dn690523.aspx

[Migrate to Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=317347
[Azure Redis Cache Samples]: http://go.microsoft.com/fwlink/?LinkId=320840
[使用资源组管理 Azure 资源]: /documentation/articles/resource-group-overview
[StackExchange.Redis]: http://github.com/StackExchange/StackExchange.Redis
[StackExchange.Redis 缓存客户端文档]: http://github.com/StackExchange/StackExchange.Redis#documentation

[Redis]: http://redis.io/documentation
[redis 数据类型]: http://redis.io/topics/data-types
[Redis 数据类型的十五分钟介绍]: http://redis.io/topics/data-types-intro
[应用程序字符串和连接字符串的工作原理]: http://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/

[Azure 试用]: /pricing/1rmb-trial/

<!---HONumber=61-->