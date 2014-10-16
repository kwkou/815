<properties pageTitle="Add paging to data (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to use paging to manage the amount of data returned to your Windows Phone app from Mobile Services." metaCanonical="" services="" documentationCenter="Mobile" title="Refine Mobile Services queries with paging" authors="glenga" solutions="" manager="" editor="" />

# 使用分页优化移动服务查询

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-add-paging-data" title="Windows Store C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-add-paging-data" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-add-paging-data" title="Windows Phone" class="current">Windows Phone</a></div>
<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-add-paging-data" title=".NET backend" class="current">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-add-paging-data"  title="JavaScript backend">JavaScript 后端</a></div>


本主题说明如何使用分页来管理从 Azure 移动服务返回给 Windows Phone 应用程序的数据量。在本教程中，你将在客户端上使用 "Take" 和 "Skip" 查询方法来请求特定的数据“页”。

> [WACOM.NOTE] 为了防止移动设备客户端上发生数据溢出，移动服务实施了自动页限制，该限制默认为每个响应中最多 50 个项。通过指定页大小，你最多可以在响应中显式请求 1,000 个项。

本教程以前一教程[数据处理入门][]中的步骤和示例应用程序为基础。在开始学习本教程之前，最起码需要先完成数据处理系列中的第一篇教程，即[数据处理入门][]。

[WACOM.INCLUDE [mobile-services-windows-dotnet-paging](../includes/mobile-services-windows-dotnet-paging.md)]

<a name="next-steps"> </a>
## 后续步骤

演示移动服务中数据处理基础知识的系列教程到此结束。建议你了解有关以下移动服务主题的详细信息：

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

  [Windows 应用商店 C#]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-add-paging-data "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-add-paging-data "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-add-paging-data "Windows Phone"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-add-paging-data ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-phone-add-paging-data "JavaScript 后端"
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-data/
  [mobile-services-windows-dotnet-paging]: ../includes/mobile-services-windows-dotnet-paging.md
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users/
  [推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/
