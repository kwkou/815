<properties pageTitle="如何使用 Azure Redis Cache" metaKeywords="" description="了解如何在 Azure Redis Cache 中创建使用缓存" metaCanonical="" services="" documentationCenter="API Management" title="如何使用 Azure Redis Cache" authors="sdanie" solutions="" manager="" editor="" />

# 如何使用 Azure Redis Cache

本指南说明如何开始使用
**Azure Redis Cache（预览版）**。相关示例用 C# 代码编写且
使用 .NET API。涉及的情况包括**创建和配置缓存**，**配置缓存客户端**，**从缓存添加和删除对象**，以及**在缓存中存储 ASP.NET 会话状态**。有关
使用 Azure Redis Cache 的详细信息，请参阅[后续步骤][后续步骤]部分。

## 目录

-   [什么是 Azure Redis Cache？][什么是 Azure Redis Cache？]
-   [开始使用 Azure Redis Cache][开始使用 Azure Redis Cache]

    -   [创建缓存][创建缓存]
    -   [配置缓存客户端][配置缓存客户端]
-   [使用缓存][使用缓存]

    -   [连接到缓存][连接到缓存]
    -   [添加和从缓存检索对象][添加和从缓存检索对象]
    -   [在缓存中指定对象的有效期][在缓存中指定对象的有效期]
    -   [在缓存中存储 ASP.NET 会话状态][在缓存中存储 ASP.NET 会话状态]
-   [后续步骤][后续步骤]

<a name="what-is"></a>

## 什么是 Azure Redis Cache？

Microsoft Azure Redis Cache 目前处于预览状态，它基于流行的开放源代码 Redis Cache。它让您访问 Microsoft 管理的安全专用的 Redis 缓存。使用 Azure Redis Cache 创建的缓存可从 Microsoft Azure 内的任何应用程序进行访问。

Microsoft Azure Redis Cache 现已推出两种层级：

-   **基本**– 单个节点。多个大小。
-   **标准**– 两个节点“主/从”。多个大小。

> 在预览期间，Azure Redis Cache 将提供 250 MB 和 1 GB 大小。在有限的时间内将提供免费缓存，每个订阅限制为两个缓存。

每个级别在功能和定价方面存在差异。在本指南的后面将介绍这些功能，而有关定价的详细信息，则请参阅[缓存定价详细信息][缓存定价详细信息]。

本指南提供了 Azure Redis Cache 的入门概述。有关超出本入门指南范围的那些功能的更多详细信息，请参阅 [Azure Redis Cache 概述][Azure Redis Cache 概述]。

<a name="getting-started-cache-service"></a>

## 开始使用 Azure Redis Cache

Azure Redis Cache 非常容易上手。若要开始使用，需要首先设置和配置缓存。接下来，配置缓存客户端，以便它们可以访问缓存。在配置了缓存客户端后，就可以开始使用它们。

-   [创建缓存][创建缓存]
-   [配置缓存][配置缓存]
-   [配置缓存客户端][配置缓存客户端]

<a name="create-cache"></a>

## 创建缓存

要创建缓存，首先登录到 Azure 管理预览门户中，然后单击**新建**、**缓存**。

![新建缓存][新建缓存]

在**创建缓存**边栏选项卡中，指定用于缓存所需的配置。

![创建缓存][1]

在 **Dns 名**中，输入要用于缓存终结点的域名。该终结点必须是长度在 6 到 20 个字符之间的字符串，仅包含小写的数字和字母，并且必须以字母开头。

使用**定价层**选择所需的缓存大小和功能。Redis Cache 提供以下两种层级。

-   **基本** - 单个节点，250 MB 或 1 GB 大小。
-   **标准** - 两个节点主/从，99.9% SLA（开机自检预览），250 MB 或 1 GB 大小。

对于**订阅**，选择要用于缓存的 Azure 订阅。

> 如果你的帐户仅具有一个订阅，将自动选择该订阅并且将不显示“订阅”下拉菜单。

在**资源组**中，为您的缓存选择或创建资源组。

> 有关详细信息，请参阅[使用资源组管理您的 Azure 资源][使用资源组管理您的 Azure 资源]。

使用**地理位置**指定在其中托管您的缓存的地理位置。Microsoft 强烈推荐的最佳做法，是在与缓存客户端应用程序相同的区域中创建缓存。

在配置了新的缓存选项后，单击**创建**。创建缓存可能耗时几分钟。要检查的状态，可以监视开始板上的进度。在创建了缓存后，您的新缓存具有**Running**状态并且可供用于默认设置。

![创建的缓存][创建的缓存]

创建缓存后，可以从**浏览**边栏选项卡访问它。

![浏览边栏选项卡][浏览边栏选项卡]

单击**缓存**查看您的缓存。

![缓存][缓存]

<a name="NuGet"></a>

## 配置缓存客户端

使用 Azure Redis Cache 创建的缓存可从任何 Azure 应用程序进行访问。在 Visual Studio 中开发的 .NET 应用程序可以使用 **StackExchange.Redis** 缓存客户端，可使用 NuGet 程序包进行配置，以简化缓存客户端应用程序的配置。

> 有关详细信息，请参阅 [StackExchange.Redis][StackExchange.Redis] github 页面和 [StackExchange.Redis 缓存客户端文档][StackExchange.Redis 缓存客户端文档]。

要使用 StackExchange.Redis NuGet 包配置客户端应用程序，请在**“解决方案资源管理器”**中右键单击项目，然后选择**“管理 NuGet 包”**。

![管理 NuGet 包][管理 NuGet 包]

将 **StackExchange.Redis** 键入**联机搜索**文本框，从结果选择它，然后单击**安装**。

![StackExchange.Redis NuGet 程序包][StackExchange.Redis NuGet 程序包]

NuGet 程序包会给客户端应用程序下载并添加所需的程序集引用，以访问带 StackExchange.Redis 缓存客户端的 Azure Redis Cache。

在配置了你的客户端项目的缓存后，你可以使用以下各节中介绍的方法来使用你的缓存。

<a name="working-with-caches"></a>

## 使用缓存

本节中的步骤介绍如何使用缓存执行常见任务。

-   [连接到缓存][连接到缓存]
-   [添加和从缓存检索对象][添加和从缓存检索对象]
-   [在缓存中存储 ASP.NET 会话状态][在缓存中存储 ASP.NET 会话状态]

<a name="connect-to-cache"></a>

## 连接到缓存

若要以编程方式使用缓存，你需要引用该缓存。以下代码添加到您想使用 StackExchange.Redis 客户端的任何文件的顶部，以访问 Azure Redis Cache：

    using StackExchange.Redis;

> StackExchange.Redis 客户端需要.NET Framework 4 或更高版本。

到 Azure Redis Cache 连接由 `ConnectionMultiplexer` 类管理。此类旨在共享并在客户端应用程序中重复使用，不需要在每次执行操作的基础上创建。

要连接到 Azure Redis Cache 并返回连接的 `ConnectionMultiplexer` 的实例，请调用静态 `Connect` 方法并传递到缓存端点和密钥中，如下例所示。

    ConnectionMultiplexer connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,ssl=true,password=...");

如果您不想使用 SSL，则设置 `ssl=false` 或只需传递到端点和密钥。

    connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,password=...");

> 有关高级连接配置选项的详细信息，请参阅 [StackExchange.Redis 配置模型][StackExchange.Redis 配置模型]。

可以从 Azure 管理预览门户边栏选项卡为您的缓存实例获得缓存端点和密钥。

![缓存属性][缓存属性]

![管理密钥][管理密钥]

一旦建立连接，通过调用 `ConnectionMultiplexer.GetDatabase` 方法将引用返回 redis cache 数据库。

    // connection referes to a previously configured ConnectionMultiplexer
    IDatabase cache = connection.GetDatabase();

> 从 `GetDatabase` 方法返回的对象是一个轻型直通对象，不需要存储。

现在您知道如何连接到 Azure Redis Cache 实例并将引用返回缓存数据库，让我们看看如何使用缓存。

<a name="add-object"></a>

## 添加和从缓存检索对象

使用 `StringSet` 和 `StringGet` 方法，可以存储并从缓存中检索项目。

    // If key1 exists, it is overwritten.
    cache.StringSet("key1", "value1");

    string value = cache.StringGet("key1");

> Redis 将大多数数据存储为 Redis 字符串，但这些字符串可能包含许多类型的数据，包括序列化的二进制数据，可在缓存中存储 .NET 对象时使用。

调用 `StringGet` 时如果对象存在，则返回它，如果对象不存在，则返回 null。在这种情况可以从所需的数据源检索值，并将其存储在缓存中供后续使用。这称为缓存端模式。

    string value = cache.StringGet("key1");
    if (value == null)
    {
        // The item keyed by "key1" is not in the cache. Obtain
        // it from the desired data source and add it to the cache.
        value = GetValueFromDataSource();

        cache.StringSet("key1", value);
    }

<a name="specify-expiration"></a>

## 在缓存中指定项目的有效期

要在缓存中指定项目的过期时间，请使用 `StringSet` 的 `TimeSpan` 参数。

    cache.StringSet("key1", "value1", TimeSpan.FromMinutes(90));

<a name="store-session"></a>

## 在缓存中存储 ASP.NET 会话状态

Azure Redis Cache 提供一个会话状态提供程序，可用于在缓存而不是内存或 SQL Server 数据库中存储您的会话状态。要使用缓存会话
状态提供程序，首先配置您的缓存，然后使用 Redis Cache 会话状态 NuGet 程序包为缓存配置 ASP.NET 应用程序。

要使用 Redis Cache 缓存会话状态 NuGet 包配置客户端应用程序，请在**“解决方案资源管理器”**中右键单击项目，然后选择**“管理 NuGet 包”**。

![管理 NuGet 包][管理 NuGet 包]

将 **Redis Cache Session State** 键入**联机搜索**文本框，从结果选择它，然后单击**安装**。

![Redis Cache 会话状态 NuGet 程序包][Redis Cache 会话状态 NuGet 程序包]

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
          />
        -->
        <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="127.0.0.1" accessKey="" ssl="false" />
      </providers>
    </sessionState>

注释部分为它们提供属性和示例设置的一个示例。

在预览门户上用来自缓存边栏选项卡的值配置属性，并根据需要配置其他值。

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
          />
        -->
        <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="contoso5.redis.cache.windows.net" 
        accessKey="..." ssl="false" />
      </providers>
    </sessionState>

> 注意重试超时以毫秒为单位。

请务必注释掉标准 **InProc** 会话状态提供程序。

    <!-- <sessionState mode="InProc" customProvider="DefaultSessionProvider">
      <providers>
        <add name="DefaultSessionProvider" type="System.Web.Providers.DefaultSessionStateProvider, System.Web.Providers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" connectionStringName="DefaultConnection" />
      </providers>
    </sessionState> -->

有关配置和使用 Azure Redis 会话状态提供程序的详细信息，请参阅 [Azure Redis 会话状态提供程序][Azure Redis 会话状态提供程序]。

<a name="next-steps"></a>

## 后续步骤

现在，您已了解 Azure Redis Cache 的基础知识，
请单击下面的链接了解如何执行更复杂的缓存任务。

-   了解有关 StackExchange.Redis 客户端的详细信息：[StackExchange.Redis 缓存客户端文档][StackExchange.Redis 缓存客户端文档]
-   请参阅 [redis][redis] 文档并阅读 [redis 数据类型][redis 数据类型]和 [Redis 数据类型的十五分钟介绍][Redis 数据类型的十五分钟介绍]。
-   查看 MSDN 参考：[Azure Redis Cache][Azure Redis Cache]

<!-- INTRA-TOPIC LINKS --> <!-- IMAGES --> <!-- LINKS -->

  [后续步骤]: #next-steps
  [什么是 Azure Redis Cache？]: #what-is
  [开始使用 Azure Redis Cache]: #getting-started-cache-service
  [创建缓存]: #create-cache
  [配置缓存客户端]: #NuGet
  [使用缓存]: #working-with-caches
  [连接到缓存]: #create-cache-object
  [添加和从缓存检索对象]: #add-object
  [在缓存中指定对象的有效期]: #specify-expiration
  [在缓存中存储 ASP.NET 会话状态]: #store-session
  [缓存定价详细信息]: http://www.windowsazure.com/zh-cn/pricing/details/cache/
  [Azure Redis Cache 概述]: http://go.microsoft.com/fwlink/?LinkId=320830
  [配置缓存]: #enable-caching
  [新建缓存]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-new-cache-menu.png
  [1]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-cache-create.png
  [使用资源组管理您的 Azure 资源]: http://azure.microsoft.com/zh-cn/documentation/articles/azure-preview-portal-using-resource-groups/
  [创建的缓存]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-cache-created.png
  [浏览边栏选项卡]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-browse-caches.png
  [缓存]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-caches.png
  [StackExchange.Redis]: http://github.com/StackExchange/
  [StackExchange.Redis 缓存客户端文档]: http://github.com/StackExchange/StackExchange.Redis#documentation
  [管理 NuGet 包]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-nuget-menu.png
  [StackExchange.Redis NuGet 程序包]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-stackexchange-redis.png
  [StackExchange.Redis 配置模型]: htts://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md
  [缓存属性]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-properties.png
  [管理密钥]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-keys.png
  [Redis Cache 会话状态 NuGet 程序包]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-session-state-provider.png
  [Azure Redis 会话状态提供程序]: http://go.microsoft.com/fwlink/?LinkId=398249
  [redis]: http://redis.io/documentation
  [redis 数据类型]: http://redis.io/topics/data-types
  [Redis 数据类型的十五分钟介绍]: http://redis.io/topics/data-types-intro
  [Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=398247
