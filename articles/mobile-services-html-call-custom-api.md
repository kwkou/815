<properties linkid="mobile-services-html-call-custom-api" urlDisplayName="从客户端调用自定义 API" pageTitle="从 HTML 客户端调用自定义 API - 移动服务" metaKeywords="" description="了解如何定义自定义 API，然后从使用 Microsoft Azure 移动服务的 HTML 应用程序调用它。" metaCanonical="" services="" documentationCenter="" title="从客户端调用自定义 API" authors=""  solutions="" writer="jparrel" manager="" editor=""  />




# 从 HTML 应用程序调用自定义 API

[WACOM.INCLUDE [mobile-services-selector-call-custom-api](../includes/mobile-services-selector-call-custom-api.md)]

本主题说明如何从 HTML 应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开未映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

使用本主题中创建的自定义 API，你可以发送单个 POST 请求，用于将表中所有 todo 项的 completed 标志设置为 `true`。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

需要将此功能添加到你在完成[移动服务入门]或[数据处理入门]教程时创建的应用程序。为此，你需要完成以下步骤：

1. [定义自定义 API]
2. [更新应用程序以调用自定义 API]
3. [测试应用程序] 

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门]或[数据处理入门]。

## <a name="define-custom-api"></a>定义自定义 API

[WACOM.INCLUDE [mobile-services-create-custom-api](../includes/mobile-services-create-custom-api.md)]

<h2><a name="update-app"></a>更新应用程序以调用自定义 API</h2>

1. 使用文本编辑器打开 index.html 文件，找到名为 `buttonRefresh` 的 **button** 元素，并紧接在该元素的后面添加以下新元素： 

		<button id="buttonCompleteAll">Complete All</button> 

	这样可将新按钮添加到页。 

2. 在 page.js 中 **refreshTodoItems** 函数的后面添加以下代码：

		var completeAllTodoItems = function () {
			// Asynchronously call the custom API using the POST method.
			client.invokeApi("completeall", {
				body: null,
				method: "post"
			}).done(function (results) {
				var message = results.result.count + " item(s) marked as complete.";
				alert(message);
				refreshTodoItems();
			}, function(error) {
				alert(error.message);
			});
		};

		$('#buttonCompleteAll').click(function () {
			completeAllTodoItems();
		});

	此方法可处理新按钮的 **Click** 事件。**invokeApi** 方法在客户端上调用，该客户端向新的自定义 API 发送 POST 请求。与任何错误相同，自定义 API 返回的结果也显示在消息对话框中。

## <a name="test-app"></a>测试应用程序

1. 刷新你的浏览器。

2. 在应用的**"插入 TodoItem"**中键入一些文本，然后单击**"保存"**。

3. 重复上述步骤，直至您将多个 ToDo 项添加到列表。

4. 单击**"完成全部"**按钮。此时会显示一个消息框，指示标记为完成的多个项，并再次执行筛选查询，将所有项从列表中清除。

## 后续步骤

创建自定义 API 并从 HTML 应用程序调用该 API 后，建议你了解有关以下移动服务主题的详细信息：

* [移动服务服务器脚本参考]
  <br/>了解有关创建自定义 API 的详细信息。

<!-- Anchors. -->
[定义自定义 API]: #define-custom-api
[更新应用程序以调用自定义 API]: #update-app
[测试应用程序]: #test-app
[后续步骤]: #next-steps

<!-- URLs. -->
[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
["我的应用程序"仪表板]: http://go.microsoft.com/fwlink/?LinkId=262039
<!-- [Get started with Mobile Services]: /zh-cn/develop/mobile/tutorials/mobile-services-html-get-started -->
