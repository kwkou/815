<properties linkid="develop-mobile-tutorials-add-paging-to-data-html" urlDisplayName="Add paging to data (HTML5)" pageTitle="为数据添加分页 (HTML 5) |移动开发人员中心" metaKeywords="" description="了解如何使用分页来管理从移动服务返回给 HTML 应用程序的数据量。" metaCanonical="" services="" documentationCenter="Mobile" title="Refine Mobile Services queries with paging" authors="glenga" solutions="" manager="" editor="" />




# 使用分页优化移动服务查询

[WACOM.INCLUDE [mobile-services-selector-add-paging-data](../includes/mobile-services-selector-add-paging-data.md)]

本主题说明如何使用分页来管理从 Azure 移动服务返回给 HTML 应用程序的数据量。在本教程中，你将在客户端上使用 **Take** 和 **Skip** 查询方法来请求特定的数据"页"。

> [AZURE.NOTE] 为了防止移动设备客户端上发生数据溢出，移动服务实施了自动页限制，该限制默认为每个响应中最多 50 个项。通过指定页大小，你最多可以在响应中显式请求 1,000 个项。

本教程以前一教程[数据处理入门]中的步骤和示例应用程序为基础。在开始学习本教程之前，最起码需要先完成数据处理系列中的第一篇教程，即[数据处理入门]。 

1. 从你在完成教程[数据处理入门]后修改的项目的 **server** 子文件夹中运行下列命令文件之一。

	+ **launch-windows**（Windows 计算机） 
	+ **launch-mac.command**（Mac OS X 计算机）
	+ **launch-linux.sh**（Linux 计算机）

	> [AZURE.NOTE] 在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入"R"。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。

	随后将在本地计算机上启动用于托管应用程序的 Web 服务器。

1. 在 Web 浏览器中，导航到 <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a>，在**"添加新任务"**中键入文本，然后单击**"添加"**。

3. 重复以上步骤至少三次，因此您将在 TodoItem 表中存储三个以上的项。 

2. 在 app.js 文件中，将 **refreshTodoItems** 方法中  `query` 变量的定义替换为以下代码行：

       
        var query = todoItemTable.where({ complete: false }).take(3);

  	执行此查询会返回未标记为已完成的前面三个项。

3. 返回到 Web 浏览器并重新加载页。

  	请注意，仅显示 TodoItem 表的前三个结果。 

4. （可选）使用消息检查软件（例如浏览器开发人员工具或 [Fiddler]）来查看发送到移动服务的请求的 URI。 

   	请注意，**take(3)** 方法已转换成查询 URI 中的查询选项 **$top=3**。

5. 使用以下代码再次更新查询：
            
        var query = todoItemTable.where({ complete: false }).skip(3).take(3);

3. 返回到 Web 浏览器并重新加载页。

   	此查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二"页"，其页大小为三个项。

    > [AZURE.NOTE] 本教程将硬编码分页值传递给 **Take** 和 **Skip** 方法，因此使用的是简化的方案。在实际应用中，您可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。您还可以调用 **includeTotalCount** 方法，以获取服务器上的可用项总数以及分页的数据。

6. （可选）再次查看发送到移动服务的请求的 URI。 

   	请注意，**skip(3)** 方法已转换成查询 URI 中的查询选项 **$skip=3**。

## <a name="next-steps"> </a>后续步骤

演示移动服务中数据处理基础知识的系列教程到此结束。接下来，请在[身份验证入门]中了解如何对应用程序的用户进行身份验证。请在[移动服务 HTML/JavaScript 操作方法概念性参考]中了解有关如何将移动服务与 HTML/JavaScript 一起使用的详细信息

<!-- Anchors. -->

[Next Steps]:#next-steps

<!-- Images. -->


<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-html
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-html
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-html


[管理门户]: https://manage.windowsazure.cn/
[移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library

