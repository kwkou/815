<properties 
   pageTitle="缓存 ASP.NET 会话状态提供程序"
   description="了解如何使用 Azure Redis 缓存存储 ASP.NET 会话状态"
   services="redis-cache"
   documentationCenter="na"
   authors="steved0x"
   manager="dwrede"
   editor="tysonn" />
<tags
	ms.service="cache"
   ms.date="04/27/2016"
	wacn.date="06/29/2016"/>

# Azure Redis 缓存的 ASP.NET 会话状态提供程序

Azure Redis Cache 提供一个会话状态提供程序，可用于在缓存而不是内存或 SQL Server 数据库中存储您的会话状态。若要使用缓存会话状态提供程序，首先配置你的缓存，然后使用 Redis 缓存会话状态 NuGet 包为缓存配置 ASP.NET 应用程序。

在实际云应用中避免存储某种形式的用户会话状态通常是不现实的，但某些方法相比其他方法而言，对性能和可伸缩性的影响更大。如果你需要存储状态，最佳解决方案是使状态量保持较小并将其存储在 Cookie 中。如果这不可行，下一个最佳解决方案是将 ASP.NET 会话状态与提供程序配合使用以实现分布式内存中缓存。从性能和可伸缩性的角度来看，最差的解决方案是使用数据库支持的会话状态提供程序。本主题提供有关使用 Azure Redis 缓存的 ASP.NET 会话状态提供程序的指南。有关其他会话状态选项的信息，请参阅 [ASP.NET 会话状态选项](#aspnet-session-state-options)。

## 在缓存中存储 ASP.NET 会话状态

若要使用 Redis 缓存会话状态 NuGet 包配置客户端应用程序，请在“解决方案资源管理器”中右键单击项目，然后选择“管理 NuGet 包”。

![Azure Redis 缓存管理 NuGet 包](./media/cache-aspnet-session-state-provider/redis-cache-manage-nuget-menu.png)

在搜索文本框中键入 **RedisSessionStateProvider**，从结果中选择它，然后单击“安装”。

>[AZURE.IMPORTANT]如果你使用高级层的聚类分析功能，则必须使用 [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) 2.0.0 或更高版本，否则会引发异常。这是一项重大更改；有关详细信息，请参阅 [2\.0.0 版重大更改详细信息](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details)。

![Azure Redis 缓存会话状态提供程序](./media/cache-aspnet-session-state-provider/redis-cache-session-state-provider.png)

Redis 会话状态提供程序 NuGet 包依赖于 StackExchange.Redis.StrongName 包。如果你的项目中没有 StackExchange.Redis.StrongName 包，则将会安装它。请注意，除了强命名的 StackExchange.Redis.StrongName 包外，还有 StackExchange.Redis 非强命名版本。如果你的项目使用的是非强命名 StackExchange.Redis 版本，则必须在安装 Redis 会话状态提供程序 NuGet 包之前或之后将其卸载，否则你的项目中将出现命名冲突。有关这些包的详细信息，请参阅[配置 .NET 缓存客户端](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#configure-the-cache-clients)。

NuGet 程序包会下载并添加所需的程序集引用，并将以下部分添加到您的 web.config 文件，包含 ASP.NET 应用程序所需的配置，以使用 Redis Cache 会话状态提供程序。

    <sessionStatemode="Custom" customProvider="MySessionStateStore">
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
		<add name="MySessionStateStore"type="Microsoft.Web.Redis.RedisSessionStateProvider"host="127.0.0.1"accessKey="" ssl="false"/>
        </providers>
    </sessionState>

注释部分提供了属性及每个属性的示例设置的一个示例。

你可以通过 Azure PowerShell 配置 Redis 缓存。有关访问缓存属性的说明，请参阅[使用 Azure PowerShell 管理 Azure Redis 缓存](/documentation/articles/cache-howto-manage-redis-cache-powershell/)。

-	**host** - 指定缓存终结点。
-	**port** - 使用非 SSL 端口或 SSL 端口，具体取决于 SSL 设置。
-	**accessKey** - 使用缓存的主密钥或辅助密钥。
-	**ssl** - 如果要使用 ssl 保护缓存/客户端通信，则为 true；否则为 false。请务必指定正确的端口。
	-	默认情况下，将为新缓存禁用非 SSL 端口。为此设置指定 true 可使用 SSL 端口。
-	**throwOnError** - 如果你想要在失败时引发异常，则为 true；如果你想要操作以静默方式失败，则为 false。可以通过检查静态 Microsoft.Web.Redis.RedisSessionStateProvider.LastException 属性来检查失败。默认值为 true。
-	**retryTimeoutInMilliseconds** - 将在此时间间隔内重试失败的操作，以毫秒为单位指定。首次重试在 20 毫秒后进行，然后重试每隔一秒进行，直到 retryTimeoutInMilliseconds 间隔到期。在此时间间隔过后，将立即重试操作最后一次。如果操作仍失败，则会将异常返回给调用方，具体取决于 throwOnError 设置。默认值为 0，这意味着不重试。
-	**databaseId** - 指定要用于缓存输出数据的数据库。如果未指定，则使用默认值 0。
-	**applicationName** - 密钥存储在 redis 中作为 `{<Application Name>_<Session ID>}_Data`。这使多个应用程序可以共享同一密钥。此参数是可选的，如果未提供它，则使用默认值。
-	**connectionTimeoutInMilliseconds** - 此设置允许你覆盖 StackExchange.Redis 客户端中的 connectTimeout 设置。如果未指定，则使用默认 connectTimeout 设置 5000。有关详细信息，请参阅 [StackExchange.Redis 配置模型](http://go.microsoft.com/fwlink/?LinkId=398705)。
-	**operationTimeoutInMilliseconds** - 此设置允许你覆盖 StackExchange.Redis 客户端中的 syncTimeout 设置。如果未指定，则使用默认 syncTimeout 设置 1000。有关详细信息，请参阅 [StackExchange.Redis 配置模型](http://go.microsoft.com/fwlink/?LinkId=398705)。
							
有关这些属性的详细信息，请参阅[宣布推出适用于 Redis 的 ASP.NET 会话状态提供程序](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的原始博客文章公告。

别忘了在 web.config 中注释掉标准 InProc 会话状态提供程序部分。

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

执行这些步骤后，你的应用程序已配置为使用 Redis 缓存会话状态提供程序。在应用程序中使用会话状态时，会话状态将存储在 Azure Redis 缓存实例中。

>[AZURE.NOTE]请注意，与可以存储在默认的内存中 ASP.NET 会话状态提供程序中的数据不同，在缓存中存储的数据必须可序列化。使用适用于 Redis 的会话状态提供程序时，请确保在会话状态中存储的数据类型可序列化。

##<a name="aspnet-session-state-options"></a> ASP.NET 会话状态选项

- 内存中会话状态提供程序 - 此提供程序将会话状态存储在内存中。使用此提供程序的好处是它简单且快速。但是，如果使用内存中提供程序，由于它不是分布式的，因此不能缩放 Web 应用。

- SQL Server 会话状态提供程序 - 此提供程序将会话状态存储在 SQL Server 中。如果要将会话状态保存在持久性存储中，则应使用此提供程序。可以缩放 Web 应用，但将 SQL Server 用于会话将对 Web 应用造成性能影响。

- 分布式内存中会话状态提供程序，如 Redis 缓存会话状态提供程序 - 此提供程序提供两全其美的功能。Web 应用可以使用简单、快速且可伸缩的会话状态提供程序。但是，你必须一直考虑此提供程序将会话状态存储在缓存中，而你的应用必须考虑到在与分布式内存中缓存通信时关联的所有特征，如暂时性网络故障。有关使用缓存的最佳实践，请参阅 Microsoft 模式和实践 [Azure 云应用程序设计和实现指南](https://github.com/mspnp/azure-guidance)中的[缓存指南](https://github.com/mspnp/azure-guidance/blob/master/Caching.md)。

有关会话状态和其他最佳实践的详细信息，请参阅 [Web 开发最佳做法（使用 Azure 构建实际的云应用）](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/web-development-best-practices)。

## 后续步骤

了解 [Azure Redis 缓存的 ASP.NET 输出缓存提供程序](/documentation/articles/cache-aspnet-output-cache-provider/)。

<!---HONumber=Mooncake_1207_2015-->