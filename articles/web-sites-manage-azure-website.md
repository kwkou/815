<properties 
	pageTitle="管理 Azure 中的 Web 应用" 
	description="用于管理 Azure 中 Web 应用的资源链接。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="erikre" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="04/27/2016"
	wacn.date="06/29/2016"/>

# 管理 Azure 中的 Web 应用

本主题包含用于管理 [Azure Web 应用](/documentation/services/web-sites/)中 Web 应用的资源链接。管理包括维持 Web 应用平稳运行的所有任务。

在整个 Web 应用使用期内，您将执行各种管理任务，从初始部署到正常操作、维护与更新。

许多 Web 应用管理任务都可在 Azure 经典管理门户中执行。

## 将 Web 应用部署到生产之前

### 选择层级

提供四个级别的 Azure Web 应用：免费、共享、基本和标准。有关各级别的特性与定价的信息，请参阅[定价详情](/home/features/web-site/pricing/)。

- 您可以在创建 Web 应用之后经常[切换层](/documentation/articles/web-sites-scale/)。

### 配置

使用 [Azure 经典管理门户](https://manage.windowsazure.cn/)设置各种配置选项。有关详细信息，请参阅[在 Azure 中配置 Web 应用](/documentation/articles/web-sites-configure/)。下面是快速核对清单：

- 如有需要，请选择针对 .NET、PHP、Java 或 Python 的**运行时版本**。
- 如果您的 Web 应用使用 WebSocket 协议，请启用 **WebSocket**。（这包括使用 [ASP.NET SignalR](http://www.asp.net/signalr) 或 [socket.io](/documentation/articles/web-sites-nodejs-chat-app-socketio/) 的应用。）
- 您是否要运行持续 web 作业？ 如果是，请启用 **Always On**。
- 设置**默认文档**，例如 index.html。

除了这些基本配置设置，您可能还需要进行下列配置：

- **安全套接字层 (SSL)**加密。若要使用带有自定义域名的 SSL，您必须获取 SSL 证书并配置 Web 应用。请参阅[在 Azure 中启用 Web 应用的 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate/)。
- **自定义域名。** Web 应用自动具有 chinacloudsites.cn 下的子域。可以关联自定义域名（如 contoso.com ）。请参阅[在 Azure 中配置自定义域名](/documentation/articles/web-sites-custom-domain-name/)。

特定于语言的配置：

- **PHP**：[在 Azure 中配置 PHP](/documentation/articles/web-sites-php-configure/)。
- **Python**：[配置 Azure Web 应用的 Python](/documentation/articles/web-sites-python-configure/)


## Web 应用运行期间

在 Web 应用运行期间，您要确保其可用性，并能够进行缩放以满足用户流量。您可能还需要解决错误。

### 监视

- 通过 Azure 经典管理门户，你可以[添加性能度量值](/documentation/articles/web-sites-monitor/)（如 CPU 使用率和客户端请求数）。
- [缩放您的 Web 应用](/documentation/articles/web-sites-scale/)以响应流量。您可以根据不同的层缩放虚拟机数量和/或虚拟机实例的大小。在标准层和高级层中，您还可以设置自动缩放，那么您的 Web 应用将能够根据固定计划，或以负载为依据进行自动缩放。  
 
### 备份

- 设置 Web 应用的[自动备份](/documentation/articles/web-sites-backup/)。
- 了解 Azure SQL 数据库的[数据库恢复](/documentation/articles/sql-database-business-continuity/)选项。

### 故障排除

- 在 Visual Studio 之外还提供了不同的诊断日志收集方法。请参阅[在 Azure 中启用 Web 应用的诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)。
- 关于 Node.js 应用程序，请参阅[如何在 Azure 中调试 Node.js Web 应用](/documentation/articles/web-sites-nodejs-debug/)。

### 还原数据

- [还原](/documentation/articles/web-sites-restore/)之前备份的 Web 应用。


## 更新 Web 应用时

如果尚未启用自动备份，您可以创建[手动备份](/documentation/articles/web-sites-backup/)。

请考虑使用[分阶段部署](/documentation/articles/web-sites-staged-publishing/)。该选项可支持您向与生产部署并排运行的分阶段部署发布更新。

<!-- Anchors. -->


[Before you deploy your site to production]: #before-you-deploy-your-site-to-production
[While your website is running]: #while-your-website-is-running
[When you update your website]: #when-you-update-your-website

  

<!---HONumber=Mooncake_1207_2015-->