<properties 
	pageTitle="在 Azure Web 应用上创建数字市场营销活动" 
	description="本指南提供如何使用 Azure Web 应用创建数字市场营销活动的技术概述。这包括部署、社交媒体集成、缩放策略和监视。" 
	editor="jimbe" 
	manager="wpickett" 
	authors="cephalin" 
	services="app-service\web" 
	documentationCenter=""/>

<tags
	ms.service="app-service-web"
	ms.date="02/26/2016" 
	wacn.date="05/24/2016"/>

# 在 Azure Web 应用上创建数字市场营销活动
[Azure Web 应用](/documentation/services/web-sites/) Web 应用是数字市场营销活动的极佳选择。数字市场营销活动的持续时间通常较短，旨在促成短期市场营销目标。有两个要考虑的主要方案。在第一个方案中，第三方市场营销公司为其客户为促销期间创建并管理市场营销活动。第二个方案涉及市场营销公司创建数字市场营销活动资源，然后将资源的所有权转移给其客户。客户然后自己运行和管理数字市场营销活动。Azure Web 应用非常适合于这两种方案。

以下是一个使用 Azure Web 应用的，全球，多频道数字市场营销活动。它展示了你简单地把 Azure Web 应用和其他服务组合起来，使用最小技术投入既能达到的效果。**点击图片中的图标即可了解更多。**

<object type="image/svg+xml" data="./media/web-sites-digital-marketing-application-solution-overview/digital-marketing-notitle.svg" width="100%" height="100%"></object>

> [AZURE.NOTE]
> 本指南介绍了一些与在 Azure 中运行数字市场营销活动一致的最常见领域和任务。但是，还有其他你可以在 Azure 中实现的常见解决方案。若要查看这些解决方案，请参阅[全球网络影响力](/documentation/articles/web-sites-global-web-presence-solution-overview/)和[业务应用程序](/documentation/articles/web-sites-business-application-solution-overview/)中的其他指南。

## 从头开始创建或引入现有资产

将使用各种语言和框架的现有 Web 资产快速引入 Azure Web 应用。

你可以使用你最喜欢的 CMS 风格创建 Web 应用。你可以从各种数据库后端选择以满足你的需要，包括 [Azure SQL 数据库]和 [MySQL]。

无论你的现有 Web 资产是 .NET、PHP、Java、Node.js 还是 Python，现在都可在 Web 应用中运行。你可以使用熟悉的 [FTP] 工具将它们移动到 Web 应用。如果频繁创建数字市场营销活动，则源代码控制系统中可能存在现有 Web 资产。你可以直接从流行的源代码管理选项部署到 Web 应用，如 [Visual Studio] 和 本地 [Git]、GitHub、Mercurial 等。

## 保持敏捷

通过直接从现有的源控件连续发布并在 Azure 中运行 A/B 测试保持敏捷

在 Web 应用的计划、原型制作和早期部署过程中，你及你的客户可以通过[将其部署到过渡槽]在市场活动应用正式运行前查看该 Web 应用的实际工作版本。通过将源控件与 Azure Web 应用相集成，你可以[连续发布]到过渡槽，并且在准备就绪时，将其切换到生产环境且无需停机。

此外，在对实时 Web 应用进行更改规划时，可以轻松地使用生产测试中的功能对建议更新[运行 A/B 测试]，分析真实用户行为，以帮助你对应用设计做出明智的决策。


## 转到社交

Azure 中的数字市场营销活动可以通过使用 Facebook 和 Twitter 等常用的提供程序进行身份验证，与社交媒体相集成。有关针对 ASP.NET 应用程序的此方法的示例，请参阅[创建包含身份验证和 SQL DB 的 ASP.NET MVC 应用并将其部署到 Azure Web 应用]。

此外，每个社交媒体站点通常都会提供与从 .NET 和从许多其他框架进行集成的其他方法的信息。

## 使用富媒体访问所有设备

使用其他 Azure 服务丰富你的数字市场营销活动：

-  使用 [Azure 媒体服务]全局上载并流式处理视频
-  使用[移动服务]在Windows、iOS 和 Android 设备上建立状态
-  使用[通知中心]向数百万台设备发送推送通知

## 转到全局

通过为区域站点提供 Azure 流量管理器服务以及使用 Azure CDN 快速传递内容来转到全局。

若要为在各自区域的全局客户提供服务，使用 [Azure 流量管理器]将站点访问者路由到可以提供最佳性能的地区站点。或者，你可以在多个区域托管的 Web 应用的多个副本中均衡分布负载。

通过[将 Web 应用与 Azure CDN 集成]向全局用户快速传递静态内容。Azure CDN 可以缓存离用户最近的 CDN 节点中的静态内容，从而最大程度减少了滞后时间和连接到 Web 应用时间。

## 优化

Web 应用可通过使用 Autoscale 进行自动缩放，使用 Azure Redis 缓存进行缓存，使用 WebJobs 运行后台任务以及使用 Azure 流量管理器维护高可用性达到优化目的。

Azure Web 应用的增加和扩大功能非常适用于不可预测的工作负载，数字市场营销活动就属于这种情况。通过 [Azure 经典管理门户](https://manage.windowsazure.cn/)手动扩大，通过[服务管理 API] 或 [PowerShell 脚本]以编程方式扩大 Web 应用，或者通过自动缩放功能自动扩大。在“标准”层，自动缩放功能使你可以基于 CPU 使用率自动扩大 Web 应用。此功能根据用户活动仅在需要时横向扩展 Web 应用，从而有助于最大限度提高灵活性并降低成本。有关最佳实践，请参阅 [Troy Hunt] 的[我所了解的有关使用 Azure 快速缩放 Web 应用的十大事项]。

使用 [Azure Redis 缓存]让你的 Web 应用响应更快。可以利用它从后端数据库和其他操作。

使用 [Azure 流量管理器]维护 Web 应用的高可用性。使用“故障转移”方法，当主站点出现问题时，流量管理器可以自动将流量路由到辅助站点。

## 监视和分析

使用 Azure 或第三方工具让 Web 应用的性能保持最新状态。接收关于关键 Web 应用事件的警报。使用 Application Insight 或 HDInsight 中的 Web 日志分析，让用户可以轻松地深入了解。

在“标准”层，当 Web 应用无法响应时，监视器应用程序响应能力会收到电子邮件通知。

## 更多资源

- [Azure Web 应用文档](/home/features/web-site/)
- [Azure Web 博客](/blog/tags/网站)

[Azure Websites]: /home/features/web-site/

[MySQL]: /documentation/articles/web-sites-php-mysql-deploy-use-git/
[Azure SQL 数据库]: /documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/
[FTP]: /documentation/articles/web-sites-deploy/#ftp
[Visual Studio]: /documentation/articles/web-sites-dotnet-get-started/
[Git]: /documentation/articles/web-sites-publish-source-control/
[将其部署到过渡槽]: /documentation/articles/web-sites-staged-publishing/
[连续发布]: http://rickrainey.com/2014/01/21/continuous-deployment-github-with-azure-web-sites-and-staged-publishing/
[运行 A/B 测试]: http://blogs.msdn.com/b/tomholl/archive/2014/11/10/a-b-testing-with-azure-websites.aspx

[创建包含身份验证和 SQL DB 的 ASP.NET MVC 应用并将其部署到 Azure Web 应用]: /documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/
[Azure 媒体服务]: http://blogs.technet.com/b/cbernier/archive/2013/09/03/windows-azure-media-services-and-web-sites.aspx
[移动服务]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/
[通知中心]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/
[Azure 流量管理器]: http://www.hanselman.com/blog/CloudPowerHowToScaleAzureWebsitesGloballyWithTrafficManager.aspx
[将 Web 应用与 Azure CDN 集成]: /documentation/articles/cdn-websites-with-cdn/

[Azure Management Portal]: http://manage.windowsazure.cn/
[服务管理 API]: http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx
[PowerShell 脚本]: http://msdn.microsoft.com/zh-cn/library/azure/jj152841.aspx
[Troy Hunt]: https://twitter.com/troyhunt
[我所了解的有关使用 Azure 快速缩放 Web 应用的十大事项]: http://www.troyhunt.com/2014/09/10-things-i-learned-about-rapidly.html
[Azure Redis 缓存]: http://azure.microsoft.com/zh-cn/blog/2014/06/05/mvc-movie-app-with-azure-redis-cache-in-15-minutes/

[Azure Application Insights]: http://blogs.msdn.com/b/visualstudioalm/archive/2015/01/07/application-insights-and-azure-websites.aspx
[New Relic]: /develop/net/how-to-guides/new-relic/

  
  [gitstaging]: http://www.bradygaster.com/post/multiple-environments-with-windows-azure-web-sites
 

<!---HONumber=79-->