<properties 
	pageTitle="从 HTML 客户端调用自定义 API - 移动服务" 
	description="了解如何定义自定义 API，然后从使用 Azure 移动服务的 HTML 应用程序调用它。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="bureado"  
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/04/2015" 
	wacn.date="10/03/2015/>

#  从 HTML 应用程序调用自定义 API

[AZURE.INCLUDE [mobile-services-selector-call-custom-api](../includes/mobile-services-selector-call-custom-api.md)]

本主题说明如何从 HTML 应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

本主题中创建的自定义 API 使你能够发送单个 POST 请求，将已完成的标记设置为 `true` 的表中的所有 todo 项。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门]或[将移动服务添加到现有应用程序]。

##  <a name="define-custom-api"></a>定义自定义 API

[AZURE.INCLUDE [mobile-services-create-custom-api](../includes/mobile-services-create-custom-api.md)]

## <a name="update-app"></a>更新应用程序以调用自定义 API

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

##  <a name="test-app"></a>测试应用程序

1. 刷新你的浏览器。

2. 在应用的“插入 TodoItem”中键入一些文本，然后单击“保存”。

3. 重复上述步骤，直至您将多个 ToDo 项添加到列表。

4. 单击“完成全部”按钮。此时会显示一个消息框，指示标记为完成的多个项，并再次执行筛选查询，将所有项从列表中清除。

##  后续步骤

本主题说明了如何使用 **invokeApi** 函数从 HTML/JavaScript 应用程序调用一个相当简单的自定义 API。若要了解有关使用 **invokeApi** 函数的详细信息，请参阅文章 [Azure 移动服务中的自定义 API](http://blogs.msdn.com/b/carlosfigueira/archive/2013/06/19/custom-api-in-azure-mobile-services-client-sdks.aspx)。

另外，建议进一步了解以下移动服务主题：

* [移动服务服务器脚本参考]<br/>了解有关创建自定义 API 的详细信息。

* [在源代码管理中存储服务器脚本]<br/>了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. -->

[Define the custom API]: #define-custom-api
[Update the app to call the custom API]: #update-app
[Test the app]: #test-app
[Next Steps]: #next-steps

<!-- URLs. -->

[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts
[移动服务入门]: /documentation/articles/mobile-services-html-get-started
[将移动服务添加到现有应用程序]: /documentation/articles/mobile-services-html-get-started-data
[在源代码管理中存储服务器脚本]: /documentation/articles/mobile-services-store-scripts-source-control
<!---HONumber=71-->