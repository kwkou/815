<properties linkid="dev-net-storage" urlDisplayName="Windows Azure Storage" pageTitle="存储 - Azure 微软云" metaKeywords="Azure Storage,Azure存储,非结构化,非关系结构化数据,SMB文件共享,复制,身份验证,度量,日志记录,结构化数据, Azure Blob,访问签名,Azure 导入/导出服务,创建存储帐户,访问密钥" description="在 Windows Azure 中配置、监视和缩放存储帐户。使用 Azure 存储服务存储和访问数据。使用 Blob 存储非结构化的二进制和文本数据。使用队列存储客户端可访问的消息。将非关系结构化数据存储在表中。" metaCanonical="" services="Storage" documentationCenter="Services" title="Configure, monitor, and scale your storage account in Azure" authors="" solutions="" manager="" editor="" />
<tags ms.service="Storage"
    ms.date=""
    wacn.date="07/23/2015"
    />

#存储

####在 Windows Azure 中配置、监视和缩放存储帐户

使用 Azure 存储服务存储和访问数据。使用 Blob 存储非结构化的二进制和文本数据。使用队列存储客户端可访问的消息。将非关系结构化数据存储在表中。

####快速链接
-   [服务概述](/home/features/storage)
-   [可交付的解决方案](/solutions/data-management)
-   [定价详细信息](/home/features/storage/#price)

####特色
-   [Azure存储简介](/documentation/articles/storage-introduction)
-   [通过文件存储创建SMB文件共享](/documentation/articles/storage-dotnet-how-to-use-files)
-   [将文件存储在Blob存储中](/documentation/articles/storage-dotnet-how-to-use-blobs)

##教程和指南

###探究
####[Azure 存储简介](/documentation/articles/storage-introduction)
详细了解 Azure 存储的优势。

####[什么是存储帐户？](/documentation/articles/storage-whatis-account)
了解 Azure 存储帐户的功能，包括复制、身份验证、度量和日志记录。

####[将文件存储在 Blob 存储中](/documentation/articles/storage-dotnet-how-to-use-blobs)
以编程方式使用 Azure Blob 服务上载和下载 Blob、在容器中列出 Blob 和执行其他常见方案。

####[使用表存储来存储结构化数据](/documentation/articles/storage-dotnet-how-to-use-tables)
以编程方式使用 Azure 表服务创建表、建立分区、添加数据和查询数据。

####[使用队列存储来可靠存储和访问消息](/documentation/articles/storage-dotnet-how-to-use-queues)
使用 Azure 队列服务创建队列、将消息插入队列中以及读取和处理消息。

####[通过文件存储在 Azure 中创建一个 SMB 文件](/documentation/articles/storage-dotnet-how-to-use-files)
就像桌面应用程序会安装一个典型的 SMB 共享一样，在 Azure 虚拟机或云服务中运行的应用程序可安装一个文件存储共享来访问文件数据。在本教程中了解使用文件存储的基础知识。

####[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage-preview-portal)
Azure 高级存储为在 Azure 虚拟机上运行的 I/O 密集型工作负荷提供高性能、低延迟的磁盘支持。

###计划
  
####[使用表、队列和 Blob 创建 .NET 多层应用程序](/documentation/articles/cloud-services-dotnet-multi-tier-app-storage-1-overview)
创建使用表、队列和 Blob 的多层 ASP.NET MVC 4 Web 应用程序。将应用程序部署到 Azure 云服务。

####<!--[-->Azure 订阅和服务限制、配额和约束条件<!--](/documentation/articles/azure-subscription-service-limits)-->
了解订阅、Web Workers、虚拟机、网络、存储以及 SQL 数据库最常见的 Windows Azure 限制。

###开发

####[如何在 Windows 应用商店应用程序中使用 Azure 存储](/documentation/articles/storage-use-store-apps)
使用 Azure Blob、队列和表可为 Windows 应用商店应用程序存储数据。

####[如何使用共享访问签名授予对存储资源的访问权限](/documentation/articles/storage-dotnet-shared-access-signature-part-1)
了解如何使用共享访问签名来授予对您的存储帐户中的 Blob、队列和表的受限访问权限。本教程的[第 1 部分](/documentation/articles/storage-dotnet-shared-access-signature-part-1)说明了共享访问签名的重要概念并提供了最佳实践。[第 2 部分](/documentation/articles/storage-dotnet-shared-access-signature-part-2)将逐步引导您完成为 Blob 资源生成共享访问签名的过程，然后演示如何从客户端应用程序使用这些签名。

###管理

####[创建存储帐户](/documentation/articles/storage-create-storage-account)
创建存储帐户，以便您能使用 Azure Blob、表和队列服务。

####[管理存储帐户](/documentation/articles/storage-manage-storage-account)
管理如何复制存储帐户，查看、复制和重新生成存储访问密钥以及删除存储帐户。

####[监视存储帐户](/documentation/articles/storage-monitor-storage-account)
在管理门户中自定义对存储帐户的监视，并配置日志记录以进行深入的故障排除。

####[配置自定义域名](/documentation/articles/storage-custom-domain-name)
配置自定义域以便访问 Azure 存储帐户中的 Blob 数据。

####[使用 Azure 导入/导出服务可将数据传输到 Blob 存储中](/documentation/articles/storage-import-export-service)
Azure 导入/导出服务可高效地将大量文件数据传输到 Azure Blob 存储中，以及从 Blob 存储传输到您的本地安装中。


##寻找更多资源？

-   [论坛-询问问题、分享见解和讨论平台](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs)
-   [参考-Azure 存储服务 REST API 参考](http://msdn.microsoft.com/zh-cn/library/dd179355.aspx)
-   [示例-浏览可下载的示例应用程序](http://code.msdn.microsoft.com/windowsazure)
-   [下载-下载用于配置 Azure 的脚本](/zh-cn/downloads/?sdk=net)
-   [MSDN参考文档-从MSDN上获取有帮助的资源](http://msdn.microsoft.com/zh-cn/library/gg433040.aspx)
   

