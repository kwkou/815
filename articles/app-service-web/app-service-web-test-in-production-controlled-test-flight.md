<!-- not suitable for Mooncake -->

<properties
	pageTitle="Azure Web 应用中的试验部署（beta 测试）"
	description="本端到端教程说明如何试验应用中的新功能或者对更新进行 beta 测试。其中汇集了持续发布、槽、流量路由和 Application Insights 集成等 Azure 功能。"
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="02/02/2016"
	wacn.date=""/>
# Azure 中的试验部署（beta 测试）

本教程说明如何通过集成 [Azure Web 应用](/documentation/services/web-sites/)和 [Azure Application Insights](/home/features/application-insights/) 的各种功能来进行试验部署。

“试验”是以有限数量的实际客户验证新功能或更改的部署过程，同时也是生产方案中的主要测试。它类似于 beta 测试，有时也称为“受控试验”。许多具有网站空间的大型企业在运用[灵活开发](https://en.wikipedia.org/wiki/Agile_software_development)时，都采用此方法提前验证其应用更新。Azure App Service 可让你通过持续性发布和 Application Insights 集成生产环境中的测试，以实现相同的 DevOps 方案。此方法的优点包括：

- **在更新发行到生产环境_之前_获得真正的反馈** - 在发行之前获得反馈的效果，仅次于在发行期间实时获得的反馈。你可以在产品生命周期中，根据需要提前以实际的用户流量和行为来测试更新。
- **增强[持续测试导向开发 (CTDD)](https://en.wikipedia.org/wiki/Continuous_test-driven_development)** - 通过 Application Insights 的持续集成和工具集成生产环境中的测试，用户验证将可在产品生命周期内提前进行。这有助于减少在手动测试执行上投入的时间。
- **优化测试工作流** - 通过持续监视工具将生产环境中的测试自动化，可让你在单个过程中达到不同测试类型的目标，例如[集成](https://en.wikipedia.org/wiki/Integration_testing)、[回归](https://en.wikipedia.org/wiki/Regression_testing)、[可用性](https://en.wikipedia.org/wiki/Usability_testing)、辅助功能、本地化、[性能](https://en.wikipedia.org/wiki/Software_performance_testing)、[安全性](https://en.wikipedia.org/wiki/Security_testing)和[接受度](https://en.wikipedia.org/wiki/Acceptance_testing)。

试验部署涉及到的不只是路由实时流量。在此类部署中，你想要尽快获取洞察信息，无论是意外的 bug、性能下降还是用户体验问题。请记住，你面对的是实际客户。因此，若要正确行事，你必须确保设置试验部署，以收集为下一步做出明智决策所需的所有数据。本教程说明如何使用 Application Insights 收集数据，但你也可以使用 New Relic 或方案所适用的其他技术。

## 执行的操作

在本教程中，你将学习如何集成以下方案，以在生产环境中测试 Azure Web 应用：

- [将生产流量路由](/documentation/articles/app-service-web-test-in-production-get-start/)到 beta 应用
- [检测应用](/documentation/articles/app-insights-web-track-usage/)以获取有用的度量值
- 持续部署 beta 应用并跟踪实时应用度量值
- 比较生产应用程序和 beta 应用程序之间的度量值，以观察代码更改的结果为何

## 所需的项目

-	一个 Azure 帐户
-	一个 [GitHub](https://github.com/) 帐户
- Visual Studio 2015 - 可以下载[社区版](https://www.visualstudio.com/products/visual-studio-express-vs.aspx)。
-	Git Shell（与 [GitHub for Windows](https://windows.github.com/) 一起安装）- 可让你在相同的会话中运行 Git 和 PowerShell 命令
-	最新的 [Azure PowerShell](https://github.com/Azure/azure-powershell/releases/download/v0.9.8-September2015/azure-powershell.0.9.8.msi) 软件
-	基本了解以下知识：
	-	[Azure Resource Manager](/documentation/articles/resource-group-overview/) 模板部署（请参阅[通过可预测的方式在 Azure 中部署复杂应用程序](/documentation/articles/app-service-deploy-complex-application-predictably/)）
	-	[Git](http://git-scm.com/documentation)
	-	[PowerShell](https://technet.microsoft.com/zh-cn/library/bb978526.aspx)

> [AZURE.NOTE] 完成本教程需要有一个 Azure 帐户：
> + 可以[免费建立一个 Azure 帐户](/pricing/1rmb-trial/)：获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用免费的 Azure 服务（如 Web Apps）。
> + 你可以[激活 Visual Studio 订户权益](/pricing/member-offers/msdn-benefits-details/)：Visual Studio 订阅每月为你提供可用来试用付费版 Azure 服务的信用额度。
>
> 如果想要在注册 Azure 帐户之前开始使用 Azure，请转到[试用 Azure Web 应用](https://tryappservice.azure.com/)，你可以在 Azure 中立即创建一个生存期较短的入门 Web 应用。你不需要使用信用卡，也不需要做出承诺。

## 设置生产 Web 应用

>[AZURE.NOTE] 本教程中使用的脚本自动从 GitHub 存储库配置连续发布。这需要你的 GitHub 凭据已存储在 Azure 中，否则，尝试配置 Web Apps 的源代码管理设置时，脚本化部署将会失败。
>
>若要在 Azure 中存储你的 GitHub 凭据，请在 [Azure 门户](https://portal.azure.cn/)中创建 Web 应用，并[配置 GitHub 部署](/documentation/articles/app-service-continuous-deployment/#Step7)。您只需执行此操作一次。

在典型的 DevOps 方案中，应用程序在 Azure 中实时运行，并且你可以通过连续发布对它进行更改。在此方案中，你要将开发并测试的模板部署到生产环境。

1.	创建 [ToDoApp](https://github.com/azure-appservice-samples/ToDoApp) 存储库的专属分叉。有关创建分叉的信息，请参阅[分叉存储库](https://help.github.com/articles/fork-a-repo/)。创建分叉后，可以在浏览器中查看它。

	![](./media/app-service-agile-software-development/production-1-private-repo.png)

2.	打开 Git Shell 会话。如果你还没有 Git Shell，请立即安装 [GitHub for Windows](https://windows.github.com/)。
3.	通过执行以下命令创建分叉的本地复本：

        git clone https://github.com/<your_fork>/ToDoApp.git

4.	创建本地复本后，请导航到 &lt;repository\_root>\\ARMTemplates 并运行具有唯一后缀的 deploy.ps1 脚本，如下所示：

        .\deploy.ps1 -RepoUrl https://github.com/<your_fork>/todoapp.git -ResourceGroupSuffix <your_suffix>

4.	出现提示时，键入所需的用户名和密码来访问数据库。请记住数据库凭据，因为在更新资源组时需要再次指定凭据。

	你应会看到各种 Azure 资源的设置进度。部署完成后，脚本将在浏览器中启动应用程序，并发出友好的提示音。
	![](./media/app-service-web-test-in-production-controlled-test-flight/00.1-app-in-browser.png)

6.	返回 Git Shell 会话，运行：

        .\swap -Name ToDoApp<your_suffix>

	![](./media/app-service-web-test-in-production-controlled-test-flight/00.2-swap-to-production.png)

7.	脚本完成后，请返回浏览到前端的地址 (http://ToDoApp*&lt;your_suffix>*.chinacloudsites.cn/)，以查看在生产环境中运行的应用程序。
5.	登录到 [Azure 门户](https://portal.azure.cn/)并查看创建的内容。

	应该可以在相同的资源组中看到两个 Web 应用，其中一个的名称具有 `Api` 后缀。当你查看资源组视图时，还会看到 SQL 数据库和服务器、App Service 计划以及 Web 应用的过渡槽。浏览不同的资源，并将它们与 *&lt;repository\_root>*\\ARMTemplates\\ProdAndStage.json 进行比较，以查看它们在模板中的配置方式。

	![](./media/app-service-web-test-in-production-controlled-test-flight/00.3-resource-group-view.png)

你已设置生产应用。现在，我们假设你收到了应用可用性不佳的反馈。因此，你决定进行调查。你要检测应用以给出你的反馈。

## 调查：检测客户端应用的监视/度量值

5. 在 Visual Studio 中打开 &lt;repository\_root>\\src\\MultiChannelToDo.sln。
6. 右键单击解决方案 >“管理解决方案的 NuGet 包”>“还原”，以还原所有 Nuget 包。
6. 右键单击“MultiChannelToDo.Web”>“添加 Application Insights 遥测”>“配置设置”>“将资源组更改为 ToDoApp*&lt;你的后缀”>*”>“将 Application Insights 添加到项目”。
7. 在 Azure 门户中，打开 MultiChannelToDo.Web Application Insight 资源的边栏选项卡。然后，在“应用程序运行状况”部分中，单击“了解如何收集浏览器页面加载数据”> 复制代码。
7. 将复制的 JS 检测代码添加到 &lt;repository\_root>\\src\\MultiChannelToDo.Web\\app\\Index.cshtml，放在右 `<heading>` 标记前面。它应包含 Application Insight 资源的唯一检测键。

        <script type="text/javascript">
        var appInsights=window.appInsights||function(config){
            function s(config){t[config]=function(){var i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var t={config:config},r=document,f=window,e="script",o=r.createElement(e),i,u;for(o.src=config.url||"//az416426.vo.msecnd.net/scripts/a/ai.0.js",r.getElementsByTagName(e)[0].parentNode.appendChild(o),t.cookie=r.cookie,t.queue=[],i=["Event","Exception","Metric","PageView","Trace"];i.length;)s("track"+i.pop());return config.disableExceptionTracking||(i="onerror",s("_"+i),u=f[i],f[i]=function(config,r,f,e,o){var s=u&&u(config,r,f,e,o);return s!==!0&&t["_"+i](config,r,f,e,o),s}),t
        }({
            instrumentationKey:"<your_unique_instrumentation_key>"
        });

        window.appInsights=appInsights;
        appInsights.trackPageView();
        </script>

11. 将以下代码添加到正文底部，以将鼠标单击自定义事件发送到 Application Insights：

        <script>
            $(document.body).find("*").click(function(event) {

                appInsights.trackEvent(event.target.tagName + ": " + event.target.className);
            });
        </script>

    此 JavaScript 代码段在每次用户单击 Web 应用中的任何位置时，将自定义事件发送到 Application Insights。

12. 在 Git Shell 中，提交更改并将其发送到在 GitHub 中的分叉。然后，等待客户端刷新浏览器。

        git add -A :/
        git commit -m "add AI configuration for client app"
        git push origin master

6.	将已部署的应用程序更改交换到生产环境：

        .\swap -Name ToDoApp<your_suffix>

13. 浏览到你配置的 Application Insights 资源。单击“自定义事件”。

    ![](./media/app-service-web-test-in-production-controlled-test-flight/01-custom-events.png)

    如果未看见自定义事件的度量值，请等待几分钟，然后单击“刷新”。

假设你看到如下所示的图表：

![](./media/app-service-web-test-in-production-controlled-test-flight/02-custom-events-chart-view.png)

以及其下事件网格：

![](./media/app-service-web-test-in-production-controlled-test-flight/03-custom-event-grid-view.png)

根据 ToDoApp 应用程序代码，**BUTTON** 事件对应于提交按钮，**INPUT** 事件对应于文本框。到目前为止一切正常。不过，点击次数似乎很多，但点击待办事项（LI 事件）的次数却很少。

根据这一点，假设有部分用户搞不清楚 UI 的哪个部分是可点击的，其原因是光标停留在列表项及其图标上时，呈现为文本选择的样式。

![](./media/app-service-web-test-in-production-controlled-test-flight/04-to-do-list-item-ui.png)

你可能认为此示例是人为的。不过，你可以通过改进应用并运行试验部署，来获取实际客户提供的使用反馈。

### 检测服务器应用的监视/度量值
这有一点离题，因为本教程中演示的方案只处理客户端应用。不过，出于完整性，我们将设置服务器端应用。

6. 右键单击“MultiChannelToDo”>“添加 Application Insights 遥测”>“配置设置”>“将资源组更改为 ToDoApp*&lt;你的后缀”>*”>“将 Application Insights 添加到项目”。
12. 在 Git Shell 中，提交更改并将其发送到在 GitHub 中的分叉。然后，等待客户端刷新浏览器。

        git add -A :/
        git commit -m "add AI configuration for server app"
        git push origin master

6.	将已部署的应用程序更改交换到生产环境：

        .\swap -Name ToDoApp<your_suffix>

就这么简单！

## 调查：将槽特定的标记添加到客户端应用度量值

在本部分中，你将配置不同的部署槽，以将槽特定的遥测数据发送到同一个 Application Insights 资源。这样，即可比较来自不同槽（部署环境）的流量之间的遥测数据，以轻松查看应用更改的影响。同时，你可以区分生产流量与其他流量，以便在需要时继续监视生产应用。

由于要收集客户端行为的相关数据，因此你要在 index.cshtml 中[将遥测初始设置式添加到 JavaScript 代码](/documentation/articles/app-insights-api-custom-events-metrics/#js-initializer)。例如，如果想要测试服务器端性能，也可以在服务器代码中运行类似的动作（请参阅[自定义事件和度量值的 Application Insights API](/documentation/articles/app-insights-api-custom-events-metrics/)）。

1. 首先，在前面添加到 `<heading>` 标记的 JavaScript 块中，在以下两个 `//` 注释之间添加代码。

        window.appInsights = appInsights;

        // Begin new code
        appInsights.queue.push(function () {
            appInsights.context.addTelemetryInitializer(function (envelope) {
                var telemetryItem = envelope.data.baseData;
                telemetryItem.properties = telemetryItem.properties || {};
                telemetryItem.properties["Environment"] = "@System.Configuration.ConfigurationManager.AppSettings["environment"]";
            });
        });
        // End new code

        appInsights.trackPageView();

    此初始值设定项代码使 `appInsights` 对象将名为 `Environment` 的自定义属性添加到它所发送的每个遥测片段。

2. 然后，在 Azure 中将此自定义属性添加为 Web 应用的[槽设置](/documentation/articles/web-sites-staged-publishing/#AboutConfiguration)。为此，请在 Git Shell 会话中运行以下命令。

        $app = Get-AzureWebsite -Name todoapp<your_suffix> -Slot production
        $app.AppSettings.Add("environment", "Production")
        $app.SlotStickyAppSettingNames.Add("environment")
        $app | Set-AzureWebsite -Name todoapp<your_suffix> -Slot production

    项目中的 Web.config 已定义 `environment` 应用设置。通过此设置，当你在本地测试应用时，度量值将加上 `VS Debugger` 标记。但是，当你将更改发送到 Azure 时，Azure 将查找并使用 Web 应用配置中的 `environment` 应用设置，因此度量值加上 `Production` 标记。

3. 提交代码更改，并将其发送到 GitHub 上的分叉，然后等待用户使用新的应用（需要刷新浏览器）。新属性大约需要 15 分钟才出现在 Application Insights `MultiChannelToDo.Web` 资源中。

        git add -A :/
        git commit -m "add environment property to AI events for client app"
        git push origin master

4. 现在，再次转到“自定义事件”边栏选项卡，并筛选 `Environment=Production` 的度量值。通过此筛选器，你现在应可看到生产槽中所有新的自定义事件。

    ![](./media/app-service-web-test-in-production-controlled-test-flight/05-filter-on-production-environment.png)

5. 单击“收藏夹”按钮，将当前的“指标资源管理器”设置保存到“自定义事件: 生产”之类的项。以后你可以在此视图与部署槽视图之间轻松切换。

    > [AZURE.TIP] 若要获得更强大的分析功能，请考虑将 [Application Insights 资源与 Power BI 集成](/documentation/articles/app-insights-export-power-bi/)。

### 将槽特定的标记添加到服务器应用度量值
同样，出于完整性，我们将设置服务器端应用。不同于以 JavaScript 进行检测的客户端应用，服务器应用的槽特定标记以 .NET 代码进行检测。

1. 打开 &lt;repository\_root>\\src\\MultiChannelToDo\\Global.asax.cs。将以下代码块添加到命名空间右括号前面。

		namespace MultiChannelToDo
		{
				...

				// Begin new code
		    public class ConfigInitializer
		    : ITelemetryInitializer
		    {
		        void ITelemetryInitializer.Initialize(ITelemetry telemetry)
		        {
		            telemetry.Context.Properties["Environment"] = System.Configuration.ConfigurationManager.AppSettings["environment"];
		        }
		    }
				// End new code
		}

2. 将以下 `using` 语句添加到文件开头，以更正名称解析错误：

		using Microsoft.ApplicationInsights.Channel;
		using Microsoft.ApplicationInsights.Extensibility;

3. 将以下代码添加到 `Application_Start()` 方法的开头：

		TelemetryConfiguration.Active.TelemetryInitializers.Add(new ConfigInitializer());

3. 提交代码更改，并将其发送到 GitHub 上的分叉，然后等待用户使用新的应用（需要刷新浏览器）。新属性大约需要 15 分钟才出现在 Application Insights `MultiChannelToDo` 资源中。

        git add -A :/
        git commit -m "add environment property to AI events for server app"
        git push origin master

## 更新：设置 beta 分支

2. 打开 &lt;repository\_root>\\ARMTemplates\\ProdAndStagetest.json，并找到 `appsettings` 资源（搜索 `"name": "appsettings"`）。资源共有 4 项，分别用于各个槽。 

2. 针对每项 `appsettings` 资源，将 `"environment": "[parameters('slotName')]"` 应用设置添加到 `properties` 数组的末尾。别忘了在上一行末尾处使用逗号。

    ![](./media/app-service-web-test-in-production-controlled-test-flight/06-arm-app-setting-with-slot-name.png)
    
    你已将 `environment` 应用设置添加到模板中的所有槽。
    
2. 在同一文件中，查找 `slotconfignames` 资源（搜索 `"name": "slotconfignames"`）。资源共有 2 项，分别用于各个应用。

2. 针对每项 `slotconfignames` 资源，将 `"environment"` 添加到 `appSettingNames` 数组的末尾。别忘了在上一行末尾处使用逗号。

    你已将 `environment` 应用设置链接到其各自用于两个应用的部署槽。

3. 在 Git Shell 会话中运行以下命令，其中具有前面使用的相同资源组后缀。

        git checkout -b beta
        git push origin beta
        .\deploy.ps1 -RepoUrl https://github.com/<your_fork>/ToDoApp.git -ResourceGroupSuffix <your_suffix> -SlotName beta -Branch beta

4. 出现提示时，请指定与前面相同的 SQL 数据库凭据。然后，当系统询问是否要更新资源组时键入 `Y`，然后按 `ENTER`。

    脚本完成后，原始资源组中的所有资源都保留下来，但在其中创建名为“beta”的新位置，且其配置与一开始创建的“过渡”槽相同。

    >[AZURE.NOTE] 这种创建不同部署环境的方法，与[使用 Azure Web 应用进行灵便软件开发](/documentation/articles/app-service-agile-software-development/)中的方法不同。使用此方法时，将创建具有部署槽的部署环境，而使用后一种方法时，将创建具有资源组的部署环境。使用资源组来管理部署环境可让你将开发人员排除在生产环境以外，但无法轻松在生产环境中进行测试，而这一点却可以通过槽来轻松做到。

如果需要，也可以通过以下代码创建 alpha 应用

    git checkout -b alpha
    git push origin alpha
    .\deploy.ps1 -RepoUrl https://github.com/<your_fork>/ToDoApp.git -ResourceGroupSuffix <your_suffix> -SlotName beta -Branch alpha

在本教程中，你只要继续使用 beta 应用即可。

## 更新：将更新发送到 beta 应用

返回到你想要改进的应用。

1. 确保目前你位于 beta 分支中

        git checkout beta

2. 在 &lt;repository\_root>\\src\\MultiChannelToDo.Web\\app\\Index.cshtml 中找到 `<li>` 标记，并添加 `style="cursor:pointer"` 属性，如下所示。

    ![](./media/app-service-web-test-in-production-controlled-test-flight/07-change-cursor-style-on-li.png)

3. 提交并推送到 Azure。

4. 通过导航到 http://todoapp*&lt;your_suffix>*-beta.chinacloudsites.cn/，验证更改是否已反映在 beta 槽中。如果未看到更改，请刷新浏览器以获取新的 javascript 代码。

    ![](./media/app-service-web-test-in-production-controlled-test-flight/08-verify-change-in-beta-site.png)

现在，你的更改已在 beta 槽中运行，因此接下来可以执行试验部署。

## 验证：将流量路由到 beta 应用

在本部分中，你要将流量路由到 beta 应用。为了能够清楚演示，请将用户流量的重要部分路由到该应用。实际上，将路由的流量取决于特定情况。例如，如果站点属于 microsoft.com 层级，则你可能只需要不到 1% 总流量即可获取有用的数据。

1. 在 Git Shell 会话中运行以下命令，将一半的生产流量路由到 beta 槽：

        $siteName = "ToDoApp<your suffix>"
        $rule = New-Object Microsoft.WindowsAzure.Commands.Utilities.Websites.Services.WebEntities.RampUpRule
        $rule.ActionHostName = "$siteName-beta.chinacloudsites.cn"
        $rule.ReroutePercentage = 50
        $rule.Name = "beta"
        Set-AzureWebsite $siteName -Slot Production -RoutingRules $rule

  `ReroutePercentage=50` 属性指定要将 50% 的生产流量路由到 beta 应用的 URL（由 `ActionHostName` 属性指定）。

2. 现在，请导航到 http://ToDoApp*&lt;your_suffix>*.chinacloudsites.cn。50% 的流量现在应该重定向到 beta 槽。

3. 在 Application Insights 资源中，使用 environment="beta" 筛选度量值。

    > [AZURE.NOTE] 如果将此筛选的视图另存为收藏夹，你可以在生产视图与 beta 视图之间轻松切换指标资源管理器视图。

假设你在 Application Insights 中看到如下所示的内容：

![](./media/app-service-web-test-in-production-controlled-test-flight/09-test-beta-site-in-production.png)

这不仅显示 `<li>` 标记的点击次数远多于一般，`<li>` 标记的点击次数似乎也有激增的现象。凭此可以推断，人们发现了新的 `<li>` 标记是可点击的，因此纷纷清除以前在应用中完成的所有任务。

根据试验部署的数据，可以确定新的 UI 已可用于生产环境。

## 实现：将新代码移到生产环境

现在可将更新移到生产环境。有利的是，现在你知道更新在发送到生产环境之前已经过验证。因此，现在可以放心进行部署。由于是对 AngularJS 客户端应用进行更新，因此只有客户端代码受到验证。如果要对后端 Web API 应用进行更改，可以用同样的方式轻松验证更改。

1. 在 Git Shell 中运行以下命令，以删除流量路由规则：

        Set-AzureWebsite $siteName -Slot Production -RoutingRules @()

2. 运行 Git 命令：

        git checkout master
        git pull origin master
        git merge beta
        git push origin master

2. 等待几分钟，让新的代码部署到过渡槽，然后启动 http://ToDoApp*&lt;your_suffix>*-staging.chinacloudsites.cn，以验证新的更新是否已在过渡槽中准备就绪。请记住，分叉的主要分支链接到应用的过渡槽。

3. 现在，将过渡槽交换到生产环境中

        cd <ROOT>\ToDoApp\ARMTemplates
        .\swap.ps1 -Name todoapp<your_suffix>

## 摘要 ##

Azure 可让中小型企业轻松地在生产环境中测试其客户端应用，这在过去只能在大型企业中进行。希望本教程提供的信息可帮助你结合 Azure 和 Application Insights 以成功进行试验部署，甚至在 DevOps 领域进行其他生产测试。

## 更多资源 ##

-   [使用 Azure Web 应用进行灵便软件开发](/documentation/articles/app-service-agile-software-development/)
-   [为 Azure 中的 Web 应用设置过渡环境](/documentation/articles/web-sites-staged-publishing/)
-	[通过可预测的方式在 Azure 中部署复杂应用程序](/documentation/articles/app-service-deploy-complex-application-predictably/)
-	[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)
-	[JSONLint – JSON 验证程序](http://jsonlint.com/)
-	[Git 分支 - 基本分支和合并](http://www.git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
-	[Azure PowerShell](/documentation/articles/powershell-install-configure/)
-	[项目 Kudu Wiki](https://github.com/projectkudu/kudu/wiki)

<!---HONumber=Mooncake_0328_2016-->