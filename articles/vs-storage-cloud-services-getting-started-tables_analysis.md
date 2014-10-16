<properties title="Getting Started with Azure Storage" pageTitle="Getting Started with Azure Storage" metaKeywords="Azure, Getting Started, Storage" description="" services="storage" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="storage" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="09/17/2014" ms.author="ghogen, kempb"></tags>

[WACOM.INCLUDE [vs-storage-cloud-services-getting-started-intro][vs-storage-cloud-services-getting-started-intro]]

### Azure 存储入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-blobs" title="Blobs" class="current">Blobs</a><a href="/zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-queues" title="Queues">Queues</a><a href="/zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-tables" title="Tables">Tables</a></div>

Azure 表存储服务使用户可以存储大量结构化数据。该服务是一个 NoSQL 数据存储，接受来自 Azure 云内部和外部的通过验证的呼叫。Azure 表最适合存储结构化非关系型数据。有关详细信息，请参阅[如何通过 .NET 使用表存储][如何通过 .NET 使用表存储]。

若要以编程方式访问云服务项目中的表，您需要执行以下任务。

1.  获取 Microsoft.WindowsAzure.Storage.dll 程序集。您可以使用 NuGet 来执行此操作。在“解决方案资源管理器”中，右键单击您的项目并选择“管理 NuGet 包”。在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装 Azure 存储包和依赖项。将对此程序集的引用添加到您的项目。
2.  在您希望以编程方式访问 Azure 存储的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部。

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Queue;

###### 获取存储连接字符串

在您可以使用表执行任何操作之前，您需要先获取将存放表的存储帐户的连接字符串。您可以使用 **CloudStorageAccount** 类型来表示您的存储帐户信息。对于云服务项目，您可以使用 **CloudConfigurationManager** 类型从 Azure 服务配置检索您的存储连接字符串和存储帐户信息，如以下代码所示。

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
      CloudConfigurationManager.GetSetting("<storageAccountName>_AzureStorageConnectionString"));

[WACOM.INCLUDE [vs-storage-getting-started-tables-include][vs-storage-getting-started-tables-include]]

  [vs-storage-cloud-services-getting-started-intro]: ../includes/vs-storage-cloud-services-getting-started-intro.md
  [Blobs]: /zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-blobs "Blobs"
  [Queues]: /zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-queues "Queues"
  [Tables]: /zh-cn/documentation/articles/vs-storage-cloud-services-getting-started-tables "Tables"
  [如何通过 .NET 使用表存储]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/#create-table "如何通过 .NET 使用表存储"
  [vs-storage-getting-started-tables-include]: ../includes/vs-storage-getting-started-tables-include.md
