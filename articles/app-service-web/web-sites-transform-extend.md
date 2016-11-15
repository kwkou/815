<properties
	pageTitle="Azure 应用服务 Web 应用高级配置和扩展"
	description="使用 XML 文档转换 (XDT) 声明可以转换 Azure 应用服务 Web 应用中的 ApplicationHost.config 文件，以及添加专用扩展来启用自定义管理操作。"
	authors="cephalin"
	writer="cephalin"
	editor="mollybos"
	manager="wpickett"
	services="app-service"
	documentationCenter=""/>

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/25/2016"
	wacn.date="10/10/2016"
	ms.author="cephalin"/>

# Azure 应用服务 Web 应用高级配置和扩展

使用 [XML 文档转换](http://msdn.microsoft.com/zh-cn/library/dd465326.aspx) (XDT) 声明，可以转换 Azure 应用服务中 Web 应用的 [ApplicationHost.config](http://www.iis.net/learn/get-started/planning-your-iis-architecture/introduction-to-applicationhostconfig) 文件。你还可使用 XDT 声明添加专用扩展，启用自定义 Web 应用管理操作。本文包括一个 PHP Manager Web 应用扩展示例，可用于通过 Web 界面管理 PHP 设置。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a id="transform"></a>通过 ApplicationHost.config 完成的高级配置
应用服务平台为 Web 应用配置提供灵活性和控制。尽管标准 IIS ApplicationHost.config 配置文件不能在应用服务中直接编辑，但该平台支持基于 XML 文档转换 (XDT) 的声明性 ApplicationHost.config 转换模型。

要利用此转换功能，请在 [Kudu 控制台](https://github.com/projectkudu/kudu/wiki/Kudu-console)中创建包含 XDT 内容的 ApplicationHost.xdt 文件，并将其放在站点根目录 (d:\\home\\site) 下。可能需要重新启动 Web 应用才能使更改生效。

以下 applicationHost.xdt 示例介绍了如何将新的自定义环境变量添加到使用 PHP 5.4 的 Web 应用。

	<?xml version="1.0"?>
	<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  		<system.webServer>
    			<fastCgi>
      				<application>
         				<environmentVariables>
            					<environmentVariable name="CONFIGTEST" value="TEST" xdt:Transform="Insert" xdt:Locator="XPath(/configuration/system.webServer/fastCgi/application[contains(@fullPath,'5.4')]/environmentVariables)" />
         				</environmentVariables>
      				</application>
    			</fastCgi>
  		</system.webServer>
	</configuration>


包含转换状态和详细信息的日志文件位于 FTP 根目录 LogFiles\\Transform 中。

有关其他示例，请参阅 [https://github.com/projectkudu/kudu/wiki/Xdt-transform-samples](https://github.com/projectkudu/kudu/wiki/Xdt-transform-samples)。

**注意**<br/>
`system.webServer` 下的模块列表中的元素无法删除或重新排序，但列表的附加内容可以删除或重新排序。


## <a id="extend"></a> 扩展你的 Web 应用

### <a id="overview"></a> 专用 Web 应用扩展概述

应用服务支持使用 Web 应用扩展作为站点管理操作的扩展点。事实上，某些应用服务平台功能是作为预安装的扩展实现的。尽管无法修改预安装的平台扩展，但可以为自己的 Web 应用创建并配置专用扩展。此功能也依赖于 XDT 声明。创建专用 Web 应用扩展的关键步骤如下：

1. Web 应用扩展**内容**：创建应用服务支持的任何 Web 应用程序
2. Web 应用扩展**声明**：创建 ApplicationHost.xdt 文件
3. Web 应用扩展**部署**：将内容存放在 `root` 下的 SiteExtensions 文件夹内

Web 应用的内部链接应指向 ApplicationHost.xdt 文件中指定的应用程序路径的相对路径。对 ApplicationHost.xdt 文件的任何更改都需要 Web 应用回收。

**注意**：有关这些关键元素的更多信息，请参阅 [https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions)。

内含一个详细示例，用于说明创建和启用专用 Web 应用扩展的步骤。可以从 [https://github.com/projectkudu/PHPManager](https://github.com/projectkudu/PHPManager) 下载后跟的 PHP Manager 示例的源代码。

### <a id="SiteSample"></a> Web 应用扩展示例：PHP Manager

PHP Manager 是一个 Web 应用扩展，通过该 Web 应用扩展，Web 应用管理员可以轻松地使用 Web 界面来查看和配置 PHP 设置，而无需直接修改 PHP.ini 文件。PHP 常用配置文件包括位于 Program Files 下的 php.ini 文件和位于 Web 应用根文件夹中的 .user.ini 文件。由于 php.ini 文件不可在 App Service 平台上直接编辑，因此 PHP Manager 扩展使用 .user.ini 文件来应用设置更改。

#### <a id="PHPwebapp"></a> PHP Manager Web 应用程序

PHP Manager 部署主页如下：

![TransformSitePHPUI][TransformSitePHPUI]

如你所见，Web 应用扩展和常规 Web 应用程序一样，只是包含一个位于 Web 应用根文件夹中的额外的 ApplicationHost.xdt 文件（有关 ApplicationHost.xdt 文件的详细信息，请见本文下一部分）。

PHP Manager 扩展是使用 Visual Studio ASP.NET MVC 4 Web 应用程序模板创建的。以下解决方案资源管理器视图显示了 PHP Manager 扩展的结构。

![TransformSiteSolEx][TransformSiteSolEx]

文件 I/O 所需的唯一特殊逻辑是表示 Web 应用的 wwwroot 目录所在的位置。如以下代码示例所示，环境变量“HOME”表示 Web 应用根路径，可以通过附加“site\\wwwroot”来构造 wwwroot 路径：

	/// <summary>
	/// Gives the location of the .user.ini file, even if one doesn't exist yet
	/// </summary>
	private static string GetUserSettingsFilePath()
	{
    		var rootPath = Environment.GetEnvironmentVariable("HOME"); // For use on Azure Websites
    		if (rootPath == null)
    		{
        		rootPath = System.IO.Path.GetTempPath(); // For testing purposes
    		};
    		var userSettingsFile = Path.Combine(rootPath, @"site\wwwroot\.user.ini");
    		return userSettingsFile;
	}


得到目录路径后，可以使用常规文件 I/O 操作来读取和写入文件。

Web 应用扩展需要注意的一点是处理内部链接。如果 HTML 文件中有任何链接给出了 Web 应用内部链接的绝对路径，必须确保这些链接前面带有扩展的名称作为根目录。这是必须的，因为扩展的根目录现在为“/`[your-extension-name]`/”而不只是“/”，因此所有内部链接都必须相应更新。例如，假设代码包含指向以下内容的链接：

`"<a href="/Home/Settings">PHP Settings</a>"`

如果链接是 Web 应用扩展的一部分，则该链接的格式必须为：

`"<a href="/[your-site-name]/Home/Settings">Settings</a>"`

可以通过只使用 Web 应用程序中的相对路径，或者如果是 ASP.NET 应用程序，通过使用 `@Html.ActionLink` 方法创建适当的链接，来满足此要求。

#### <a id="XDT"></a>applicationHost.xdt 文件

Web 应用扩展的代码位于 %HOME%\\SiteExtensions[你的扩展名称] 下。该目录称为扩展根目录。

若要使用 applicationHost.config 文件注册 Web 应用扩展，需要将 ApplicationHost.xdt 文件放在扩展根目录中。ApplicationHost.xdt 文件的内容应如下所示：

	<?xml version="1.0"?>
	<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  		<system.applicationHost>
    			<sites>
      				<site name="%XDT_SCMSITENAME%" xdt:Locator="Match(name)">
						<!-- NOTE: Add your extension name in the application paths below -->
        				<application path="/[your-extension-name]" xdt:Locator="Match(path)" xdt:Transform="Remove" />
        				<application path="/[your-extension-name]" applicationPool="%XDT_APPPOOLNAME%" xdt:Transform="Insert">
          					<virtualDirectory path="/" physicalPath="%XDT_EXTENSIONPATH%" />
        				</application>
      				</site>
    			</sites>
  		</system.applicationHost>
	</configuration>

选择作为扩展名称的名称应与扩展根文件夹的名称相同。

这样做可以将新应用程序路径添加到 SCM 站点下的 `system.applicationHost` 站点列表中。SCM 站点为具有特定访问凭据的站点管理终结点。其 URL 为 `https://[your-site-name].scm.chinacloudsites.cn`。

	<system.applicationHost>
  		...
  		<site name="~1[your-website]" id="1716402716">
      			<bindings>
        			<binding protocol="http" bindingInformation="*:80: [your-website].scm.chinacloudsites.cn" />
        			<binding protocol="https" bindingInformation="*:443: [your-website].scm.chinacloudsites.cn" />
      			</bindings>
      			<traceFailedRequestsLogging enabled="false" directory="C:\DWASFiles\Sites\[your-website]\VirtualDirectory0\LogFiles" />
      			<detailedErrorLogging enabled="false" directory="C:\DWASFiles\Sites\[your-website]\VirtualDirectory0\LogFiles\DetailedErrors" />
      			<logFile logSiteId="false" />
      			<application path="/" applicationPool="[your-website]">
        			<virtualDirectory path="/" physicalPath="D:\Program Files (x86)\SiteExtensions\Kudu\1.24.20926.5" />
      			</application>
				<!-- Note the custom changes that go here -->
      			<application path="/[your-extension-name]" applicationPool="[your-website]">
        			<virtualDirectory path="/" physicalPath="C:\DWASFiles\Sites\[your-website]\VirtualDirectory0\SiteExtensions\[your-extension-name]" />
      			</application>
    		</site>
  	</sites>
	  ...
	</system.applicationHost>

### <a id="deploy"></a> Web 应用扩展部署

若要安装 Web 应用扩展，你可以使用 FTP 将 Web 应用程序的所有文件复制到要安装该扩展的 Web 应用的 `\SiteExtensions[your-extension-name]` 文件夹中。请确保也将 ApplicationHost.xdt 文件复制到此位置。重新启动 Web 应用以启用该扩展。

Web 应用扩展位于：

`https://[your-site-name].scm.chinacloudsites.cn/[your-extension-name]`

请注意，该 URL 类似于你的 Web 应用的 URL，只不过它使用 HTTPS，且包含“.scm”。

在开发和调查过程中，可以通过添加使用键 `WEBSITE_PRIVATE_EXTENSIONS` 和值 `0` 的应用设置，来禁用 Web 应用的所有专用（非预装）扩展。

## 发生的更改
* 有关从网站更改为 App Service 的指南，请参阅 [Azure App Service 及其对现有 Azure 服务的影响](/documentation/articles/app-service-changes-existing-services/)

<!-- IMAGES -->
[TransformSitePHPUI]: ./media/web-sites-transform-extend/TransformSitePHPUI.png
[TransformSiteSolEx]: ./media/web-sites-transform-extend/TransformSiteSolEx.png
 

<!---HONumber=Mooncake_0926_2016-->