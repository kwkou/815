<properties title="How to create a PHP web site in Azure Web Sites" pageTitle="How to create a PHP web site in Azure Web Sites" metaKeywords="PHP Azure Web Sites" description="Learn how to create a PHP web site in Azure Web Sites" documentationCenter="PHP" services="Web Sites" editor="mollybos" manager="bjsmith" authors="" />

# 如何在 Azure 网站中创建 PHP 网站

本文将说明如何使用 [Azure 管理门户][Azure 管理门户]、[针对 Mac 和 Linux 的 Azure 命令行工具][针对 Mac 和 Linux 的 Azure 命令行工具]或 [Azure PowerShell cmdlet][Azure PowerShell cmdlet] 来在 [Azure 网站][Azure 网站]中创建 PHP 网站。

通常，在 Azure 网站中创建 PHP 网站与创建*任何* 其他网站并无差异。默认情况下，将为所有网站启用 PHP。有关配置 PHP（或提供你自定义的 PHP 运行时）的信息，请参阅[如何在 Azure 网站中配置 PHP][如何在 Azure 网站中配置 PHP]。

下述每个选项将向你演示如何在共享的托管环境中免费创建网站，但在 CPU 使用率和带宽使用率上有一些限制。有关详细信息，请参阅 [Azure 网站定价][Azure 网站定价]。有关如何升级和缩放你的网站的信息，请参阅[如何缩放网站][如何缩放网站]。

## 目录

-   [使用 Azure 管理门户创建网站][使用 Azure 管理门户创建网站]
-   [使用针对 Mac 和 Linux 的 Azure 命令行工具创建网站][使用针对 Mac 和 Linux 的 Azure 命令行工具创建网站]
-   [使用 Azure PowerShell cmdlet 创建网站][使用 Azure PowerShell cmdlet 创建网站]

## <a name="portal"></a>使用 Azure 管理门户创建 PHP 网站

在 Azure 管理门户中创建网站时，你有三个选项：“快速创建”、“与数据库一起创建”和“从库中”。以下说明将介绍“快速创建”选项。有关其他两个选项的信息，请参阅[创建 PHP-MySQL Azure 网站并使用 Git 进行部署][创建 PHP-MySQL Azure 网站并使用 Git 进行部署]和[在 Azure 中从库中创建 WordPress 网站][在 Azure 中从库中创建 WordPress 网站]。

若要使用 Azure 管理门户创建 PHP 网站，请执行以下操作：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在页面底部，依次单击“新建”、“计算”、“网站”和“快速创建”。提供网站的“URL”，并选择网站的“区域”。最后，单击“创建网站”。

    ![选择“快速创建”网站][选择“快速创建”网站]

## <a name="XplatTools"></a>使用针对 Mac 和 Linux 的 Azure 命令行工具创建 PHP 网站

若要使用针对 Mac 和 Linux 的 Azure 命令行工具创建 PHP 网站，请执行以下操作：

1.  按照此处的说明进行操作来安装 Azure 命令行工具：[如何安装针对 Mac 和 Linux 的 Azure 命令行工具][如何安装针对 Mac 和 Linux 的 Azure 命令行工具]。

2.  按照此处的说明进行操作来下载和导入你的发布设置文件：[如何下载和导入发布设置][如何下载和导入发布设置]。

3.  从命令提示符处运行以下命令：

        azure site create MySiteName

新创建的网站的 URL 将为 `http://MySiteName.chinacloudsites.cn` 。

请注意，你可将`azure site create` 命令与下列任一选项一起执行：

-   `--location [location name]`：此选项允许你指定创建网站的数据中心的位置（例如“China East”）。如果忽略此选项，系统将提示你选择一个位置。
-   `--hostname [custom host name]`：此选项允许你指定网站的自定义主机名。
-   `--git`：此选项允许你使用 Git 通过在本地应用程序目录和网站的数据中心创建 Git 存储库来发布到网站。请注意，如果本地文件夹已是 Git 存储库，则此命令会将新的远程添加到现有存储库，并指向网站数据中心内的存储库。

有关其他选项的信息，请参阅[如何创建和管理 Azure 网站][如何创建和管理 Azure 网站]。

## <a name="PowerShell"></a>使用 Azure PowerShell cmdlet 创建 PHP 网站

若要使用 Azure PowerShell cmdlet 创建 PHP 网站，请执行以下操作：

1.  按照此处的说明进行操作来安装 Azure PowerShell cmdlet：[Azure PowerShell 入门][Azure PowerShell 入门]。

2.  按照此处的说明进行操作来下载和导入你的发布设置文件：[如何：导入发布设置][如何：导入发布设置]。

3.  打开 PowerShell 命令提示符，并执行以下命令：

        New-AzureWebSite MySiteName

新创建的网站的 URL 将为 `http://MySiteName.chinacloudsites.cn`.

请注意，你可将`New-AzureWebSite` 命令与下列任一选项一起执行：

-   `-Location [location name]`. 此选项允许你指定创建网站的数据中心的位置（例如“China East”）。如果忽略此选项，系统将提示你选择一个位置。
-   `-Hostname [custom host name]`. 此选项允许你指定网站的自定义主机名。
-   `-Git`. 此选项允许你使用 Git 通过在本地应用程序目录和网站的数据中心创建 Git 存储库来发布到网站。请注意，如果本地文件夹已是 Git 存储库，则此命令会将新的远程添加到现有存储库，并指向网站数据中心内的存储库。

有关其他选项的信息，请参阅[如何：创建和管理 Azure 网站][如何：创建和管理 Azure 网站]。

## <a name="NextSteps"></a>后续步骤

现在你已在 Azure 网站中创建 PHP 网站，可以管理、配置、监视、部署到和缩放你的网站。有关详细信息，请参阅以下链接：

-   [如何配置网站][如何配置网站]
-   [如何在 Azure 网站中配置 PHP][如何在 Azure 网站中配置 PHP]
-   [如何管理网站][如何管理网站]
-   [如何监视网站][如何监视网站]
-   [如何缩放网站][如何缩放网站]
-   [使用 Git 进行发布][使用 Git 进行发布]

有关端到端教程，请访问 [PHP 开发中心 - 教程][PHP 开发中心 - 教程]页。

  [Azure 管理门户]: http://manage.windowsazure.cn/
  [针对 Mac 和 Linux 的 Azure 命令行工具]: /en-us/develop/php/how-to-guides/command-line-tools/
  [Azure PowerShell cmdlet]: /en-us/develop/php/how-to-guides/powershell-cmdlets/
  [Azure 网站]: /en-us/manage/services/web-sites/
  [如何在 Azure 网站中配置 PHP]: /en-us/develop/php/common-tasks/configure-php-web-site/
  [Azure 网站定价]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [如何缩放网站]: /en-us/manage/services/web-sites/how-to-scale-websites/
  [使用 Azure 管理门户创建网站]: #portal
  [使用针对 Mac 和 Linux 的 Azure 命令行工具创建网站]: #XplatTools
  [使用 Azure PowerShell cmdlet 创建网站]: #PowerShell
  [创建 PHP-MySQL Azure 网站并使用 Git 进行部署]: /en-us/develop/php/tutorials/website-w-mysql-and-git/
  [在 Azure 中从库中创建 WordPress 网站]: /en-us/develop/php/tutorials/website-from-gallery/
  [选择“快速创建”网站]: ./media/web-sites-php-create-web-sites/select-quickcreate-website.png
  [如何安装针对 Mac 和 Linux 的 Azure 命令行工具]: /en-us/develop/php/how-to-guides/command-line-tools/#Download
  [如何下载和导入发布设置]: /en-us/develop/php/how-to-guides/command-line-tools/#Account
  [如何创建和管理 Azure 网站]: /en-us/develop/php/how-to-guides/command-line-tools/#WebSites
  [Azure PowerShell 入门]: /en-us/develop/php/how-to-guides/powershell-cmdlets/#GetStarted
  [如何：导入发布设置]: /en-us/develop/php/how-to-guides/powershell-cmdlets/#ImportPubSettings
  [如何：创建和管理 Azure 网站]: /en-us/develop/php/how-to-guides/powershell-cmdlets/#WebSite
  [如何配置网站]: /en-us/manage/services/web-sites/how-to-configure-websites/
  [如何管理网站]: /en-us/manage/services/web-sites/how-to-manage-websites/
  [如何监视网站]: /en-us/manage/services/web-sites/how-to-monitor-websites/
  [使用 Git 进行发布]: /en-us/develop/php/common-tasks/publishing-with-git/
  [PHP 开发中心 - 教程]: /en-us/develop/php/tutorials/
