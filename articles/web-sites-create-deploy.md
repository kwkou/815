<properties linkid="manage-services-how-to-create-websites" urlDisplayName="如何创建" pageTitle="如何创建网站 - Windows Azure 服务管理" metaKeywords="Azure 创建网站, Azure 删除网站" description="了解如何使用 Windows Azure 管理门户创建网站。" metaCanonical="" services="web-sites" documentationCenter="" title="如何创建和部署网站" authors=""  solutions="" writer="timamm" manager="" editor=""  />

#如何创建网站

本主题说明如何从库或通过使用管理门户创建网站。

有关如何将内容部署到您创建的 Windows Azure 网站的信息，请参见 [Windows Azure 网站](/zh-cn/documentation/services/web-sites/) 中的“部署”部分。

## 目录##

- [如何使用管理门户创建网站](#createawebsiteportal)
- [如何从库中创建网站](#howtocreatefromgallery)
- [如何删除网站](#deleteawebsite)
- [后续步骤](#nextsteps)

##<a name="createawebsiteportal"></a>如何使用管理门户创建网站

按照下列步骤操作可在 Windows Azure 中创建网站。
	
1. 登录到 [Windows Azure 管理门户](http://manage.windowsazure.com/)。

2. 单击该管理门户左下角的“新建”图标。

3. 单击“网站”图标，再单击“快速创建”图标，输入 URL 的值，然后单击页面右下角的“创建网站”旁边的复选标记。

4. 创建网站后，您会看到文本“创建网站 <*web site name*> 已成功”。您可以通过单击门户页底部的“浏览”，浏览到该网站。

5. 在门户中，单击在网站列表中显示的网站的名称以打开该网站的“快速启动”管理页面。

6. 在“快速启动”页上，将向您提供用于获取网站开发工具、为您的网站设置发布或从源代码管理提供程序（例如 TFS 或 Git）设置部署的选项。默认为网站设置 FTP 发布，并且 FTP 主机名显示在“仪表板”页的“速览”部分中的“FTP 主机名”下方。在使用 FTP 或 Git 进行发布前，选择“仪表板”页上的“重置部署凭据”选项，以便您可以在将内容部署到您的网站时对 FTP 主机或 Git 存储库进行身份验证。

7. “配置”管理页将为您的网站公开以下类别中的设置：

 - **常规**：设置 Web 应用程序所需版本的 .NET Framework 或 PHP。对于“标准”模式下的网站，“平台”选项可用于选择 32 位或 64 位环境。

- **证书**：在“标准”模式下，您可为自定义域上载 SSL 证书。

- **域名**：在“共享”和“标准”模式下，可以查看并管理您的网站的自定义域名。

- **SSL 绑定**：在“标准”模式下，您可以使用基于 IP 或基于 SNI 的 SSL 将 SSL 证书分配给您的自定义域名。

 - **部署**：在您从源代码管理设置部署时，可在此处配置部署。

 - **应用程序诊断**：设置选项以便从已使用跟踪进行了检测的 Web 应用程序收集诊断跟踪。

- **网站诊断**：设置用于收集网站的诊断信息的日志记录选项。选项包括 Web 服务器日志记录、详细错误消息以及失败的请求跟踪。

- **监视**：对于“标准”模式下的网站，可以使用该功能测试 HTTP 或 HTTPS 终结点的可用性。

- **应用设置**：指定 Web 应用程序在启动时将加载的名称/值对。对于 .NET 网站，这些设置将在运行时注入到 .NET 配置 AppSettings 中，并且将重写现有设置。对于 PHP 和 Node 网站，这些设置将在运行时作为环境变量提供。

 - **连接字符串**：查看和编辑连接字符串。对于 .NET 网站，这些连接字符串将在运行时注入到 .NET 配置 connectionStrings 设置中，并且将重写其中的键等于链接的数据库名称的所有现有条目。对于 PHP 和 Node 网站，这些设置将在运行时作为环境变量提供。

 - **默认文档**：网站的默认文档是指当用户导航到网站时默认显示的网页。在 Web 应用程序的默认文档未在列表存在中时将该文档添加到该列表中。您的网站的默认文档应位于列表的顶部。

- **处理程序映射**：指定将处理针对特定文件扩展名（例如 *.php）的请求的脚本处理器。

##<a name="howtocreatefromgallery"></a>如何从库中创建网站

[WACOM.INCLUDE [website-from-gallery](../includes/website-from-gallery.md)]

##<a name="deleteawebsite"></a>如何删除网站
在 Windows Azure 管理门户中使用“删除”图标删除网站。在单击“网站”以列出所有网站时，“删除”图标位于 Windows Azure 门户中，并且将位于每个网站管理页面的底部。

##<a name="nextsteps"></a>后续步骤

有关更多信息，请参见 [Windows Azure 网站](/zh-cn/documentation/services/web-sites/)。

