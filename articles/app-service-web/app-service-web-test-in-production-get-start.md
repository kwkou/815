<properties
	pageTitle="在生产环境中 Web Apps 的测试入门"
	description="了解 Azure App Service Web Apps 的生产中测试 (TiP) 功能。"
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="01/13/2016"
	wacn.date=""/>

# 在生产环境中 Web Apps 的测试入门

在生产环境中测试，或者使用实时客户流量实测 Web 应用，是应用开发人员逐渐集成到其[灵活开发](https://en.wikipedia.org/wiki/Agile_software_development)方法的测试策略。它可让你在生产环境中使用实际用户流量测试应用的质量，而不是在测试环境中以综合数据进行测试。通过将新的应用公开给实际用户，得知应用在部署之后可能发生的实际问题。你可以根据实际用户流量的数量、速度和变化来确认应用更新的功能、性能和价值，这些都是在测试环境中无法测得的。

## App Service Web Apps 中的流量路由

通过 [Azure App Service](/documentation/services/web-sites/) 中的流量路由功能，可以将部分实时用户流量定向到一个或多个[部署槽](/documentation/articles/web-sites-staged-publishing/)，然后使用 [Azure Application Insights](/home/features/application-insights/) 或 [Azure HDInsight](/home/features/hdinsight/) 来分析应用，或使用第三方工具（例如 [New Relic](/marketplace/partners/newrelic/newrelic/)）来验证更改。例如，可以使用 App Service 实现以下方案：

- 在部署到整个站点之前找出更新中的功能错误或性能瓶颈
- 对 beta 应用测量可用性度量值，以执行更改的“受控试验”。
- 逐渐构建出新的更新，并在错误发生时正常恢复为当前版本 
- 在多个部署位置中运行 [A/B 测试](https://en.wikipedia.org/wiki/A/B_testing)或[多元测试](https://en.wikipedia.org/wiki/Multivariate_testing_in_marketing)，以优化应用的业务效果

### 在 Web Apps 中使用流量路由的要求

- Web 应用必须在“标准”或“高级”层运行，因为多个部署槽有此要求。
- 若要正常使用流量路由，需要在用户的浏览器中启用 Cookie。在客户端会话生存期内，流量路由使用 Cookie 将客户端浏览器固定到部署槽。
- 流量路由可通过 Azure PowerShell cmdlet 支持高级 TiP 方案。

## 将流量段路由到部署槽

在每个 TiP 方案的基本级别中，将预定义百分比的实际流量路由到非生产部署槽。为此，请执行以下步骤：

>[AZURE.NOTE] 此处的步骤假设已有[非生产部署槽](/documentation/articles/web-sites-staged-publishing/)，并且所需的 Web 应用内容[已部署](/documentation/articles/web-sites-deploy/)给它。

1. 登录到 [Azure 门户](https://portal.azure.cn/)。
2. 在 Web 应用的边栏选项卡中，单击“设置”>“流量路由”。![](./media/app-service-web-test-in-production/01-traffic-routing.png)
3. 选择要将流量路由到的槽以及所需的总流量百分比，然后单击“保存”。

	![](./media/app-service-web-test-in-production/02-select-slot.png)

4. 转到部署槽的边栏选项卡。现在你应会看到路由到该处的实时流量。

	![](./media/app-service-web-test-in-production/03-traffic-routed.png)

配置流量路由后，指定百分比的客户端将随机路由到非生产槽。但是，要特别注意的是，一旦客户端自动路由到特定槽，它将在客户端会话生存期内都“固定”到该槽。这是使用 Cookie 固定用户会话来实现的。如果你查看 HTTP 请求，就会发现每个后续请求中都有一个 `TipMix` Cookie。

![](./media/app-service-web-test-in-production/04-tip-cookie.png)

## 强制将客户端请求发送到特定槽

除了自动流量路由以外，App Service 也可以将请求路由到特定槽。如果想要让用户能够选择加入或退出 beta 应用，这就非常有用。为此，请使用 `x-ms-routing-name` 查询参数。

若要使用 `x-ms-routing-name` 将用户重新路由到特定槽，必须确保该槽已添加到流量路由列表。由于你想要显式路由到某个槽，因此设置的实际路由百分比并不重要。如果需要，可以创建用户单击即可访问 beta 应用的“beta 链接”。

![](./media/app-service-web-test-in-production/06-enable-x-ms-routing-name.png)

### 让用户选择退出 beta 应用

例如，若要让用户选择退出 beta 应用，你可以在网页中插入以下链接：

    <a href="<webappname>.chinacloudsites.cn/?x-ms-routing-name=self">Go back to production app</a>

字符串 `x-ms-routing-name=self` 指定生产槽。一旦客户端浏览器访问此链接，不仅浏览器会重定向到生产槽，后续每个请求也都包含将会话固定到生产槽的 `x-ms-routing-name=self` Cookie。

![](./media/app-service-web-test-in-production/05-access-production-slot.png)

### 让用户选择加入 beta 应用

若要让用户选择加入 beta 应用，请将相同的查询参数设置为非生产槽的名称，例如：

		<webappname>.chinacloudsites.cn/?x-ms-routing-name=staging

## 更多资源 ##

-   [为 Azure App Service 中的 Web 应用设置过渡环境](/documentation/articles/web-sites-staged-publishing/)
-	[通过可预测的方式在 Azure 中部署复杂应用程序](/documentation/articles/app-service-deploy-complex-application-predictably/)
-   [使用 Azure App Service 进行灵便软件开发](/documentation/articles/app-service-agile-software-development/)
-	[对 Web 应用有效使用 DevOps 环境](/documentation/articles/app-service-web-staged-publishing-realworld-scenarios/)
<!---HONumber=Mooncake_0328_2016-->