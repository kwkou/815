<properties 
	pageTitle="将图像从通用 Windows 应用上载到 Azure Blob 存储 | Azure" 
	description="了解如何使用 JavaScript 后端移动服务将图像上载到 Azure Blob 存储和从通用 Windows 应用访问图像。" 
	services="mobile-services,storage" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="11/16/2015" 
	wacn.date="01/29/2016"/>

# 使用移动服务将图像上载到 Azure Blob 存储

[AZURE.INCLUDE [mobile-services-selector-upload-data-blob-storage](../includes/mobile-services-selector-upload-data-blob-storage.md)]

##概述 

本主题说明如何借助 Azure 移动服务，使应用程序能够在 Azure 存储空间中上载和存储用户生成的图像。移动服务使用 SQL 数据库存储数据。但是，将二进制大型对象 (BLOB) 数据存储在 Azure Blob 存储服务中可以提高效率。

你无法使用客户端应用程序安全地分发所需的凭据，因此无法安全地将数据上载到 Blob 存储服务。你必须将这些凭据存储在移动服务中，并使用它们来生成用于上载新图像的共享访问签名 (SAS)。移动服务会向客户端应用程序安全返回 SAS（一个凭据，其过期时间较短 &mdash; 在本例中为 5 分钟）。然后，应用程序将使用此临时凭据来上载图像。在此示例中，公众可以从 Blob 服务下载。

在本教程中，你将要向移动服务快速入门应用程序添加功能，使用户能够拍摄照片，并使用移动服务生成的 SAS 将图像上载到 Azure。

##先决条件

本教程需要的内容如下：

+ Microsoft Visual Studio 2013 Update 3 或更高版本
+ [Azure 存储帐户](/documentation/articles/storage-create-storage-account/)
+ 连接到你的计算机的照相机或其他图像捕获设备。

本教程基于移动服务快速入门。在开始本教程之前，必须先[完成移动服务入门]。

##在 Azure 经典门户中更新已注册的插入脚本

[AZURE.INCLUDE [mobile-services-configure-blob-storage](../includes/mobile-services-configure-blob-storage.md)]

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-upload-to-blob-storage](../includes/mobile-services-windows-universal-dotnet-upload-to-blob-storage.md)]

##后续步骤

现在，你已能够通过将移动服务与 Blob 服务集成安全地上载图片，请查看一些其他的后端服务和集成主题：


+ [移动服务服务器脚本参考]

    参考使用服务器脚本执行服务器端任务，并与其他 Azure 组件和外部资源集成的主题。
 
+ [移动服务 .NET 操作方法概念性参考]

    了解有关如何将移动服务与 .NET 一起使用的详细信息
  
 
<!-- Anchors. -->
[Install the Storage Client library]: #install-storage-client
[Update the client app to capture images]: #add-select-images
[Update the insert script to generate an SAS]: #update-scripts
[Upload images to test the app]: #test
[Next Steps]: #next-steps

<!-- Images. -->

[2]: ./media/mobile-services-windows-store-dotnet-upload-data-blob-storage/mobile-add-storage-nuget-package-dotnet.png


<!-- URLs. -->
[Send email from Mobile Services with SendGrid]: /documentation/articles/store-sendgrid-mobile-services-send-email-scripts/

[Send push notifications to Windows Store apps using Service Bus from a .NET back-end]: http://go.microsoft.com/fwlink/?LinkId=277073&clcid=0x409
[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[完成移动服务入门]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/

[Azure Management Portal]: https://manage.windowsazure.cn
[How To Create a Storage Account]: /documentation/articles/storage-create-storage-account/
[Azure Storage Client library for Store apps]: http://go.microsoft.com/fwlink/p/?LinkId=276866
[移动服务 .NET 操作方法概念性参考]: /documentation/articles/mobile-services-dotnet-how-to-use-client-library/
[App settings]: http://msdn.microsoft.com/zh-cn/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7
 

<!---HONumber=Mooncake_0118_2016-->