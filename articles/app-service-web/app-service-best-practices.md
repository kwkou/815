<properties
	pageTitle="有关 Azure App Service 的最佳实践"
	description="了解有关 Azure App Service 的最佳实践和故障排除步骤。"
	services="app-service"
	documentationCenter=""
	authors="dariagrigoriu"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.date="06/30/2016"
	wacn.date="09/26/2016"/>
    
# 有关 Azure App Service 的最佳实践

本文汇总了有关使用 [Azure App Service](/documentation/articles/app-service-changes-existing-services/) 的最佳实践。

## <a name="colocation"></a>共置
如果将编写解决方案（例如 Web 应用和数据库）的 Azure 资源定位在不同的区域，则影响可能包括：

*  增大资源之间通信的延迟
*  [Azure 定价页](/pricing/details/data-transfer/)中列出了跨区域出站数据传输的收费

相同区域中的共置最适合用于组成解决方案的 Azure 资源（例如 Web 应用），以及用于保存内容或数据的数据库或存储帐户。创建资源时，应确保它们位于同一个 Azure 区域，除非有具体的业务或设计理由需要将它们放在不同的区域。可以利用高级应用服务计划应用当前可用的[应用服务克隆功能](/documentation/articles/app-service-web-app-cloning-portal/)，将应用服务应用移至数据库所在的区域。

## <a name="memoryresources"></a>当应用消耗的内存超出预期时
如果通过监视或者参考服务建议，发现应用消耗的内存超出指定的预期值，请考虑使用[应用服务自动修复功能](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites)。自动修复功能的选项之一是根据内存阈值采取自定义操作。这些操作的范围包括发出电子邮件通知、通过内存转储提供调查依据，以及通过回收工作进程在现场消除问题。可以根据这篇有关[应用服务支持站点扩展](https://azure.microsoft.com/blog/additional-updates-to-support-site-extension-for-azure-app-service-web-apps)的博文中所述，通过 web.config 或者友好的用户界面来配置自动修复。

## <a name="CPUresources"></a>当应用消耗的 CPU 超出预期时
如果你通过监视或者参考服务建议，发现应用消耗的 CPU 超出预期，或者反复出现 CPU 高峰，请考虑向上缩放或向外缩放 App Service 计划。如果你的应用程序是有状态的，则向上缩放是唯一选项；如果你的应用程序是无状态的，则向外缩放可以提供更高的灵活性和更大的缩放潜力。

有关“有状态”与“无状态”应用程序的详细信息，请观看此视频：[Planning a Scalable End-to-End Multi-Tier Application on Azure Web 应用](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DEV-B414#fbid=?hashlink=fbid)（在 Azure Web 应用中规划可缩放的端到端多层应用程序）。有关 App Service 缩放和自动缩放选项的详细信息，请阅读：[在 Azure App Service 中缩放 Web 应用](/documentation/articles/web-sites-scale/)。

## <a name="socketresources"></a>当套接字资源耗尽时
耗尽出站 TCP 连接的一个常见原因是使用的客户端库未实施为重复使用 TCP 连接，或者使用了较高级别的协议（如 HTTP），因而未利用 Keep-Alive。请查看 App Service 计划中的应用引用的每个库，以确保在代码中配置或访问这些库时，能够有效地重复使用出站连接。此外，请遵循有关正确执行创建和发布或清理操作的库指导文档，以避免连接泄漏。在展开此类客户端库调查的过程中，可以通过向外扩展到多个实例来消除影响。

## <a name="appbackup"></a>当应用备份开始失败时
应用备份失败的两个最常见原因：存储设置无效和数据库配置无效。这些失败通常发生在对存储或数据库资源或其访问方式进行了更改（例如更新了备份设置中所选数据库的凭据）时。备份通常按计划运行并且只需访问存储（以便输出备份后的文件）和数据库（以便复制和读取备份中要包含的内容）。其中任一资源访问失败将导致持续备份失败。

出现备份失败时，请查看最新结果以了解所出现失败的类型。如果存储访问失败，请查看并更新备份配置中使用的存储设置。如果数据库访问失败，请查看并更新应用设置中的连接字符串，然后继续将备份配置更新为正确地包括所需数据库。有关应用备份的详细信息，请参阅[在 Azure App Service 中备份 Web 应用](/documentation/articles/web-sites-backup/)文档。

## <a name="nodejs"></a>当新的 Node.js 应用部署到 Azure App Service 时
适用于 Node.js 应用的 Azure App Service 默认配置旨在符合最常见应用的需求。如果 Node.js 应用的配置可从个性化调整中受益，使性能提升或 CPU/内存/网络资源的资源使用情况得到优化，用户可以查看我们的最佳实践和故障排除步骤。这篇文章介绍了可能需要为 Node.js 应用配置的 iisnode 设置、应用可能面临的各种案例或问题，并说明了如何解决这些问题：[有关 Azure App Service 上 Node 应用程序的最佳实践和故障排除指南](/documentation/articles/app-service-web-nodejs-best-practices-and-troubleshoot-guide/)。

<!---HONumber=Mooncake_0919_2016-->