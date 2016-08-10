<properties
	pageTitle="有关 Azure Web 应用的最佳实践"
	description="了解有关 Azure Web 应用的最佳实践和故障排除方法。"
	services="app-service"
	documentationCenter=""
	authors="dariagrigoriu"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.date="05/19/2016"
	wacn.date="07/04/2016"/>
    
# Azure 最佳实践

本文汇总了有关使用 [Azure Web 应用](/documentation/services/web-sites/)的最佳实践。

## <a name="colocation"></a>共置
如果将编写解决方案（例如 Web 应用和数据库）的 Azure 资源定位在不同的区域，则影响可能包括：

*  资源间通信延迟加大
*  [Azure 定价页](/pricing/details/data-transfer/)中列出了跨区域出站数据传输的收费

相同区域中的共置最适合用于组成解决方案的 Azure 资源（例如 Web 应用），以及用于保存内容或数据的数据库或存储帐户。创建资源时，应确保它们位于同一个 Azure 区域，除非有具体的业务或设计理由需要将它们放在不同的区域。

## <a name="memoryresources"></a>当应用消耗的内存超出预期时
如果你通过监视或者参考服务建议，发现应用消耗的内存超出指定的预期值，请考虑使用 [Azure 自动修复功能](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites)。自动修复功能的选项之一是根据内存阈值采取自定义操作。这些操作包括电子邮件通知、内存转储调查，以及通过回收工作进程在现场消除问题。

## <a name="CPUresources"></a>当应用消耗的 CPU 超出预期时
如果你通过监视或者参考服务建议，发现应用消耗的 CPU 超出预期，或者反复出现 CPU 高峰，请考虑扩大或扩展 App Service 计划。如果你的应用程序是有状态的，则扩大是唯一选项；如果你的应用程序是无状态的，则扩展可以提供更好的灵活性和更大的缩放潜力。

有关“有状态”与“无状态”应用程序的详细信息，请观看此视频：[Planning a Scalable End-to-End Multi-Tier Application on Azure Web 应用（在 Azure Web 应用中规划可缩放的端到端多层应用程序）](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DEV-B414#fbid=?hashlink=fbid)。有关 Azure 缩放和自动缩放选项的详细信息，请阅读：[在 Azure 中缩放 Web 应用](/documentation/articles/web-sites-scale/)。

## <a name="socketresources"></a>当套接字资源耗尽时
耗尽出站 TCP 连接的一个常见原因是客户端库未重复使用TCP连接，或者使用了较高级别的协议（如 HTTP），未利用 Keep-Alive。请查看 App Service 计划中的应用引用的每个库，确保在代码中配置或访问这些库时，能够有效地重复使用出站连接。此外，请遵循有关正确执行创建和发布或清理操作的库指导文档，避免连接泄漏。在展开此类客户端库调查的过程中，可以通过向外扩展到多个实例来消除影响。



<!---HONumber=Mooncake_0627_2016-->