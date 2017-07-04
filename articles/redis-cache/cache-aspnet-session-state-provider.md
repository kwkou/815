<properties
    pageTitle="缓存 ASP.NET 会话状态提供程序 | Azure"
    description="了解如何使用 Azure Redis 缓存存储 ASP.NET 会话状态"
    services="redis-cache"
    documentationcenter="na"
    author="steved0x"
    manager="douge"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="192f384c-836a-479a-bb65-8c3e6d6522bb"
    ms.service="cache"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="cache-redis"
    ms.workload="tbd"
    ms.date="03/22/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="dc3a54edf5d8ce17df83c2e5fc24e4937c33fb2d"
    ms.lasthandoff="04/22/2017" />

# <a name="aspnet-session-state-provider-for-azure-redis-cache"></a>Azure Redis 缓存的 ASP.NET 会话状态提供程序

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Azure Redis 缓存提供一个会话状态提供程序，可用于在缓存而不是内存或 SQL Server 数据库中存储会话状态。 若要使用缓存会话状态提供程序，首先配置缓存，然后使用 Redis 缓存会话状态 NuGet 包为缓存配置 ASP.NET 应用程序。

在实际云应用中避免存储某种形式的用户会话状态通常是不现实的，但某些方法相比其他方法而言，对性能和可伸缩性的影响更大。 如果你需要存储状态，最佳解决方案是使状态量保持较小并将其存储在 Cookie 中。 如果这不可行，下一个最佳解决方案是将 ASP.NET 会话状态与提供程序配合使用以实现分布式内存中缓存。 从性能和可伸缩性的角度来看，最差的解决方案是使用数据库支持的会话状态提供程序。 本主题提供有关使用 Azure Redis 缓存的 ASP.NET 会话状态提供程序的指南。 有关其他会话状态选项的信息，请参阅 [ASP.NET 会话状态选项](#aspnet-session-state-options)。

## <a name="store-aspnet-session-state-in-the-cache"></a>在缓存中存储 ASP.NET 会话状态
若要使用 Redis 缓存会话状态 NuGet 包在 Visual Studio 中配置客户端应用程序，请在“工具”菜单中依次单击“NuGet 包管理器”和“包管理器控制台”。

从 `Package Manager Console` 窗口运行以下命令。

    Install-Package Microsoft.Web.RedisSessionStateProvider

> [AZURE.IMPORTANT]
> 如果使用高级层的聚类分析功能，则必须使用 [RedisSessionStateProvider 2.0.1](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) 或更高版本，否则会引发异常。 移到 2.0.1 或更高版本是一项重大更改；有关详细信息，请参阅 [v2.0.0 Breaking Change Details](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details)（2.0.0 版重大更改详细信息）。 本文更新时，此包的当前版本是 2.2.3。
> 
> 

Redis 会话状态提供程序 NuGet 包依赖于 StackExchange.Redis.StrongName 包。 如果项目中没有 StackExchange.Redis.StrongName 包，则会安装它。

>[AZURE.NOTE]
>除了强命名的 StackExchange.Redis.StrongName 包外，还有 StackExchange.Redis 非强命名版本。 如果项目使用非强命名的 StackExchange.Redis 版本，则必须将其卸载，否则会在项目中出现命名冲突。 有关这些包的详细信息，请参阅[配置 .NET 缓存客户端](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#configure-the-cache-clients)。
>
>

NuGet 包会下载并添加所需的程序集引用，并将以下节添加到 web.config 文件中。 此节包含的配置是 ASP.NET 应用程序使用 Redis 缓存会话状态提供程序所必需的。

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
        <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="127.0.0.1" accessKey="" ssl="false"/>
        </providers>
    </sessionState>

注释部分提供了属性及每个属性的示例设置的一个示例。

使用来自 Azure 门户中的缓存边栏选项卡的值配置属性，并根据需要配置其他值。 有关访问缓存属性的说明，请参阅[配置 Redis 缓存设置](/documentation/articles/cache-configure/#configure-redis-cache-settings)。

* **host** - 指定缓存终结点。
* **port** - 使用非 SSL 端口或 SSL 端口，具体取决于 SSL 设置。
* **accessKey** - 使用缓存的主密钥或辅助密钥。
* **ssl** - 如果要使用 SSL 保护缓存/客户端通信，则为 true；否则为 false。 请务必指定正确的端口。
    * 默认情况下，将为新缓存禁用非 SSL 端口。 为此设置指定 true 可使用 SSL 端口。 有关启用非 SSL 端口的详细信息，请参阅[配置缓存](/documentation/articles/cache-configure/)主题中的[访问端口](/documentation/articles/cache-configure/#access-ports)部分。
* **throwOnError** - 如果要在失败时引发异常，则为 true；如果要操作以静默方式失败，则为 false。 可以通过检查静态 Microsoft.Web.Redis.RedisSessionStateProvider.LastException 属性来检查失败。 默认值为 true。
* **retryTimeoutInMilliseconds** - 将在此时间间隔内重试失败的操作，以毫秒为单位指定。 首次重试在 20 毫秒后进行，然后重试每隔一秒进行，直到 retryTimeoutInMilliseconds 间隔到期。 在此时间间隔过后，将立即重试操作最后一次。 如果操作仍失败，则会将异常返回给调用方，具体取决于 throwOnError 设置。 默认值为 0，这意味着不重试。
* **databaseId** - 指定要用于缓存输出数据的数据库。 如果未指定，则使用默认值 0。
* **applicationName** - 密钥存储在 Redis 中作为 `{<Application Name>_<Session ID>}_Data`。 此命名方案使多个应用程序可以共享同一密钥。 此参数是可选的，如果未提供它，则使用默认值。
* **connectionTimeoutInMilliseconds** - 此设置允许你覆盖 StackExchange.Redis 客户端中的 connectTimeout 设置。 如果未指定，则使用默认 connectTimeout 设置 5000。
* **operationTimeoutInMilliseconds** - 此设置允许你覆盖 StackExchange.Redis 客户端中的 syncTimeout 设置。 如果未指定，则使用默认 syncTimeout 设置 1000。

有关这些属性的详细信息，请参阅 [宣布推出适用于 Redis 的 ASP.NET 会话状态提供程序](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的原始博客文章公告。

别忘了在 web.config 中注释禁止标准 InProc 会话状态提供程序部分。

    <!-- <sessionState mode="InProc"
         customProvider="DefaultSessionProvider">
         <providers>
            <add name="DefaultSessionProvider"
                  type="System.Web.Providers.DefaultSessionStateProvider,
                        System.Web.Providers, Version=1.0.0.0, Culture=neutral,
                        PublicKeyToken=31bf3856ad364e35"
                  connectionStringName="DefaultConnection" />
          </providers>
    </sessionState> -->

执行这些步骤后，你的应用程序已配置为使用 Redis 缓存会话状态提供程序。 在应用程序中使用会话状态时，会话状态存储在 Azure Redis 缓存实例中。

> [AZURE.IMPORTANT]
> 与可以存储在默认的内存中 ASP.NET 会话状态提供程序中的数据不同，在缓存中存储的数据必须可序列化。 使用适用于 Redis 的会话状态提供程序时，请确保在会话状态中存储的数据类型可序列化。
> 
> 

## <a name="aspnet-session-state-options"></a> ASP.NET 会话状态选项
* 内存中会话状态提供程序 - 此提供程序将会话状态存储在内存中。 使用此提供程序的好处是它简单且快速。 但是，如果使用内存中提供程序，由于它不是分布式的，因此不能缩放 Web 应用。
* SQL Server 会话状态提供程序 - 此提供程序将会话状态存储在 SQL Server 中。 如果要将会话状态保存在持久性存储中，请使用此提供程序。 可以缩放 Web 应用，但将 SQL Server 用于会话会对 Web 应用造成性能影响。
* 分布式内存中会话状态提供程序，如 Redis 缓存会话状态提供程序 - 此提供程序提供两全其美的功能。 Web 应用可以使用简单、快速且可缩放的会话状态提供程序。 由于此提供程序将会话状态存储在缓存中，应用必须考虑到在与分布式内存中缓存通信时关联的所有特征，如暂时性网络故障。

有关会话状态和其他最佳实践的详细信息，请参阅 [Web Development Best Practices (Building Real-World Cloud Apps with Azure)](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/web-development-best-practices)（Web 开发最佳做法（使用 Azure 构建实际的云应用））。

## <a name="next-steps"></a>后续步骤
请查看 [Azure Redis 缓存的 ASP.NET 输出缓存提供程序](/documentation/articles/cache-aspnet-output-cache-provider/)。

<!--Update_Description: change nuget package install method-->