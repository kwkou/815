<properties linkid="develop-net-how-to-guides-new-relic" urlDisplayName="New Relic" pageTitle="Using New Relic with Azure - Azure feature guides" metaKeywords="" description="Learn how to use the New Relic service to manage and monitor your Azure application." metaCanonical="" services="" documentationCenter=".NET" title="New Relic Application Performance Management on Azure" authors="" solutions="" manager="" editor="" />

# Azure 中的 New Relic 应用程序性能管理

本指南介绍如何向 Azure 托管应用程序中添加 New Relic 的一流性能监视。我们将介绍向你的应用程序中添加 New Relic 的快速且简单的过程，并向你介绍 New Relic 的一些功能。有关使用 New Relic 的更多信息，请参阅[使用 New Relic][使用 New Relic]。

## 什么是 New Relic？

New Relic 是以开发人员为主的工具，用于监视生产应用程序，并使用户能够深入了解应用程序的性能和可靠性。其目的在于缩短你识别和诊断性能问题的时间，并使你能够轻松获得解决这些问题所需的信息。

New Relic 可从服务器和你的用户的浏览器跟踪你的 Web 事务的加载时间和吞吐量。它可显示你在数据库方面花费的时间，分析缓慢查询和 Web 请求，提供运行时间监视和通知，跟踪应用程序异常，还有其他更多功能。

## Azure 应用商店中的 New Relic 特价优惠

New Relic Standard 可供 Azure 用户免费使用，New Relic Pro 基于 Azure 云服务的实例大小提供

有关定价信息，请参阅 [Azure 应用商店中的 New Relic 页][Azure 应用商店中的 New Relic 页]。

<div class="dev-callout"> 
<strong>注意：</strong>
<p>只列出了 10 个以下的计算实例的价格。如果计数大于 10，请联系 New Relic (sales@newrelic.com) 获取批量定价。</p>
</div>

Azure 客户在部署 New Relic 代理时可获得一个为期两周的 New Relic Pro 试用订阅。

## 使用 Azure 应用商店注册 New Relic

New Relic 可与 Azure Web 角色和辅助角色无缝集成。

若要直接从 Azure 应用商店注册 New Relic，请按照以下三个简单步骤操作。

### 步骤 1. 通过 Azure 应用商店进行注册

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在该管理门户的下方窗格中，单击“新建”。
3.  单击“应用商店”。
4.  在“选择外接程序”对话框中，选择“New Relic”，然后单击“下一步”。
5.  在“个性化外接程序”对话框中，选择你需要的 New Relic 计划。
6.  输入促销代码（如果适用）。
7.  输入 New Relic 服务将在你的 Azure 设置中显示的
    名称，或者使用默认值“NewRelic”。此名称在你已订阅的 Azure 应用
    商店项目列表中必须是唯一的。
8.  选择区域值；例如，“美国西部”。
9.  单击“下一步”。
10. 在“查看购买”对话框中，查看计划和定价信息
    以及法律条款。如果你同意这些条款，请单击“购买”。
11. 单击“购买”后，你的 New Relic 将开始创建过程。你可以在 Azure 管理门户中监视状态。
12. 若要检索你的 New Relic 许可证密钥，请单击“输出值”。
13. 复制显示的许可证密钥。安装 New Relic Nuget 程序包时，你将需要输入该密钥。

### 步骤 2. 安装 Nuget 程序包

1.  打开你的 Visual Studio 解决方案，或者通过选择“文件”\>“新建”\>“项目”创建一个新的解决方案。

    ![Visual Studio][Visual Studio]

2.  如果你的解决方案中尚不具有 Azure 云服务项目，则通过在解决方案资源管理器中右键单击你的应用程序并选择“添加 Azure 云服务项目”来添加一个。

    ![创建云服务][创建云服务]

3.  通过选择“工具”\>“库程序包管理器”\>“程序包管理器控制台”****打开程序包管理器控制台。在“程序包管理器控制台”窗口顶部，将你的项目
    设置为默认项目。

    ![程序包管理器控制台][程序包管理器控制台]

4.  在程序包管理器命令提示符处，键入`Install-Package    NewRelicWindowsAzure` 然后按 **Enter** 键。

    ![在程序包管理器中安装][在程序包管理器中安装]

5.  在许可证密钥提示符处，输入你从 Azure 应用商店获取的许可证密钥。

    ![输入许可证密钥][输入许可证密钥]

6.  可选：在应用程序名称提示符处，输入将显示在 New Relic 仪表板中的你的应用程序的名称。或者，将你的解决方案名称用作默认值。

    ![输入应用程序名称][输入应用程序名称]

7.  从“解决方案资源管理器”中，右键单击你的 Azure 云服务项目，并选择“发布”。

    ![发布云项目][发布云项目]

**注意：**如果这是你第一次将此应用程序部署到 Azure，系统将提示你输入你的 Azure 凭据。有关更多信息，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站][将 ASP.NET Web 应用程序部署到 Azure 网站]。

![发布设置][发布设置]

### 步骤 3. 在 New Relic 中查看你的应用程序的性能。

查看 New Relic 仪表板：

1.  从 Azure 门户中，单击“管理”按钮。
2.  使用你的 New Relic 帐户电子邮件地址和密码进行登录。
3.  从 New Relic 菜单栏中，选择“Applications”（应用程序）\>“（应用程序名称）”。

    将自动显示“Monitoring”（监视）\>“Overview”（概览）仪表板。

    ![New Relic Monitorin 仪表板][New Relic Monitorin 仪表板]

    从你的“Applications”（应用程序）菜单上的列表中选择应用程序后，“Overview”（概览）仪表板将显示当前的应用程序服务器和浏览器信息。

### <span id="using-new-relic"></span></a>使用 New Relic

从“Overview”（概览）菜单上的列表中选择你的应用程序后，“Overview”（概览）仪表板将显示当前的应用程序服务器和浏览器信息。若要在两个视图之间切换，请单击“App server”（应用程序服务器）或“Browser”（浏览器）按钮。

除了 [standard New Relic UI][standard New Relic UI] 和 [dashboard drill-down][dashboard drill-down] 功能之外，Applications Overview（应用程序概览）仪表板还具有其他一些功能。

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><strong>如果你需要…</strong></th>
<th align="left"><strong>采取的措施：</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">为所选应用程序服务器或浏览器显示仪表板信息</td>
<td align="left">单击“App server”（应用程序服务器）或“Browser”（浏览器）按钮。<strong></strong></td>
</tr>
<tr class="even">
<td align="left">查看你的应用程序的 <a href="https://newrelic.com/docs/site/apdex">Apdex</a> 得分的阈值级别</td>
<td align="left">指向“Apdex score?”（Apdex 得分?）图标。<strong><strong></strong></strong></td>
</tr>
<tr class="odd">
<td align="left">查看 Worldwide Apdex 详细信息</td>
<td align="left">从“Overview”（概览）的“Browser”（浏览器）视图中，指向“Global Apdex”图上的任何位置。<strong></strong><br /><strong>提示</strong>：若要直接转到所选应用程序的 <a href="https://docs.newrelic.com/docs/new-relic-browser/geography-dashboard">Geography</a> 仪表板，请单击“Global Apdex”标题，或单击“Global Apdex”图上的任何位置。<strong></strong></td>
</tr>
<tr class="even">
<td align="left">查看 <a href="https://newrelic.com/docs/applications-dashboards/web-transactions">Web Transactions</a> 仪表板</td>
<td align="left">单击“Applications Overview”（应用程序概览）仪表板上的“Web Transactions”（Web 事务）表。或者，若要查看有关特定 Web 事务（包括 <a href="https://newrelic.com/docs/site/key-transactions">Key Transactions</a>）的更多信息，请单击其名称。</td>
</tr>
<tr class="odd">
<td align="left">查看 <a href="https://newrelic.com/docs/site/errors">Errors</a>仪表板</td>
<td align="left">单击“Applications Overview”（应用程序概览）仪表板上的“Error rate”（错误率）图表的标题。<br /><strong>提示</strong>：你还可以从“应用程序”&gt;“（你的应用程序）”&gt;“事件”&gt;“错误”中查看“错误”仪表板。<strong></strong></td>
</tr>
<tr class="even">
<td align="left">查看应用程序服务器详细信息</td>
<td align="left"><p>执行以下任何操作：</p>
<p></p>
<ul>
<li>在多个主机的表视图或每个主机的分类度量值详细信息之间进行切换。</li>
<li>单击单个服务器的名称。</li>
<li>指向单个服务器的“Apdex score”（Apdex 得分）。</li>
<li>单击单个服务器的“CPU usage”（CPU 使用率）或“Memory”（内存）。</li>
</ul>
</p>
</p></td>
</tr>
</tbody>
</table>

下面是当选择“Browser”（浏览器）视图时“Applications Overview”（应用程序概览）仪表板的示例。

![程序包管理器控制台][1]

## 后续步骤

有关更多信息，请查看以下其他资源：

-   [在 Azure 上安装 .NET 代理（可能为英文页面）][在 Azure 上安装 .NET 代理（可能为英文页面）]：New Relic .NET 代理安装过程
-   [New Relic 用户界面][New Relic 用户界面]：
    大致介绍 New Relic UI、设置用户权限和配置文件、使用标准功能和仪表板向下钻取详细信息
-   [应用程序概览（可能为英文页面）][应用程序概览（可能为英文页面）]：使用 New Relic 的“Applications Overview”仪表板时的特性和功能
-   [Apdex][Apdex]：大致介绍 Apdex 如何衡量最终用户对你的应用程序的满意度
-   [实际用户监视（可能为英文页面）][实际用户监视（可能为英文页面）]：大致介绍 RUM 如何详细记录你的用户的浏览器加载你的
    网页所需的时间、这些用户所在的位置以及他们使用的浏览器
-   [寻求帮助（可能为英文页面）][寻求帮助（可能为英文页面）]：通过 New Relic 的联机帮助中心提供的资源

  [使用 New Relic]: #using-new-relic
  [Azure 应用商店中的 New Relic 页]: http://www.windowsazure.com/zh-cn/gallery/store/new-relic/new-relic/
  [Azure 管理门户]: https://manage.windowsazure.cn
  [Visual Studio]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget01.png
  [创建云服务]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget02.png
  [程序包管理器控制台]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget04.png
  [在程序包管理器中安装]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget06.png
  [输入许可证密钥]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget07.png
  [输入应用程序名称]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget08.png
  [发布云项目]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget09.png
  [将 ASP.NET Web 应用程序部署到 Azure 网站]: /zh-cn/develop/net/tutorials/get-started/
  [发布设置]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelicAzureNuget10.png
  [New Relic Monitorin 仪表板]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelic_app.png
  [standard New Relic UI]: https://newrelic.com/docs/site/the-new-relic-ui#functions
  [dashboard drill-down]: https://newrelic.com/docs/site/the-new-relic-ui#drilldown
  [Apdex]: https://newrelic.com/docs/site/apdex
  [Geography]: https://docs.newrelic.com/docs/new-relic-browser/geography-dashboard
  [Web Transactions]: https://newrelic.com/docs/applications-dashboards/web-transactions
  [Key Transactions]: https://newrelic.com/docs/site/key-transactions
  [Errors]: https://newrelic.com/docs/site/errors
  [1]: ./media/store-new-relic-cloud-services-dotnet-application-performce-management/NewRelic_app_browser.png
  [在 Azure 上安装 .NET 代理（可能为英文页面）]: https://newrelic.com/docs/dotnet/installing-the-net-agent-on-azure
  [New Relic 用户界面]: https://newrelic.com/docs/site/the-new-relic-ui
  [应用程序概览（可能为英文页面）]: https://newrelic.com/docs/site/applications-overview
  [实际用户监视（可能为英文页面）]: https://newrelic.com/docs/features/real-user-monitoring
  [寻求帮助（可能为英文页面）]: https://newrelic.com/docs/site/finding-help
