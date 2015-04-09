<properties title="How to create web test" pageTitle="How to create web test" description="Learn how to create web tests in Azure." authors="stepsic"  />

# Web 测试

你的 Azure 网站是否仍在运行？它是否可正常响应，速度是否够快？请配置 Web 测试，以定期测试你的网站。如果站点关闭，或者响应速度缓慢或不正常，你将会收到电子邮件警报。此外，你还会收到显示网站在一段时间内的可用性和响应速度的图形。

![浏览中心][浏览中心]

你可以为任何使用基本或标准计划的 Azure 网站设置可用性监视。最多可以创建 3 个 Web 测试，并且最多可以从 3 个地理位置运行其中的每项测试。你无需对网站做出任何形式的更改。

你还可以在部署或计划的服务中断期间暂停 Web 测试，使整体可用性不受影响。整体可用性基于所有 Web 测试（包括选择的不同位置）计算得出。

## 如何设置 Web 测试

1.  若要配置 Web 测试，请先确保你的网站使用“基本”或“标准”计划。
2.  然后，在“网站”分页上选择“Web 测试”部件：

   ![配置 Web 测试][配置 Web 测试]

3.  在“创建 Web 测试”分页中为 Web 测试命名，然后指定要针对其运行测试的 URL。

   ![创建 Web 测试][创建 Web 测试]

1.  接下来，请从 8 个位置中选择最多 3 个位置

2.  指定成功标准，包括 HTTP 状态码检查，或站点本身包含的字符串。

3.  然后选择警报设置，包括敏感度和电子邮件的收件人。

   ![警报][警报]

    - High sensitivity will create an alert whenever a test failure is detected in just 1 location.
    - Medium sensitivity requires at least half of the locations have seen a failure in 10 minutes.
    - Low sensitivity requires that the test at all locations have failed within 15 minutes.

完成设置后，请单击“创建”按钮。创建 Web 测试后，它每隔 5 分钟就会从指定的位置执行，因此可能需要经过一段时间才会显示数据。

### 位置能够反映什么问题？

我们从这些位置向网站发送请求，用户将以相同的方式从世界各地访问站点。如果你的站点在美国不可访问，但在欧洲仍可访问，那么，你就知道这是网络问题，而不是服务器的问题。

### 使用成功标准

通常，你需要测试 HTTP 状态代码是否为 200，这表示服务器已识别 URI 并返回了一个页面。

不能在内容匹配字符串中使用通配符，但可以测试任何纯文本。

## 哎呀，我的站点关闭了！

如果你的 Web 测试达不到成功标准，则会被标记为失败的测试，因而会降低网站的整体可用性。失败的测试（以及成功的测试）显示在特定 Web 测试分页上的散点图中。

![失败的测试][失败的测试]

你可以分析失败的测试以确定失败的原因。请钻取到失败的 Web 测试，然后打开 Visual Studio 的 Web 测试结果文件，以分析并了解测试失败的原因。

![在 VS 中打开][在 VS 中打开]

  [浏览中心]: ./media/insights-create-web-tests/Inisghts_WebTestBlade.png
  [配置 Web 测试]: ./media/insights-create-web-tests/Insights_ConfigurePart.png
  [创建 Web 测试]: ./media/insights-create-web-tests/Insights_CreateTest.png
  [警报]: ./media/insights-create-web-tests/Inisghts_AlertCreation.png
  [失败的测试]: ./media/insights-create-web-tests/Insights_FailedWebTest.png
  [在 VS 中打开]: ./media/insights-create-web-tests/Insights_OpenInVS.png
