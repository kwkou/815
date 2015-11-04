<properties title="How to create a PHP web site in Azure Web Sites" pageTitle="How to create a PHP web site in Azure Web Sites" metaKeywords="PHP Azure Web Sites" description="Learn how to create a PHP web site in Azure Web Sites" documentationCenter="PHP" services="Web Sites" editor="mollybos" manager="bjsmith" authors="" />
<tags ms.service="Web Sites"
    ms.date="02/05/2015"
    wacn.date="11/02/2015"
    />

#如何在 Azure 网站中创建 PHP 网站

本文将说明如何使用 [Azure 管理门户]、[针对 Mac 和 Linux 的 Azure 命令行工具][xplat-tools]或 [Azure PowerShell cmdlets][powershell-cmdlets] 来在 [Azure 网站][waws]中创建 PHP 网站。

通常，在 Azure 网站中创建 PHP 网站与创建任何其他网站并无差异。默认情况下，PHP 是针对所有网站启用的。有关配置 PHP（或提供你自定义的 PHP 运行时）的信息，请参阅 [如何在 Azure 网站中配置 PHP][configure-php]。

下述每个选项将向你演示如何在共享的托管环境中免费创建网站，但在 CPU 使用率和带宽使用率上有一些限制。有关详细信息，请参阅 [Azure 网站定价][websites-pricing]。有关如何升级和扩展你的网站的信息，请参阅 [如何扩展网站][scale-websites]。


##目录
* [使用 Azure 管理门户创建网站](#portal)
* [使用适用于 Mac 和 Linux 的 Azure 命令行工具创建网站](#XplatTools)
* [使用 Azure PowerShell cmdlet 创建网站](#PowerShell)

<h2>使用 Azure 管理门户创建 PHP 网站</h2>

在 Azure 管理门户中创建网站时，你有两个选项：“快速创建”和“与数据库一起创建”。以下说明将介绍 “快速创建”选项。有关其他两个选项的信息，请参阅 [创建 PHP-MySQL Azure 网站并使用 Git 进行部署] 和 [ Website-mysql-git]。

若要使用 Azure 管理门户创建 PHP 网站，请执行以下操作：

1. 登录到 [Azure 管理门户]。
1. 在页面底部，依次单击“新建”、“计算”、“网站”和“快速创建”。提供网站的“URL”，并选择网站的“区域”。最后，单击“创建网站”。

![选择“快速创建”网站](./media/web-sites-php-create-web-sites/select-quickcreate-Website.png)

<h2>使用适用于 Mac 和 Linux 的 Azure 命令行工具创建 PHP 网站</h2>

若要使用适用于 Mac 和 Linux 的 Azure 命令行工具创建 PHP 网站，请执行以下操作：

1. 按照此处的说明进行操作来安装 Azure 命令行工具：[如何安装面向 Mac 和 Linux 的 Azure 命令行工具](/documentation/articles/xplat-cli/#Download)。

1. 按照此处的说明进行操作来下载和导入你的发布设置文件：[如何下载和导入发布设置](/documentation/articles/xplat-cli/#Account)。

1. 从命令提示符处运行以下命令：

		azure site create MySiteName

新创建的网站的 URL 将为 `http://MySiteName.chinacloudsites.cn`。
 
请注意，你可将 `azure site create` 命令与下列任一选项一起执行：

* `--location [location name]`。此选项允许你指定创建网站的数据中心的位置（例如“China East”）。如果忽略此选项，系统将提示你选择一个位置。
* `--hostname [custom host name]`。此选项允许你指定网站的自定义主机名。
* `--git`。此选项允许你使用 Git 通过在本地应用程序目录和网站的数据中心创建 Git 存储库来发布到网站。请注意，如果本地文件夹已是 Git 存储库，则此命令会将新的远程添加到现有存储库，并指向网站数据中心内的存储库。

有关其他选项的信息，请参阅 [如何创建和管理 Azure 网站](/documentation/articles/xplat-cli/# Websites)。

<h2>使用 Azure PowerShell cmdlet 创建 PHP 网站</h2>

若要使用 Azure PowerShell cmdlet 创建 PHP 网站，请执行以下操作：

1. 按照此处的说明进行操作来安装 Azure PowerShell cmdlet：[Azure PowerShell 入门](/documentation/articles/install-configure-powershell/#GetStarted)。

1. 按照此处的说明进行操作来下载和导入你的发布设置文件：[如何：导入发布设置](/documentation/articles/install-configure-powershell/#ImportPubSettings)。

1. 打开 PowerShell 命令提示符，并执行以下命令：

		New-Azure Website MySiteName

新创建的网站的 URL 将为 `http://MySiteName.chinacloudsites.cn`。
 
请注意，你可将 `New-Azure Website` 命令与下列任一选项一起执行：

* `-Location [location name]`。此选项允许你指定创建网站的数据中心的位置（例如“China East”）。如果忽略此选项，系统将提示你选择一个位置。
* `-Hostname [custom host name]`。此选项允许你指定网站的自定义主机名。
* `-Git`。此选项允许你使用 Git 通过在本地应用程序目录和网站的数据中心创建 Git 存储库来发布到网站。请注意，如果本地文件夹已是 Git 存储库，则此命令会将新的远程添加到现有存储库，并指向网站数据中心内的存储库。

有关其他选项的信息，请参阅 [如何：创建和管理 Azure 网站](/develop/php/how-to-guides/powershell-cmdlets/# Website)。

<h2><a name="NextSteps"></a>后续步骤</h2>

现在你已在 Azure 网站中创建 PHP 网站，可以管理、配置、监视、部署到和扩展你的网站。有关详细信息，请参阅以下链接：

* [如何配置网站](/documentation/articles/web-sites-configure//)
* [如何在 Azure 网站中配置 PHP][configure-php]
* [如何管理网站](/documentation/articles/web-sites-manage/)
* [如何监视网站](/documentation/articles/web-sites-monitor/)
* [如何扩展网站](/documentation/articles/web-sites-scale/)
* [使用 Git 进行发布](/documentation/articles/web-sites-publish-source-control//)

有关端到端教程，请访问 [PHP 开发中心 - 教程](/develop/php//)页。

[waws]: /zh-cn/documentation/services/web-sites
[Azure 管理门户]: http://manage.windowsazure.cn/
[xplat-tools]: /documentation/articles/xplat-cli/
[powershell-cmdlets]: /documentation/articles/install-configure-powershell/
[configure-php]: /documentation/articles/web-sites-php-configure/
[Website-mysql-git]: /documentation/articles/web-sites-php-mysql-deploy-use-git/
[Websites-pricing]: /zh-cn/pricing/overview/
[scale-Websites]: /documentation/articles/web-sites-scale

<!---HONumber=76-->