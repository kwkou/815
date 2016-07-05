<properties 
	pageTitle="将 Azure 中的 Azure Redis 缓存用于会话状态" 
	description="了解如何使用 Azure 缓存服务来支持 ASP.NET 会话状态缓存。" 
	services="app-service\web" 
	documentationCenter=".net" 
 	authors="Rick-Anderson" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web"
	ms.date="02/27/2016" 
	wacn.date="04/26/2016"/>


# 将 Azure 中的 Azure Redis 缓存用于会话状态


本主题说明如何将 Azure Redis Cache 服务（预览版）用于会话状态。

如果你的 ASP.NET Web 应用使用会话状态，则你需要设置外部会话状态提供程序（可为 Redis Cache 服务或 SQL Server 会话状态提供程序）。如果你使用会话状态，但未使用外部提供程序，则你的 Web 应用只能限定为一个实例。Redis Cache 服务是最快最简单的启用方式。

## <a id="configureproject"></a>将 RedisSessionStateProvider NuGet 包添加到 Web 应用
安装 NuGet `RedisSessionStateProvider` 包。从包管理器控制台（“工具”>“NuGet 包管理器”>“包管理器控制台”）使用以下命令进行安装：

  `PM> Install-Package Microsoft.Web.RedisSessionStateProvider`
  
若要从“工具”>“NuGet 包管理器”>“管理解决方案的 NuGet 包”进行安装，请搜索 `RedisSessionStateProvider`。

有关详细信息，请参阅 [NuGet RedisSessionStateProvider 页](http://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider/)。

## <a id="configurewebconfig"></a>修改 Web.config 文件。
除了为缓存生成程序集引用，NuGet 包还在 *web.config* 文件中添加存根项。

1. 打开 *web.config* 并查找 **sessionState** 元素。

1. 输入 `host`、`accessKey`、 `port` 的值（SSL 端口应为 6380），并将 `SSL` 设置为 `true`。可以从 [Azure 经典管理门户](http://manage.windowsazure.cn)边栏选项卡为你的缓存实例获取这些值。请注意，默认情况下，将为新缓存禁用非 SSL 端口。以下标记显示了对 *web.config* 文件所做的更改，具体而言，是对 *port*、*host*、accessKey* 和 *ssl* 的更改。

		  <system.web>;
		    <customErrors mode="Off" />;
		    <authentication mode="None" />;
		    <compilation debug="true" targetFramework="4.5" />;
		    <httpRuntime targetFramework="4.5" />;
		    <sessionState mode="Custom" customProvider="RedisSessionProvider">;
		      <providers>;  
		          <!--<add name="RedisSessionProvider" 
		            host = "127.0.0.1" [String]
		            port = "" [number]
		            accessKey = "" [String]
		            ssl = "false" [true|false]
		            throwOnError = "true" [true|false]
		            retryTimeoutInMilliseconds = "0" [number]
		            databaseId = "0" [number]
		            applicationName = "" [String]
		          />;-->;
		         <add name="RedisSessionProvider" 
		              type="Microsoft.Web.Redis.RedisSessionStateProvider" 
		              port="6380"
              		      host="movie2.redis.cache.chinacloudapi.cn" 
		              accessKey="m7PNV60CrvKpLqMUxosC3dSe6kx9nQ6jP5del8TmADk=" 
		              ssl="true" />;
		      <!--<add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider" host="127.0.0.1" accessKey="" ssl="false" />;-->;
		      </providers>;
		    </sessionState>;
		  </system.web>;


## <a id="usesessionobject"></a>在代码中使用 Session 对象
最后一步是开始在 ASP.NET 代码中使用 Session 对象。通过使用 **Session.Add** 方法将对象添加到会话状态。此方法使用键值对在会话状态缓存中存储项。

    string strValue = "yourvalue";
	Session.Add("yourkey", strValue);

下面的代码从会话状态中检索该值。

    object objValue = Session["yourkey"];
    if (objValue != null)
       strValue = (string)objValue;	

还可以使用 Redis Cache 在 Web 应用中缓存对象。有关详细信息，请参阅 [15 分钟学会创建包含 Azure Redis 缓存的 MVC 影片应用](http://azure.microsoft.com/blog/2014/06/05/mvc-movie-app-with-azure-redis-cache-in-15-minutes/)。
有关如何使用 ASP.NET 会话状态的详细信息，请参阅 [ASP.NET 会话状态概述][]。

  
  
  [installed the latest]: /zh-cn/downloads/?sdk=net
  [ASP.NET 会话状态概述]: http://msdn.microsoft.com/zh-cn/library/ms178581.aspx

  [NewIcon]: ./media/web-sites-dotnet-session-state-caching/CacheScreenshot_NewButton.png
  [NewCacheDialog]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CreateOptions.png
  [CacheIcon]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CacheIcon.png
  [NuGetDialog]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_NuGet.png
  [OutputConfig]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_OC_WebConfig.png
  [CacheConfig]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_CacheConfig.png
  [EndpointURL]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_EndpointURL.png
  [ManageKeys]: ./media/web-sites-dotnet-session-state-caching/CachingScreenshot_ManageAccessKeys.png

<!---HONumber=74-->