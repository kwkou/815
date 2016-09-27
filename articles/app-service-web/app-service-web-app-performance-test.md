<properties
   pageTitle="测试 Azure Web 应用的性能 | Azure"
   description="运行 Azure Web 应用性能测试以检查你的应用如何处理用户负载。测量响应时间，并查找可能表明出现问题的失败状况。"
   services="app-service\web"
   documentationCenter=""
   authors="ecfan"
   manager="douge"
   editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="05/25/2016"
	wacn.date="09/26/2016"/>

# 测试 Azure Web 应用在低负载下的性能

在推出 Web 应用或将更新部署到生产环境之前，先检查该应用的性能。这样，便可更妥善地评估应用是否已准备好发布。对应用更具信心，其可在尖峰使用期间或在下一波推式营销时处理流量。

在公开预览期间，可以在 Azure 门户中免费测试应用的性能。这些测试将模拟应用在特定期间内的用户负载并测量应用的响应度。例如，测试结果显示应用对指定数量的用户的响应速度。它们还会显示有多少个请求失败（这可能表示应用有问题）。

![在 Web 应用中查找性能问题](./media/app-service-web-app-performance-test/azure-np-perf-test-overview.png)

## 开始之前

* 你需要一个 [Azure 订阅](https://account.windowsazure.cn/subscriptions)（如果还没有的话）。了解如何[免费注册 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

* 需要一个 [Visual Studio Team Services](https://www.visualstudio.com/products/what-is-visual-studio-online-vs) 帐户才能保留性能测试历史记录。在你设置性能测试时，系统将自动创建一个合适的帐户。你也可以创建一个新帐户或使用现有的帐户（如果你是帐户所有者）。

* 部署应用以便在非生产环境中进行测试。让应用使用生产环境中所用方案以外的 App Service 计划。这样就不会影响任何现有的客户或让应用在生产环境中变慢。

## 设置和运行性能测试

0.  登录到 [Azure 门户](https://portal.azure.cn)。若要使用你拥有的 Visual Studio Team Services 帐户，请以帐户所有者的身份登录。

0.  转到你的 Web 应用。

    ![转到“全部浏览”、“Web Apps”、你的 Web 应用](./media/app-service-web-app-performance-test/azure-np-web-apps.png)

0.  转到“性能测试”。

    ![转到“工具”、“性能测试”](./media/app-service-web-app-performance-test/azure-np-web-app-details-tools-expanded.png)
 
0. 现在请链接 [Visual Studio Team Services](https://www.visualstudio.com/products/what-is-visual-studio-online-vs) 帐户以保留性能测试历史记录。

    如果你已有 Team Services 帐户，请选择该帐户。如果没有，请创建新帐户。

    ![选择现有的 Team Services 帐户，或创建新的帐户](./media/app-service-web-app-performance-test/azure-np-no-vso-account.png)

0.  创建性能测试。设置详细信息并运行测试。

可以一边运行测试，一边实时查看结果。

例如，假设我们有在去年节日特卖时发放优惠券的应用。此活动持续了 15 分钟，高峰负载为 100 位并发客户。我们希望今年的客户数变成两倍。我们还想让页面加载时间从 5 秒降低为 2 秒，以改善客户满意度。因此，我们将以 250 名用户测试更新后应用的性能达 15 分钟之久。

我们生成同时访问我们的网站的虚拟用户（客户），以此模拟应用的负载。这显示有多少个请求失败或响应速度较慢。

  ![创建、设置并运行性能测试](./media/app-service-web-app-performance-test/azure-np-new-performance-test.png)

   *  系统自动添加 Web 应用的默认 URL。可以更改此 URL 以测试其他页面（仅限 HTTP GET 请求）。

   *  若要模拟本地情况并降低延迟，请选择最靠近用户的位置来生成负载。

  以下是正在进行的测试。在第一分钟内，我们的页面加载速度低于预期。

  ![正在进行的包含实时数据的性能测试](./media/app-service-web-app-performance-test/azure-np-running-perf-test.png)

  测试完成后，我们了解页面加载速度在第一分钟后变快许多。这有助于找出我们要开始排查问题的位置。

  ![已完成的性能测试将显示结果，包括失败的请求](./media/app-service-web-app-performance-test/azure-np-perf-test-done.png)

## 测试多个 URL

也可以通过上载 Visual Studio Web 测试文件来运行包含多个 URL 的性能测试，这些 URL 表示一个端到端的用户方案。可通过以下几种方法创建 Visual Studio Web 测试文件：

* [使用 Fiddler 捕获流量，并导出为 Visual Studio Web 测试文件](http://docs.telerik.com/fiddler/Save-And-Load-Traffic/Tasks/VSWebTest)
* [在 Visual Studio 中创建负载测试文件](https://www.visualstudio.com/docs/test/performance-testing/run-performance-tests-app-before-release)

若要上载并运行 Visual Studio Web 测试文件，请执行以下操作：
 
0. 按照上述步骤打开“新建性能测试”边栏选项卡。在此边栏选项卡中，选择“使用以下项目配置测试”选项来打开“使用以下项目配置测试”边栏选项卡。  

    ![打开“配置负载测试”边栏选项卡](./media/app-service-web-app-performance-test/multiple-01-authoring-blade.png)

0. 确保“测试类型”设置为“Visual Studio Web 测试”并选择 HTTP 存档文件。使用“文件夹”图标打开文件选择器对话框。

    ![上载多个 URL Visual Studio Web 测试文件](./media/app-service-web-app-performance-test/multiple-01-authoring-blade2.png)

    上载文件后，“URL 详细信息”部分中将显示待测试 URL 的列表。
 
0. 指定用户负载和测试持续时间，然后选择“运行测试”。

    ![选择用户负载和持续时间](./media/app-service-web-app-performance-test/multiple-01-authoring-blade3.png)

    测试完成后，将在两个窗格中显示结果。左窗格通过一系列图表显示性能信息。

    ![性能结果窗格](./media/app-service-web-app-performance-test/multiple-01a-results.png)

    右窗格显示失败请求以及错误类型和发生次数的列表。

    ![请求失败窗格](./media/app-service-web-app-performance-test/multiple-01b-results.png)

0. 通过选择右窗格顶部的“重新运行”图标重新运行测试。

    ![重新运行测试](./media/app-service-web-app-performance-test/multiple-rerun-test.png)

##  问题解答

#### 问：持续运行测试的时间是否有限制？ 

**答**：是，最多可以在 Azure 门户中运行一小时的测试。

#### 问：可运行性能测试的时间有多久？ 

**答**：公共预览版发布后，通过 Visual Studio Team Services 帐户每个月可免费获取 20,000 虚拟用户分钟数 (VUM)。VUM 是虚拟用户数乘以测试中的分钟数。如果要求超过免费限制，可以购买更多时间，你只需支付所用的时间。

#### 问：哪里可以检查到目前为止我已使用多少 VUM？

**答**：可以在 Azure 门户中检查此数量。

![转到你的 Team Services 帐户](./media/app-service-web-app-performance-test/azure-np-vso-accounts.png)

![检查已用的 VUM](./media/app-service-web-app-performance-test/azure-np-vso-accounts-vum-summary.png)

#### 问：默认选项是什么？我现有的测试会受影响吗？

**答**：性能负载测试的默认选项是手动测试 — 与向门户添加多个 URL 测试选项之前相同。现有测试将继续使用配置的 URL，并像以前一样工作。

#### 问：Visual Studio Web 测试文件不支持哪些功能？

**答**：目前，此功能不支持 Web 测试插件、数据源和提取规则。必须编辑 Web 测试文件以删除这些功能。我们希望在未来的更新中添加对这些功能的支持。

#### 问：它是否支持任何其他 Web 测试文件格式？
  
**答**：目前仅支持 Visual Studio Web 测试格式文件。如果需要对其他文件格式的支持，欢迎告知我们。请通过 [vsoloadtest@microsoft.com](mailto:vsoloadtest@microsoft.com) 向我们发送电子邮件。

#### 问：Visual Studio Team Service 帐户还有什么用途？

**答**：若要查找新帐户，请转到 ```https://{accountname}.visualstudio.com```。使用任何工具或语言来共享代码、构建、测试、跟踪工作和交付软件 – 一切尽在云中进行。详细了解 [Visual Studio Team Services](https://www.visualstudio.com/products/what-is-visual-studio-online-vs) 功能与服务如何帮助你的团队更轻松地协作以及持续进行部署。

## 另请参阅

* [Run simple cloud performance tests（运行简单的云性能测试）](https://www.visualstudio.com/docs/test/performance-testing/getting-started/get-started-simple-cloud-load-test)
* [Run Apache Jmeter performance tests（运行 Apache Jmeter 性能测试）](https://www.visualstudio.com/docs/test/performance-testing/getting-started/get-started-jmeter-test)
* [Record and replay cloud-based load tests（记录和重播基于云的负载测试）](https://www.visualstudio.com/docs/test/performance-testing/getting-started/record-and-replay-cloud-load-tests)
* [Performance test your app in the cloud（在云中测试应用的性能）](https://www.visualstudio.com/docs/test/performance-testing/getting-started/getting-started-with-performance-testing)

<!---HONumber=Mooncake_0627_2016-->