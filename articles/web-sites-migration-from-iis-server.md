<properties 
	pageTitle="将企业 Web 应用迁移至 Azure App Service" 
	description="演示如何使用 Web Apps 迁移助手快速将现有 IIS 网站迁移至 Azure App Service Web Apps" 
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	writer="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web"  
	ms.date="07/07/2015" 
	wacn.date="08/29/2015"/>

# 将企业 Web 应用迁移至 Azure App Service

您可以轻松将运行于 Internet 信息服务 (IIS) 6 或更高版本的现有网站迁移至 <!--[-->App Service Web Apps<!--](http://go.microsoft.com/fwlink/?LinkId=529714)-->。[Web Apps 迁移助手](https://www.movemetothecloud.net/)可以分析您的 IIS 服务器安装，确定哪些网站可以迁移至 App Service、突出显示所有不能迁移或不受平台支持的元素，然后将您的网站迁移到与 Azure 相关联的数据库。

>[AZURE.IMPORTANT]Windows Server 2003 将于 2015 年 7 月 14 终止支持。如果您的网站目前正运行于 IIS 服务器（即 Windows Server 2003），Web Apps 将能够以较低的风险、成本和摩擦保持网站联机，此外，Web Apps 迁移助手还可帮助您实现迁移过程自动化。

## 兼容性分析期间所验证的元素 ##
迁移助手能够创建准备情况报告，以确定任何可能导致从本地 IIS 迁移至 Azure App Service Web Apps 失败的潜在隐患或妨碍性问题。需注意以下几个要点：

-	端口绑定 - Web Apps 仅支持用于 HTTP 的端口 80 和用于 HTTPS 通信的端口 443。不同的端口配置将会被忽略，且流量将路由到 80 或 443。 
-	身份验证 – Web Apps 默认支持匿名身份验证，并在应用程序指定的情况下支持表单身份验证。Windows 身份验证仅可通过集成 Azure Active Directory 和 ADFS 加以使用。其他形式的身份验证 - 目前暂不予以支持，例如基本身份验证。 
-	全局程序集缓存 (GAC) – GAC 不受 Web Apps 的支持。如果您的应用程序引用您通常部署到 GAC 的程序集，您需要部署到 Web Apps 的应用程序 bin 文件夹。 
-	IIS5 兼容性模式 – Web Apps 不支持该模式。 
-	应用程序池 - 在 Web Apps 中，各站点及其子应用程序在同一个应用程序池中运行。如果您的站点有多个子应用程序利用多个应用程序池，请将它们整合到带有常用设置的单个应用程序池，或者将各应用程序迁移到另一个 Web 应用。
-	COM 组件 – Web Apps 不允许 COM 组件在平台上注册。如果您的网站或应用程序使用任何 COM 组件，您必须在托管代码中对它们进行重写，并将其与网站或应用程序一同部署。
-	ISAPI 筛选器 – Web Apps 支持使用 ISAPI 筛选器。您需要执行以下操作：
	-	将 DLL与您的 Web Apps 一同部署 
	-	使用 [Web.config](http://www.iis.net/configreference/system.webserver/isapifilters) 注册 DLL
	-	将 applicationHost.xdt 文件放置在带有下列内容的站点根目录中：

			<?xml version="1.0"?>
			<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
			<configSections>
			    <sectionGroup name="system.webServer">
			      <section name="isapiFilters" xdt:Transform="SetAttributes(overrideModeDefault)" overrideModeDefault="Allow" />
			    </sectionGroup>
			  </configSections>
			</configuration>

		有关如何使用 XML 文档转换网站的更多示例，请参阅[转换您的 Windows Azure Web Site](http://blogs.msdn.com/b/waws/archive/2014/06/17/transform-your-microsoft-azure-web-site.aspx)。

-	其他组件，比如 SharePoint、首页、server extensions (FPSE)、FTP，SSL 证书等，将不予迁移。

## 如何使用 Web Apps 迁移助手 ##
本部分将举例逐步说明如何迁移使用 SQL Server 数据库并运行于本地 Windows Server 2003 R2 (IIS 6.0) 计算机的网站：

1.	在 IIS 服务器或客户端计算机上导航到[https://www.movemetothecloud.net/](https://www.movemetothecloud.net/) 

	![](./media/web-sites-migration-from-iis-server/migration-tool-homepage.png)

2.	通过单击**专用 IIS 服务器**按钮安装 Web Apps 迁移助手。未来将提供更多选项。
4.	单击**安装工具**按钮，在您的计算机上安装 Web Apps 迁移助手。

	![](./media/web-sites-migration-from-iis-server/install-page.png)

	>[AZURE.NOTE]您也可以单击**脱机安装下载**，以下载在服务器未联网状态下进行安装的 ZIP 文件。您还可以单击**上载现有迁移准备情况报告**，该高级选项适用于您之前生成的现有迁移准备情况报表（稍后进行说明）。

5.	在**应用程序将安装**屏幕，单击**安装**以在您的计算机上进行安装。如果需要，它还会安装比如 Web 部署、DacFX，和 IIS 等相应的依赖项。

	![](./media/web-sites-migration-from-iis-server/install-progress.png)

	安装后，Web Apps 迁移助手将自动启动。
  
6.	选择**将站点和数据库从远程服务器迁移到 Azure**。输入远程服务器的管理凭据，然后单击**继续**。

	![](./media/web-sites-migration-from-iis-server/migrate-from-remote.png)

	当然，您可以选择从本地服务器迁移。如果希望从 IIS 生产服务器迁移网站，远程选项将发挥作用。
 
	此时，迁移工具将检查您的 IIS 服务器配置，如站点、应用程序、应用程序池和依赖项，以确定用于迁移的候选网站。

8.	下列屏幕快照显示了三个网站 – **Default Web Site**、**TimeTracker** 和 **CommerceNet4**。他们均有我们希望迁移的关联数据库。选择您想要评估的所有站点，然后单击**下一步**。

	![](./media/web-sites-migration-from-iis-server/select-migration-candidates.png)
 
9.	单击**上载**以上载准备情况报表。单击**本地保存文件**，您可以在以后重新运行迁移工具并上载已保存的准备情况报告，如前文所述。

	![](./media/web-sites-migration-from-iis-server/upload-readiness-report.png)
 
	准备情况报告上载完成后，Azure 将执行准备情况分析，并显示结果。读取各网站的评估详情，并确保您了解或在继续操作之前已解决所有问题。
 
	![](./media/web-sites-migration-from-iis-server/readiness-assessment.png)

12.	单击**开始迁移**以启动迁移。您现在会重定向到 Azure 以登录帐户。您需要采用带有有效 Azure 订阅的帐户进行登录。如果没有 Azure 帐户，您可以注册免费试用。

13.	依次选择用于已迁移的 Azure Web 应用和数据库的租户帐户、Azure 订阅和重新登录，然后单击**开始迁移**。您可以选择以后要迁移的网站。

	![](./media/web-sites-migration-from-iis-server/choose-tenant-account.png)

14.	在下一个屏幕上，您可以更改为默认迁移设置，如：

	- 使用现有 Azure SQL Database 或创建新的 Azure SQL Database，并配置凭据
	- 选择要迁移的网站
	- 定义用于 Azure Web 应用及其链接的 SQL 数据库的名称
	- 自定义全局设置和站点级设置

	以下屏幕快照显示了所有已选定要迁移并带有默认设置的网站。

	![](./media/web-sites-migration-from-iis-server/migration-settings.png)

	>[AZURE.NOTE]自定义设置中的**启用 Azure Active Directory** 复选框集成了带有 [Azure Active Directory](/documentation/articles/active-directory-whatis)（**默认目录**）的 Azure Web 应用。更多关于同步 Azure Active Directory 与您的本地 Active Directory 的详细信息，请参阅[目录集成](https://msdn.microsoft.com/zh-cn/library/jj573653)。

16.	 所需的更改完成后，请单击**创建**开始迁移过程。迁移工具将创建 Azure SQL 数据库和 Azure Web 应用，然后发布网站内容和数据库。迁移工具可清晰地显示迁移进度，结束时您将看到一个摘要屏幕，详细显示迁移的站点（无论是否成功迁移），并链接到新创建的 Azure Web 应用。

	如果迁移过程中出现任何错误，迁移工具将明确显示迁移失败，并回滚到所做的更改。您还将可以通过单击**发送错误报告**按钮直接向工程团队发送错误报告，以及捕获的失败调用堆栈，并构建消息正文。

	![](./media/web-sites-migration-from-iis-server/migration-error-report.png)

	如果迁移成功，没有出错，还可以单击**提供反馈**按钮直接提供任何反馈信息。
 
20.	单击 Azure Web 应用链接，并验证迁移是否成功。

21. 您现在可以 Azure App Service 中已迁移的 Web 应用。为此，请登录到 [Azure 门户](https://manage.windowsazure.cn)。

22. 在 Azure 门户中，打开 Web Apps 边栏选项卡以查看已迁移的网站（显示为 Web 应用），然后单击其中任何一个以开始管理 Web 应用，例如配置连续发布、创建备份、自动缩放、监视使用情况或性能。

	![](./media/web-sites-migration-from-iis-server/TimeTrackerMigrated.png)

## 发生的更改
* 有关从网站更改为 App Service 的指南，请参阅：[Azure App Service 及其对现有 Azure 服务的影响](http://go.microsoft.com/fwlink/?LinkId=529714)
* 有关从旧门户更改为新门户的指南，请参阅：[有关在预览门户中导航的参考](http://go.microsoft.com/fwlink/?LinkId=529715)
 

<!---HONumber=67-->