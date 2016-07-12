<properties 
	pageTitle="使用 .NET 连接到媒体服务帐户" 
	description="本主题演示如何使用 .NET 连接到媒体服务。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="04/18/2016"
	wacn.date="06/20/2016"/>


# 使用适用于 .NET 的媒体服务 SDK 连接到媒体服务帐户

> [AZURE.SELECTOR]
- [REST](/documentation/articles/media-services-rest-connect-programmatically/)
- [.NET](/documentation/articles/media-services-dotnet-connect_programmatically/)


本主题介绍如何在使用适用于 .NET 的媒体服务 SDK 编程时获取与 Azure 媒体服务的编程连接。


## 连接到媒体服务

若要以编程方式连接到媒体服务，你必须在以前设置了一个 Azure 帐户，并在该帐户上配置了媒体服务，然后设置一个 Visual Studio 项目，以便通过适用于 .NET 的媒体服务 SDK 进行开发。有关详细信息，请参阅“设置以使用适用于 .NET 的媒体服务 SDK 进行开发”。

在媒体服务帐户设置过程结束时，你获得以下必需的连接值。请使用这些值以编程方式连接到媒体服务。

- 你的媒体服务帐户名。

- 你的媒体服务帐户密钥。

若要查找这些值，请转到 Azure 管理门户，选择你的媒体服务帐户，然后单击门户窗口底部的“管理密钥”图标。单击每个文本框旁边的图标将值复制到系统剪贴板中。


## 创建 CloudMediaContext 实例

若要开始针对媒体服务编程，你需要创建一个代表服务器上下文的 **CloudMediaContext** 实例。**CloudMediaContext** 包括对各种重要集合的引用，这些集合包括作业、资产、文件、访问策略和定位符。

>[AZURE.NOTE] **CloudMediaContext** 类不是线程安全的。每个线程或每组操作均应创建一个新 CloudMediaContext。


CloudMediaContext 具有五个构造函数重载。建议使用以 **MediaServicesCredentials** 为参数的构造函数。有关详细信息，请参阅下面的**重复使用访问控制服务令牌**。

以下示例使用 public CloudMediaContext(MediaServicesCredentials credentials) 构造函数：

	// _cachedCredentials and _context are class member variables. 
	_cachedCredentials = new MediaServicesCredentials(
	                _mediaServicesAccountName,
	                _mediaServicesAccountKey);
	
	_context = new CloudMediaContext(_cachedCredentials);


## 重复使用访问控制服务令牌

本部分说明如何通过使用以 MediaServicesCredentials 为参数的 CloudMediaContext 构造函数重复使用访问控制服务令牌。


[Azure Active Directory 访问控制](https://msdn.microsoft.com/zh-cn/library/hh147631.aspx)（也称为访问控制服务或 ACS）是一个基于云的服务，可轻松对用户进行身份验证和授权以使用户获得访问其 Web 应用程序的权限。Azure 媒体服务通过需要 ACS 令牌的 OAuth 协议控制对其服务的访问。媒体服务从授权服务器接收 ACS 令牌。

使用媒体服务 SDK 进行开发时，可选择不处理令牌，而是由 SDK 代码为你进行管理。不过，将 ACS 令牌完全交由 SDK 管理会导致不必要的令牌请求。请求令牌将耗用一定的时间并消耗客户端和服务器资源。此外，如果速度过快，ACS 服务器还会限制请求。上限为每秒钟 30 条请求，请参阅 [ACS 服务限制](https://msdn.microsoft.com/zh-cn/library/gg185909.aspx)了解更多详细信息。

自媒体服务 SDK 版本 3.0.0.0 起，可以重复使用 ACS 令牌。以 **MediaServicesCredentials** 作为参数的 **CloudMediaContext** 构造函数可实现在多个上下文之间共享 ACS 令牌。MediaServicesCredentials 类中封装有媒体服务凭据。如果 ACS 令牌可用且其过期时间已知，则可以使用该令牌创建一个新的 MediaServicesCredentials 实例并将其传递给 CloudMediaContext 的构造函数。请注意，媒体服务 SDK 将在每次令牌过期时自动刷新令牌。有两种方法可重复使用 ACS 令牌，如以下示例中所示。

- 你可以在内存中（例如，在静态类变量中）缓存 **MediaServicesCredentials** 对象。然后，将缓存的对象传递给 CloudMediaContext 构造函数。MediaServicesCredentials 对象包含一个 ACS 令牌，如果该令牌仍然有效，则可重复使用。如果该令牌无效，则会使用提供给 MediaServicesCredentials 构造函数的凭据通过媒体服务 SDK 刷新该令牌。

	请注意，在调用 RefreshToken 后，**MediaServicesCredentials** 对象将获得有效的令牌。**CloudMediaContext** 将调用构造函数中的 **RefreshToken** 方法。如果你计划将令牌值保存到外部存储中，请确保在保存令牌数据之前检查 TokenExpiration 值是否有效。如果该值无效，请在进行缓存前调用 RefreshToken。

		// Create and cache the Media Services credentials in a static class variable.
		_cachedCredentials = new MediaServicesCredentials(_mediaServicesAccountName, _mediaServicesAccountKey);

		
		// Use the cached credentials to create a new CloudMediaContext object.
		if(_cachedCredentials == null)
		{
		    _cachedCredentials = new MediaServicesCredentials(_mediaServicesAccountName, _mediaServicesAccountKey);
		}
		
		CloudMediaContext context = new CloudMediaContext(_cachedCredentials);

- 你也可以缓存 AccessToken 字符串和 TokenExpiration 值。这些值以后可以与缓存的令牌数据一起用于创建一个新的 MediaServicesCredentials 对象。这对于令牌可以在多个进程或多台计算机之间安全共享的方案尤其有用。

	以下代码段调用了未在此示例中定义的 SaveTokenDataToExternalStorage、GetTokenDataFromExternalStorage 和 UpdateTokenDataInExternalStorageIfNeeded 方法。你可以定义这些方法以在外部存储中存储、检索和更新令牌数据。

		CloudMediaContext context1 = new CloudMediaContext(_mediaServicesAccountName, _mediaServicesAccountKey);
		
		// Get token values from the context.
		var accessToken = context1.Credentials.AccessToken;
		var tokenExpiration = context1.Credentials.TokenExpiration;
		
		// Save token values for later use. 
		// The SaveTokenDataToExternalStorage method should check 
		// whether the TokenExpiration value is valid before saving the token data. 
		// If it is not valid, call MediaServicesCredentials’s RefreshToken before caching.
		SaveTokenDataToExternalStorage(accessToken, tokenExpiration);
		
	使用保存的令牌值可创建 MediaServicesCredentials。


		var accessToken = "";
		var tokenExpiration = DateTime.UtcNow;
		
		// Retrieve saved token values.
		GetTokenDataFromExternalStorage(out accessToken, out tokenExpiration);
		
		// Create a new MediaServicesCredentials object using saved token values.
		MediaServicesCredentials credentials = new MediaServicesCredentials(_mediaServicesAccountName, _mediaServicesAccountKey)
		{
		    AccessToken = accessToken,
		    TokenExpiration = tokenExpiration
		};
		
		CloudMediaContext context2 = new CloudMediaContext(credentials);

	在令牌已由媒体服务 SDK 更新的情况下，更新令牌副本。
	
		if(tokenExpiration != context2.Credentials.TokenExpiration)
		{
		    UpdateTokenDataInExternalStorageIfNeeded(accessToken, context2.Credentials.TokenExpiration);
		}
		

- 如果你具有多个媒体服务帐户（例如，用于负载共享目的或地域分布，则可以使用 System.Collections.Concurrent.ConcurrentDictionary 集合（ConcurrentDictionary 集合表示可由多个线程同时访问的密钥/值对的线程安全集合）缓存 MediaServicesCredentials 对象。然后可以使用 GetOrAdd 方法获得缓存凭据。

		// Declare a static class variable of the ConcurrentDictionary type in which the Media Services credentials will be cached.  
		private static readonly ConcurrentDictionary<string, MediaServicesCredentials> mediaServicesCredentialsCache = 
		    new ConcurrentDictionary<string, MediaServicesCredentials>();
		

		// Cache (or get already cached) Media Services credentials. Use these credentials to create a new CloudMediaContext object.
		static public CloudMediaContext CreateMediaServicesContext(string accountName, string accountKey)
		{
		    CloudMediaContext cloudMediaContext;
		    MediaServicesCredentials mediaServicesCredentials;
		
		    mediaServicesCredentials = mediaServicesCredentialsCache.GetOrAdd(
		        accountName,
		        valueFactory => new MediaServicesCredentials(accountName, accountKey));
		
		    cloudMediaContext = new CloudMediaContext(mediaServicesCredentials);
		
		    return cloudMediaContext;
		}
		
## 连接到中国北部地区的媒体服务帐户

如果你的帐户位于中国北部地区，请使用以下构造函数：

	public CloudMediaContext(Uri apiServer, string accountName, string accountKey, string scope, string acsBaseAddress)

例如：


	_context = new CloudMediaContext(
	    new Uri("https://wamsbjbclus001rest-hs.chinacloudapp.cn/API/"),
	    _mediaServicesAccountName,
	    _mediaServicesAccountKey,
	    "urn:WindowsAzureMediaServices",
	    "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn");


## 将连接值存储到配置中

强烈建议你将连接值（特别是敏感值，如你的帐户名和密码）存储到配置中。此外，建议你对敏感的配置数据加密。你可以使用 Windows 加密文件系统 (EFS) 加密整个配置文件。要对某个文件启用 EFS，请右键单击该文件，选择“属性”，然后在“高级”设置选项卡中启用加密。此外，你也可以使用受保护的配置来创建自定义解决方案，以便加密配置文件的所选部分。请参阅[使用受保护的配置来加密配置信息](https://msdn.microsoft.com/zh-cn/library/53tyfkaw.aspx)。

以下 App.config 文件包含了必需的连接值。<appSettings> 元素中的值是你从媒体服务帐户设置过程中获取的必需值。

	<configuration>
	  <appSettings>
	    <add key="MediaServicesAccountName" value="Media-Services-Account-Name" />
	    <add key="MediaServicesAccountKey" value="Media-Services-Account-Key" />
	  </appSettings>
	</configuration>


若要从配置中检索连接值，你可以使用 **ConfigurationManager** 类，然后将相关值分配给代码中的字段：
	
	private static readonly string _accountName = ConfigurationManager.AppSettings["MediaServicesAccountName"];
	private static readonly string _accountKey = ConfigurationManager.AppSettings["MediaServicesAccountKey"];





<!---HONumber=Mooncake_0613_2016-->