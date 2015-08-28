<properties 
	pageTitle="在 Azure App Service 中的 .NET Web 应用上使用 New Relic 应用程序性能管理" 
	description="了解如何对 Azure App Service 上运行的 ASP.NET 应用程序使用 New Relic 的性能监视。" 
	services="app-service\web" 
	documentationCenter=".net" 
	authors="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 	
	ms.date="04/17/2015" 
	wacn.date=""/>



# 在 Azure App Service 中的 .NET Web 应用上使用 New Relic 应用程序性能管理

本指南介绍如何向 [Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714) 中的 Web 应用添加 New Relic 的一流性能监视。我们将介绍向您的应用程序中添加 New Relic 的快速且简单的过程，并向您介绍 New Relic 的一些功能。有关使用 New Relic 的更多信息，请参阅[使用 New Relic](#using-new-relic)。

## 什么是 New Relic？

New Relic 是以开发人员为主的工具，用于监视生产应用程序，并使用户能够深入了解应用程序的性能和可靠性。其目的在于缩短您识别和诊断性能问题的时间，并使您能够轻松获得解决这些问题所需的信息。

New Relic 可从服务器和您的用户的浏览器跟踪您的 Web 事务的加载时间和吞吐量。它可显示您在数据库方面花费的时间，分析缓慢查询和 Web 请求，提供运行时间监视和通知，跟踪应用程序异常，还有其他更多功能。

## Azure 应用商店中的 New Relic 特价优惠

New Relic Standard 可供 Azure 用户免费使用。New Relic Pro 基于你所使用的网站模式和实例大小（如果你使用的是专属模式）在多个包中提供。

<!--For pricing information see the [New Relic page in the Azure Marketplace](/marketplace/partners/newrelic/newrelic).-->

> [AZURE.NOTE]只列出了 10 个以下的计算实例的价格。如果计数大于 10，请联系 New Relic (sales@newrelic.com) 获取批量定价。

Azure 客户在部署 New Relic 代理时可获得一个为期两周的 New Relic Pro 试用订阅。

使用 Azure 应用商店注册 New Relic 
--

New Relic 可与 Azure Web 角色、辅助角色和 Azure App Service 无缝集成。

若要直接从 Azure 应用商店注册 New Relic，请按照以下四个简单步骤操作。

## 步骤 1.创建 New Relic 帐户

1. 登录到 [Azure 门户](https://manage.windowsazure.cn)，然后单击左下角的“新建”。
3. 单击“开发人员服务”>“New Relic APM”。
4. 指定以下信息来配置 New Relic 帐户，然后单击“创建”。
	- **Name**
	- **定价层**
	- **资源组**
	- **订阅**
	- **位置**
	- **法律条款**

11. 单击“创建”之后，New Relic 帐户的创建过程就会开始。你可以单击“通知”按钮来监视状态。创建后，即会显示 New Relic 帐户的边栏选项卡。

12. 若要检索你的 New Relic 许可密钥，请查看边栏选项卡顶部的“Essentials”面板。将 Web 应用与 New Relic 帐户集成时，Web Apps 实例将在其应用设置中自动注册此许可密钥。

## 步骤 2：配置 Web 应用的 New Relic 集成

2. 在 [Azure 门户](https://manage.windowsazure.cn)中，打开 Web 应用的边栏选项卡。
3. 单击“应用程序监视”>“New Relic”。选择你在前一个步骤中创建的文件，然后单击“确定”。 

	![](./media/store-new-relic-web-sites-dotnet-application-performance-management/configure-new-relic-integration.png)

	完成保存操作后，在 Web 应用的边栏选项卡中单击“所有设置”，然后单击“应用程序设置”。你应会看到 **NEWRELIC_LICENSEKEY** 设置已添加到边栏选项卡的“应用设置”部分，以支持 New Relic：

	>[AZURE.NOTE]新的应用设置最多可能需要 30 秒的时间才能生效。若要强制使设置立即生效，请重新启动 Web 应用。

## 步骤 3：发布 ASP.NET Web 应用

使用 Visual Studio 或 WebMatrix 发布 Web 应用。如果你先前已发布 Web 应用，请再次发布以便 Web Apps 实例添加所需的 New Relic NuGet 包用于启用 New Relic 监视。

## 步骤 4.在 New Relic 中检查应用程序的性能。

查看 New Relic 仪表板：

2. 在 [Azure 门户](https://manage.windowsazure.cn)中，打开 Web 应用的边栏选项卡。
3. 单击“应用程序监视”>“应用程序名称”>“在 New Relic 查看”。

	![](./media/store-new-relic-web-sites-dotnet-application-performance-management/view-new-relic-data.png)

3. 如果这是你首次使用帐户，请配置你的帐户信息。
3. 从 New Relic 菜单栏中，选择“应用程序”>“（应用程序名称）”。

	将自动显示“监视”>“概览”仪表板。

	![New Relic Monitorin 仪表板](./media/store-new-relic-web-sites-dotnet-application-performance-management/NewRelic_app.png)

	从你的“应用程序”菜单上的列表中选择应用程序后，“概览”仪表板将显示当前的应用程序服务器和浏览器信息。

### <a id="using-new-relic"></a>使用 New Relic

从“概览”菜单上的列表中选择你的应用程序后，“概览”仪表板将显示当前的应用程序服务器和浏览器信息。若要在两个视图之间切换，请单击“应用程序服务器”或“浏览器”按钮。

除了 <a href="https://newrelic.com/docs/site/the-new-relic-ui#functions">standard New Relic UI</a> 和 <a href="https://newrelic.com/docs/site/the-new-relic-ui#drilldown">dashboard drill-down</a> 功能之外，“应用程序概览”仪表板还具有其他一些功能。

<table border="1">
  <thead>
    <tr>
      <th><b>如果你需要…</b></th>
      <th><b>采取的措施：</b></th>
    </tr>
  </thead>
  <tbody>
    <tr>
       <td>为所选应用程序服务器或浏览器显示仪表板信息</td>
       <td>单击“应用程序服务器”或“浏览器”按钮<b></b><b></b>。</td>
    </tr>
     <tr>
       <td>查看你的应用程序的 <a href="https://newrelic.com/docs/site/apdex" target="_blank">Apdex</a> 得分的阈值级别</td>
       <td>指向“Apdex 得分<b>?<b>”图标。</b></b></td>
    </tr>
    <tr>
       <td>查看 Worldwide Apdex 详细信息</td>
       <td>从“概览”的“浏览器”视图中，指向“Global Apdex”图上的任何位置<b></b>。<br /><b>提示：</b>若要直接转到所选应用程序的<a href="https://newrelic.com/docs/site/geography" target="_blank">“地理”</a> 仪表板，请单击“全球 Apdex”标题，或单击“全球 Apdex”图上的任何位置<b></b>。</td>
    </tr>
    <tr>
       <td>查看<a href="https://docs.newrelic.com/docs/applications-menu/transactions-dashboard" target="_blank">“Web 事务”</a>仪表板</td>
       <td>单击“应用程序概览”仪表板上的“Web 事务”表。或者，若要查看有关特定 Web 事务（包括<a href="https://newrelic.com/docs/site/key-transactions" target="_blank">“关键事务”</a>）的详细信息，请单击其名称。</td>
    </tr>
    <tr>
       <td>查看<a href="https://newrelic.com/docs/site/errors" target="_blank">“错误”</a>仪表板</td>
       <td>单击“应用程序概览”仪表板上的“错误率”图表的标题。<br /><b>提示：</b>你还可以从“应用程序”>“（你的应用）”>“事件”>“错误”中查看“错误”仪表板<b></b>。</td>
    </tr>
    <tr>
       <td>查看应用程序服务器详细信息</td>
       <td><p>执行以下任何操作：<p>
        <ul>
          <li>在多个主机的表视图或每个主机的分类度量值详细信息之间进行切换。</li>
          <li>单击单个服务器的名称。</li>
          <li>指向单个服务器的“Apdex score”（Apdex 得分）。</li>
          <li>单击单个服务器的“CPU usage”（CPU 使用率）或“Memory”（内存）。</li>
        </ul>
       </p></p></td>
    </tr>
  </tbody>
</table>

下面是当选择“Browser”（浏览器）视图时“Applications Overview”（应用程序概览）仪表板的示例。

![程序包管理器控制台](./media/store-new-relic-web-sites-dotnet-application-performance-management/NewRelic_app_browser.png)

## 后续步骤

有关更多信息，请查看以下其他资源：

 * [为 Azure 网站安装 .NET 代理](https://docs.newrelic.com/docs/agents/net-agent/azure-installation/azure-websites#manual)：New Relic .NET 代理安装过程 
 * [New Relic 用户界面](https://newrelic.com/docs/site/the-new-relic-ui)：概述 New Relic UI、设置用户权限和配置文件、使用标准功能和仪表板向下钻取详细信息
 * [应用程序概览](https://newrelic.com/docs/site/applications-overview)：使用 New Relic 的“应用程序概览”仪表板时的特性和功能
 * [Apdex](https://newrelic.com/docs/site/apdex)：概述 Apdex 如何衡量最终用户对你的应用程序的满意度
 * [实时用户监视](https://newrelic.com/docs/features/real-user-monitoring)：概述 RUM 如何详细记录你的用户的浏览器加载你的网页所需的时间、这些用户所在的位置以及他们使用的浏览器
 * [查找帮助](https://newrelic.com/docs/site/finding-help)：通过 New Relic 的联机帮助中心提供的资源

>[AZURE.NOTE]如果你想要在注册帐户之前开始使用 Azure App Service，请转到[“试用 App Service”](http://go.microsoft.com/fwlink/?LinkId=523751)，你可以通过该网站在 App Service 中创建一个生存期较短的入门站点。你不需要使用信用卡，也不需要做出承诺。

## 发生的更改
* 有关从网站更改为 App Service 的指南，请参阅：[Azure App Service 及其对现有 Azure 服务的影响](http://go.microsoft.com/fwlink/?LinkId=529714)
* 有关从 Azure 门户更改为 Azure 预览门户的指南，请参阅：[有关在预览门户中导航的参考](http://go.microsoft.com/fwlink/?LinkId=529715)


[webmatrixwebsite]: /documentation/articles/web-sites-dotnet-using-webmatrix
[vswebsite]: /documentation/articles/web-sites-dotnet-get-started

[wmnugetbutton]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrwmnugetbutton.png
[wmnugetgallery]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrwmnugetgallery.png

[newrelicconf]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrwmlicensekey.png
[vslicensekey]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrvslicensekey.png
[add-on]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nraddon.png
[custom]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrcustom.png
 

<!---HONumber=66-->