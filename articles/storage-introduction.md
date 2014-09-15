<properties linkid="storage-introduction" urlDisplayName="Introduction to Azure Storage" pageTitle="Introduction to Storage | Microsoft Azure" metaKeywords="Get started  Azure storage introduction  Azure storage overview  Azure blob   Azure unstructured data   Azure unstructured storage   Azure blob   Azure blob storage  Azure queue   Azure asynchronous processing   Azure queue   Azure queue storage Azure table   Azure nosql   Azure large structured data store   Azure table   Azure table storage   Azure " description="An overview of Microsoft Azure Storage." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter="" title="Introduction to Microsoft Azure Storage" authors="tamram" manager="mbaldwin" editor="cgronlun" />

# Microsoft Azure 存储空间简介

本文面向开发人员、IT 专业人员和业务决策者介绍了 Microsoft Azure 存储空间。通过阅读本文，你将会了解：

-   什么是 Azure 存储空间，以及如何在你的云、移动、服务器和桌面应用程序中利用它
-   你可以使用 Azure 存储服务存储什么类型的数据：Blob、表和队列存储
-   在 Azure 存储空间中如何管理对你的数据的访问
-   如何通过冗余和复制保护你的 Azure 存储数据
-   接下来要到何处去构建你的第一个 Azure 存储应用程序

## 什么是 Azure 存储空间？

对于需要为其数据使用可伸缩的、持久的且具有高可用性的存储的应用程序，云计算使其有了新的方案可供选择，这正是 Microsoft 开发 Azure 存储空间的原因。除了使开发人员可以构建大型应用程序来支持新方案之外，Azure 存储空间还为 Azure 虚拟机提供了存储基础，进一步证明其可靠性。

Azure 存储空间可以大规模伸缩，因此你可以存储和处理数百 TB 的数据来支持科学、财务分析和媒体应用程序所需的大数据方案。你也可以存储小型商业网站所需的少量数据。当你的需求降低时，你只需要为你当前存储的数据支付费用。Azure 存储空间当前存储了数十万亿个唯一的客户对象，平均每秒处理数百万个请求。

Azure 存储空间是弹性的，因此你可以针对大量的全球受众设计应用程序，并根据需要伸缩这些应用程序 - 在存储的数据量和针对它发出的请求数两个方面。你只需要为你使用的内容付费，并且只需要在你使用它时付费。

Azure 存储空间使用了一个自动分区系统，它可以根据流量自动对你的数据进行负载平衡。这意味着当你的应用程序上的需求增长时，Azure 存储空间会自动分配合适的资源来满足这些需求。

可以从世界上的任何位置从任何类型的应用程序访问 Azure 存储空间，无论该应用程序是在云中、在桌面上、在本地服务器上运行还是在移动设备或平板电脑设备上运行。你可以将 Azure 存储空间用于移动方案，在这类方案中，应用程序在设备上存储一部分数据，并将其与存储在云中的完整数据集进行同步。

Azure 存储空间支持使用各种操作系统（包括 Windows 和 Linux）及各种编程语言（包括 .NET、Java 和 C++）的客户端以方便开发。Azure 存储空间还通过简单的 REST API 公开数据资源，这些 REST API 可供能够通过 HTTP/HTTPS 发送和接收数据的任何客户端使用。

## Azure 存储服务介绍

Azure 存储服务包括 Blob 存储、表存储和队列存储。每个存储帐户中都包括这三项服务：

-   **Blob 存储**用于存储文件数据。Blob 可以是任何类型的文本或二进制数据，例如文档、媒体文件或应用程序安装程序。
-   **表存储**用于存储结构化数据集。表存储是一个 NoSQL“键-属性”数据存储，可以用于实现快速开发以及快速访问大量数据。
-   **队列存储**为云服务的各个组件之间的工作流处理和通信提供可靠的消息传送。

存储帐户是一个唯一的命名空间，它向你授予对 Azure 存储空间的访问权限。每个存储帐户可以容纳高达 500 TB 混合的 Blob、队列和表数据。

下图显示了各种 Azure 存储资源之间的关系：

![Azure 存储资源][]

你必须具有 Azure 订阅（这是允许你访问各种 Azure 服务的计划），然后才能创建存储帐户。通过单个订阅，你最多可以创建 20 个唯一的命名存储帐户。

你可以从[免费试用版][]开始使用 Azure。一旦决定购买某个计划，你可以从各种[购买选项][]进行选择。如果你是 [MSDN 订户][]，则可以获得免费的月度信用，你可以将其用于各种 Azure 服务，包括 Azure 存储空间。

## Blob 存储

对于有大量非结构化数据要存储在云中的用户，Blob 存储提供了一种经济高效且可伸缩的解决方案。你可以使用 Blob 存储来存储如下内容：

-   文档
-   社交数据，例如照片、视频、音乐和博客
-   文件、计算机、数据库和设备的备份
-   Web 应用程序的图像和文本
-   云应用程序的配置数据
-   大数据，例如日志和其他大型数据集

每个 Blob 都组织到一个容器中。容器还提供了一种有用的方式来向对象组分配安全策略。一个存储帐户可以包含任意数目的容器，一个容器可以包含任意数目的 Blob，直至达到存储帐户的容量限制 500 TB。

Blob 存储提供了两种类型的 Blob：块 Blob 和页 Blob（磁盘）。块 Blob 进行了相应的优化来流化和存储云对象，并且是用于存储文档、介质文件和备份等对象的不错选择。块 Blob 最大可以为 200 GB。页 Blob 进行了相应的优化来表示 IaaS 磁盘和支持随机写入，并且最大可以为 1 TB。Azure 虚拟机网络连接的 IaaS 磁盘是一个 VHD，存储为页 Blob。

对于网络限制使得通过线缆向 Blob 存储上载或从其下载数据不可行的每个大型数据集，你可以将硬盘驱动器发运到 Microsoft 以使用 [Azure 导入/导出服务][]直接通过数据中心导入或导出数据。你还可以在存储帐户内或者在存储帐户之间复制 Blob 数据。

## 表存储

与前几代的必需软件相比，现代应用程序通常要求数据存储具有更高的可伸缩性和灵活性。表存储提供了具有高可用性且可大规模伸缩的存储，因此你的应用程序可以自动伸缩以满足用户需求。表存储是 Microsoft 的 NoSQL 键/属性存储 - 它具有无模式的设计，使其不同于传统的关系数据库。采用无模式的数据存储，可以很容易地随着你的应用程序需求的发展使数据适应存储。表存储易于使用，因此开发人员可以快速创建应用程序。对于所有类型的应用程序，都可以快速并经济高效地访问数据。对于相似的数据量，表存储的成本通常显著低于传统的 SQL。

表存储是一种“键-属性”存储，这意味着表中的每个值都是随所键入的一个属性名称存储的。属性名称可以用来筛选和指定选择条件。属性集合及其值构成了实体。因为表存储是无模式的，因此同一表中的两个实体可以包含不同的属性集合，并且这些属性可以属于不同的类型。

你可以使用表存储来存储灵活的数据集，例如 Web 应用程序的用户数据、通讯簿、设备信息，以及你的服务需要的任何其他类型的元数据。你可以在一个表中存储任意数目的实体，并且一个存储帐户可以包含任意数目的表，直至达到存储帐户的容量限制 200 TB。

像 Blob 和队列一样，开发人员可以使用标准 REST 协议来管理和访问表存储，不过，表存储还支持 OData 协议的一个子集，这简化了高级查询功能并支持 JSON 和 AtomPub（基于 XML）格式。

对于当前的基于 Internet 的应用程序，NoSQL 数据库（例如表存储）提供了一种用于替代传统的关系数据库的主流方式。

## 队列存储

在设计应用程序以实现可伸缩性时，通常要将各个应用程序组件分离，使其可以独立地进行伸缩。队列存储为在应用程序组件之间进行异步通信提供了一种可靠的消息传送解决方案，无论这些应用程序组件是在云中、在桌面上、在本地服务器上运行还是在移动设备上运行。队列存储还支持管理异步任务以及构建过程工作流。

一个存储帐户可以包含任意数目的队列。一个队列可以包含任意数目的消息，直至达到存储帐户的容量限制 200 TB。每条消息最大可以为 64 KB。

## 对 Blob、表和队列资源的访问

默认情况下，只有存储帐户所有者可以访问存储帐户中的资源。为保证你的数据的安全性，对你帐户中的资源发出的每个请求都必须进行身份验证。身份验证依赖于一个共享密钥模型。还可以将 Blob 配置为支持异步身份验证。

在创建你的存储帐户时为其分配了两个用于身份验证的私有访问密钥。设置两个密钥可以确保你的应用程序在你定期重新生成密钥（这是一种常用的安全密钥管理做法）时仍然保持可用。

如果你不需要为你的存储资源实施用户受控访问，则可以创建一个[共享访问签名][]。共享访问签名是一个可以附加到 URL 的令牌，可以实现对容器、Blob、表或队列的委托访问。持有令牌的任何人都可以在令牌有效期间使用它指定的权限访问它指向的资源。

最后，你可以指定一个容器及其 Blob 或某个特定的 Blob 可供公开访问。当你指定某个容器或 Blob 为公用的时，任何人都可以匿名读取它，不需要进行身份验证。公用容器和 Blob 非常适用于公开在网站上托管的资源，例如媒体和文档。若要降低全球受众的网络延迟，你可以通过 Azure CDN 来缓存网站使用的 Blob 数据。

## 用于实现持久性和高可用性的复制

可以对你存储帐户中的数据进行复制以确保数据的持久性及其高可用性，即使在发生短暂的硬件故障时也可以满足 [Azure 存储 SLA][]。Azure 存储空间部署在世界各地的 15 个区域中并且还支持在各个区域之间复制数据。你有三个选项用于复制你的存储帐户中的数据：

-   **本地冗余存储 (LRS)，将在单个数据中心内复制三次。当你向 Blob、队列或表写入数据时，将在所有三个副本中同步执行写入操作。LRS 可以保护你的数据免受普通硬件故障的危害。
-   **地域冗余存储 (GRS)，将在单个区域内复制三次，并且还将异步复制到离主区域数百英里的另一个区域中。GRS 将为你的数据保留 6 个相同副本（每个区域中 3 个）。GRS 使 Microsoft 在由于严重中断或灾难而无法还原第一个区域时能够故障转移到另一个区域。建议使用 GRS 而非本地冗余存储。
-   **读取访问地域冗余存储 (RA-GRS)，具有上面所述的地域冗余存储的所有优点，当主区域变得不可用时还可对第二个区域中的数据进行读取访问。要实现最大可用性以及持久性，建议使用读取访问地域冗余存储。

可以在[存储定价详细信息][]页上找到 LRS、GRS 和 RA-GRS 之间的价格差异。

## 定价

对于 Azure 存储空间，将根据四个因素向客户收费：使用的存储容量、选择的复制选项、对服务发出的请求数，以及数据流出量。

存储容量指的是存储帐户中用来存储数据的配额。对数据进行简单存储时，其成本取决于存储的数据量和数据复制方式。针对 Azure 存储空间的每个读取和写入操作还将针对服务发出一个请求。数据流出量是指从某个 Windows Azure 区域传出的数据。当不在同一区域中的应用程序访问你的存储帐户中的数据时，无论该应用程序是云服务还是某个其他类型的应用程序，都将会针对数据流出量向你收费。（对于 Windows Azure 服务，你可以采取措施将数据和服务通过分组分到相同的数据中心内，从而降低或避免进程和数据流出量费用。）

[存储定价详细信息][]页提供了针对存储容量、复制和事务的详细定价信息。[数据传输定价详细信息][]提供了针对数据流出量的详细定价信息。你可以使用 [Azure 存储空间定价计算器][]来帮助估算成本。

## 针对存储进行开发

Azure 存储空间通过一个 [REST API][] 来公开存储资源，任何可以发出 HTTP/HTTPS 请求的语言都可以调用该 REST API。另外，Azure 存储空间还为多种主流语言提供了编程库。这些库通过对细节进行处理简化了使用 Azure 存储空间的许多方面，这些细节包括同步和异步调用、操作的批处理、异常管理、自动重试、操作行为，等等。这些库当前可供下列语言和平台以及正在筹备的其他语言和平台使用：

-   [.NET][]
-   [本机代码][]
-   [Java][]
-   [Node.js][]
-   [PHP][]
-   [Ruby][]
-   [Python][]
-   [PowerShell][]

## 后续步骤

在开始使用 Azure 存储空间之前，请浏览以下资源：

-   [Azure 存储文档][]
-   [Azure 存储空间可伸缩性和性能目标][]

### 对于 .NET 开发人员

-   [如何通过 .NET 使用 Blob 存储][]
-   [如何通过 .NET 使用表存储][]
-   [如何通过 .NET 使用队列存储][]

### 对于 Java 开发人员

-   [如何通过 Java 使用 Blob 存储][]
-   [如何通过 Java 使用表存储][]
-   [如何通过 Java 使用队列存储][]

### 对于 Node.js 开发人员

-   [如何通过 Node.js 使用 Blob 存储][]
-   [如何通过 Node.js 使用表存储][]
-   [如何通过 Node.js 使用队列存储][]

### 对于 PHP 开发人员

-   [如何通过 PHP 使用 Blob 存储][]
-   [如何通过 PHP 使用表存储][]
-   [如何通过 PHP 使用队列存储][]

### 对于 Ruby 开发人员

-   [如何通过 Ruby 使用 Blob 存储][]
-   [如何通过 Ruby 使用表存储][]
-   [如何通过 Ruby 使用队列存储][]

### 对于 Python 开发人员

-   [如何通过 Python 使用 Blob 存储][]
-   [如何通过 Python 使用表存储][]
-   [如何通过 Python 使用队列存储][]

  [Azure 存储资源]: ./media/storage-introduction/storage-concepts.png
  [免费试用版]: /en-us/pricing/free-trial/
  [购买选项]: /en-us/pricing/purchase-options/
  [MSDN 订户]: /en-us/pricing/member-offers/msdn-benefits-details/
  [Azure 导入/导出服务]: http://azure.microsoft.com/zh-cn/documentation/articles/storage-import-export-service/
  [共享访问签名]: ../storage-dotnet-shared-access-signature-part-1/
  [Azure 存储 SLA]: /en-us/support/legal/sla/
  [存储定价详细信息]: /en-us/pricing/details/storage/
  [数据传输定价详细信息]: /en-us/pricing/details/data-transfers/
  [Azure 存储空间定价计算器]: /en-us/pricing/calculator/?scenario=data-management
  [REST API]: http://msdn.microsoft.com/library/windowsazure/dd179355.aspx
  [.NET]: http://msdn.microsoft.com/zh-cn/library/dn495001.aspx
  [本机代码]: http://msdn.microsoft.com/zh-cn/library/dn495438.aspx
  [Java]: /en-us/develop/java/
  [Node.js]: ../storage/#node
  [PHP]: ../storage/#php
  [Ruby]: ../storage/#ruby
  [Python]: ../storage/#python
  [PowerShell]: http://msdn.microsoft.com/zh-cn/library/dn495240.aspx
  [Azure 存储文档]: /en-us/documentation/services/storage/
  [Azure 存储空间可伸缩性和性能目标]: http://msdn.microsoft.com/zh-cn/library/azure/dn249410.aspx
  [如何通过 .NET 使用 Blob 存储]: ../storage-dotnet-how-to-use-blobs/
  [如何通过 .NET 使用表存储]: ../storage-dotnet-how-to-use-tables/
  [如何通过 .NET 使用队列存储]: ../storage-dotnet-how-to-use-queues/
  [如何通过 Java 使用 Blob 存储]: ../storage-java-how-to-use-blob-storage/
  [如何通过 Java 使用表存储]: ../storage-java-how-to-use-table-storage/
  [如何通过 Java 使用队列存储]: ../storage-java-how-to-use-queue-storage/
  [如何通过 Node.js 使用 Blob 存储]: ../storage-nodejs-how-to-use-blob-storage/
  [如何通过 Node.js 使用表存储]: ../storage-nodejs-how-to-use-table-storage/
  [如何通过 Node.js 使用队列存储]: ../storage-nodejs-how-to-use-queues/
  [如何通过 PHP 使用 Blob 存储]: ../storage-php-how-to-use-blobs/
  [如何通过 PHP 使用表存储]: ../storage-php-how-to-use-table-storage/
  [如何通过 PHP 使用队列存储]: ../storage-php-how-to-use-queues/
  [如何通过 Ruby 使用 Blob 存储]: ../storage-ruby-how-to-use-blob-storage/
  [如何通过 Ruby 使用表存储]: ../storage-ruby-how-to-use-table-storage/
  [如何通过 Ruby 使用队列存储]: ../storage-ruby-how-to-use-queue-storage/
  [如何通过 Python 使用 Blob 存储]: ../storage-python-how-to-use-blob-storage/
  [如何通过 Python 使用表存储]: ../storage-python-how-to-use-table-storage/
  [如何通过 Python 使用队列存储]: ../storage-python-how-to-use-queue-storage/
