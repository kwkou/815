<properties title="Getting Started with Azure Storage" pageTitle="Getting Started with Azure Storage" metaKeywords="Azure, Getting Started, Storage" description="" services="storage" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="storage" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/10/2014" ms.author="ghogen, kempb"></tags>

[WACOM.INCLUDE [vs-storage-aspnet-getting-started-intro][vs-storage-aspnet-getting-started-intro]]

### Azure 存储入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-blobs" title="Blobs" class="current">Blobs</a><a href="/zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-queues" title="Queues">Queues</a><a href="/zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-tables" title="Tables">Tables</a></div>

Azure 队列存储是一项可存储大量消息的服务，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。一条队列消息的大小可达 64 KB，一个队列中可以包含数百万条消息，直至达到存储帐户的总容量限值。有关详细信息，请参阅[如何通过 .NET 使用队列存储][如何通过 .NET 使用队列存储]。

若要以编程方式访问云服务项目中的队列，您需要执行以下任务。

1.  获取 Microsoft.WindowsAzure.Storage.dll 程序集。您可以使用 NuGet 来执行此操作。在“解决方案资源管理器”中，右键单击您的项目并选择“管理 NuGet 包”。在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装 Azure 存储包和依赖项。将对此程序集的引用添加到您的项目。
2.  在您希望以编程方式访问 Azure 存储的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部。

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Queue;

###### 获取存储连接字符串

在您可以使用队列执行任何操作之前，您需要先获取将存放队列的存储帐户的连接字符串。您可以使用 **CloudStorageAccount** 类型来表示您的存储帐户信息。对于云服务项目，您可以使用 **CloudConfigurationManager** 类型从 Azure 服务配置检索您的存储连接字符串和存储帐户信息，如以下代码所示。

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
      CloudConfigurationManager.GetSetting("<storageAccountName>_AzureStorageConnectionString"));

[WACOM.INCLUDE [vs-storage-getting-started-queues-include][vs-storage-getting-started-queues-include]]

  [vs-storage-aspnet-getting-started-intro]: ../includes/vs-storage-aspnet-getting-started-intro.md
  [Blobs]: /zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-blobs "Blobs"
  [Queues]: /zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-queues "Queues"
  [Tables]: /zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-tables "Tables"
  [如何通过 .NET 使用队列存储]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/ "如何通过 .NET 使用队列存储"
  [vs-storage-getting-started-queues-include]: ../includes/vs-storage-getting-started-queues-include.md
