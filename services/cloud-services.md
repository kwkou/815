<properties linkid="dev-net-Cloud-Service" urlDisplayName="Windows Azure Cloud Service" pageTitle="Windows Azure 服务管理：云服务" metaKeywords="Cloud Service" description="" metaCanonical="" services="Cloud Service" documentationCenter="Services" title="Configure, monitor, and scale your cloud services in Azure" authors="" solutions="" manager="" editor="" />


#云服务

####在 Azure 中配置、监视和缩放云服务
使用云服务部署和管理功能强大的应用程序和服务。上载应用程序，Azure 将处理部署细节（从设置和负载平衡到运行状况监视）以实现持续可用性。

####快速链接

-   [服务概述](/home/features/cloud-services/)
-   [可交付的解决方案](/solutions/web/)
-   [定价信息](/pricing/details/cloud-services/)

####特色

-   [Azure 云服务和 ASP.NET 入门](/zh-cn/documentation/articles/cloud-services-dotnet-get-started/)

##教程和指南

###计划

####[什么是云服务？](/zh-cn/documentation/articles/cloud-services-what-is/)

当您在 Azure 中创建应用程序并运行它时，您的代码（以及配置代码的方式）被称为云服务。本主题包括有关云服务的基本概念。

####[Azure 执行模型](zh-cn/documentation/articles/fundamentals-application-models/)

Azure 提供了三种可用于承载 Web 应用程序的计算模型：网站、云服务和虚拟机。本主题概述了三种模型和信息，以帮助您确定适用于您的应用程序的模型。

###开发

####[创建云服务](/zh-cn/documentation/articles/cloud-services-how-to-create-deploy/)

使用“快速创建”创建新的云服务。完成此操作后，您可以在 Azure 中上载和部署云服务包。

####[Azure 云服务和 ASP.NET 入门](/zh-cn/documentation/articles/cloud-services-dotnet-get-started/)

####[将资源链接到云服务](/zh-cn/documentation/articles/cloud-services-how-to-manage/#linkresources)

要揭示云服务对其他资源的依赖性，您可以将 Azure SQL数据库 实例或存储帐户链接到云服务。

####[创建存储帐户](/zh-cn/documentation/articles/storage-create-storage-account/)

要在 Azure 中存储 Blob、表和队列服务中的文件和数据，您必须在要存储数据的地理区域创建存储帐户。存储帐户可以容纳高达 100 TB 的 Blob、表和队列数据。

###部署

####[部署云服务](/zh-cn/documentation/articles/cloud-services-how-to-create-deploy/)

创建云服务后，您可以使用 Azure 管理门户上载、测试和部署新的服务包。

####[缩放应用程序](/zh-cn/documentation/articles/cloud-services-how-to-scale/)

可以通过添加或删除角色实例来缩放云服务。如果将 Azure SQL数据库 实例链接到云服务，则还可以缩放数据库。

###管理

####[配置云服务](/zh-cn/documentation/articles/cloud-services-how-to-configure)

您可以在 Azure 管理门户中配置最常使用的云服务设置。或者，如果您希望直接更新配置文件，则可以下载要更新的服务配置文件，然后上载更新文件并使用配置更改更新云服务。无论使用哪种方法，配置更新都将应用于所有角色实例。

####[更新云服务角色或部署](/zh-cn/documentation/articles/cloud-services-how-to-manage/#updaterole)

如果您需要为云服务更新应用程序中的代码，则将需要上载一些内容。本主题将指导您上载新的服务包和服务配置文件。

####[监视云服务](/zh-cn/documentation/articles/cloud-services-how-to-monitor/)

如果您需要为云服务更新应用程序中的代码，则将需要上载一些内容。本主题将指导您上载新的服务包和服务配置文件。

####[在 Azure 中启用诊断](/zh-cn/documentation/articles/cloud-services-dotnet-diagnostics/)

从 Azure 中运行的辅助角色、Web 角色或虚拟机收集诊断数据以排查问题。

####[在云服务中使用 Windows 错误报告](http://download.microsoft.com/download/C/4/8/C48CAA93-537E-453B-A3EE-55AC0300BD95/WER-in-Azure_Aug2014.pdf)

了解如何在 Azure 云服务中配置 Windows 错误报告，以及如何使用 Windows 错误报告来收集 Azure 平台组件的故障排除信息。

##寻找更多资源？

-   [论坛---询问问题、分享见解和讨论平台](https://social.msdn.microsoft.com/Forums/azure/zh-CN/home?forum=windowsazurezhchs)
-   [参考---查找针对客户端库和服务器脚本的文档](http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460812)
-   [示例---浏览可下载的示例应用程序](http://code.msdn.microsoft.com/windowsazure/site/search?query=cloud%20services&amp;f%5B0%5D.Value=cloud%20services&amp;f%5B0%5D.Type=SearchText&amp;ac=5)
-   [下载---下载用于配置 Azure 的脚本](/zh-cn/downloads/?sdk=net)

