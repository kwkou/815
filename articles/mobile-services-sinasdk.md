<properties urlDisplayName="使用新浪微博帐户登录进行第三方身份认证" 
			pageTitle="使用新浪微博帐户登录进行第三方身份认证 - Azure 教程" 
			metaKeywords="移动服务，第三方身份认证，新浪微博" 
			description="认证和授权是当今移动应用都需要的关键功能。Azure Mobile Service 的一个最强有力的功能就是使得向移动应用中添加用户认证变得容易。随着新浪微博团队和微软 （DX TE） 的全力合作，首个中国第三方 Azure Mobile Service 认证 SDK——新浪微博认证 SDK 上线了。" 
			metaCanonical="" 
			services="" 
			documentationCenter="Mobile" 
			title="" 
			authors="" 
			solutions="" 
			manager="" editor="EricChen" />

<tags wacn.date="05/18/2015" ms.service="mobile-services" ms.date=""/>

#使用新浪微博帐户登录进行第三方身份认证
本主题将说明如何注册你的应用程序，以便能够使用新浪微博作为 Azure Mobile Service 的身份验证提供程序。认证和授权是当今移动应用都需要的关键功能。Azure Mobile Service 的一个最强有力的功能就是使得向移动应用中添加用户认证变得容易。
这一功能的实现基于 Mobile Service 对于跨平台的支持，以及对于三方身份认证服务（例如 Facebook、Google、Microsoft、Twitter 和 Azure Active Directory）的支持。**随着新浪微博团队和微软 （DX TE） 的全力合作，首个中国第三方 Azure Mobile Service 认证 SDK——新浪微博认证 SDK 上线了**。
![][1]
##Mobile Service结合新浪微博认证实现流程概述
认证的第一步是将应用注册到新浪微博开发者门户，从而获应用ID和应用密钥。随后将这些信息填入 Mobile Service 管理门户，应用才能使用这一身份认证提供商进行认证。
接下来的流程如下图所示。认证使用的 Oauth 通常需要一个复杂的配置和认证流程，但是 Mobile Service 代办了所有的操作细节。具体过程如下：
1.	向 Mobile Service 注册认证提供商以获得认证功能，随后客户端会调用认证方法，将认证名称传入。
2.	Mobile Service 和提供商使用 Oauth 认证用户。
3.	用户 ID 和 token 被返回客户端存储，进而可被 Mobile Service 访问。
	![][2]##使用新浪微博第三方身份认证实现步骤1.	 将应用注册到微博开发者门户。
	应用注册通过两步完成。第一步，将应用名称注册到新浪微博的开发者网站。注册成功后，即可通过开发者网站查看应用 ID 和客户端密钥。例如，新浪微博开发者门户中，在创建了一个移动应用后，可在下图位置中可以找到关于应用 ID 和应用密钥的信息。	![][3]

	然后将应用 ID 和密钥保存在 web.config 文件中以供应用后端逻辑使用。
	![][4]
2.	 在应用后端工程中安装新浪微博 Nuget 包。通过 Visual Studio 的支持，这一安装过程会比较便捷。
	**注意：在项目中引用 Windows Azure Mobile Service SDK 的方法有两种**。
	* 下载新浪微博 Windows Azure Mobile Service SDK 源代码，并在项目中引用。这样的做法更易于进行调试。
	* 在 Visual Studio 中通过 Nuget 直接下载 DLL 使用。这个方法更简便，但无法调试修改与 Sina 接口部分代码。
		![][5]3.	提供定制化 Login Provider 的实现。完成这一 Provider 的第一步是注册微博 OWIN 兼容性中间件，这样就可以参与认证，并在运行时刻序列化或反序列化访问 token。
	![][6]
	![][7]4.	在后端通过 WebAPIConfig 注册微博 SDK 类。
	![][8]5.	部署应用程序到Mobile Service。
	导入Windows Azure 订阅文件：
	![][9]	点击发布来部署应用程序：
	![][10]**这一微博认证SDK证明了微软云平台的开放性，同时也表明中国开发者在微软平台上使用中国本土提供商的需求。所以这个SDK将是中国本土云服务提供商和DX为微软云平台带来更多三方服务的合作开端**。##相关资源
1.	[新浪官方SDK页面的链接](http://open.weibo.com/wiki/SDK)
2.	[Github上的源代码](https://github.com/SinaWeiBoAuth/MobileServiceAppsUsingSinaweiboAccountAuthorize)
3.	[Nuget 程序包](https://www.nuget.org/packages/SinaWeiboAuthenticationSDK_AzureMobileService/1.0.0)
4.	[该SDK具体实现过程的指导博客](http://www.cnblogs.com/sonic1abc/p/4308994.html )
<!-- images -->
[1]: ./media/mobile-services-sinasdk/sinasdk-01.png
[2]: ./media/mobile-services-sinasdk/sinasdk-02.png
[3]: ./media/mobile-services-sinasdk/sinasdk-03.png
[4]: ./media/mobile-services-sinasdk/sinasdk-04.png
[5]: ./media/mobile-services-sinasdk/sinasdk-05.png
[6]: ./media/mobile-services-sinasdk/sinasdk-06.png
[7]: ./media/mobile-services-sinasdk/sinasdk-07.png
[8]: ./media/mobile-services-sinasdk/sinasdk-08.png
[9]: ./media/mobile-services-sinasdk/sinasdk-09.png
[10]: ./media/mobile-services-sinasdk/sinasdk-10.png