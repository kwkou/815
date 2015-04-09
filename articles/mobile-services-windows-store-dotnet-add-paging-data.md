<properties pageTitle="使用分页优化移动服务查询 (Windows Store) | 移动开发人员中心" metaKeywords="" description="本主题说明如何使用分页来管理从移动服务返回给 Windows 应用商店应用程序的数据量。" metaCanonical="" services="" documentationCenter="Mobile" title="Refine Mobile Services queries with paging" authors="glenga" solutions="" manager="" editor="" />



# 使用分页优化移动服务查询

[WACOM.INCLUDE [mobile-services-selector-add-paging-data](../includes/mobile-services-selector-add-paging-data.md)]

本主题说明如何使用分页来管理从 Azure 移动服务返回给 Windows 应用商店应用程序的数据量。在本教程中，将在客户端上使用 **Take** 和 **Skip** 查询方法来请求特定的数据"页"。

>[WACOM.NOTE]为了防止移动设备客户端上发生数据溢出，移动服务实施了自动页限制，该限制默认为每个响应中最多 50 个项。通过指定页大小，你最多可以在响应中显式请求 1,000 个项。

本教程以前一教程[数据处理入门]中的步骤和示例应用程序为基础。在开始学习本教程之前，最起码需要先完成数据处理系列中的第一篇教程，即[数据处理入门]。 

[WACOM.INCLUDE [mobile-services-windows-dotnet-paging](../includes/mobile-services-windows-dotnet-paging.md)]

## <a name="next-steps"> </a>后续步骤

演示移动服务中数据处理基础知识的系列教程到此结束。建议进一步了解以下移动服务主题：

* [身份验证入门]
  <br/>了解如何使用 Windows 帐户对应用程序的用户进行身份验证。

* [推送通知入门] 
  <br/>了解如何将非常简单的推送通知发送到您的应用程序。
  
* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。
  
<!-- Anchors. -->

[后续步骤]:#next-steps

<!-- Images. -->


<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push/

[管理门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
