<properties linkid="develop-net-how-to-guides-new-relic-app" urlDisplayName="New Relic App Performance Management" pageTitle="New Relic App Performance Management on Azure" metaKeywords="new relic Azure, performance azure" description="Learn how to use New Relic's performance monitoring on Azure." metaCanonical="" services="web-sites" documentationCenter=".NET" title="New Relic Application Performance Management on Azure Web Sites" authors="larryfr" solutions="" manager="" editor="" />

# Azure 网站中的 New Relic 应用程序性能管理

本指南介绍如何向 Azure 网站添加 New Relic 的一流性能监视。我们将介绍向你的应用程序中添加 New Relic 的快速且简单的过程，并向你介绍 New Relic 的一些功能。有关使用 New Relic 的更多信息，请参阅[使用 New Relic][使用 New Relic]。

## 什么是 New Relic？

New Relic 是以开发人员为主的工具，用于监视生产应用程序，并使用户能够深入了解应用程序的性能和可靠性。其目的在于缩短你识别和诊断性能问题的时间，并使你能够轻松获得解决这些问题所需的信息。

New Relic 可从服务器和你的用户的浏览器跟踪你的 Web 事务的加载时间和吞吐量。它可显示你在数据库方面花费的时间，分析缓慢查询和 Web 请求，提供运行时间监视和通知，跟踪应用程序异常，还有其他更多功能。

## Azure 应用商店中的 New Relic 特价优惠

New Relic Standard 免费供 Azure 用户使用。
New Relic Pro 基于你所使用的网站模式和实例大小（如果你使用的是专属模式）在多个包中提供。

有关定价信息，请参阅 [Azure 应用商店中的 New Relic 页][Azure 应用商店中的 New Relic 页]。

<div class="dev-callout"> 
<strong>注意：</strong>
<p>只列出了 10 个以下的计算实例的价格。如果计数大于 10，请联系 New Relic (sales@newrelic.com) 获取批量定价。</p>
</div>

Azure 客户在部署 New Relic 代理时可获得一个为期两周的 New Relic Pro 试用订阅。

## 使用 Azure 应用商店注册 New Relic

New Relic 可与 Azure Web 角色、辅助角色和网站无缝集成。

若要直接从 Azure 应用商店注册 New Relic，请按照以下四个简单步骤操作。

### 步骤 1. 通过 Azure 应用商店进行注册

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在该管理门户的下方窗格中，单击“新建”。
3.  单击“应用商店”。
4.  在“选择外接程序”对话框中，选择“New Relic”，然后单击“下一步”。
5.  在“个性化外接程序”对话框中，选择你需要的 New Relic 计划。
6.  输入 New Relic 服务将在你的 Azure 设置中显示的
    名称，或者使用默认值“NewRelic”。此名称在你已订阅的 Azure 应用
    商店项目列表中必须是唯一的。
7.  选择区域值；例如，“美国西部”。
8.  单击“下一步”。
9.  在“查看购买”对话框中，查看计划和定价信息
    以及法律条款。如果你同意这些条款，请单击“购买”。
10. 单击“购买”后，你的 New Relic 将开始创建过程。你可以在 Azure 管理门户中监视状态。
11. 若要检索你的 New Relic 许可证密钥，请单击你刚刚创建的外接程序，然后单击“连接信息”。
12. 复制显示的许可证密钥。安装 New Relic Nuget 程序包时，你将需要输入该密钥。

### 步骤 2. 安装 New Relic 程序包

New Relic 网站代理作为 NuGet 程序包分发，可使用 Visual Studio 或 WebMatrix 将其添加到你的网站。如果你不熟悉如何将 Visual Studio 或 WebMatrix 用于 Azure 网站，请参阅[使用 Visual Studio 将 ASP.NET Web 应用程序部署到 Azure 网站（可能为英文页面）][使用 Visual Studio 将 ASP.NET Web 应用程序部署到 Azure 网站（可能为英文页面）]或[使用 Microsoft WebMatrix 开发和部署网站（可能为英文页面）][使用 Microsoft WebMatrix 开发和部署网站（可能为英文页面）]。

为你正在使用的特定开发环境执行下列步骤：

**Visual Studio**

1.  打开 Visual Studio 网站解决方案。

2.  通过选择“工具”\>“库程序包管理器”\>“程序包管理器控制台” 打开程序包管理器控制台。在“程序包管理器控制台”窗口顶部，将你的项目
    设置为默认项目。

    ![程序包管理器控制台][程序包管理器控制台]

3.  在程序包管理器命令提示符处，使用以下命令安装程序包：

        Install-Package NewRelic.Azure.WebSites

4.  在许可证密钥提示符处，输入你从 Azure 应用商店获取的许可证密钥。

    ![输入许可证密钥][输入许可证密钥]

<!--5. Optional: At the application name prompt, enter your app's name as it will    appear in New Relic's dashboard. Or, use your solution name as the default.      ![enter application name](./media/store-new-relic-web-sites-dotnet-application-performce-management/NewRelicAzureNuget08.png)-->

**WebMatrix**

1.  使用 WebMatrix 打开网站。

2.  在功能区的“主页”选项卡上，选择“NuGet”。

    ![“主页”选项卡上的“NuGet”按钮][“主页”选项卡上的“NuGet”按钮]

3.  在 NuGet 库中，将源设置为“NuGet 正式程序包来源”，然后搜索 NewRelic.Azure.WebSites。

    ![搜索 NewRelic.Azure.WebSites 的 nuget 库][搜索 NewRelic.Azure.WebSites 的 nuget 库]

4.  选择“用于 Azure 网站的 New Relic”条目，然后单击“安装”。

5.  在安装程序包后，该网站此时将包含名为“newrelic”的文件夹。展开此文件夹，并打开“newrelic.config”文件。在此文件中，将值 **REPLACE\_WITH\_LICENSE\_KEY** 替换为你从 Azure 应用商店获取的许可证密钥。

    ![包含所选 newrelic.conf 的已展开的 newrelic 文件夹][包含所选 newrelic.conf 的已展开的 newrelic 文件夹]

    添加许可证密钥信息后，将更改保存到 **newrelic.config** 文件。

### 步骤 3. 配置网站和发布应用程序。

在上一步中添加到你的应用程序的 New Relic 程序包将由添加到 Azure 网站的“应用程序设置”配置。执行以下步骤来添加这些设置。

1.  登录到 [Azure 管理门户][Azure 管理门户]并导航到你的网站。

2.  从网站中，选择“配置”。在“开发人员分析”部分，选择“外接程序”或“自定义”。上述任一方法都将生成相同的输出，但需要输入的内容稍有差别。“外接程序”将列出当前 New-Relic 许可证并允许你选择一个，而“自定义”要求你手动指定许可证密钥。

    如果你选择了“外接程序”，请使用“选择外接程序”字段来选择 New-Relic 许可证。

    ![外接程序字段的图像][外接程序字段的图像]

    如果你选择了“自定义”，请在“提供程序”中选择“New-Relic”，然后在“提供程序密钥”字段中输入许可证。

    ![自定义字段的图像][自定义字段的图像]

3.  在“开发人员分析”中指定许可证后，单击“保存”。保存操作完成后，以下值将会添加到页面的“应用程序设置”部分来支持 New-Relic：

    <table border="1">
    <thead>
    <tr>
    <td>
    密钥

    </td>
    <td>
    值

    </td>
    </tr>
    </thead>
    <tbody>
    <tr>
    <td>
    COR\\\_ENABLE\\\_PROFILING

    </td>
    <td>
    1

    </td>
    </tr>
    <tr>
    <td>
    COR\\\_PROFILER

    </td>
    <td>
    {71DA0A04-7777-4EC6-9643-7D28B46A8A41}

    </td>
    </tr>
    <tr>
    <td>
    COR\\\_PROFILER\\\_PATH

    </td>
    <td>
    C:\\Home\\site\\wwwroot\\newrelic\\NewRelic.Profiler.dll

    </td>
    </tr>
    <tr>
    <td>
    NEWRELIC\\\_HOME

    </td>
    <td>
    C:\\Home\\site\\wwwroot\\newrelic

    </td>
    </tr>
    <tr>
    <td>
    NEWRELIC\\\_LICENSEKEY

    </td>
    <td>
    你的许可证密钥

    </td>
    </tr>
    </tbody>
    </table>

    <div class="dev-callout"> 
<strong>说明</strong> 
<p>新的&ldquo;应用程序设置&rdquo;最多需要 30 秒即可生效。若要强制这些设置立即生效，请重新启动网站。</p> 
</div>

4.  通过使用 Visual Studio 或 WebMatrix 来发布应用程序。

### 步骤 4. 在 New Relic 中查看你的应用程序的性能。

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
<td align="left">从“Overview”（概览）的“Browser”（浏览器）视图中，指向“Global Apdex”图上的任何位置。<strong></strong><br /><strong>提示</strong>：若要直接转到所选应用程序的 <a href="https://newrelic.com/docs/site/geography">Geography</a> 仪表板，请单击“Global Apdex”标题，或单击“Global Apdex”图上的任何位置。<strong></strong></td>
</tr>
<tr class="even">
<td align="left">查看 <a href="https://docs.newrelic.com/docs/applications-menu/transactions-dashboard">Web Transactions</a> 仪表板</td>
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

-   [安装 Azure 网站的 .NET 代理][安装 Azure 网站的 .NET 代理]：New Relic .NET 代理安装过程
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
  [使用 Visual Studio 将 ASP.NET Web 应用程序部署到 Azure 网站（可能为英文页面）]: http://www.windowsazure.com/zh-cn/develop/net/tutorials/get-started/
  [使用 Microsoft WebMatrix 开发和部署网站（可能为英文页面）]: http://www.windowsazure.com/zh-cn/develop/net/tutorials/website-with-webmatrix/
  [程序包管理器控制台]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/NewRelicAzureNuget04.png
  [输入许可证密钥]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrvslicensekey.png
  [“主页”选项卡上的“NuGet”按钮]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrwmnugetbutton.png
  [搜索 NewRelic.Azure.WebSites 的 nuget 库]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrwmnugetgallery.png
  [包含所选 newrelic.conf 的已展开的 newrelic 文件夹]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrwmlicensekey.png
  [外接程序字段的图像]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nraddon.png
  [自定义字段的图像]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/nrcustom.png
  [New Relic Monitorin 仪表板]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/NewRelic_app.png
  [standard New Relic UI]: https://newrelic.com/docs/site/the-new-relic-ui#functions
  [dashboard drill-down]: https://newrelic.com/docs/site/the-new-relic-ui#drilldown
  [Apdex]: https://newrelic.com/docs/site/apdex
  [Geography]: https://newrelic.com/docs/site/geography
  [Web Transactions]: https://docs.newrelic.com/docs/applications-menu/transactions-dashboard
  [Key Transactions]: https://newrelic.com/docs/site/key-transactions
  [Errors]: https://newrelic.com/docs/site/errors
  [1]: ./media/store-new-relic-web-sites-dotnet-application-performce-management/NewRelic_app_browser.png
  [安装 Azure 网站的 .NET 代理]: https://newrelic.com/docs/dotnet/azure-web-sites-beta#manual_install
  [New Relic 用户界面]: https://newrelic.com/docs/site/the-new-relic-ui
  [应用程序概览（可能为英文页面）]: https://newrelic.com/docs/site/applications-overview
  [实际用户监视（可能为英文页面）]: https://newrelic.com/docs/features/real-user-monitoring
  [寻求帮助（可能为英文页面）]: https://newrelic.com/docs/site/finding-help
