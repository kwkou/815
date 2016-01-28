<properties 
	pageTitle="在 Azure 中配置 Web 应用" 
	description="如何在 Azure 中配置 Web 应用" 
	services="app-service\web" 
	documentationCenter="" 
	authors="erikre" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="web-sites" 
	ms.date="09/16/2015" 
	wacn.date="01/21/2016"/>
	
# 如何配置 Web 应用 #
在 Azure 管理门户中，你可以更改 Web 应用的配置选项并链接到其他 Azure 资源，如数据库。

## 目录 ##
- [如何更改 Web 应用的配置选项](#howtochangeconfig)
- [如何配置 Web 应用以使用 SQL 数据库](#howtoconfigSQL)
- [如何配置 Web 应用以使用 MySQL 数据库](#howtoconfigMySQL)
- [如何配置自定义域名](#howtodomain)
- [如何配置 Web 应用以使用 SSL](#howtoconfigSSL)
- [后续步骤](#next)


## <a name="howtochangeconfig"></a>如何更改 Web 应用的配置选项

<!-- HOW TO: CHANGE CONFIGURATION OPTIONS FOR A WEBSITE -->

若要设置 Web 应用的配置选项，请执行以下操作：

1. 在[管理门户](https://manage.windowsazure.cn/)中，打开 Web 应用的管理页。
1. 单击“配置”选项卡。

“配置”选项卡包含以下部分：

### 常规

**框架版本**。如果你的应用程序使用下列任一框架，请设置这些选项：

- **.NET Framework 版本**：设置 .NET Framework 版本。 
- **PHP 版本**：设置 PHP 版本，或设为“关闭”****以禁用 PHP。
- **Java 版本**：选择显示的版本以启用 Java，或设为“关闭”以禁用 Java。 
- 如果你启用 Java，通过“Web 容器”选项，你可以选择 Tomcat 版本或是 Jetty 版本。
- **Python 版本**：选择 Python 版本，或设为“关闭”以禁用 Python。

	出于技术原因，为你的 Web 应用启用 Java 会禁用 .NET、PHP 和 Python 选项。

- <strong>托管管道模式</strong>。设置 IIS [管道模式](http://www.iis.net/learn/get-started/introduction-to-iis/introduction-to-iis-architecture#Application)。请保持设置为“集成(默认)”，除非你的旧版 Web 应用需要旧版 IIS。

- <strong>平台</strong>。选择是要在 32 位还是 64 位环境中运行应用程序。64 位环境需要“基本”或“标准”模式。“免费”和“共享”模式始终在 32 位环境下运行。

- <strong>Web Socket</strong>。设为“打开”以启用 WebSocket 协议；例如，如果你的 Web 应用使用 [ASP.NET SignalR](http://www.asp.net/signalr) 或 [socket.io](/documentation/articles/web-sites-nodejs-chat-app-socketio/)。

- <strong>始终可用</strong>。默认情况下， Web 应用如果已处于空闲状态相当一段时间，则是未加载的状态。这样可以让系统节省资源。在“基本”或“标准”模式下，你可以启用“始终可用”，以始终保持加载站点。如果你的站点运行连续的 Web 作业，则应启用“始终可用”，否则这些 Web 作业可能无法可靠运行

- <strong>在 Visual Studio Online 中编辑</strong>。通过 Visual Studio Online 启用实时代码编辑。如果启用，“仪表板”选项卡的<strong></strong>“速览”部分下则会显示名为“在 Visual Studio Online 编辑”<strong></strong>的链接。单击此连结可直接联机编辑 Web 应用。如果你需要进行身份验证，可以使用你的基本部署凭据。

	>[AZURE.NOTE]
	> 此功能还在预览中。

注意：如果通过源代码管理你启用部署，则部署可能会覆盖你在 Visual Studio Online 编辑器中进行的更改。


### 证书

在“基本”或“标准”模式下，你可为自定义域上载 SSL 证书。有关详细信息，请参阅[为 Azure Web 应用启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate)。

此处列出已上载的证书。在你上载某一证书后，可以将其分配给你的订阅和区域中的任何 Web 应用。通配符证书可用于接受此证书的域中的任何站点。仅当该证书不具有有效绑定关系时，才能删除证书。

### 域名

查看或添加 Web 应用的其他域名。有关详细信息，请参阅[为 Azure Web 应用配置自定义域名](/documentation/articles/web-sites-custom-domain-name)。

### SSL 绑定

如果已上载 SSL 证书，可以将其绑定到自定义域名。有关详细信息，请参阅[为 Azure Web 应用启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate)

### 部署

只有在你已经从源代码管理启用了部署后，此部分才出现。使用这些设置配置部署。

- <strong>Git URL</strong>。如果你已为 Azure Web 应用创建 Git 存储库，则 Git URL 就是推送内容的 URL。
- <strong>部署触发器 URL</strong>。可以在 GitHub、CodePlex、Bitbucket 或其他存储库上设置此 URL，将提交操作推送到存储库时就能触发部署。
- <strong>要部署的分支</strong>。指定推送内容时要部署的分支。

若要通过源代码管理设置部署，请查看“仪表板”选项卡，然后单击“通过源代码管理设置部署”。

### 应用程序诊断

从支持日志记录的 Web 应用写入诊断日志的选项：

- <strong>文件系统</strong>。将日志写入 Web 应用的文件系统。文件系统日志记录将持续 12 小时。你可以从 Web 应用的 FTP 共享访问这些日志。
- <strong>表存储</strong>。将日志写入 Azure 表存储。该存储没有时限，除非你禁用日志记录，否则会一直启用日志记录。 
- <strong>Blob 存储</strong>。将日志写入 Azure Blob 存储。该存储没有时限，除非你禁用日志记录，否则会一直启用日志记录。

<strong>日志记录级别</strong>。启用日志记录时，此选项指定要记录的信息量（“错误”、“警告”、“信息”或“详细”）。

**管理表存储**。启用表存储时，单击此按钮可设置存储帐户和表名称。

**管理 blob 存储。** 启用 Blob 存储时，单击此按钮可设置存储帐户和 blob 存储名称。

### 应用程序诊断

用于收集 Web 应用诊断信息的选项。

<strong>Web 服务器日志记录</strong>。启用 Web 服务器日志记录。日志以 W3C 扩展日志文件格式保存。可将日志保存到 Azure 存储空间或 Web 应用文件系统。
 
- 如果你选择“文件系统”，日志会保存到“仪表板”页上“FTP 诊断日志”下列出的 FTP 站点。
- 如果你选择“文件系统”，请使用“配额”框设置日志文件的最大磁盘空间量。空间量最小为 25MB，最大为 100MB。默认值为 35MB.。在达到了配额后，最旧的文件将被最新的文件依次覆盖。如果你需要保留超过 100MB 的历史记录，请使用 Azure 存储空间，它的存储容量更大。
- 或者，单击“设置保留”，以在一段时间后自动删除文件。默认情况下，永不删除日志。   

<strong>详细的错误消息</strong>。如果启用，详细的错误消息会另存为 .htm 文件。若要查看文件，请转到“仪表板”页上“FTP 诊断日志”下列出的 FTP 站点。文件保存在 FTP 站点中的 /LogFiles/DetailedErrors 下。

<strong>未能请求跟踪</strong>。如果启用，失败的请求将记录到 XML 文件。若要查看文件，请转到“仪表板”页上“FTP 诊断日志”下列出的 FTP 站点。） 这些文件保存在 /LogFiles/W3SVCxxx 下，其中 xxx 是唯一标识符。此文件夹包含一个 XSL 文件和一个或多个 XML 文件。请务必下载 XSL 文件，因为 XSL 文件提供格式化和筛选 XML 文件内容的功能。

<strong>远程调试</strong> 启用远程调试。如果启用，你可以使用 Visual Studio 中的远程调试器直接连接到 Azure Web 应用。远程调试将保持启用状态 48 小时。

**注意**：远程调试不适用于长度超过 20 个字符的站点名称或用户名。


### 开发人员分析

选择“加载项”可从列表中选择分析外接程序，目前 Azure 中国上不支持 Azure 应用商店。选择“自定义”可从列表中选择 New Relic 之类的分析提供程序。如果你使用某一自定义提供程序，则必须在“提供程序密钥”框中输入许可证密钥。

有关 New Relic 与 Azure Web 应用配合使用的详细信息，请参阅 [Azure Web 应用上的 New Relic 应用程序性能管理](/documentation/articles/store-new-relic-web-sites-dotnet-application-performance-management)。

### 应用设置

你的 Web 应用在启动时将加载的名称/值对。

- 对于 .NET Web 应用，这些设置将在运行时注入到 .NET 配置 AppSettings 中，并且将重写现有设置。 

- PHP、Python、Java 和 Node 应用程序可以在运行时以环境变量的形式访问这些设置。系统将为每个应用程序设置创建两个环境变量，一个变量具有由应用程序设置条目指定的名称，另一个具有 APPSETTING\_ 前缀。这两个变量都包含相同的值。

### 连接字符串

链接资源的连接字符串。

对于 .NET Web 应用，这些连接字符串将在运行时注入到 .NET 配置 connectionStrings 设置中，并且将重写其中的键等于链接的数据库名称的所有现有条目。

对于 PHP、Python、Java 和 Node 应用程序，这些设置将在运行时作为环境变量提供，并且用连接类型作为前缀。下面列出了环境变量前缀：

- SQL Server：SQLCONNSTR\_
- MySQL：MYSQLCONNSTR\_
- SQL 数据库：SQLAZURECONNSTR\_
- 自定义：CUSTOMCONNSTR\_

例如，如果将 MySql 连接字符串命名为 connectionstring1，则将通过环境变量 <code>MYSQLCONNSTR_connectionString1</code> 访问该字符串。

<strong>请注意</strong>：将数据库资源链接到 Web 应用时，也会创建连接字符串。在配置管理页上查看时，以此方式创建的连接字符串为只读形式。

### 默认文档

Web 应用的默认文档是未指定 Web 应用上的特定页面时显示的网页。如果 Web 应用包含列表中的多个文件，请确保将默认文件放置在列表的顶部。

Web 应用可能会使用根据 URL 路由的模块，而不是提供静态内容，在此情况下，将没有此类默认文档。

### 处理程序映射

使用此区域可添加自定义脚本处理器，以处理特定文件扩展名的请求。

- **扩展**。要处理的扩展名，例如 *.php 或 handler.fcgi。
- **脚本处理器路径**。脚本处理器的绝对路径。匹配该扩展名的文件请求将由脚本处理器处理。使用路径 <code>D:\\home\\site\\wwwroot</code> 来引用站点的根目录。
- **其他参数**。脚本处理器的可选命令行参数 


### 虚拟应用程序和目录 

若要配置与你的 Web 应用关联的虚拟应用程序和目录，请指定每个虚拟目录及其对应于 Web 应用根目录的物理路径。或者，你也可以选中“应用程序”<strong></strong>复选框，在站点配置中将虚拟目录标记为应用程序。

	

<!-- HOW TO: CONFIGURE A WEBSITE TO USE A SQL DATABASE -->
## <a name="howtoconfigSQL"></a>如何配置 Web 应用以使用 SQL 数据库

执行以下步骤可将 Web 应用链接到 SQL 数据库：

1. 在[管理门户](http://manage.windowsazure.cn)中，选择 “Web 应用”显示当前登录的帐户创建的网站的列表。

2. 从 Web 应用列表中选择 Web 应用，以打开该 Web 应用的“管理”页。

3. 单击“链接的资源”选项卡，“链接的资源”页上会显示一条消息：“没有链接资源”。

4. 单击“链接资源”，以打开“链接资源”向导。

5. 单击“创建新资源”，以显示可连接到 Web 应用的资源类型的列表。

6. 单击“SQL 数据库”，以显示“链接数据库”向导。

7. 完成“链接数据库”向导的第 3 页和第 4 页上的必填字段，然后单击第 4 页上的“完成”复选标记。

Azure 将使用指定的参数创建 SQL 数据库并将该数据库链接到 Web 应用。

## <a name="howtodomain"></a>如何配置自定义域名

有关配置 Web 应用以使用自定义域名的信息，请参阅[为 Azure Web 应用配置自定义域名](/documentation/articles/web-sites-custom-domain-name/)。

## <a name="howtoconfigSSL"></a>如何配置 Web 应用以使用 SSL##

有关在 Azure 上为自定义域配置 SSL 的信息，请参阅[为 Azure Web 应用启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate/)。

## <a name="next"></a>后续步骤

* [如何缩放 Web 应用](/documentation/articles/web-sites-scale/)

* [如何监视 Web 应用](/documentation/articles/web-sites-monitor/)

<!---HONumber=74-->