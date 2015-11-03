<properties
	pageTitle="保护 Azure 网站中的 Web 应用"
	description="了解如何保护 Azure Web 应用安全。"
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="09/16/2015"
	wacn.date="11/02/2015"/>


#保护 Azure 网站中的 Web 应用

开发 Web 应用时所要面对的一大难题是，如何为您的客户提供安全服务。在本文中，你将了解到可以保护你的 Web 应用的 [Azure 网站](/documentation/services/web-sites/)功能的相关信息。

> [AZURE.NOTE] 关于基于 web 的应用程序安全注意事项的全面讨论超出了本文的范围。有关保护 Web 应用程序安全的更多指导，请参阅[打开 Web 应用程序安全项目 (OWASP)](https://www.owasp.org/index.php/Main_Page)（尤其是[前 10 大项目](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)），其中列出了前 10 大 Web 应用程序安全缺陷（由 OWASP 成员确定）。

###目录

* [安全通信](#https)
* [安全开发](#develop)
* [后续步骤](#next)
 
##<a name="https"></a>安全通信

如果你使用为 Web 应用创建的 ***.chinacloudsites.cn** 域名，则可以立即使用 HTTPS，因为 SSL 证书是针对所有 ***.chinacloudsites.cn** 域名提供的。如果您的网站使用[自定义域名](/documentation/articles/web-sites-custom-domain-name)，则可以上载 SSL 证书，为自定义域[启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate)。

##<a name="develop"></a>安全开发

### 发布配置文件和发布设置

在开发应用程序、执行管理任务或使用实用程序（如 **Visual Studio**、**Web Matrix**、**Azure PowerShell** 或 **Azure 命令行界面 (Azure CLI)**）自动执行任务时，您可以使用*发布设置*文件或*发布配置文件*。两者都向 Azure 验证您的身份，并应受保护以防止未经授权的访问。

* **发布设置**文件包含

	* 您的 Azure 订阅 ID

	* 管理证书，允许您为订阅执行管理任务，*而无需提供帐户名称或密码*。

* **发布配置文件**包含

	* 用于发布到 Web 应用的信息

如果您使用的实用程序利用发布设置或发布配置文件，请将包含发布设置或配置文件的文件导入实用程序，然后**删除**文件。如果您必须保留文件（例如，要与处理项目的其他人共享），请将文件存储在安全的位置上，如权限受限的**加密**目录。

此外，应确保已导入的凭证受到保护。例如，**Azure PowerShell** 和 **Azure 命令行界面 (Azure CLI)** 都将导入的信息存储在您的**主目录**中（在 Linux 或 OS X 系统中，为*~*；在 Windows 系统中，为 */users/yourusername*）。 为了多一层安全保障，您可能希望使用操作系统支持的加密工具来**加密**这些位置。

### 配置设置和连接字符串
常见的做法是，将连接字符串、 身份验证凭证和其他敏感信息存储在配置文件中。遗憾的是，这些文件可能会在您的网站上被公开或将其检入一个公共存储库，从而公开此类信息。

Azure 网站允许你将配置信息作为“应用设置”和“连接字符串”存储为 Web Apps 运行时环境的一部分。对于大多数编程语言，这些值通过*环境变量*在运行时向您的应用程序公开。对于 .NET 应用程序，在运行时这些值被注入到.NET 配置。

可以使用 [Azure 门户](https://manage.windowsazure.cn)或实用程序（如 PowerShell 或 Azure CLI）配置**应用设置**和**连接字符串**。

有关应用设置和连接字符串的更多信息，请参阅[配置 Web 应用](/documentation/articles/web-sites-configure)。

### FTPS

Azure 通过 **FTPS** 提供对您 Web 应用文件系统的安全 FTP 访问。这样一来，您可以安全地访问 Web 应用上的应用程序代码以及诊断日志。按照以下步骤操作，可以找到 Web 应用的 FTPS 链接：

1. 打开 [Azure 门户](https://manage.windowsazure.cn)。
2. 单击“浏览所有”。
3. 在“浏览”边栏选项卡中，选择“Web 应用”。
4. 在“Web 应用”边栏选项卡中，选择所需的 Web 应用。
5. 在 Web 应用的边栏选项卡中，选择“所有设置”。
6. 在“设置”边栏选项卡中，选择“属性”。
7. “设置”边栏选项卡上显示了 FTP 和 FTPS 链接。 

有关 FTPS 的更多信息，请参阅[文件传输协议](http://zh.wikipedia.org/wiki/File_Transfer_Protocol)。

## 后续步骤

若要详细了解 Azure 平台安全、如何举报**安全事件或滥用行为**，或者如何通知 Microsoft 你将对网站执行**渗透测试**，请参阅 [Windows Azure 信任中心](http://azure.microsoft.com/support/trust-center/security/)的安全部分。

有关 Web 应用中 **web.config** 或 **applicationhost.config** 文件的更多信息，请参阅 [Azure 网站中解锁的配置选项](http://azure.microsoft.com/blog/2014/01/28/more-to-explore-configuration-options-unlocked-in-windows-azure-web-sites/)。

若要了解 Web 应用的日志记录（可能在检测攻击时很有用），请参阅[启用诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log)。
 

<!---HONumber=76-->