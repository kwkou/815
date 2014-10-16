<properties linkid="develop-mobile-tutorials-add-paging-to-data-xamarin-android" urlDisplayName="Add paging to data" pageTitle="Add paging to data (Xamarin Android) | Mobile Dev Center" metaKeywords="" description="Learn how to use paging to manage the amount of data returned to your Xamarin Android app from Mobile Services." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Refine Mobile Services queries with paging" authors="" />

# 使用分页优化移动服务查询

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-xamarin-ios" title="Xamarin.iOS">iOS C#</a><a href="/zh-cn/develop/mobile/tutorials/add-paging-to-data-xamarin-android" title="Xamarin.Android" class="current">Android C#</a>
</div>

本主题说明如何使用分页来管理从 Azure 移动服务返回给 Xamarin.Android 应用程序的数据量。在本教程中，你将在客户端上使用 "Take" 和 "Skip" 查询方法来请求特定的数据“页”。

<div class="dev-callout"><b>说明</b>

<p>为了防止移动设备客户端上发生数据溢出，移动服务实施了自动页限制，该限制默认为每个响应中最多 50 个项。通过指定页大小，你最多可以在响应中显式请求 1,000 个项。</p>
</div>

本教程以前一教程[数据处理入门][]中的步骤和示例应用程序为基础。在开始学习本教程之前，最起码需要先完成数据处理系列中的第一篇教程，即[数据处理入门][]。

1.  在 Xamarin Studio 中，打开你在完成[数据处理入门][]教程后创建的项目。

2.  单击“运行” 以启动应用程序，然后在文本框中输入文本，然后单击“添加” 按钮。

3.  重复以上步骤至少三次，因此你将在 TodoItem 表中存储三个以上的项。

4.  在 "TodoActivity.cs" 文件中，将 "RefreshItemsFromTableAsync" 方法中的 LINQ 查询替换为以下查询：

        var list = await todoTable.Where(item => item.Complete == false)
        .Take(3)
        .ToListAsync();

    此查询将返回未标记为已完成的前面三个项。

5.  重新生成并启动应用程序。

    请注意，仅显示 TodoItem 表的前三个结果。

6.  （可选）使用消息检查软件（例如浏览器开发人员工具或 [Fiddler]）来查看发送到移动服务的请求的 URI。

    请注意，`Take(3)` 方法已转换成查询 URI 中的查询选项 `$top=3`。

7.  使用以下查询再次更新 "RefreshItemsFromTableAsync" 方法中的 LINQ 查询：

            var list = await todoTable.Where(item => item.Complete == false)
        .Skip(3)
        .Take(3)
        .ToListAsync();

    此查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二“页”，其页大小为三个项。

    <div class="dev-callout"><b>说明</b>

    <p>本教程将硬编码分页值传递给 <b>Take</b> 和 <b>Skip</b> 方法，因此使用的是简化的方案。在实际应用程序中，你可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。你还可以调用 <b>IncludeTotalCount</b> 方法，以获取服务器上的可用项总数，以及分页的数据。</p>
	</div>

8.  （可选）再次查看发送到移动服务的请求的 URI。

    请注意，`Skip(3)` 方法已转换成查询 URI 中的查询选项 `$skip=3`。

<a name="next-steps"> </a>
## 后续步骤

演示移动服务中数据处理基础知识的系列教程到此结束。建议你了解有关以下移动服务主题的详细信息：

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-html "HTML"
  [iOS C\#]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-xamarin-ios "Xamarin.iOS"
  [Android C\#]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-xamarin-android "Xamarin.Android"
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android
