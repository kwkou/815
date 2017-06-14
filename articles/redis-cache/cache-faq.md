<properties
    pageTitle="Azure Redis 缓存常见问题 | Azure"
    description="了解有关 Azure Redis 缓存的常见问题解答、模式和最佳实践"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor="" />
<tags
    ms.assetid="c2c52b7d-b2d1-433a-b635-c20180e5cab2"
    ms.service="cache"
    ms.workload="tbd"
    ms.tgt_pltfrm="cache-redis"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/27/2017"
    wacn.date="05/31/2017"
    ms.author="sdanie"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="1183246b6c6ff8b6d60e8b17cef0a7f4b8dd2293"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="azure-redis-cache-faq"></a>Azure Redis 缓存常见问题
了解有关 Azure Redis 缓存的常见问题解答、模式和最佳实践。

## <a name="what-if-my-question-isnt-answered-here"></a>如果未在此处找到相关问题怎么办？
如果未在此处找到相关问题，请联系我们，我们会帮助你找到答案。

* 你可以在此常见问题末尾的评论处发布问题，并与 Azure 内容团队和其他社区成员就本文进行讨论。
* 若希望更多的人看到问题，可以将问题发布在 [Azure Cache MSDN Forum](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=azurecache)（Azure 缓存 MSDN 论坛）并与 Azure 缓存团队和社区的其他成员讨论。

## <a name="azure-redis-cache-basics"></a>Azure Redis 缓存基础知识
本部分中的常见问题介绍了 Azure Redis 缓存的一些基础知识。

* [什么是 Azure Redis 缓存？](#what-is-azure-redis-cache)
* [使用 Azure Redis 缓存？](#how-can-i-get-started-with-azure-redis-cache)

下面的常见问题介绍了有关 Azure Redis 缓存的基本概念和问题，并在另一个常见问题部分列出了相应回答。

* [我应使用哪种 Redis 缓存产品和大小？](#what-redis-cache-offering-and-size-should-i-use)
* [可以使用哪些 Redis 缓存客户端？](#what-redis-cache-clients-can-i-use)
* [Azure Redis 缓存是否有本地模拟器？](#is-there-a-local-emulator-for-azure-redis-cache)
* [如何监视缓存的运行状况和性能？](#how-do-i-monitor-the-health-and-performance-of-my-cache)

## <a name="planning-faqs"></a>有关规划的常见问题
* [我应使用哪种 Redis 缓存产品和大小？](#what-redis-cache-offering-and-size-should-i-use)
* [Azure Redis 缓存性能](#azure-redis-cache-performance)
* [我应该将缓存放在哪个区域？](#in-what-region-should-i-locate-my-cache)
* [Azure Redis 缓存如何计费？](#how-am-i-billed-for-azure-redis-cache)

## <a name="development-faqs"></a>有关开发的常见问题
* [StackExchange.Redis 配置选项有什么作用？](#what-do-the-stackexchangeredis-configuration-options-do)
* [可以使用哪些 Redis 缓存客户端？](#what-redis-cache-clients-can-i-use)
* [Azure Redis 缓存是否有本地模拟器？](#is-there-a-local-emulator-for-azure-redis-cache)
* [如何运行 Redis 命令？](#how-can-i-run-redis-commands)
* [Azure Redis 缓存为何不像某些其他 Azure 服务一样提供 MSDN 类库参考？](#why-doesnt-azure-redis-cache-have-an-msdn-class-library-reference-like-some-of-the-other-azure-services)
* [是否可将 Azure Redis 缓存用作 PHP 会话缓存？](#can-i-use-azure-redis-cache-as-a-php-session-cache)
* [什么是 Redis 数据库？](#what-are-redis-databases)

## <a name="security-faqs"></a>安全常见问题
* [何时应启用非 SSL 端口来连接 Redis？](#when-should-i-enable-the-non-ssl-port-for-connecting-to-redis)

## <a name="production-faqs"></a>有关生产的常见问题
* [生产的一些最佳做法是什么？](#what-are-some-production-best-practices)
* [使用常见 Redis 命令时要注意哪些问题？](#what-are-some-of-the-considerations-when-using-common-redis-commands)
* [如何制定基准和测试缓存的性能？](#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* [有关线程池增长的重要详细信息](#important-details-about-threadpool-growth)
* [启用服务器 GC，以便在使用 StackExchange.Redis 时在客户端上获取更多吞吐量](#enable-server-gc-to-get-more-throughput-on-the-client-when-using-stackexchangeredis)
* [围绕连接的性能注意事项](#performance-considerations-around-connections)

## <a name="monitoring-and-troubleshooting-faqs"></a>有关监视和故障排除的常见问题
本部分中的常见问题包括常见的监视和故障排除问题。 有关 Azure Redis 缓存实例监视和故障排除的详细信息，请参阅[如何监视 Azure Redis 缓存](/documentation/articles/cache-how-to-monitor/)以及[如何排查 Azure Redis 缓存问题](/documentation/articles/cache-how-to-troubleshoot/)。

* [如何监视缓存的运行状况和性能？](#how-do-i-monitor-the-health-and-performance-of-my-cache)
* [缓存诊断存储帐户的设置为何会更改？](#my-cache-diagnostics-storage-account-settings-changed-what-happened)
* [为何有些新缓存启用了诊断，但其他一些缓存却未启用诊断？](#why-are-diagnostics-enabled-for-some-new-caches-but-not-others)
* [为何会出现超时？](#why-am-i-seeing-timeouts)
* [客户端为何与缓存断开连接？](#why-was-my-client-disconnected-from-the-cache)

## <a name="prior-cache-offering-faqs"></a>有关之前缓存产品的常见问题
* [哪种 Azure 缓存产品适合我？](#which-azure-cache-offering-is-right-for-me)

### <a name="what-is-azure-redis-cache"></a>什么是 Azure Redis 缓存？
Azure Redis 缓存以常用的开放源 [Redis 缓存](http://redis.io)为基础。 这使你可以访问安全、专用的 Redis 缓存，该缓存由 Azure.cn 管理并可从 Azure 内的任何应用程序进行访问。 有关更详细的概述，请参阅 Azure.cn 上的 [Azure Redis 缓存](/home/features/redis-cache/)产品页。

### <a name="how-can-i-get-started-with-azure-redis-cache"></a>如何开始使用 Azure Redis 缓存？
可通过多种方法来开始使用 Azure Redis 缓存。

* 用户可以查看适用于 [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)、[ASP.NET](/documentation/articles/cache-web-app-howto/)、[Java](/documentation/articles/cache-java-get-started/)、[Node.js](/documentation/articles/cache-nodejs-get-started/) 和 [Python](/documentation/articles/cache-python-get-started/) 的教程之一。
* 可以查看与你项目的开发语言对应的面向客户的客户端文档，了解如何使用 Redis。 许多 Redis 客户端都可用于 Azure Redis 缓存。 有关 Redis 客户端的列表，请参阅 [http://redis.io/clients](http://redis.io/clients)。

如果还没有 Azure 帐户，可以：

* [建立 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=redis_cache_hero)。 获取可用来尝试付费版 Azure 服务的信用额度。 即使在信用额度用完之后，你也可以保留帐户并使用免费的 Azure 服务和功能。

### <a name="cache-size" id="what-redis-cache-offering-and-size-should-i-use"></a> 我应使用哪种 Redis 缓存产品和大小？
每个 Azure Redis 缓存产品/服务提供不同级别的**大小**、**带宽**、**高可用性**和 **SLA** 选项。

以下是有关选择缓存产品的注意事项。

* **内存**：基本级别和标准级别提供 250 MB - 53 GB。 高级层最多提供 530 GB。 有关详细信息，请参阅 [Azure Redis 缓存定价](/pricing/details/redis-cache/)。
* **网络性能**：如果工作负荷需要较高的吞吐量，则可使用高级层，该层可提供比标准层或基本层更高的带宽。 另外，在每个层中，缓存大小越大，带宽越高，因为是由基础 VM 托管缓存。 有关详细信息，请参阅 [下表](#cache-performance) 。
* **吞吐量**：高级级别提供的可用吞吐量最大。 如果缓存服务器或客户端达到带宽限制，客户端可能会出现超时。 有关详细信息，请参阅下表。
* **高可用性/SLA**：Azure Redis 缓存保证标准/高级缓存在至少 99.9% 的时间内都可用。 若要了解有关 SLA 的详细信息，请参阅 [Azure Redis 缓存定价](/support/sla/redis-cache/)。 SLA 仅涉及与缓存终结点的连接。 SLA 不涉及对数据丢失的防护。 我们建议使用高级层中的 Redis 数据暂留功能来增加灵活性，防止数据丢失。
* **Redis 数据持久性**：高级级别允许你将缓存数据暂留在 Azure 存储帐户中。 在基本/标准缓存中，所有数据只存储在内存中。 如果底层基础结构出现问题，可能会导致数据丢失。 我们建议使用高级层中的 Redis 数据暂留功能来增加灵活性，防止数据丢失。 Azure Redis 缓存提供可在 Redis 暂留中使用的 RDB 和 AOF（即将推出）选项。 有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置持久性](/documentation/articles/cache-how-to-premium-persistence/)。
* **Redis 缓存**：若要创建大于 53 GB 的缓存，或要将数据通过分片的方式分散到多个 Redis 节点中，可以使用在高级层中提供的 Redis 群集功能。 每个节点都包含一个主/副缓存对，目的是提高可用性。 有关详细信息，请参阅 [如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)。
* **增强的安全性和独立性**：Azure 虚拟网络 (VNET) 部署为 Azure Redis 缓存提供增强的安全性和隔离性，并提供子网、访问控制策略和进一步限制访问的其他功能。 有关详细信息，请参阅 [如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)。
* **配置 Redis**：在标准级别和高级级别，你都可以针对 Keyspace 通知来配置 Redis。
* **客户端连接的最大数量**：高级级别提供的可以连接到 Redis 的客户端数量是最大的，缓存大小越大，连接数量越大。 有关详细信息，请参阅 [Azure Redis 缓存定价](/pricing/details/redis-cache/)。
* **专用 Redis 服务器核心**：高级层的所有缓存大小都有针对 Redis 的专用核心。 在基本层/标准层中，C1 以上（含）大小均有针对 Redis 服务器的专用核心。
* **Redis 是单线程的** ，因此与仅使用两个内核相比，使用两个以上的内核并没有额外的优势，但大型 VM 通常提供比小型 VM 更高的带宽。 如果缓存服务器或客户端达到带宽限制，客户端会出现超时。
* **性能改进**：相较于基本层或标准层，高级层中的缓存部署在处理器速度更快且性能更高的硬件上。 高级层缓存的吞吐量更高，延迟更低。

### <a name="cache-performance" id="azure-redis-cache-performance"></a> Azure Redis 缓存性能
下表显示了在 Iaas VM 中使用 `redis-benchmark.exe` 针对 Azure Redis 缓存终结点测试各种大小的标准缓存和高级缓存时，所观测到的最大带宽值。 

>[AZURE.NOTE] 
>这些值是没有保证的，并且我们不针对这些数字提供 SLA，但它们反映了典型的情况。 你应该对自己的应用程序进行负载测试，以确定适合应用程序的缓存大小。
>
>

我们可以从此表得出以下结论：

* 在缓存大小相同的情况下，高级层中的缓存吞吐量要高于标准层中的缓存吞吐量。 例如，以 6 GB 缓存为例，P1 的吞吐量为 140K RPS，而 C3 的吞吐量为 49K。
* 启用 Redis 群集功能时，增加群集中分片（节点）的数量会导致吞吐量线性提高。 如果创建了一个包含 10 个分片的 P4 群集，则可用吞吐量为 250K*10 = 每秒 250 万个请求。
* 如果增加密钥大小，则高级层的吞吐量要高于标准层。

| 定价层 | 大小 | CPU 核心数 | 可用带宽 | 1 KB 值大小 |
| --- | --- | --- | --- | --- |
| **标准缓存大小** | | |**兆位/秒（Mb/秒）/兆字节/秒（MB/秒）** |**请求数/秒 (RPS)** |
| C0 |250 MB |共享 |5/0.625 |600 |
| C1 |1 GB |1 |100/12.5 |12200 |
| C2 |2.5 GB |2 |200/25 |24000 |
| C3 |6 GB |4 |400/50 |49000 |
| C4 |13 GB |2 |500/62.5 |61000 |
| C5 |26 GB |4 |1000/125 |115000 |
| C6 |53 GB |8 |2000/250 |150000 |
| **高级缓存大小** | |**每个分片的 CPU 核心数** | |**每分片每秒请求数 (RPS)** |
| P1 |6 GB |2 |1000/125 |140000 |
| P2 |13 GB |4 |2000/250 |220000 |
| P3 |26 GB |4 |2000/250 |220000 |
| P4 |53 GB |8 |4000/500 |250000 |

有关下载 Redis 工具（例如 `redis-benchmark.exe`）的说明，请参阅 [如何运行 Redis 命令？](#cache-commands) 部分。

### <a name="cache-region" id="in-what-region-should-i-locate-my-cache"></a> 我应该将缓存放在哪个区域？
为了获得最佳性能并最大程度地降低延迟，请在缓存客户端应用程序所在的区域放置 Azure Redis 缓存。

### <a name="cache-billing" id="how-am-i-billed-for-azure-redis-cache"></a> Azure Redis 缓存如何计费？
[此处](/pricing/details/redis-cache/)提供了 Azure Redis 缓存定价。 定价页列出了每小时费率。 缓存按分钟计费，从创建缓存时开始，到删除缓存时为止。 没有提供用于停止或暂停缓存的计费选项。

### <a name="cache-configuration" id="what-do-the-stackexchangeredis-configuration-options-do"></a> StackExchange.Redis 配置选项有什么作用？
StackExchange.Redis 有很多选项。 本部分将介绍一些常用设置。 有关 StackExchange.Redis 选项的详细详细，请参阅 [StackExchange.Redis 配置](https://stackexchange.github.io/StackExchange.Redis/Configuration)。

| 配置选项 | 说明 | 建议 |
| --- | --- | --- |
| AbortOnConnectFail |如果设置为 true，则发生网络故障后不会重新建立连接。 |设置为 false，让 StackExchange.Redis 自动重新连接。 |
| ConnectRetry |初始连接期间重试连接的次数。 |请参阅下面的注释寻求指导。 |
| ConnectTimeout |连接操作的超时，以毫秒为单位。 |请参阅下面的注释寻求指导。 |

通常情况下，使用客户端的默认值便已足够。 你可以根据工作负荷微调选项。

* **重试**
    * 对于 ConnectRetry 和 ConnectTimeout，一般指导原则是快速失败并重试。 该指导原则取决于工作负荷，以及客户端发出 Redis 命令和接收响应平均花费的时间。
    * 让 StackExchange.Redis 自动重新连接，而不是检查连接状态，然后由你自己重新连接。 **避免使用 ConnectionMultiplexer.IsConnected 属性**。
    * 雪球效应 - 有时可能会遇到这样的问题：不断地重试，但问题不断累积，导致永远无法恢复。
* **超时值**
    * 根据工作负荷相应地设置值。 如果要存储较大值，应将超时设置为较大值。
    * 将 `AbortOnConnectFail` 设置为 false，让 StackExchange.Redis 为你重新连接。
    * 使用应用程序的单个 ConnectionMultiplexer 实例。 可以使用 LazyConnection 创建 Connection 属性返回的单个实例，如[使用 ConnectionMultiplexer 类连接到缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#connect-to-the-cache)中所示。
    * 将 `ConnectionMultiplexer.ClientName` 属性设置为应用程序实例的唯一名称以进行诊断。
    * 对自定义工作负载使用多个 `ConnectionMultiplexer` 实例。
        * 如果应用程序中的负载不同，你可以遵循此模型。 例如：
        * 可以使用一个多路复用器来处理大键。
        * 可以使用一个多路复用器来处理小键。
        * 可为连接超时设置不同的值，并为使用的每个 ConnectionMultiplexer 设置重试逻辑。
        * 在每个多路复用器上设置 `ClientName` 属性以帮助进行诊断。
        * 该指导原则可以更好地改进每个 `ConnectionMultiplexer`的延迟。

### <a name="what-redis-cache-clients-can-i-use"></a>可以使用哪些 Redis 缓存客户端？
Redis 的一大优势是有许多客户端，支持许多不同的开发语言。 如需最新的客户端列表，请参阅 [Redis 客户端](http://redis.io/clients)。 若需涵盖多种不同语言和客户端的教程，请参阅[如何使用 Azure Redis 缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)，然后单击文章顶部语言切换器中的所需语言。

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

### <a name="cache-emulator" id="is-there-a-local-emulator-for-azure-redis-cache"></a> Azure Redis 缓存是否有本地模拟器？
Azure Redis 缓存没有本地模拟器，但可以在本地计算机上从 [Redis 命令行工具](https://github.com/MSOpenTech/redis/releases/) 运行 MSOpenTech 版本的 redis-server.exe 并连接到它，以获得与本地缓存模拟器相似的体验，如以下示例所示：

    private static Lazy<ConnectionMultiplexer>
          lazyConnection = new Lazy<ConnectionMultiplexer>
        (() =>
        {
            // Connect to a locally running instance of Redis to simulate a local cache emulator experience.
            return ConnectionMultiplexer.Connect("127.0.0.1:6379");
        });

        public static ConnectionMultiplexer Connection
        {
            get
            {
                return lazyConnection.Value;
            }
        }

如果需要，可以选择配置 [redis.conf](http://redis.io/topics/config) 文件，以更好地匹配联机 Azure Redis 缓存的[默认缓存设置](/documentation/articles/cache-configure/#default-redis-server-configuration)。

### <a name="cache-commands" id="how-can-i-run-redis-commands"></a> 如何运行 Redis 命令？
可以使用 [Redis 命令](http://redis.io/commands#)中列出的任何命令，但 [Azure Redis 缓存中不支持的 Redis 命令](/documentation/articles/cache-configure/#redis-commands-not-supported-in-azure-redis-cache)所列的命令除外。 可以使用多个选项来运行 Redis 命令。

* 如果采用标准或高级缓存，可以使用 [Redis 控制台](/documentation/articles/cache-configure/#redis-console)运行 Redis 命令。 可以通过 Redis 控制台在 Azure 门户中安全地运行 Redis 命令。
* 还可以使用 Redis 命令行工具。 若要使用这些工具，请执行以下步骤：
* 下载 [Redis 命令行工具](https://github.com/MSOpenTech/redis/releases/)。
* 使用 `redis-cli.exe`连接到缓存。 使用 -h 开关传入缓存终结点，使用 -a 传入密钥，如以下示例中所示：
* `redis-cli -h <your cache="" name="">
  .redis.cache.chinacloudapi.cn -a <key>
  `

> [AZURE.NOTE]
> Redis 命令行工具对 SSL 端口不起作用，但是，可以[根据适用于 Redis 预览版的 ASP.NET 会话状态提供程序通告](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的说明，使用 `stunnel` 等实用程序安全地将这些工具连接到 SSL。
>
>

### <a name="cache-reference" id="why-doesnt-azure-redis-cache-have-an-msdn-class-library-reference-like-some-of-the-other-azure-services"></a> Azure Redis 缓存为何不像某些其他 Azure 服务一样提供 MSDN 类库参考？
Azure Redis 缓存基于流行的开源 Redis 缓存，可以通过各种 [Redis 客户端](http://redis.io/clients) 进行访问，这些客户端适用于许多编程语言。 每个客户端有自身的 API，用于通过 [Redis 命令](http://redis.io/commands)调用 Redis 缓存实例。

由于客户端各不相同，因此 MSDN 上未提供统一的类引用，每个客户端都有自己的参考文档。 除了参考文档以外，还可以参阅多个教程，这些教程介绍了如何通过不同的语言和缓存客户端来开始使用 Azure Redis 缓存。 若要访问这些教程，请参阅[如何使用 Azure Redis 缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)，然后单击文章顶部语言切换器中的所需语言。

### <a name="can-i-use-azure-redis-cache-as-a-php-session-cache"></a>是否可将 Azure Redis 缓存用作 PHP 会话缓存？
可以。若要使用 Azure Redis 缓存作为 PHP 会话缓存，请在 `session.save_path` 中指定 Azure Redis 缓存实例的连接字符串。

> [AZURE.IMPORTANT]
> 使用 Azure Redis 缓存作为 PHP 会话缓存时，必须对用于连接到缓存的安全密钥进行 URL 编码，如以下示例所示：
> <p>
> `session.save_path = "tcp://mycache.redis.cache.chinacloudapi.cn:6379?auth=<url encoded primary or secondary key here>";`
> <p>
> 如果未对密钥进行 URL 编码，可能会收到类似于以下消息的异常： `Failed to parse session.save_path`
>
>

有关在 PhpRedis 客户端中使用 Redis 缓存作为 PHP 会话缓存的详细信息，请参阅 [PHP Session handler](https://github.com/phpredis/phpredis#php-session-handler)（PHP 会话处理程序）。

### <a name="what-are-redis-databases"></a>什么是 Redis 数据库？

Redis 数据库就是同一 Redis 实例中的数据的逻辑隔离。 缓存内存在所有数据库之间共享，给定数据库的实际内存消耗取决于该数据库中存储的键/值。 例如，C6 缓存具有 53 GB 的内存。 可以选择将全部 53 GB 放置在一个数据库中，也可以将其分配到多个数据库中。 

> [AZURE.NOTE]
> 使用启用了群集功能的高级 Azure Redis 缓存时，仅数据库 0 可用。 此限制是固有的 Redis 限制，并不特定于 Azure Redis 缓存。 有关详细信息，请参阅[使用群集功能时，是否需要对客户端应用程序进行更改？](/documentation/articles/cache-how-to-premium-clustering/#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
> 
> 

### <a name="cache-ssl" id="when-should-i-enable-the-non-ssl-port-for-connecting-to-redis"></a> 何时应启用非 SSL 端口来连接 Redis？
Redis 服务器本身不支持 SSL，但 Azure Redis 缓存可以。 如果你要连接到 Azure Redis 缓存并且客户端支持 SSL（如 StackExchange.Redis），则你应使用 SSL。

>[AZURE.NOTE]
>默认情况下，为新的 Azure Redis 缓存实例禁用了非 SSL 端口。 如果客户端不支持 SSL，则必须根据[在 Azure Redis 缓存中配置缓存](/documentation/articles/cache-configure/)一文中的[访问端口](/documentation/articles/cache-configure/#access-ports)部分中的说明启用非 SSL 端口。
>
>

`redis-cli` 等 Redis 工具对 SSL 端口不起作用，但是，可以[根据适用于 Redis 预览版的 ASP.NET 会话状态提供程序通告](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的说明，使用 `stunnel` 等实用程序安全地将这些工具连接到 SSL。

有关下载 Redis 工具的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。

### <a name="what-are-some-production-best-practices"></a>生产的一些最佳做法是什么？
* [StackExchange.Redis 最佳做法](#stackexchangeredis-best-practices)
* [配置和概念](#configuration-and-concepts)
* [性能测试](#performance-testing)

#### <a name="stackexchangeredis-best-practices"></a>StackExchange.Redis 最佳做法
* 将 `AbortConnect` 设置为 false，然后使 ConnectionMultiplexer 自动重新连接。 [请参阅此处了解详细信息](https://gist.github.com/JonCole/36ba6f60c274e89014dd#file-se-redis-setabortconnecttofalse-md)。
* 重复使用 ConnectionMultiplexer - 不要为每个请求创建一个新的 ConnectionMultiplexe。 建议使用[此处所示](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#connect-to-the-cache)的 `Lazy<ConnectionMultiplexer>` 模式。
* 具有较小值的 Redis 工作性能最佳，因此请考虑将较大数据分成多个密钥。 阅读[本文](https://gist.github.com/JonCole/db0e90bedeb3fc4823c2#large-requestresponse-size)了解较大值可能引起的问题示例。
* 配置 [ThreadPool 设置](#important-details-about-threadpool-growth) ，以免超时。
* 将默认 connectTimeout 至少设置为 5 秒。 出现网络故障时，此时间间隔会给 StackExchange.Redis 足够的时间来重新建立连接。
* 注意与正在运行的不同操作相关的性能成本。 例如，`KEYS` 命令是 O(n) 操作，应当避免。 [redis.io](http://redis.io/commands/) 站点具有关于其支持的每个操作的时间复杂性的详细信息。 单击每个命令以查看每个操作的复杂程度。

#### <a name="configuration-and-concepts"></a>配置和概念
* 针对生产系统使用标准层或高级层。 基本层是没有数据复制和 SLA 的单节点系统。 此外，使用至少一个 C1 缓存。 C0 缓存通常用于简单的开发/测试方案。
* 请记住，Redis 是 **内存中** 数据存储区。 阅读[本文](https://gist.github.com/JonCole/b6354d92a2d51c141490f10142884ea4#file-whathappenedtomydatainredis-md)，了解可能出现数据丢失的情况。
* 开发您的系统，以便它可以处理 [由于修补和故障转移](https://gist.github.com/JonCole/317fe03805d5802e31cfa37e646e419d#file-azureredis-patchingexplained-md)出现的连接故障。

#### <a name="performance-testing"></a>性能测试
* 使用 `redis-benchmark.exe` 启动以在编写您自己的性能测试前感受可能的吞吐量。 由于 `redis-benchmark` 不支持 SSL，因此在运行测试之前必须[通过 Azure 门户启用非 SSL 端口](/documentation/articles/cache-configure/#access-ports)。 例如，请参阅 [如何制定基准和测试缓存的性能？](#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* 用于测试的客户端 VM 应与 Redis 缓存实例位于同一区域。
* 建议为客户端使用 Dv2 VM 系列，因为该系列具有更好的硬件，会提供最佳的结果。
* 确保选择的客户端 VM 至少与正在测试的缓存拥有相同的计算和带宽容量。
* 如果是在 Windows 设备上操作，请在客户端计算机上启用 VRSS。 [请参阅此处了解详细信息](https://technet.microsoft.com/zh-cn/library/dn383582.aspx)。
* 高级层 Redis 实例具有更好的网络延迟和吞吐量，因为它们是在 CPU 和网络两方面都更好的硬件上运行的。

### <a name="cache-redis-commands" id="what-are-some-of-the-considerations-when-using-common-redis-commands"></a> 使用常见 Redis 命令时要注意哪些问题？
* 对于某些需要较长时间才能完成的 Redis 命令，在未了解这些命令造成的影响的情况下，不应运行这些命令。
    * 例如，不要在生产环境中运行 [KEYS](http://redis.io/commands/keys) 命令，因为它可能需要很长时间才能返回，具体时间取决于键数。 Redis 是单线程服务器，每次只能处理一个命令。 如果在 KEYS 后面发出了其他命令，则这些命令只会在处理完 KEYS 命令后才会得到处理。 [redis.io 站点](http://redis.io/commands/) 具有关于其支持的每个操作的时间复杂性的详细信息。 单击每个命令以查看每个操作的复杂程度。
* 键大小 - 应使用小键/值还是大键/值？ 通常这取决于方案。 如果方案需要较大的键，则可调整 ConnectionTimeout 和重试值，并调整重试逻辑。 从 Redis 服务器的角度来看，值越小，性能就越好。
* 这些考量并不意味着不能在 Redis 中存储较大值，只是要注意以下事项。 延迟将会提高。 如果你有一个较大的数据集和一个较小的数据集，则可以使用多个 ConnectionMultiplexer 实例，并根据 [StackExchange.Redis 配置选项有什么作用](#cache-configuration) 部分中所述，为每个实例配置一组不同的超时和重试值。

### <a name="cache-benchmarking" id="how-can-i-benchmark-and-test-the-performance-of-my-cache"></a> 如何制定基准和测试缓存的性能？
* [启用缓存诊断](/documentation/articles/cache-how-to-monitor/#enable-cache-diagnostics)，以便可以[监视](/documentation/articles/cache-how-to-monitor/)缓存的运行状况。 可以在 Azure 门户中查看指标，也可以使用所选的工具 [下载和查看](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring) 这些指标。
* 可以使用 redis-benchmark.exe 对 Redis 服务器进行负载测试。
* 确保负载测试客户端和 Redis 缓存位于同一区域。
* 使用 redis-cli.exe，并使用 INFO 命令监视缓存。
* 如果你的负载导致出现大量内存碎片，则你应该扩展为更大的缓存大小。
* 有关下载 Redis 工具的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。

下面的命令提供了有关如何使用 redis-benchmark.exe 的示例。 为获得准确的结果，请从与缓存位于同一区域的 VM 运行这些命令。

* 使用 1 k 有效负载测试管道 SET 请求

    `redis-benchmark.exe -h **yourcache**.redis.cache.chinacloudapi.cn -a **yourAccesskey** -t SET -n 1000000 -d 1024 -P 50`
* 使用 1 k 有效负载测试管道 GET 请求。
  注意：首先运行上面显示的 SET 测试以填充缓存

    `redis-benchmark.exe -h **yourcache**.redis.cache.chinacloudapi.cn -a **yourAccesskey** -t GET -n 1000000 -d 1024 -P 50`

### <a name="threadpool" id="important-details-about-threadpool-growth"></a> 有关线程池增长的重要详细信息
CLR 线程池具有两种类型的线程 —“辅助角色”和“I/O 完成端口”（又称为 IOCP）线程。

* 对于诸如处理 `Task.Run(…)` 或 `ThreadPool.QueueUserWorkItem(…)` 方法这类事务，使用辅助角色线程。 需要在后台线程上进行工作时，CLR 中的各种组件也会使用这些线程。
* 进行异步 IO（例如从网络进行读取）时，使用 IOCP 线程。

线程池按需提供新的辅助角色线程或 I/O 完成线程（没有任何限制），直到它达到每种线程类型的“最小值”设置。 默认情况下，最小线程数设置为系统上的处理器数。

一旦现有（忙碌）线程数达到“最小”线程数，ThreadPool 便会将插入新线程的速率限制为每 500 毫秒一个线程。 通常情况下，如果系统中出现需要 IOCP 线程的突发工作，则它会非常快速地处理该工作。 但是，如果突发工作多于配置的“最小值”设置，则在处理某些工作时会出现一定的延迟，因为 ThreadPool 会等待发生以下两种情况之一。

1. 一个现有线程释放，以便处理工作。
2. 在 500 毫秒内没有任何现有线程释放，因此会创建一个新线程。

基本上，这意味着忙碌线程数大于最小线程数，在应用程序处理网络流量之前可能需要付出 500 毫秒延迟。 此外请务必注意，当现有线程保持空闲状态的时间超过 15 秒（基于我记得的内容）时，会清理它，并且这种增长和收缩的循环可能会重复。

如果我们考虑一个来自 StackExchange.Redis（内部版本 1.0.450 或更高版本）的示例错误消息，会看到它现在会打印 ThreadPool 统计信息（请参阅下面的 IOCP 和辅助角色详细信息）。

    System.TimeoutException: Timeout performing GET MyKey, inst: 2, mgr: Inactive,
    queue: 6, qu: 0, qs: 6, qc: 0, wr: 0, wq: 0, in: 0, ar: 0,
    IOCP: (Busy=6,Free=994,Min=4,Max=1000),
    WORKER: (Busy=3,Free=997,Min=4,Max=1000)

在上面的示例中，可以看到对于 IOCP 线程有 6 个忙碌线程，而系统配置为允许最少 4 个线程。 在这种情况下，客户端可能会遇到两个 500 毫秒延迟，因为 6 > 4。

请注意，如果 IOCP 或辅助角色线程受到限制，则 StackExchange.Redis 可以会超时。

### <a name="recommendation"></a>建议
考虑到此信息，我们强烈建议客户将 IOCP 和辅助角色线程的最小配置值设置为大于默认值。 我们无法提供有关此值应是多少的通用指导，因为一个应用程序的合适值对于另一个应用程序会太高/低。 此设置还可能会影响复杂应用程序其他部分的性能，因此每个客户需要按照其特定需求来微调此设置。 开始时设置为 200 或 300 会比较好，随后可进行测试并根据需要进行调整。

如何配置此设置：

* 在 ASP.NET 中，可在 web.config 中的 `<processModel>` 配置元素下使用“minIoThreads”配置设置。 如果在 Azure 网站内部运行，则此设置不会通过配置选项进行公开。 但是，你应该仍然能够通过 global.asax.cs 中的 Application_Start 方法以编程方式配置此设置置（请参阅下文）。

    > [AZURE.NOTE] 
    > 此配置元素中指定的值是*按核心*设置。 例如，如果使用 4 核计算机，并且希望 minIOThreads 设置在运行时为 200，则使用 `<processModel minIoThreads="50"/>`。
    >

* 在 ASP.NET 外部，可使用 [ThreadPool.SetMinThreads(…)](https://msdn.microsoft.com/zh-cn/library/system.threading.threadpool.setminthreads.aspx) API。

### <a name="server-gc" id="enable-server-gc-to-get-more-throughput-on-the-client-when-using-stackexchangeredis"></a> 启用服务器 GC，以便在使用 StackExchange.Redis 时在客户端上获取更多吞吐量
启用服务器 GC 可以在使用 StackExchange.Redis 时优化客户端并提供更好的性能和吞吐量。 有关服务器 GC 以及如何启用它的详细信息，请参阅以下文章：

* [若要启用服务器 GC](https://msdn.microsoft.com/zh-cn/library/ms229357.aspx)
* [垃圾回收基础](https://msdn.microsoft.com/zh-cn/library/ee787088.aspx)
* [垃圾回收和性能](https://msdn.microsoft.com/zh-cn/library/ee851764.aspx)

### <a name="performance-considerations-around-connections"></a>围绕连接的性能注意事项

每个定价层都有不同的客户端连接、内存和带宽的限制。 虽然每个缓存大小*最多*允许一定数量的连接，但与 Redis 的每个连接都具有其关联的开销。 此类开销的一个示例是，由于 TLS/SSL 加密而导致的 CPU 和内存使用。 给定缓存大小的最大连接限制假定轻负载缓存。 如果连接开销的负载*和*客户端操作的负载超出了系统容量，那么即使未超出当前缓存大小的连接限制，缓存也可能会遇到容量问题。

有关每个层的不同连接限制的详细信息，请参阅 [Azure Redis 缓存定价](/pricing/details/redis-cache/)。 有关连接和其他默认配置的详细信息，请参阅[默认 Redis 服务器配置](/documentation/articles/cache-configure/#default-redis-server-configuration)。

### <a name="cache-monitor" id="how-do-i-monitor-the-health-and-performance-of-my-cache"></a> 如何监视缓存的运行状况和性能？
可以在 [Azure 门户](https://portal.azure.cn)中监视 Azure Redis 缓存实例。 可以查看度量值、将度量值图表固定到启动板、自定义监视图表的日期和时间范围、在图表中添加和删除度量值，以及设置符合特定条件时发出的警报。 有关详细信息，请参阅[监视 Azure Redis 缓存](/documentation/articles/cache-how-to-monitor/)。

Redis 缓存**资源菜单**中还包含了用于对缓存进行监视和故障排除的几个工具。

* “诊断并解决问题”提供常见问题的相关信息，以及解决问题的策略。
* “资源运行状况”会监视你的资源，并告知资源是否按预期运行。
* “新建支持请求”提供用于建立缓存支持请求的选项。

借助这些工具，可以监视 Azure Redis 缓存实例的运行状况，以及管理缓存应用程序。 有关详细信息，请参阅[如何配置 Azure Redis 缓存](/documentation/articles/cache-configure/)的“支持和故障排除设置”部分。

### <a name="my-cache-diagnostics-storage-account-settings-changed-what-happened"></a>缓存诊断存储帐户的设置为何会更改？
同一区域和订阅中的缓存共享诊断存储设置，当配置更改（启用/禁用诊断或更改存储帐户）时，所做的更改将应用于订阅中位于该区域的所有缓存。 如果缓存的诊断设置已更改，请检查同一订阅和区域中其他缓存的诊断设置是否也已更改。 检查方法之一是查看 `Write DiagnosticSettings` 事件的缓存审核日志。 有关使用审核日志的详细信息，请参阅[查看事件和审核日志](/documentation/articles/insights-debugging-with-events/)以及[使用 Resource Manager 执行审核操作](/documentation/articles/resource-group-audit/)。 有关监视 Azure Redis 缓存事件的详细信息，请参阅[操作和警报](/documentation/articles/cache-how-to-monitor/#operations-and-alerts)。

### <a name="why-are-diagnostics-enabled-for-some-new-caches-but-not-others"></a>为何有些新缓存启用了诊断，但其他一些缓存却未启用诊断？
在同一区域和订阅中，缓存共享相同的诊断存储设置。 如果有其他缓存已启用诊断，并在与该缓存相同的区域和订阅中创建新缓存，将在新缓存中使用相同的设置来启用诊断。

### <a name="cache-timeouts" id="why-am-i-seeing-timeouts"></a> 为何会出现超时？
超时发生在用来与 Redis 通信的客户端中。 将某个命令发送到 Redis 服务器后，该命令将会排队，Redis 服务器最终会提取该命令并执行它。 但是，客户端在此过程中可能会超时，在这种情况下，会在调用端引发异常。 有关排查超时问题的详细信息，请参阅[客户端故障排除](/documentation/articles/cache-how-to-troubleshoot/#client-side-troubleshooting)和 [StackExchange.Redis 超时异常](/documentation/articles/cache-how-to-troubleshoot/#stackexchangeredis-timeout-exceptions)。

### <a name="cache-disconnect" id="why-was-my-client-disconnected-from-the-cache"></a> 客户端为何与缓存断开连接？
下面是缓存断开连接的一些常见原因。

* 客户端的原因
    * 已重新部署客户端应用程序。
    * 客户端应用程序执行了缩放操作。
        * 对于云服务或 Web 应用，原因可能在于自动缩放。
    * 客户端上的网络层已更改。
    * 客户端中或客户端与服务器之间的网络节点中发生暂时性错误。
    * 已达到带宽阈值限制。
    * 占用大量 CPU 的操作花费了太长时间才完成。
* 服务器端的原因
    * 在标准缓存产品上，Azure Redis 缓存服务启动了从主节点到辅助节点的故障转移。
    * Azure 正在修补已部署缓存的实例
        * 原因可能是 Redis 服务器更新或常规 VM 维护。

### <a name="which-azure-cache-offering-is-right-for-me"></a>哪种 Azure 缓存产品适合我？
> [AZURE.IMPORTANT]
> 按照去年的 [公告](https://azure.microsoft.com/zh-cn/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)，已于 2016 年 11 月 30 日 **停用** Azure 托管缓存服务和 Azure 角色中缓存服务。 我们建议使用 [Azure Redis 缓存](/home/features/redis-cache/)。
>
>

### <a name="azure-redis-cache"></a>Azure Redis 缓存
Azure Redis 缓存已正式发布，最大大小为 53 GB，且其可用性 SLA 为 99.9%。 全新[高级级别](/documentation/articles/cache-premium-tier-intro/)提供的最大大小为 530 GB，且支持群集、VNET 和持久性，并附带 99.9% SLA。

Azure Redis 缓存使客户能够使用 Azure.cn 管理的安全专用 Redis 缓存。 有了此产品，用户就可以利用 Redis 提供的丰富功能集和生态系统，并可以从 Azure.cn 获得可靠的托管和监控。

与仅处理键/值对的传统缓存不同，Redis 因其高性能的数据类型而受欢迎。 Redis 还支持对这些类型运行原子操作，如在字符串后面追加内容；递增哈希中的值；推送到列表；计算交集、并集和差集，或者获取排序集中排名最高的成员。 其他功能包括支持事务、发布/订阅、Lua 脚本、具有有限生存时间的键和配置设置，使 Redis 在行为上更类似于传统缓存。

Redis 取得成功的另一个重要方面是围绕它构建了健康而充满活力的开源生态系统。 这反映在可通过多种语言使用各种不同的 Redis 客户端。 此生态系统和广泛的客户端使得 Azure Redis 缓存几乎可供你在 Azure 内生成的任何工作负荷使用。

有关如何开始使用 Azure Redis 缓存的详细信息，请参阅[如何使用 Azure Redis 缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)和 [Azure Redis 缓存文档](/documentation/services/redis-cache/)。

### <a name="managed-cache-service"></a>托管缓存服务
[托管缓存服务已于 2016 年 11 月 30 日停用。](https://azure.microsoft.com/zh-cn/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)

### <a name="in-role-cache"></a>角色中缓存
[角色中缓存已于 2016 年 11 月 30 日停用。](https://azure.microsoft.com/zh-cn/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)

<!--Update_Description: add question "performance considerations around connections"-->