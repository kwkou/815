<properties title="Getting Started with Azure Storage" pageTitle="Getting Started with Azure Storage" metaKeywords="Azure, Getting Started, Storage" description="" services="storage" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="storage" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/10/2014" ms.author="ghogen, kempb"></tags>

[WACOM.INCLUDE [vs-storage-aspnet-getting-started-intro][vs-storage-aspnet-getting-started-intro]]

### Azure 存储入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/vs-storage-aspnet-getting-started-blobs" title="Blobs" class="current">Blobs</a><a href="/zh-cn/documentation/articles/vs-storage-aspnet-getting-started-queues" title="Queues">Queues</a><a href="/zh-cn/documentation/articles/vs-storage-aspnet-getting-started-tables" title="Tables">Tables</a></div>

Azure Blob 存储是一项可存储大量非结构化数据的服务，用户可在世界任何地方通过 HTTP 或 HTTPS 访问这些数据。单个 Blob 可以是任意大小。Blob 可以是图像、音频和视频文件、原始数据以及文档文件等。

若要开始使用，您需要创建一个 Azure 存储帐户，然后在存储中创建一个或多个容器。例如，您可以将存储命名为“Scrapbook”，然后在该存储中创建名为“图像”的容器以存储图片，然后创建名为“音频”的容器以存储音频文件。创建这些容器后，您可以向它们上载单独的 Blob 文件。有关以编程方式操作 Blob 的详细信息，请参阅[如何通过 .NET 使用 Blob 存储][如何通过 .NET 使用 Blob 存储]。

若要以编程方式访问 ASP.NET 项目中的 Blob，您需要执行以下任务。

1.  获取 Microsoft.WindowsAzure.Storage.dll 程序集。您可以使用 NuGet 来执行此操作。在“解决方案资源管理器”中，右键单击您的项目并选择“管理 NuGet 包”。在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装 Azure 存储包和依赖项。将对此程序集的引用添加到您的项目。
2.  在您希望以编程方式访问 Azure 存储的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部。

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Queue;

###### 获取存储连接字符串

在您可以使用 Blob 执行任何操作之前，您需要先获取将存放 Blob 的存储帐户的连接字符串。您可以使用 **CloudStorageAccount** 类型来表示您的存储帐户信息。对于 ASP.NET 项目，您可以使用 **ConfigurationManager** 类型从 Azure 服务配置检索您的存储连接字符串和存储帐户信息，如以下代码所示。

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
      ConfigurationManager.GetSetting("<storageAccountName>_AzureStorageConnectionString"));

[WACOM.INCLUDE [vs-storage-getting-started-blobs-include][vs-storage-getting-started-blobs-include]]

  [vs-storage-aspnet-getting-started-intro]: ../includes/vs-storage-aspnet-getting-started-intro.md
  [Blobs]: /zh-cn/documentation/articles/vs-storage-aspnet-getting-started-blobs "Blobs"
  [Queues]: /zh-cn/documentation/articles/vs-storage-aspnet-getting-started-queues "Queues"
  [Tables]: /zh-cn/documentation/articles/vs-storage-aspnet-getting-started-tables "Tables"
  [如何通过 .NET 使用 Blob 存储]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/ "如何通过 .NET 使用 Blob 存储"
  [vs-storage-getting-started-blobs-include]: ../includes/vs-storage-getting-started-blobs-include.md
