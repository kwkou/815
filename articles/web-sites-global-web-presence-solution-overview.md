<properties linkid="websites-global-web-presence" urlDisplayName="Create a Global Web Presence on Azure Web Sites" pageTitle="Create a Global Web Presence on Azure Web Sites" metaKeywords="" description="This guide provides a technical overview of how to host your organization's (.COM) site on Azure Web Sites. This includes deployment, custom domains, SSL, and monitoring." metaCanonical="http://www.windowsazure.com/en-us/documentation/articles/web-sites-global-web-presence-solution-overview/" services="" documentationCenter="" title="Create a Global Web Presence on Azure Web Sites" authors="jroth" solutions="" manager="paulettm" editor="mollybos" />

# 在 Azure 网站上打造全球网络影响力

本指南提供了如何在 Azure 上托管你的组织的 (.COM) 网站的技术概述。此方案也称作全球网络影响力。本指南重点介绍使用 [Azure 网站][Azure 网站]，因为网站是在 Azure 上创建、迁移、缩放和管理 Web 应用程序的最快和最简单方法。但是，某些应用程序要求适合于运行 IIS 的 [Azure 云服务][Azure 云服务]或 [Azure 虚拟机][Azure 虚拟机]。它们还是托管 Web 应用程序的极佳选择。如果你处于初始计划阶段，请查看文档 [Azure 网站、云服务和虚拟机：何时使用何种产品？][Azure 网站、云服务和虚拟机：何时使用何种产品？]。如果没有针对使用云服务或虚拟机的要求，我们建议使用网站来发挥你的全球网络影响力。本文的剩余部分将提供将网站用于此情形的指南。

在本指南中将针对以下方面：

-   [创建 Azure 网站][创建 Azure 网站]
-   [部署网站][部署网站]
-   [添加自定义域][添加自定义域]
-   [使用 SSL 保护网站][使用 SSL 保护网站]
-   [监视网站][监视网站]

<div class="dev-callout">
<strong>说明</strong>
<p>本指南展示为面向公众的 .COM 站点开发而进行调整的一些最常见领域和任务。但是，还有可在特定实施中使用的其他一些 Azure 网站功能。若要查看这些功能，另请参阅<a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-digital-marketing-application-solution-overview/">数字市场营销活动</a>和<a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-business-application-solution-overview/">业务应用程序</a>中的其他指南。</p>
</div>

## <a name="createwebsite"></a>创建 Azure 网站

使用 Azure 管理门户，你可以通过几种方式创建新的 Azure 网站。当你在门户的底部单击“新建”按钮时，将向你提供以下对话框：

![GlobalWebCreate][GlobalWebCreate]

有两个用于创建新网站的选项：“快速创建”和“自定义创建”。对于上述每个选项，你都应选择适合于你的主要用户群的 Azure 区域。

如果你在迁移现有网站，则“自定义创建”选项允许你创建或关联 SQL Database。此选项还提供为部署指定若干源代码管理选项的功能，例如 GitHub 或 Team Foundation Server (TFS)。如果你已在使用源代码管理机制管理你的网站，则这是一种针对部署设置 Azure 网站的快速方法。

<!-- The **From Gallery** option allows you to setup a new site with one of several frameworks, such as Drupal or WordPress. This can be helpful to quickly set up a new site that you can then customize within the chosen framework. -->

与 Azure 中的大多数服务相似，你必须为新网站选择 Azure 区域。Azure 在世界各地有多个区域。一旦你将你的网站部署到任何一个区域，就能够在 Internet 上从全球访问该网站。但是，多个区域提供更高的灵活性。一个明显的好处是在最接近用户的区域中部署网站。

有关创建新网站的步骤的详细信息，请参见 [Azure 网站和 ASP.NET 入门][Azure 网站和 ASP.NET 入门]。

## <a name="deploywebsite"></a>部署网站

有几种方法可以将你的网站部署到 Azure。如果你从库中选择了某一框架，就已经部署了入门网站。但是，若要进一步发展，你仍必须设置某种类型的编辑和部署过程。一些部署选项包括：

-   使用 FTP 客户端。
-   从源代码管理部署。
-   从 Visual Studio 发布。
-   从 [WebMatrix][WebMatrix] 发布。

上述每个选项都具有不同的能力。能够从 FTP 客户端进行发布是将新文件推送到你的网站的简单、直接的解决方案。它还意味着依赖于 FTP 的任何现有发布工具或进程都可以继续使用 Azure 网站。源代码管理提供对网站内容发布的最佳控制，因为可以跟踪和发布更改，以及根据需要将更改回滚到之前的版本。直接从 Visual Studio 或 Web Matrix 进行发布的选项为使用这两个工具之一的开发人员带来方便。针对该功能的一个有用的情形是在项目的早期阶段中或用于原型制作。在这两种情形下，频繁的发布和测试可能从开发环境内更方便。

这里的许多部署任务都涉及使用 Azure 管理门户中的信息。转到你的网站，选择“仪表板”选项卡，然后查看“速览”部分。下面的屏幕快照显示若干选项。

![GlobalWebQuickGlance][GlobalWebQuickGlance]

某些源代码管理工具和 FTP 客户端要求用户名/密码访问。对于新网站，不自动创建凭据。不过，你可以通过单击“重置部署凭据”轻松地创建凭据。在完成后，你可以通过在同一个“仪表板”页将这些凭据与“FTP 主机名”一起使用，使用任何 FTP 客户端部署你的网站。

![GlobalWebFTPSettings][GlobalWebFTPSettings]

请注意，部署/FTP 用户名是你提供的网站名称和用户名的组合。因此，如果你的网站是“<http://contoso.chinacloudsites.cn>”并且你的用户名是“myuser”，则部署和 FTP 的用户名将是“contoso\\myuser”。

还可以选择通过源代码管理服务（例如 GitHub 或 TFS Online）进行部署。单击针对“从源代码管理设置部署”的选项。然后，按照针对你选择的源代码管理系统或服务的说明执行。有关从本地 Git 存储库进行发布的分步说明，请参见[从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。

如果你计划使用 Visual Studio 创建和管理你的网站，则可以选择直接从 Visual Studio 发布。一个方法是单击“下载发布配置文件”选项。这允许你保存可导入到 Visual Studio 中以便进行 Web 发布的 publishsettings 文件。

<div class="dev-callout">
<strong>说明</strong>
<p>务必要确保 <i>publishsettings</i> 文件安全并且处于源代码管理之外，因为它包含针对部署以及任何链接的数据库连接字符串的用户名和密码。</p>
</div>

也可以将订阅信息直接导入到 Visual Studio 中。有关示例，请考虑 Visual Studio 中的本地 ASP.NET 项目。右键单击 Web 项目，然后选择“发布”。通过“发布 Web”对话框中的“导入”按钮，你可以导入包含你的 Azure 订阅设置的文件或者导入从网站仪表板下载的 publishsettings 文件。下面的屏幕快照显示这些选项。

![GlobalWebVSPublish][GlobalWebVSPublish]

有关从 Visual Studio 发布到 Azure 的更多信息，请参见“将 ASP.NET Web 应用程序部署到 Azure 网站”。

用于开发和部署的另一个选项是 Azure 管理门户中的 WebMatrix。

![GlobalWebWebMatrix][GlobalWebWebMatrix]

有关此选项的详细信息，请参阅[使用 Microsoft WebMatrix 开发和部署网站][使用 Microsoft WebMatrix 开发和部署网站]。

尽管这些步骤提供了你部署 .COM 网站所需项目，但还应为管理后续内容发布周期创建计划。这些选项可以涵盖从开始执行自定义解决方案到不频繁更改的网站的定期重新部署再到功能完备的内容管理系统 (CMS) 的方方面面。

## <a name="customdomain"></a>添加自定义域

如果需要在这里体现你的全球网络影响力，你需要将你的已注册域名与网站相关联。有许多提供域注册服务的第三方提供程序。其中的每个提供程序都支持创建不同类型的 DNS 记录以便管理你的域。DNS 记录有助于将用户友好的 URL（例如“www.contoso.com”）映射到托管网站的实际 URL 或 IP 地址。

<div class="dev-callout">
<strong>说明</strong>
<p>在下面的论述中，有两个令人感兴趣的 DNS 记录类型。首先，CNAME 记录可以从一个 URL（例如&ldquo;www.contoso.com&rdquo;）映射到其他 URL（例如&ldquo;contoso.chinacloudsites.cn&rdquo;）。其次，A 记录可以将 URL（例如&ldquo;www.contoso.com&rdquo;）映射到 IP 地址（例如 172.16.48.1）。</p>
</div>

对于 Azure 网站，你必须首先对 Azure 网站创建一个 CNAME 记录。该设置通过第三方注册机构的网站实现。以下是一个示例 CNAME 记录。
<table cellspacing="0" border="1">
<tr>
<th align="left" valign="top">类型</th>
<th align="left" valign="top">Host</th>
<th align="left" valign="top">Answer</th>
<th align="left" valign="top">TTL</th>
</tr>
<tr>
<td valign="top"><strong>CNAME</strong></td>
<td valign="top">www.contoso.com</td>
<td valign="top">contoso.chinacloudsites.cn</td>
   <td valign="top">8000</td>
</tr>
</table>

如果你的域是新注册的，该域可能会用一天或更长时间完成所有 DNS 服务器上的解析，对缓存的 DNS 条目进行操作。但是，如果你的域已存在，则 CNAME 更改应该会在一分钟内发生。请注意，该 CNAME 记录在你的域（必须用子域别名进行限定，例如“www”）和 Azure 网站 URL 之间提供映射。CNAME 记录的任何一侧都不包含“<http://>”前缀。

在 Azure 管理门户的“缩放”选项卡上确认你在“共享”或“标准”模式下运行（“免费”网站不支持自定义域）。然后转到“配置”选项卡，单击“管理域”按钮。这使你能够将网站域自定义域名相关联。

![GlobalWebWebMatrix][GlobalWebWebMatrix]

在列表中放置你的自定义域之前，必须首先转到你的 DNS 提供程序并且创建你的自定义域 (www.contoso.com) 的 CNAME 记录，该记录指向你的 Azure 网站的 URL (contoso.chinacloudsites.cn)。在此传播后，你可以在前面屏幕快照中显示的对话框中输入自定义域。对于指向此网站的 www.contoso.com 提供 CNAME 记录确保你有权将该域名用于此网站。此时，你可以在对话框的底部创建具有 IP 地址的 A 记录。

<table cellspacing="0" border="1">
<tr>
<th align="left" valign="top">类型</th>
<th align="left" valign="top">Host</th>
<th align="left" valign="top">Answer</th>
<th align="left" valign="top">TTL</th>
</tr>
<tr>
<td valign="top"><strong>A</strong></td>
<td valign="top">contoso.com</td>
   <td valign="top">172.16.48.1</td>
   <td valign="top">8000</td>
</tr>
</table>

有关更多信息，请参见[为 Azure 网站配置自定义域名][为 Azure 网站配置自定义域名]。

## <a name="ssl"></a>使用 SSL 保护网站

如果你的网站包含只读信息，则无需提供对该网站的安全访问。但是，如果你收集任何用户信息、执行电子商务或管理任何其他敏感数据，则应确保网站的安全。安全性是一个很大的主题，本文无法涵盖所有最佳实践和技术。但是，着重说明为你的网站实现安全套接字层 (SSL) 的过程是很重要的。SSL 允许用户以加密的方式使用 HTTPS 地址而非 HTTP 连接到你的网站。有一些将 SSL 用于 Azure 网站所需的特定步骤。

Azure 网站自动提供与实际网站 URL 的安全连接。例如，如果你的网站是 <http://contoso.chinacloudsites.cn>，则只需通过将“http”更改为“https”，例如 **https**://contoso.chinacloudsites.cn，就可以通过 SSL 进行连接。

但是，如果你在使用自定义域名，则必须采取措施通过网站的 Azure 管理门户上载证书和启用 SSL。下面的步骤提供此过程的摘要，但你可以在主题[为 Azure 网站配置 SSL 证书][为 Azure 网站配置 SSL 证书]中找到详细说明。

首先，从证书颁发机构获取一个 SSL 证书。如果你要确保具有多个子域（例如 www.contoso.com 和 staging.contoso.com）的域的安全，你将需要获取通配符证书 (\*.contoso.com)。它们可能会导致更高成本，因此，你必须确定此类型证书的灵活性是否值得付出该成本。

一旦从证书颁发机构获取证书后，就不能简单地以相同方式将其上载到 Azure。你必须使用 openssl 命令生成 .pfx 文件。该 openssl 命令是 OpenSSL 项目的一部分。这些源在 [OpenSSL 网站][OpenSSL 网站]上分发，但你通常也可以在 Internet 上找到该工具的预先编译的版本。在下面的示例中，使用证书 myserver.crt 和私钥文件 myserver.key 创建了一个 .pfx 文件。

    openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt

若要将该证书上载到 Azure，请首先转到“缩放”选项卡，确认你正在“标准”模式下运行。“免费”或“共享”模式不支持针对自定义域的 SSL。在“配置”选项卡上，选择“上载证书”按钮。

![GlobalWebUplodateCert][GlobalWebUplodateCert]

然后在“ssl 绑定”部分中，将证书映射到它保护的域名。有两个用于此映射的选项：SNI SSL 和基于 IP 的 SSL。

![GlobalWebSSLBindings][GlobalWebSSLBindings]

“基于 IP 的 SSL”选项是用于将公共的专用 IP 地址映射到域名的传统方法。它适用于所有浏览器。“SNI SSL”选项允许多个域共享相同 IP 地址，但对每个域具有不同的关联 SSL 证书。SNI SSL 不适用于某些较旧的浏览器（有关兼容性的更多信息，请参见[针对 SNI SSL 的 Wikipedia 条目][针对 SNI SSL 的 Wikipedia 条目]）。存在与每个 SSL 证书相关联的每月费用（每小时按比例分摊），并且定价因选择的是基于 IP 的 SSL 还是 SNI SSL 而异。有关定价信息，请参见[网站定价详细信息][网站定价详细信息]。有关此过程的详细信息，请参阅[为 Azure 网站配置 SSL 证书][为 Azure 网站配置 SSL 证书]。

## <a name="monitor"></a>监视网站

在你的网站主动处理用户请求后，使用监视十分重要。例如，你可能想要知道用户负荷是否正在导致高 CPU 时间，这可能指示需要扩展网站。或者应用程序效率低下可能增加了响应时间或导致错误。本节介绍一些 Azure 管理门户上的某些内置监视功能。

“监视器”选项卡以图形格式包含针对你的网站的某些关键度量值。

![GlobalWebMonitor1][GlobalWebMonitor1]

你可以使用“添加度量值”按钮自定义此图形中的度量值。

![GlobalWebMonitor2][GlobalWebMonitor2]

对于在“标准”模式下运行的网站，你还可以启用终结点监视和警报。在“配置”选项卡上，转到“监视”部分，并且配置终结点。此终结点从你指定的一个或多个位置运行，并且定期尝试访问你的网站。时间和错误信息都会收集。

在“监视器”选项卡中，此终结点在出现时会显示响应时间。如果你选择终结点度量值，则可以通过单击“添加规则”图标，添加警报规则。

![GlobalWebMonitor3][GlobalWebMonitor3]

在响应时间超出了指定的阈值时，该规则可通过电子邮件通知管理员或其他个体。

![GlobalWebMonitor4][GlobalWebMonitor4]

如果你发现网站要求缩放，则可以在“缩放”选项卡上手动执行缩放，或者通过自动缩放预览执行缩放。该“缩放”选项卡提供向上扩展（更大的专用计算机）或向外扩展（相同大小的附加共享实例或专用实例）的选择。但是，自动缩放预览仅支持向外扩展。有关此选项的更详细信息，请参见有关网站监视的更多信息，以及参见[数字市场营销活动][1]方案的“根据用户要求进行缩放”部分。另请参见[如何监视网站][如何监视网站]。

## <a name="summary"></a>摘要

若要创建你的组织的 (.COM) 网站，标准任务包括选择开发框架、网站创建、部署、自定义域分配和监视。对于必须确保用户数据安全的网站，高度推荐 SSL。本文概述了如何使用 Azure 网站执行这些任务。有关详细信息，请参阅本文引用的以下技术文章。

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
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-get-started/">Azure 网站和 ASP.NET 入门</a></td>
</tr>
<tr>
<td valign="middle"><strong>部署</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-publish-source-control/">从源代码管理发布到 Azure 网站</a><br/>- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-get-started/">将 ASP.NET Web 应用程序部署到 Azure 网站</a><br/>- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-using-webmatrix/">使用 Microsoft WebMatrix 开发和部署网站</a></td>
</tr>
<tr>
<td valign="middle"><strong>自定义域</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-custom-domain-name/">为 Azure 网站配置自定义域名</a></td>
</tr>
<tr>
<td valign="middle"><strong>SSL</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-configure-ssl-certificate/">为 Azure 网站配置 SSL 证书</a></td>
</tr>
<tr>
<td valign="middle"><strong>监视</strong></td>
<td valign="top">- <a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-monitor/">如何监视网站</a></td>
</tr>
</table>

<!--   [drupal]:https://drupal.org/   [umbraco]:http://umbraco.com/  -->

  [Azure 网站]: /en-us/documentation/services/web-sites/
  [Azure 云服务]: /en-us/documentation/services/cloud-services/
  [Azure 虚拟机]: /en-us/documentation/services/virtual-machines/
  [Azure 网站、云服务和虚拟机：何时使用何种产品？]: /en-us/manage/services/web-sites/choose-web-app-service
  [创建 Azure 网站]: #createwebsite
  [部署网站]: #deploywebsite
  [添加自定义域]: #customdomain
  [使用 SSL 保护网站]: #ssl
  [监视网站]: #monitor
  [数字市场营销活动]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-digital-marketing-application-solution-overview/
  [业务应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-business-application-solution-overview/
  [GlobalWebCreate]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_Create.png
  [Azure 网站和 ASP.NET 入门]: /en-us/documentation/articles/web-sites-dotnet-get-started
  [WebMatrix]: http://www.microsoft.com/web/webmatrix/
  [GlobalWebQuickGlance]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_QuickGlance.png
  [GlobalWebFTPSettings]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_FTPSettings.png
  [从源代码管理发布到 Azure 网站]: /en-us/develop/net/common-tasks/publishing-with-git/
  [GlobalWebVSPublish]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_VS_Publish.png
  [GlobalWebWebMatrix]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_WebMatrix.png
  [使用 Microsoft WebMatrix 开发和部署网站]: /en-us/develop/net/tutorials/get-started/
  [为 Azure 网站配置自定义域名]: /en-us/develop/net/common-tasks/custom-dns-web-site/
  [为 Azure 网站配置 SSL 证书]: /en-us/develop/net/common-tasks/enable-ssl-web-site/
  [OpenSSL 网站]: http://www.openssl.org/
  [GlobalWebUplodateCert]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_Uplodate_Cert.png
  [GlobalWebSSLBindings]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_SSL_Bindings.png
  [针对 SNI SSL 的 Wikipedia 条目]: http://en.wikipedia.org/wiki/Server_Name_Indication
  [网站定价详细信息]: /en-us/pricing/details/web-sites/#service-ssl
  [GlobalWebMonitor1]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_Monitor1.png
  [GlobalWebMonitor2]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_Monitor2.png
  [GlobalWebMonitor3]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_Monitor3.png
  [GlobalWebMonitor4]: ./media/web-sites-global-web-presence-solution-overview/GlobalWeb_Monitor4.png
  [1]: /en-us/manage/services/web-sites/digital-marketing-campaign-solution-overview
  [如何监视网站]: /en-us/manage/services/web-sites/how-to-monitor-websites/
  [2]: http://azure.microsoft.com/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/
  [3]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-get-started/
  [4]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-publish-source-control/
  [5]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-using-webmatrix/
  [6]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-custom-domain-name/
  [7]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-configure-ssl-certificate/
  [8]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-monitor/
