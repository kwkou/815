<properties pageTitle="Use Mobile Services to upload images to blob storage (Windows Store) | Mobile Services" metaKeywords="" description="Learn how to use Mobile Services to upload images to Azure Blob Storage and access the images from your Windows Store app." metaCanonical="" services="" documentationCenter="Mobile" title="Upload images to Azure Storage by using Mobile Services" authors="glenga" solutions="mobile" manager="" editor="" />

# 使用移动服务将图像上载到 Azure 存储空间

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-upload-data-blob-storage" title="Windows Store C#" class="current">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-upload-data-blob-storage" title="Windows Phone">Windows Phone</a></div>
<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-upload-data-blob-storage" title=".NET backend" >.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-upload-data-blob-storage" title="JavaScript backend" class="current">JavaScript 后端</a></div>

本主题说明如何借助 Azure 移动服务，使应用程序能够在 Azure 存储空间中上载和存储用户生成的图像。移动服务使用 SQL Database 存储数据。但是，将二进制大型对象 (BLOB) 数据存储在 Azure Blob 存储服务中可以提高效率。

你无法使用客户端应用程序安全地分发所需的凭据，因此无法安全地将数据上载到 Blob 存储服务。你必须将这些凭据存储在移动服务中，并使用它们来生成用于上载新图像的共享访问签名 (SAS)。移动服务会向客户端应用程序安全返回 SAS（一个凭据，其过期时间较短 - 在本例中为 5 分钟）。然后，应用程序将使用此临时凭据来上载图像。在此示例中，公众可以从 Blob 服务下载。

在本教程中，你将要向移动服务快速入门应用程序添加功能，使用户能够拍摄照片，并使用移动服务生成的 SAS 将图像上载到 Azure。本教程将指导你完成以下基本步骤，让你更新移动服务快速入门项目，以将图像上载到 Blob 存储服务：

1.  [安装存储客户端库][]
2.  [更新插入脚本以生成 SAS][]
3.  [更新客户端应用程序以捕获图像][]
4.  [上载图像以测试应用程序][]

本教程需要的内容如下：

-   Microsoft Visual Studio 2012 Express for Windows 8 或更高版本
-   [Azure 存储帐户][]
-   连接到你的计算机的照相机或其他图像捕获设备。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]。

<a name="install-storage-client"></a>
## 安装 Windows 应用商店应用程序的存储客户端

若要使用 SAS 将图像上载到 Blob 存储，必须先添加 NuGet 包，该包用于安装 Windows 应用商店应用程序的存储客户端库。

1.  在 Visual Studio 的“解决方案资源管理器” 中，右键单击项目名称，然后选择“管理 NuGet 包” 。

2.  在左窗格中，选择“联机” 类别，搜索 `WindowsAzure.Storage`，在“Azure 存储空间” 包上单击“安装” ，然后接受许可协议。

    ![][2]

    随即会将 Azure 存储服务的客户端库添加到项目。

接下来，你要更新快速入门应用程序以捕获和上载图像。

<a name="update-scripts"></a>
## 在管理门户中更新已注册的插入脚本

[WACOM.INCLUDE [mobile-services-configure-blob-storage](../includes/mobile-services-configure-blob-storage.md)]

[WACOM.INCLUDE [mobile-services-windows-store-dotnet-upload-to-blob-storage](../includes/mobile-services-windows-store-dotnet-upload-to-blob-storage.md)]

<a name="next-steps"> </a>
## 后续步骤

现在，你已能够通过将移动服务与 Blob 服务集成安全地上载图片，请查看一些其他的后端服务和集成主题：

-   [使用 SendGrid 从移动服务发送电子邮件][]

了解如何使用 SendGrid 电子邮件服务为你的移动服务添加电子邮件功能。本主题演示如何添加服务器端脚本，以使用 SendGrid 发送电子邮件。

-   [在移动服务中计划后端作业][]

了解如何使用移动服务作业计划程序功能，定义按你定义的计划执行的服务器脚本代码。

-   [移动服务服务器脚本参考][]

参考使用服务器脚本执行服务器端任务，并与其他 Azure 组件和外部资源集成的主题。

-   [移动服务 .NET 操作方法概念性参考][]

了解有关如何将移动服务与 .NET 一起使用的详细信息

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-upload-data-blob-storage "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-upload-data-blob-storage "Windows Phone"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-upload-data-blob-storage ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-upload-data-blob-storage "JavaScript 后端"
  [安装存储客户端库]: #install-storage-client
  [更新插入脚本以生成 SAS]: #update-scripts
  [更新客户端应用程序以捕获图像]: #add-select-images
  [上载图像以测试应用程序]: #test
  [Azure 存储帐户]: /zh-cn/manage/services/storage/how-to-create-a-storage-account
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started
  [2]: ./media/mobile-services-windows-store-dotnet-upload-data-blob-storage/mobile-add-storage-nuget-package-dotnet.png
  [mobile-services-configure-blob-storage]: ../includes/mobile-services-configure-blob-storage.md
  [mobile-services-windows-store-dotnet-upload-to-blob-storage]: ../includes/mobile-services-windows-store-dotnet-upload-to-blob-storage.md
  [使用 SendGrid 从移动服务发送电子邮件]: /zh-cn/develop/mobile/tutorials/send-email-with-sendgrid/
  [在移动服务中计划后端作业]: /zh-cn/documentation/articles/mobile-services-schedule-recurring-tasks
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/p/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library
