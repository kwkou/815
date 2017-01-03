<properties 
	pageTitle="在 Azure 应用服务中配置 Web 应用" 
	description="如何在 Azure 应用服务中配置 Web 应用" 
	services="app-service\web" 
	documentationCenter="" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/01/2016" 
	wacn.date="12/30/2016" 
	ms.author="robmcm"/>

# 在 Azure 应用服务中配置 Web 应用 #

本主题介绍如何使用 [Azure 门户预览]配置 Web 应用。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## 应用程序设置

1. 在 [Azure 门户预览]中，打开 Web 应用边栏选项卡。
2. 单击**“所有设置”**。
3. 单击“应用程序设置”。

![应用程序设置][configure01]

“应用程序设置”边栏选项卡包含多个类别的设置。

### 常规设置

**框架版本**。如果你的应用使用下列任一框架，请设置这些选项：

- **.NET Framework**：设置 .NET Framework 版本。
- **PHP**：设置 PHP 版本，或设为“关”以禁用 PHP。
- **Java**：选择 Java 版本，或设为“关”以禁用 Java。利用“Web 容器”选项来选择 Tomcat 或 Jetty 版本。
- **Python**：选择 Python 版本，或设为“关”以禁用 Python。

出于技术原因，为你的应用启用 Java 会禁用 .NET、PHP 和 Python 选项。

<a name="platform"></a>
**平台**。选择是要在 32 位还是 64 位环境中运行 Web 应用。64 位环境需要“基本”或“标准”模式。“免费”和“共享”模式始终在 32 位环境下运行。

**Web 套接字**。设为“开”以启用 WebSocket 协议；例如，如果 Web 应用使用 [ASP.NET SignalR] 或 [socket.io]。

<a name="alwayson"></a>
**AlwaysOn**。默认情况下，如果 Web 应用已处于空闲状态相当一段时间，则是未加载的状态。这样可以让系统节省资源。在“基本”或“标准”模式下，可启用“AlwaysOn”以保证始终加载应用。如果应用运行连续的 Web 作业，应启用“AlwaysOn”；否则，这些 Web 作业可能无法可靠运行。

**托管管道版本**。设置 IIS [管道模式]。将此设置保留为“集成(默认)”，除非旧版应用需要旧版 IIS。

**自动交换**。如果启用部署槽的自动交换，则在向该槽推送更新时，应用服务会自动将 Web 应用交换到生产。有关详细信息，请参阅[为 Azure 应用服务中的 Web 应用部署到过渡槽](/documentation/articles/web-sites-staged-publishing/)。

### 调试

**远程调试**。启用远程调试。启用后，可使用 Visual Studio 中的远程调试器直接连接到 Web 应用。远程调试将保持启用状态 48 小时。

### 应用设置

本部分包含你的 Web 应用启动时将要加载的名称/值对。

- 对于 .NET 应用，这些设置会在运行时注入到 .NET 配置 `AppSettings` 中，重写现有设置。

- PHP、Python、Java 和 Node 应用程序可以在运行时以环境变量的形式访问这些设置。系统将为每个应用设置创建两个环境变量，一个变量具有由应用设置条目指定的名称，另一个具有 APPSETTING\_ 前缀。这两个变量都包含相同的值。

### 连接字符串

链接资源的连接字符串。

对于 .NET 应用，这些连接字符串会在运行时注入到 .NET 配置 `connectionStrings` 设置中，重写其中的键等于链接的数据库名称的所有现有条目。

对于 PHP、Python、Java 和 Node 应用程序，这些设置将在运行时作为环境变量提供，并且用连接类型作为前缀。下面列出了环境变量前缀：

- SQL Server：`SQLCONNSTR_`
- MySQL：`MYSQLCONNSTR_`
- SQL 数据库：`SQLAZURECONNSTR_`
- 自定义：`CUSTOMCONNSTR_`

例如，如果 MySql 连接字符串命名为 `connectionstring1`，则会通过环境变量 `MYSQLCONNSTR_connectionString1` 访问该字符串。

### 默认文档

默认文档是在网站的根 URL 下显示的网页。使用列表中第一个匹配文件。

Web 应用可能会使用根据 URL 路由的模块，而不是提供静态内容，在此情况下，将没有此类默认文档。

### 处理程序映射

使用此区域可添加自定义脚本处理器，以处理特定文件扩展名的请求。

- **扩展名**。要处理的扩展名，例如 \*.php 或 handler.fcgi。
- **脚本处理器路径**。脚本处理器的绝对路径。与文件扩展名匹配的文件请求将由脚本处理器处理。使用路径 `D:\home\site\wwwroot` 表示应用的根目录。
- **其他参数**。脚本处理器的可选命令行参数


### 虚拟应用程序和目录 
 
若要配置虚拟应用程序和目录，请指定每个虚拟目录及其对应于网站根目录的物理路径。还可选中“应用程序”复选框，将虚拟目录标记为应用程序。


## 启用诊断日志

启用诊断日志：

1. 在 Web 应用边栏选项卡上，单击“所有设置”。
2. 单击“诊断日志”.

从支持日志记录的 Web 应用程序写入诊断日志的选项：

- **应用程序日志记录**。将应用程序日志写入文件系统。日志记录将持续 12 小时。

**级别**。启用应用程序日志记录时，此选项指定要记录的信息量（“错误”、“警告”、“信息”或“详细”）。

**Web 服务器日志记录**。日志以 W3C 扩展日志文件格式保存。

**详细的错误消息**。保存详细的错误消息 .htm 文件。文件保存在 /LogFiles/DetailedErrors 下。

**失败请求跟踪**。将失败请求记录到 XML 文件中。这些文件保存在 /LogFiles/W3SVC *xxx* 下，其中 xxx 是唯一标识符。此文件夹包含一个 XSL 文件和一个或多个 XML 文件。请务必下载 XSL 文件，因为 XSL 文件提供格式化和筛选 XML 文件内容的功能。

若要查看日志文件，必须按以下方式创建 FTP 凭据：

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。在“Web 应用”页上，选择要为其安装连续部署的 Web 应用，然后选择“仪表板”选项卡。

1. 在“速览”部分中，单击“重置部署凭据”设置菜单项，提供用于将文件发布到应用的用户名和密码。

1. 回到 [Azure 门户预览](https://portal.azure.cn)。完整的 FTP 用户名是“app\\username”，其中 *app* 是 Web 应用的名称。用户名列在 Web 应用边栏选项卡的“Essentials”下。

![FTP 部署凭据][configure02]

## 其他配置任务

### SSL 

在“基本”或“标准”模式下，你可为自定义域上载 SSL 证书。有关详细信息，请参阅[为 Web 应用启用 HTTPS]。

若要查看上传的证书，请单击“所有设置”>“自定义域和 SSL”。

### 域名

添加 Web 应用的自定义域名。有关详细信息，请参阅[为 Azure 应用服务中的 Web 应用配置自定义域名]。

若要查看域名，请单击“所有设置”>“自定义域和 SSL”。

### 部署

- 设置连续部署。请参阅[使用 Git 在 Azure 应用服务中部署 Web 应用]
- 部署槽。请参阅[为 Azure 应用服务中的 Web 应用部署到过渡环境]。

若要查看部署槽，请单击“所有设置”>“部署槽”。

### 监视

在“基本”或“标准”模式下，你可以测试 HTTP 或 HTTPS 终结点的可用性，最多可测试两个地理分散的位置。如果 HTTP 响应码为错误（4xx 或 5xx），或者响应时间超过 30 秒，则表示监视测试失败。如果从所有指定的位置监视测试均成功，则将终结点视为可用。

## 后续步骤

- [在 Azure 应用服务中配置自定义域名]
- [Enable HTTPS for an app in Azure App Service（为 Azure 应用服务中的应用启用 HTTPS）]
- [在 Azure 应用服务中缩放 Web 应用]
- [Azure 应用服务中 Web 应用的监视基础知识]

<!-- URL List -->

[ASP.NET SignalR]: http://www.asp.net/signalr
[Azure 门户预览]: https://portal.azure.cn/
[在 Azure 应用服务中配置自定义域名]: /documentation/articles/web-sites-custom-domain-name/
[为 Azure 应用服务中的 Web 应用部署到过渡环境]: /documentation/articles/web-sites-staged-publishing/
[Enable HTTPS for an app in Azure App Service（为 Azure 应用服务中的应用启用 HTTPS）]: /documentation/articles/web-sites-configure-ssl-certificate/
[如何监视 Web 终结点状态]: http://go.microsoft.com/fwLink/?LinkID=279906
[Azure 应用服务中 Web 应用的监视基础知识]: /documentation/articles/web-sites-monitor/
[管道模式]: http://www.iis.net/learn/get-started/introduction-to-iis/introduction-to-iis-architecture#Application
[在 Azure 应用服务中缩放 Web 应用]: /documentation/articles/web-sites-scale/
[socket.io]: /documentation/articles/web-sites-nodejs-chat-app-socketio/
[试用应用服务]: https://tryappservice.azure.com/

<!-- IMG List -->

[configure01]: ./media/web-sites-configure/configure01.png
[configure02]: ./media/web-sites-configure/configure02.png
[configure03]: ./media/web-sites-configure/configure03.png

<!---HONumber=Mooncake_Quality_Review_1118_2016-->