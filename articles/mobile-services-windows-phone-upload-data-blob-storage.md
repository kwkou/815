<properties 
	pageTitle="将图像从 Windows Phone Silverlight 应用上载到 Azure 存储空间 | Windows Azure" 
	description="了解如何使用移动服务将图像从 Windows Phone Silverlight 应用上载到 Azure Blob 存储。" 
	documentationCenter="windows" 
	authors="ggailey777" 
	services="mobile-services" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="07/21/2015" 
	wacn.date="10/22/2015"/>

#  使用移动服务将图像上载到 Azure 存储空间

[AZURE.INCLUDE [mobile-services-selector-upload-data-blob-storage](../includes/mobile-services-selector-upload-data-blob-storage.md)]

## 概述

本主题说明如何借助 Azure 移动服务，使应用程序能够在 Azure 存储空间中上载和存储用户生成的图像。移动服务使用 SQL 数据库存储数据。但是，将二进制大型对象 (BLOB) 数据存储在 Azure Blob 存储服务中可以提高效率。

你无法使用客户端应用程序安全地分发所需的凭据，因此无法安全地将数据上载到 Blob 存储服务。你必须将这些凭据存储在移动服务中，并使用它们来生成用于上载新图像的共享访问签名 (SAS)。移动服务会向客户端应用程序安全返回 SAS（一个凭据，其过期时间较短 &mdash; 在本例中为 5 分钟）。然后，应用程序将使用此临时凭据来上载图像。在此示例中，公众可以从 Blob 服务下载。

在本教程中，你将要向 [GetStartedWithData 示例应用程序项目](/documentation/articles/mobile-services-windows-phone-get-started-data)添加功能，使用户能够拍摄照片，并使用移动服务生成的 SAS 将图像上载到 Azure。


## 先决条件

本教程需要的内容如下：

+ Microsoft Visual Studio 2012 Express for Windows 8 或更高版本
+ [Windows Phone SDK 8.0] 或更高版本
+ 为 Microsoft Visual Studio 安装 Nuget Package Manager。
+ [Azure 存储帐户][How To Create a Storage Account]
+ 完成教程[将移动服务添加到现有应用程序](/documentation/articles/mobile-services-windows-phone-get-started-data)  


## 安装 Windows Phone 应用程序的存储客户端

若要使用 SAS 将图像上载到 Blob 存储，必须先添加 NuGet 包，该包用于安装 Windows Phone 应用程序的存储客户端库。

1. 在 Visual Studio 的“解决方案资源管理器”中，右键单击项目名称，然后选择“管理 NuGet 包”。

2. 在左窗格中，选择“联机”类别，选择“包括预发行版”，搜索 **WindowsAzure.Storage-Preview**，在“Azure 存储空间”包上单击“安装”，然后接受许可协议。

  	![添加 Azure 存储空间 NuGet](./media/mobile-services-windows-phone-upload-data-blob-storage/mobile-add-storage-nuget-package-dotnet.png)

  	随即会将 Azure 存储服务的客户端库添加到项目。

接下来，你要更新快速入门应用程序以捕获和上载图像。

## 在管理门户中更新已注册的插入脚本


[AZURE.INCLUDE [mobile-services-configure-blob-storage](../includes/mobile-services-configure-blob-storage.md)]

>[AZURE.NOTE]若要将新属性添加到 TodoItem 对象，您必须在移动服务中启用动态架构。启用动态架构时，新列自动添加到映射到这些新属性的 TodoItem 表。

[AZURE.INCLUDE [mobile-services-windows-phone-upload-to-blob-storage](../includes/mobile-services-windows-phone-upload-to-blob-storage.md)]


## 后续步骤

现在，你已能够通过将移动服务与 Blob 服务集成安全地上载图片，请查看一些其他的后端服务和集成主题：

+ [使用 SendGrid 从移动服务发送电子邮件]
 
  了解如何使用 SendGrid 电子邮件服务为你的移动服务添加电子邮件功能。本主题演示如何添加服务器端脚本，以使用 SendGrid 发送电子邮件。

+ [在移动服务中计划后端作业]

  了解如何使用移动服务作业计划程序功能，定义按你定义的计划执行的服务器脚本代码。

## 另请参阅

+ [移动服务服务器脚本参考]

  参考使用服务器脚本执行服务器端任务，并与其他 Azure 组件和外部资源集成的主题。
 
+ [移动服务 .NET 操作方法概念性参考]

  了解有关如何将移动服务与 .NET 一起使用的详细信息
  
<!-- Images. -->

<!-- URLs. -->
[使用 SendGrid 从移动服务发送电子邮件]: /documentation/articles/store-sendgrid-mobile-services-send-email-scripts
[在移动服务中计划后端作业]: /documentation/articles/mobile-services-schedule-recurring-tasks
[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts
[Get started with Mobile Services]: /documentation/articles/mobile-services-windows-phone-get-started
[Azure Management Portal]: https://manage.windowsazure.cn/
[How To Create a Storage Account]: /documentation/articles/storage-create-storage-account
[Azure Storage Client library for Store apps]: http://go.microsoft.com/fwlink/p/?LinkId=276866
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[Windows Phone SDK 8.0]: http://www.microsoft.com/zh-cn/download/details.aspx?id=35471

<!---HONumber=74-->