<properties title="How to debug with events" pageTitle="How to debug with events" description="Learn how to see events in Azure." authors="hanikn"  />

# 监视影响 Azure 网站或资源组的事件

1.  登录到 [Azure 门户预览版][Azure 门户预览版]。

2.  在左侧导航栏（又称为**跳转栏**）中单击“浏览”按钮。

    ![浏览中心][浏览中心]

1.  然后选择“资源组”或“网站”（具体取决于你感兴趣的事件）。为方便演示，本文档中的屏幕截图包含了资源组的事件。

2.  在“资源组”分页上，单击资源组的名称。此时你将会导航到资源组分页。

    ![资源组][资源组]

3.  资源组分页包含一个名为“过去一周的事件”的部件。该部件上的每个长条表示过去一周每一天发生的事件数。每个长条具有两种不同的颜色：蓝色和粉红色。粉红色表示当天**失败的**事件，蓝色表示所有其他事件。

    ![资源组][1]

4.  现在，请单击“过去一周的事件”部件。你将会看到名为“事件”的新分页，其中包含过去一周对你的资源组造成了影响的所有事件。

    ![资源组][2]

1.  单击其中一个事件。

    ![资源组][3]

将会打开一个新分页，其中包含有关该事件的许多详细信息。对于**失败的**事件，此页通常会显示“子状态”和“属性”部分，其中包含了可用于调试的详细信息。

  [Azure 门户预览版]: https://portal.azure.com/
  [浏览中心]: ./media/insights-debugging-with-events/Insights_Browse.png
  [资源组]: ./media/insights-debugging-with-events/Insights_SelectRG.png
  [1]: ./media/insights-debugging-with-events/Insights_RGBlade.png
  [2]: ./media/insights-debugging-with-events/Insights_AllEvents.png
  [3]: ./media/insights-debugging-with-events/Insights_EventDetails.png
