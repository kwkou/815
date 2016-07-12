<properties
	pageTitle="在 Azure Web 应用中保护应用"
	description="了解如何在 Azure Web 应用中保护 Web 应用、移动应用后端或 API 应用。"
	services="app-service"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="01/12/2016"
	wacn.date="02/26/2016"/>


#在 Azure 中保护应用

本文可帮助你开始在 Azure Web 应用中保护你的 Web 应用、移动应用后端或 API 应用。

Azure 中的安全性有两个级别：

- **基础结构和平台安全性** - 你相信 Azure 提供你所需的服务以实际在云中安全地运行任务。
- **应用程序安全性** - 你需要安全地设计应用本身。这包括如何与 Azure Active Directory 集成、如何管理证书，以及如何确保你可以安全地与不同服务通信。

#### 基础结构和平台安全性
由于 Azure 维护 Azure VM、存储空间、网络连接、web 框架、管理和集成功能以及更多内容，因此它主动受到保护和强化，并审查有力的合规性，并不断检查以确保：

- 你的 Azure Web 应用同时与 Internet 以及其他客户的 Azure 资源隔离。
- 你的 Azure Web 应用与资源组中的其他 Azure 资源（例如 SQL 数据库）之间的机密（如连接字符串）通信保持在 Azure 中，并且未跨越任何网络边界。机密始终加密。
- 你的 Azure Web 应用与外部资源（例如 PowerShell 管理、命令行界面、Azure SDK、REST API 和混合连接）之间的所有通信都经过正确加密。
- 24 小时威胁管理可保护 Azure 资源免受恶意软件、分布式拒绝服务 (DDoS)、中间人 (MITM) 和其他威胁的危害。 

有关 Azure 中的基础结构和平台安全性的详细信息，请参阅 [Azure 信任中心](/support/trust-center/security/)。

#### 应用程序安全性

Azure 负责保护运行应用程序的基础结构和平台，而你负责保护你的应用程序本身。换而言之，你需要以安全方式开发、部署和管理你的应用程序代码和内容。无此安全性，你的应用程序代码或内容仍然容易受到如下威胁的危害：

- SQL 注入
- 会话劫持
- 跨站点脚本攻击
- 应用程序级 MITM
- 应用程序级 DDoS

关于基于 web 的应用程序安全注意事项的全面讨论超出了本文的范围。作为保护应用程序安全的更多指导的起点，请参阅[打开 Web 应用程序安全项目 (OWASP)](https://www.owasp.org/index.php/Main_Page)（尤其是[前 10 大项目](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)），其中列出了当前前 10 大 Web 应用程序安全缺陷（由 OWASP 成员确定）。

##<a name="https"></a> 保护与客户的通信

如果你使用为 Azure Web 应用创建的 **\*.chinacloudsites.cn** 域名，则可以立即使用 HTTPS，因为 SSL 证书是针对所有 **\*.chinacloudsites.cn** 域名提供的。如果您的网站使用[自定义域名](/documentation/articles/web-sites-custom-domain-name/)，则可以上载 SSL 证书，为自定义域[启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate/)。

启用 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 可帮助防范对你的应用与其用户之间的通信进行的 MITM 攻击。

## 保护数据层

Azure 与 SQL 数据库高度集成，以便所有连接字符串都全面加密，并且仅在运行应用的 VM 上解密 *且* 仅在应用运行时解密。此外，Azure SQL 数据库还提供许多安全功能来帮助你保护应用程序数据免受网络威胁的危害，这些功能包括[静态加密](https://msdn.microsoft.com/zh-cn/library/dn948096.aspx)、[始终加密](https://msdn.microsoft.com/zh-cn/library/mt163865.aspx)、[动态数据屏蔽](/documentation/articles/sql-database-dynamic-data-masking-get-started-portal/)。如果你有敏感数据或合规性要求，请参阅[保护 SQL 数据库](/documentation/articles/sql-database-security/)，了解有关如何保护数据的详细信息。

如果你使用第三方数据库提供程序（如 ClearDB），则应直接查阅提供程序文档以了解安全性最佳实践。

##<a name="develop"></a> 安全开发和部署

### 发布配置文件和发布设置

在开发应用程序、执行管理任务或使用实用程序（如 **Visual Studio**、**Web Matrix**、**Azure PowerShell** 或 **Azure 命令行界面 (Azure CLI)**）自动执行任务时，您可以使用 *发布设置* 文件或 *发布配置文件* 。这两种文件类型都使用 Azure 验证你的身份，并应受保护以防止未经授权的访问。

* **发布设置**文件包含

	* 您的 Azure 订阅 ID

	* 管理证书，允许您为订阅执行管理任务， *而无需提供帐户名称或密码* 。

* **发布配置文件**包含

	* 用于发布到应用的信息

如果你使用的实用程序利用发布设置文件或发布配置文件，请将包含发布设置或配置文件的文件导入实用程序，然后**删除**该文件。如果您必须保留文件（例如，要与处理项目的其他人共享），请将文件存储在安全的位置上，如权限受限的 *加密* 目录。

此外，应确保已导入的凭证受到保护。例如，**Azure PowerShell** 和 **Azure 命令行界面 (Azure CLI)** 都将导入的信息存储在您的**主目录**中（在 Linux 或 OS X 系统中，为 *~* ；在 Windows 系统中，为 */users/yourusername* ）。 为了多一层安全保障，您可能希望使用操作系统支持的加密工具来**加密**这些位置。

### 配置设置和连接字符串
常见的做法是，将连接字符串、 身份验证凭证和其他敏感信息存储在配置文件中。遗憾的是，这些文件可能会在您的网站上被公开或将其检入一个公共存储库，从而公开此类信息。例如，在 [GitHub](https://github.com) 上执行简单搜索，可以发现无数个配置文件，其中包含公共存储库中公开的机密。

最佳做法是将此信息保存在应用的配置文件以外。Azure 允许你将配置信息作为“应用设置”和“连接字符串”存储为运行时环境的一部分。对于大多数编程语言，这些值通过 *环境变量* 在运行时向您的应用程序公开。对于 .NET 应用程序，在运行时这些值被注入到.NET 配置。除了这些情况外，除非你使用 [Azure 经典管理门户](http://manage.windowsazure.cn)或实用程序（例如 PowerShell 或 Azure CLI）查看或配置这些配置设置，否则这些配置设置将保持加密状态。

在 Azure 中存储配置信息允许应用的管理员对生产应用锁定敏感信息。开发人员可以对应用开发使用一组单独的配置设置，并且这些设置可以自动由 Azure Web 应用中配置的设置取代。甚至开发人员也无需知道为生产应用配置的机密。有关在 Azure Web 应用中配置应用设置和连接字符串的更多信息，请参阅[配置 Web 应用](/documentation/articles/web-sites-configure/)。

<!-- be to customize -->
### FTPS

Azure 通过 **FTPS** 提供对你的应用文件系统的安全 FTP 访问。这样一来，您可以安全地访问 Web 应用上的应用程序代码以及诊断日志。建议你始终使用 FTPS 而不是 FTP。

按照以下步骤操作，可以找到你的应用的 FTPS 链接：

1. 打开 [Azure 经典管理门户](http://manage.windowsazure.cn)。
2. 选择“Web 应用”。
4. 选择所需的应用。
5. 单击“仪表板”
6. 你可以在该处找到 **FTPS 主机名**。

有关 FTPS 的更多信息，请参阅[文件传输协议](http://en.wikipedia.org/wiki/File_Transfer_Protocol)。

##<a name="next"></a>后续步骤

若要详细了解 Azure 平台安全、如何举报**安全事件或滥用行为**，或者如何通知 Microsoft 你将对站点执行**渗透测试**，请参阅 [Azure 信任中心](/support/trust-center/security/)的安全部分。

有关 Azure Web 应用中 **web.config** 或 **applicationhost.config** 文件的更多信息，请参阅 [Azure Web 应用中解锁的配置选项](/blog/2014/01/28/more-to-explore-configuration-options-unlocked-in-windows-azure-web-sites/)。

有关 Azure Web 应用的日志记录信息（可能在检测攻击时很有用），请参阅[启用诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)。

<!---HONumber=Mooncake_0215_2016-->