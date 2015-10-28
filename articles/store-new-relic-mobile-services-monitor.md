<properties 
	pageTitle="在源代码管理中存储服务器脚本 - Azure 移动服务" 
	description="了解如何使用 New Relic 外接程序来监视你的移动服务。" 
	documentationCenter="" 
	authors="stepsic-microsoft-com" 
	manager="carolz"
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services"  
	ms.date="03/16/2015" 
	wacn.date="08/29/2015"/>

# 使用 New Relic 监视移动服务

本主题展示如何配置第三方 New Relic 外接程序，以便使用 Azure 移动服务增强对你的移动服务的监视。

本教程将指导你完成以下步骤：

1. [使用 Azure 应用商店注册 New Relic]。
2. [安装 New Relic 模块]。
3. [启用针对移动服务的 New Relic 开发人员分析]。
4. [在 New Relic 仪表板中监视移动服务]。

若要完成本教程，你必须事先参考[移动服务入门]或[数据处理入门教程]创建一个移动服务。

## <a name="sign-up"></a>使用 Azure 应用商店注册 New Relic

第一步是购买 New Relic 服务。本教程演示如何从 Azure 应用商店购买该服务。移动服务支持在 Azure 应用商店外购买的 New Relic 订阅。

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。

2. 在该管理门户的下方窗格中，单击“新建”。

3. 单击“应用商店”。

4. 在“选择外接程序”对话框中，选择“New Relic”，然后单击“下一步”。

5. 在“个性化外接程序”对话框中，选择你需要的 New Relic 计划。

7. 输入 New Relic 服务将在你的 Azure 设置中显示的名称，或者使用默认值“NewRelic”。此名称在你已订阅的 Azure 应用商店项目列表中必须是唯一的。

8. 选择区域值，例如美国西部。

9. 单击**“下一步”**。

10. 在“查看购买”对话框中，查看计划和定价信息以及法律条款。如果你同意这些条款，请单击“购买”。

11. 单击“购买”后，你的 New Relic 将开始创建过程。你可以在 Azure 管理门户中监视状态。

## <a name="install-module"></a>安装 New Relic 模块

在你注册 New Relic 服务后，需要在你的移动服务中安装 New Relic Node.js 模块。你必须针对你的移动服务启用了源代码管理，才能上载此模块。

1. 如果你尚未这样做，请按照教程[在源代码管理中存储服务器脚本]中的步骤针对你的移动服务启用源代码管理，克隆存储库，并且安装 <a href="http://nodejs.org/" target="_blank">Node 包管理器 (NPM)</a>。

2. 导航到本地 Git 存储库的 `.\service` 文件夹，然后在命令提示符下运行以下命令：

		npm install newrelic

	NPM 将 [New Relic 模块][newrelic]安装在 `\newrelic` 子目录中。

3. 打开一个 Git 命令行工具，例如 **GitBash** (Windows) 或 **Bash** (Unix Shell)，然后在 Git 命令提示符处键入以下命令：

		$ git add .
		$ git commit -m "added newrelic module"
		$ git push origin master
		
	这会将新的 `newrelic` 模块上载到你的移动服务。

接下来，你将在[管理门户][Azure Management Portal]中启用针对你的移动服务的 New Relic 监视。

## <a name="enable-service"></a>启用针对移动服务的 New Relic 开发人员分析

1. 在[管理门户][Azure Management Portal]中，选择你的移动服务，然后单击“配置”选项卡。

	![][0]

2. 向下滚动到“开发人员分析”，然后根据你购买 New Relic 订阅的方式，执行以下操作之一：

	+ 在 Azure 应用商店中购买的：

		单击“外接程序”，从“选择外接程序”中选择 New Relic 外接程序，然后单击“保存”。

		![][1]

	+ 直接从 New Relic 购买的：

		单击“自定义”，从“提供程序”中选择 New Relic，输入密钥，然后单击“保存”。

		![][2]

		可从 New Relic 仪表板中获取密钥。

3. 在完成注册后，你将在“应用程序设置”中看到新值：

	![][3]

## <a name="monitor"></a>在 New Relic 仪表板中监视移动服务

1. 运行你的客户端应用程序以便生成对你的移动服务的读取、创建、更新和删除请求。

2. 等待几分钟以便处理数据，然后导航到 New Relic 仪表板。

	如果你的 New Relic 订阅是作为外接程序购买的，则在[管理门户][Azure Management Portal]中选择它，然后单击“管理”。

3. 在 New Relic 中，单击“应用程序”，然后单击你的移动服务。

	![][4]

4. 单击“Web 事务”可查看刚对你的移动服务发出的最近的请求：

	![][5]

## <a name="next-steps"></a>后续步骤

+ 若要优化你的 **iOS**/**Android** 移动应用程序性能，请参阅 [New Relic 移动]。
+ 有关定价信息，请参阅 <!--[-->Azure 应用商店中的 New Relic 页<!--]-->。
+ 有关使用 New Relic 的更多信息，请参阅 New Relic 文档中的[应用程序概述]。 

<!-- Anchors. -->
[使用 Azure 应用商店注册 New Relic]: #sign-up
[安装 New Relic 模块]: #install-module
[启用针对移动服务的 New Relic 开发人员分析]: #enable-service
[在 New Relic 仪表板中监视移动服务]: #monitor
[Next steps]: #next-steps

<!-- Images. -->
[0]: ./media/store-new-relic-mobile-services-monitor/mobile-configure-tab.png
[1]: ./media/store-new-relic-mobile-services-monitor/mobile-configure-new-relic-monitoring.png
[2]: ./media/store-new-relic-mobile-services-monitor/mobile-configure-new-relic-monitoring-custom.png
[3]: ./media/store-new-relic-mobile-services-monitor/mobile-configure-new-relic-monitoring-complete.png
[4]: ./media/store-new-relic-mobile-services-monitor/mobile-new-relic-dashboard.png
[5]: ./media/store-new-relic-mobile-services-monitor/mobile-new-relic-dashboard-2.png

<!-- URLs. -->
[Source control]: http://msdn.microsoft.com/zh-cn/library/windowsazure/c25aaede-c1f0-4004-8b78-113708761643
[Work with server scripts in Mobile Services]: /documentation/articles/work-with-server-scripts
[Azure Management Portal]: https://manage.windowsazure.cn
[Node.js API Documentation: Modules]: http://nodejs.org/api/modules.html
[在源代码管理中存储服务器脚本]: /documentation/articles/mobile-services-store-scripts-source-control
[newrelic]: https://npmjs.org/package/newrelic
[Azure 应用商店中的 New Relic 页]: /gallery/store/new-relic/new-relic/
[应用程序概述]: https://docs.newrelic.com/docs/applications-dashboards/applications-overview
[移动服务入门]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started
[数据处理入门教程]: /develop/mobile/tutorials/get-started-with-data-dotnet
[New Relic 移动]: http://newrelic.com/mobile-monitoring

 

<!---HONumber=67-->