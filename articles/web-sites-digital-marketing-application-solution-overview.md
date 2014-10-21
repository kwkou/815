<properties linkid="websites-digital-marketing" urlDisplayName="Resources" pageTitle="Create a Digital Marketing Campaign on Azure Web Sites" metaKeywords="" description="This guide provides a technical overview of how to use Azure Web Sites to create digital marketing campaigns. This includes deployment, social media integration, scaling strategies, and monitoring." metaCanonical="" services="" documentationCenter="" title="Create a Digital Marketing Campaign on Azure Web Sites" authors="jroth" solutions="" manager="paulettm" editor="mollybos" />

# 在 Azure 网站上创建数字市场营销活动

本指南提供如何使用 Azure 网站创建数字市场营销活动的技术概述。数字市场营销活动通常是一种短期的实体，旨在促进短期市场营销目标。有两个要考虑的主要方案。在第一个方案中，第三方市场营销公司为其客户为促销期间创建并管理市场营销活动。第二个方案涉及市场营销公司创建数字市场营销活动资源，然后将资源的所有权转移给其客户。客户然后自己运行和管理数字市场营销活动。

[Azure 网站][Azure 网站]非常适合于这两种方案。它提供快速创建、支持多个框架和语言、可缩放以满足用户要求并且适应多种部署和源代码管理系统。通过使用 Azure，你还能够访问媒体服务之类的其他 Azure 服务，从而提高市场营销活动。

尽管可以使用 [Azure 云服务][Azure 云服务]或 [Azure 虚拟机][Azure 虚拟机]托管网站，但它不是此方案的最佳选择，除非存在 Azure 网站不提供的必需功能。要了解这些选项，请参阅 [Azure 网站、云服务和虚拟机：何时使用何种产品？][Azure 网站、云服务和虚拟机：何时使用何种产品？]。

在本指南中将针对以下方面：

-   [部署现有网站][部署现有网站]
-   [与社交媒体相集成][与社交媒体相集成]
-   [根据用户要求进行缩放][根据用户要求进行缩放]
-   [与其他服务集成][与其他服务集成]
-   [监视活动][监视活动]

<div class="dev-callout">
<strong>说明</strong>
<p>本指南展示为面向公众的 .COM 站点开发而进行调整的一些最常见领域和任务。但是，还有可在特定实施中使用的其他一些 Azure 网站功能。若要查看这些功能，另请参阅<a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-global-web-presence-solution-overview/">全球网络影响力</a>和<a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-business-application-solution-overview/">业务应用程序</a>中的其他指南。</p>
</div>

## <a name="deployexisting"></a>部署现有网站

在“全球网络影响力”方案中，我们探讨了用于创建和部署新网站的不同选项。如果你是 Azure 网站的新手，最好[查看该信息][查看该信息]。如果你频繁创建数字市场营销活动，则可能具有为不同促销自定义的现有 Web 资产。在本节中，我们将更进一步地看一下用于从源代码管理部署不同类型的网站的选项。

如果你出于多种目的重复使用 Web 资产，并且还没有使用源代码管理系统，则强烈建议你考虑源代码管理系统。这允许你存储可为特定客户分支出来和自定义的常见 Web 解决方案的模板。网站提供用于与许多不同的源代码存储库同步的选项。在“仪表板”选项卡上，选择“从源代码管理设置部署”链接。

![DigitalMarketingDeploy1][DigitalMarketingDeploy1]

这将显示一个具有多个源代码管理选项的对话框。这包括功能完备的源代码管理系统（例如 TFS）以及简单的部署解决方案（例如 Dropbox）。

![DigitalMarketingDeploy2][DigitalMarketingDeploy2]

你可以使用不同的源代码管理技术来管理基于现有基准的新项目。例如，你可以复制以前保存的基准存储库以便开始使用新项目，或者可以创建一个新分支以便跟踪针对当前项目的自定义。有关使用分支从相同的源代码管理存储库管理不同部署的很棒的示例，请参阅[具有 Azure 网站的多个环境][具有 Azure 网站的多个环境]。这篇文章演示了如何使用 git 分支管理过渡和生产环境。

在你将网站连接到源代码管理后，可以从门户配置和跟踪部署。有关将源代码管理用于网站的更多信息，请参阅[从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。

在使用现有 Web 资产时，能够灵活地托管多种不同网站也很重要。在“配置”选项卡上，你可为网站选择 .NET 和 PHP 支持。

![DigitalMarketingFrameworkVersions][DigitalMarketingFrameworkVersions]

除了这些配置选项之外，网站自动提供对 Python 2.7 和 Node.js 的支持。默认 Node.js 版本为 0.10.5。

Azure 网站的另外一个优势是加快将过渡网站部署到 Web 的速度。在网站的计划、原型制作和早期部署过程中，代理及其客户可以在市场活动网站正式运行前查看该网站的实际工作版本。在该网站可供生产后，代理可以管理客户的生产部署，或者向客户提供 Web 资产以便进行部署和管理。

## <a name="socialmedia"></a>与社交媒体相集成

大多数数字市场营销活动都利用社交媒体网站，例如 Facebook 或 Twitter。一个集成点是将社交媒体标识用于身份验证。有关针对 ASP.NET 应用程序的此方法的示例，请参阅[使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用程序部署到 Azure 网站][使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用程序部署到 Azure 网站]。

但是，许多数字市场营销活动都超出了身份验证并且使用社交媒体集成作为其策略的关键部分。社交媒体网站通常具有开发人员部分，说明与其服务相集成的应用程序的不同方式。可以从几乎所有 Web 框架使用提供 REST API 的服务。但是，通常具有特定于你所选语言的信息。你应该从你选择作为目标的每个社交媒体网站直接查找类似的开发人员指导。

<!-- For ASP.NET developer targeting Facebook, Visual Studio provides a template for an MVC 4 Facebook Application.     ![DigitalMarketingFacebook][DigitalMarketingFacebook]     For a quick walkthrough of using this template with Web Sites, see [Creating a Facebook app using ASP.NET MVC Facebook Templates and hosting them for free on Azure Websites][fbtutorial]. For a more detailed tutorial and example application, see [ASP.NET MVC Facebook Birthday App][fbbirthdayapp].  -->

如果你是 ASP.NET 开发人员，认识到与社交媒体的集成未受到 Visual Studio 提供的模板的限制十分重要。模板只是帮助你简化流程。但如前所述，每个社交媒体网站通常都提供与从 .NET 和从许多其他语言和框架进行连接的其他方法有关的信息。

## <a name="scale"></a>根据用户要求进行缩放

云计算适用于无法预测的工作负荷。数字市场营销活动就属于此类别。很难预测相对短期的市场营销网站的流行情况，因为它主要依赖于吸引用户兴趣以及驱动对网站的更多流量的相关社交媒体交互。Azure 提供若干选项用于缩放网站和云服务。

-   通过 [Azure 管理门户][Azure 管理门户]手动缩放。
-   通过[服务管理 API][服务管理 API]或 [PowerShell 脚本编写][PowerShell 脚本编写]以编程方式缩放。
-   通过自动缩放（预览）功能自动缩放。

管理门户中，转到你的网站的“缩放”选项卡。有几个用于缩放的选项：第一个选项确定网站模式，它设置为“免费”、“共享”或“标准”。

![DigitalMarketingScale][DigitalMarketingScale]

对于每个层，可伸缩性功能和选项逐步增加。例如，“免费”模式网站不能向外扩展到多个实例，但“共享”可以向外扩展到 6 个实例。你还可以通过选择“标准”模式具有向上扩展的选项。此模式在专用虚拟机上运行你的网站，并且提供小、中和大的虚拟机大小选项。在确定了虚拟机的大小后，你还可以选择向外扩展到多个实例。在“标准”模式中，你可以向外扩展到 10 个实例。在[网站定价准则][网站定价准则]中可以找到各模式之间差异的完整列表。

在你选择“标准”模式后，将具有启用自动缩放（预览）功能的附加选项。这使你能够基于 CPU 进行缩放。“目标 CPU”百分比是 Azure 针对你的网站实例的处理器使用率范围。如果该平均 CPU 低于此目标，Azure 会降低实例计数，因为针对较少实例的相同负荷将导致增加对剩余实例的使用率。但是，它不能将实例计数降低到低于最小“实例计数”值。同样，如果该平均 CPU 高于“目标 CPU”，则 Azure 将增加实例数目。跨附加虚拟机分布的相同负荷将导致降低每个虚拟机上的 CPU 使用率。添加的实例数目受到最大“实例计数”值的限制。

![DigitalMarketingAutoScale][DigitalMarketingAutoScale]

自动缩放能够大大提高资源使用效率。例如，在晚上和周末，数字市场营销活动可能最活跃。这还将有效处理某一营销活动的受欢迎程度意外增加的情形。自动缩放有助于有效处理增加的负荷，然后在负荷降低时减少实例（和成本）。

有关缩放网站的详细信息，请参阅[如何扩展网站][如何扩展网站]。此主题还与监视紧密关联。有关更多信息，请参阅本指南中有关[监视你的活动][监视活动]的部分。

<div class="dev-callout">
<strong>说明</strong>
<p>对于选择使用云服务和 Web 角色的 Web 应用程序，还有其他可基于队列中项的长度进行缩放的选项。在云服务中，处理后端队列的角色是常见的体系结构模式。有关云服务缩放的更多信息，请参阅<a href="http://www.windowsazure.cn/zh-cn/manage/services/cloud-services/how-to-scale-a-cloud-service/">如何缩放云服务</a>。</p>
</div>

<!-- ##<a name="integrate"></a>Integrate with Other Services A digital marketing site will often incorporate rich media, such as video streaming. Hosting these sites in Azure provides close integration to related Azure services. For example, you can use Azure Media Services to encode and stream video for your web site. For more information on Media Services, see [Introduction to Azure Media Services Concepts and Scenarios][mediaservices].  Other Azure services can be used to create a more robust application. For example, Web Sites can use distributed caching provided by the new [Azure Cache Service (Preview)][caching]. Or you can use Azure Storage Services to store application data and resources. For example, graphics, videos, and other large files can be durably stored in blobs. Database services, such as Azure SQL Database and MySQL, are also available for relational data requirements. -->

## <a name="monitor"></a>监视活动

“监视器”选项卡包含多种度量，可帮助你评估数字市场营销网站成功与否以及性能。

![DigitalMarketingMonitor][DigitalMarketingMonitor]

例如，你可以通过查看“CPU 时间”、“请求数”和“输出数据量”之类的度量，查看使用模式和级别。随着市场营销活动变得日益受欢迎，所有这些度量都将增加。可以使用与使用率有关的信息确定何时向外扩展或向上扩展。有关详细信息，请参阅[如何监视网站][如何监视网站]。

对于“免费”和“共享”模式，你还应知道资源配额。在“仪表板”选项卡上，门户将显示若干配额的当前使用情况统计信息以及何时将重置这些配额。这些使用情况统计信息将因你所选模式而异。下面的屏幕快照显示“免费”模式的统计信息。在“共享”模式下，你不具有针对“输出数据量”的配额。在“标准”模式下，它将仅显示“文件系统存储”和“大小”。

![DigitalMarketingUsageOverview][DigitalMarketingUsageOverview]

如果你发现正接近于用尽这些配额，则考虑从“免费”迁移到“共享”，或者从“共享”迁移到“标准”。在“标准”模式下，你将在一个或多个小、中或大型虚拟机上具有专用资源。

<!-- In addition to using the Management Portal for this information, there are a number of third-party tools that provide advanced monitoring data for your web sites. For example, you can install a New Relic add-on from the Azure Store. For a walkthrough on using New Relic for monitoring, see [New Relic Application Performance Management on Azure][newrelic].  -->

最后，“标准”模式网站可以实现终结点监视和警报。终结点监视器定期尝试接入你的网站，然后报告响应时间。如果响应时间超出指定阈值，警报将提供电子邮件通知。有关更多信息，请参阅[全球网络影响力][查看该信息]方案的监视部分以及主题[如何：在 Azure 中接收警报通知和管理警报规则][如何：在 Azure 中接收警报通知和管理警报规则]。

## <a name="summary"></a>摘要

Azure 网站十分适合于为单独市场营销活动定制的可重复使用的 Web 内容。网站支持许多流行语言、框架和源代码管理系统，从而更便于将这些资产和工作流迁移到云。你可以使用支持 Web 接口的几乎任何第三方社交媒体集成。Azure 上的其他相关服务提供附加的工具，用于构建设计良好的活动网站。并且可以使用多种手动和自动缩放选项来处理用户需求，而这种需求是很难预测的。有关详细信息，请参阅以下技术文章。

<table cellspacing="0" border="1">
<tr>
<th align="left" valign="top">区域</th>
<th align="left" valign="top">资源</th>
</tr>
<tr>
<td valign="middle"><strong>计划</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/">Azure 网站、云服务和虚拟机：何时使用何种产品？</a></td>
</tr>
<tr>
<td valign="middle"><strong>创建</strong></td>
<td valign="top">- <a href="http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-create-websites/">如何创建和部署网站</a></td>
</tr>
<tr>
<td valign="middle"><strong>部署</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-deploy/">如何部署 Azure 网站</a><br/>- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-publish-source-control/">从源代码管理发布到 Azure 网站</a></td>
</tr>
<tr>
<td valign="middle"><strong>社交媒体</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/">使用成员资格、OAuth 和 SQL Database 部署安全的 ASP.NET MVC 应用程序</a><br/></td>
</tr>
<tr>
<td valign="middle"><strong>缩放</strong></td>
<td valign="top">- <a href="http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-scale-websites/">如何缩放网站</a></td>
</tr>

<!-- <tr>    <td valign="middle"><strong>Rich Media</strong></td>    <td valign="top">- <a href="http://msdn.microsoft.com/en-us/library/windowsazure/dn223282.aspx">Introduction to Azure Media Services Concepts and Scenarios</a></td> </tr>  -->
 
<tr>
<td valign="middle"><strong>监视</strong></td>
<td valign="top">- <a href="http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-monitor-websites/">如何监视网站</a><br/>- <a href="http://msdn.microsoft.com/library/windowsazure/dn306638.aspx">如何：在 Azure 中接收警报通知和管理警报规则</a></td>
</tr>
</table>
<!--   [twitter]:https://dev.twitter.com/docs/twitter-libraries#dotnet   [fbtutorial]:http://blogs.msdn.com/b/africaapps/archive/2013/02/20/creating-a-facebook-app-using-asp-net-mvc-facebook-templates-and-hosting-them-for-free-on-windows-azure-websites.aspx   [fbbirthdayapp]:http://www.asp.net/mvc/tutorials/mvc-4/aspnet-mvc-facebook-birthday-app   [fbvstemplate]:http://blogs.msdn.com/b/webdev/archive/2012/12/13/the-new-facebook-application-template-and-library-for-asp.net-mvc.aspx  -->

<!--[mediaservices]:http://msdn.microsoft.com/en-us/library/windowsazure/dn223282.aspx   [caching]:http://msdn.microsoft.com/en-us/library/windowsazure/dn386094.aspx   -->

<!--[DigitalMarketingFacebook]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_Facebook.png -->

  [Azure 网站]: /en-us/documentation/services/web-sites/
  [Azure 云服务]: /en-us/documentation/services/cloud-services/
  [Azure 虚拟机]: /en-us/documentation/services/virtual-machines/
  [Azure 网站、云服务和虚拟机：何时使用何种产品？]: /en-us/manage/services/web-sites/choose-web-app-service
  [部署现有网站]: #deployexisting
  [与社交媒体相集成]: #socialmedia
  [根据用户要求进行缩放]: #scale
  [与其他服务集成]: #integrate
  [监视活动]: #monitor
  [全球网络影响力]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-global-web-presence-solution-overview/
  [业务应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-business-application-solution-overview/
  [查看该信息]: /en-us/manage/services/web-sites/global-web-presence-solution-overview/
  [DigitalMarketingDeploy1]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_Deploy1.png
  [DigitalMarketingDeploy2]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_Deploy2.png
  [具有 Azure 网站的多个环境]: http://www.bradygaster.com/post/multiple-environments-with-windows-azure-web-sites
  [从源代码管理发布到 Azure 网站]: /en-us/develop/net/common-tasks/publishing-with-git/
  [DigitalMarketingFrameworkVersions]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_FrameworkVersions.png
  [使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用程序部署到 Azure 网站]: /en-us/develop/net/tutorials/web-site-with-sql-database/
  [Azure 管理门户]: http://manage.windowsazure.cn/
  [服务管理 API]: http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx
  [PowerShell 脚本编写]: http://msdn.microsoft.com/zh-cn/library/azure/jj152841.aspx
  [DigitalMarketingScale]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_Scale.png
  [网站定价准则]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [DigitalMarketingAutoScale]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_AutoScale.png
  [如何扩展网站]: /en-us/manage/services/web-sites/how-to-scale-websites/
  [如何缩放云服务]: http://www.windowsazure.cn/zh-cn/manage/services/cloud-services/how-to-scale-a-cloud-service/
  [DigitalMarketingMonitor]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_Monitor.png
  [如何监视网站]: /en-us/manage/services/web-sites/how-to-monitor-websites/
  [DigitalMarketingUsageOverview]: ./media/web-sites-digital-marketing-application-solution-overview/DigitalMarketing_UsageOverview.png
  [如何：在 Azure 中接收警报通知和管理警报规则]: http://msdn.microsoft.com/zh-cn/library/azure/dn306638.aspx
  [1]: http://azure.microsoft.com/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/
  [如何创建和部署网站]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-create-websites/
  [如何部署 Azure 网站]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-deploy/
  [2]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-publish-source-control/
  [使用成员资格、OAuth 和 SQL Database 部署安全的 ASP.NET MVC 应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/
  [如何缩放网站]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-scale-websites/
  [3]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-monitor-websites/
  [4]: http://msdn.microsoft.com/library/windowsazure/dn306638.aspx
