<properties 
	pageTitle="在 Azure 中将 WordPress 转换为 Multisite" 
	description="了解如何采用通过 Azure 中的库创建的现有 WordPress Web 应用并将其转换为 WordPress Multisite" 
	services="app-service\web" 
	documentationCenter="php" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="04/08/2016"
	wacn.date="05/24/2016"/>



# 在 Azure 中将 WordPress 转换为 Multisite

## 概述

*作者：[Ben Lobaugh][ben-lobaugh]，[Microsoft Open Technologies Inc.][ms-open-tech]*

在本教程中，你将了解如何采用通过 Azure 中的库创建的现有 WordPress Web 应用并将其转换为 WordPress Multisite 安装。此外，你还将了解如何将自定义域分配给安装中的每个子 Web 应用。

假定你现在具有 WordPress 安装。如果你没有，请按照 [在 Azure 中从库中创建 WordPress Web 应用][website-from-gallery] 中提供的指导操作。

通常，将现有 WordPress 单站点安装转换为 Multisite 非常简单，此处的许多初始步骤直接来自在 [WordPress Codex](http://codex.wordpress.org) 上[创建网络][wordpress-codex-create-a-network]页面。

下面我们来开始操作。

## 允许 Multisite

你首先需要通过带有 **WP\_ALLOW\_MULTISITE** 常量的 `wp-config.php` 文件启用 Multisite。编辑 Web 应用文件有两种方法：第一种是通过 FTP，第二种是通过 Git。如果你不熟悉如何设置这两种方法，请参考以下教程：

* [带 MySQL 和 FTP 的 PHP Web 应用][website-w-mysql-and-ftp-ftp-setup]

* [带 MySQL 和 Git 的 PHP Web 应用][website-w-mysql-and-git-git-setup]

使用所选编辑器打开 `wp-config.php` 文件并在 `/* That's all, stop editing! Happy blogging. */` 行的上方添加以下内容。

	/* Multisite */

	define( 'WP_ALLOW_MULTISITE', true );

请务必保存该文件并将其上载回服务器！

## 网络设置

登录到你 Web 应用的 *wp-admin* 区域，你应在“工具”菜单的下方看到一个名为“网络设置”的新项目。单击“网络设置”并填写你网络的详细信息。

![“网络设置”屏幕][wordpress-network-setup]

本教程使用*子目录*站点架构，因为它应始终运行，我们在本教程的后面将为每个子站点设置自定义域。但是，如果你通过 [Azure 经典管理门户](https://manage.windowsazure.cn/)映射域并正确设置通配符 DNS，则应能够设置子域安装。

有关子域和子目录设置的详细信息，请参阅 WordPress Codex 上的[多站点网络的类型][wordpress-codex-types-of-networks]一文。

## 启用网络

现已在数据库中配置网络，但剩下的一个步骤是启用网络功能。完成 `wp-config.php` 设置并确保 `web.config` 正确路由每个站点。


在“网络设置”页上单击“安装”按钮后，WordPress 将尝试更新 `wp-config.php` 和 `web.config` 文件。不过，你应该始终检查这些文件，以便确保更新成功。如果更新不成功，此屏幕将向你显示必要的更新。编辑并保存文件。


进行这些更新后，你需要注销并重新登录到 wp-admin 仪表板。

现在，管理栏上应额外显示一个标记为“我的 Web 应用”的菜单。利用此菜单，可以通过“网络管理员”仪表板管理新的网络。

## 添加自定义域

利用 [WordPress MU 域映射][wordpress-plugin-wordpress-mu-domain-mapping]插件，可以轻松向网络中的任何站点添加自定义域。若要使该插件正常运行，你需要额外对经典管理门户和域注册机构进行一些设置。

## 启用到 Web 应用的域映射

**免费**的 [Azure Web 应用](/documentation/services/web-sites/)计划模式不支持向 Web 应用添加自定义域。你需要切换到“共享”或“标准”模式。为此，请按以下步骤操作：

* 登录到 Azure 经典管理门户并找到你的 Web 应用。 
* 单击主要内容区域中的“缩放”选项卡
* 在“常规”下，选择“共享”或“标准”
* 单击“保存”

你可能收到一条消息，要求你验证更改并确认 Web 应用现在可能会产生费用，具体取决于使用情况和你设置的其他配置。

由于处理新的设置需要花费几秒钟的时间，因此现在正好来开始设置域。

## 验证域

在 Azure Web 应用允许你将域映射到站点前，你先需要验证你是否有映射域的权限。为此，你必须将新的 CNAME 记录添加到 DNS 项。

* 登录到域的 DNS 管理器
* 创建新的 CNAME *awverify*
* 让 *awverify* 指向 *awverify.YOUR\_DOMAIN.chinacloudsites.cn*

由于 DNS 更改可能需要过段时间才能生效，因此，如果后续步骤无法立即运行，你可以先去冲杯咖啡，然后回来重试。

## 将域添加到 Web 应用

通过 Azure 经典管理门户返回你的 Web 应用，此时请单击“配置”选项卡。“管理域”按钮应可用。请单击此按钮。

这将显示“管理自定义域”对话框。可在其中输入要分配给 Web 应用的所有域。如果某个域未在此处列出，则无法在 WordPress 中将该域用于映射，无论设置域 DNS 的方式如何。

![“管理自定义域”对话框][wordpress-manage-domains]

将域键入文本框后，Azure 将验证你之前创建的 CNAME 记录。如果 DNS 尚未完全传播，则会显示一个红色指示器。如果已成功传播，你将看到一个绿色复选标记。

记下该对话框底部列出的 IP 地址。你需要此地址来设置域的 A 记录。

## 设置域 A 记录

如果已成功执行其他步骤，则你现在可以通过 DNS A 记录将域分配给 Azure Web 应用。

此处请务必记住，Azure Web 应用同时接受 CNAME 和 A 记录，但你*必须*使用 A 记录才能启用正确的域映射。CNAME 无法转发到 Azure 使用 YOUR\_DOMAIN.chinacloudsites.cn 为你创建的其他 CNAME。

使用上一个步骤中的 IP 地址可返回你的 DNS 管理器并将 A 记录设置为指向该 IP。


## 安装和设置插件

WordPress Multisite 当前没有用于映射自定义域的内置方法。但是，你可以利用一个名为[WordPress MU 域映射][wordpress-plugin-wordpress-mu-domain-mapping]的插件来为你添加该功能。登录到你 Web 应用的“网络管理员”部分，并安装“WordPress MU 域映射”插件。

安装并激活该插件后，请访问 “设置”>“域映射”来配置插件。在第一个文本框“服务器 IP 地址”中，输入用于设置域的 A 记录的 IP 地址。设置所需的任何“域选项”（通常使用默认值即可）并单击“保存”。

## 映射域

访问你希望将域映射到的 Web 应用的**仪表板**。单击“工具”>“域映射”，在文本框中键入新域，然后单击“添加”。

默认情况下，新域将重写到自动生成的站点域。若要将所有流量发送到新域，请在保存前选中“此博客的主域”框。可以向一个站点添加无数个域，但只有一个域可作为主域。

## 再执行一次此操作

利用 Azure Web 应用，你可以向一个 Web 应用添加无数个域。若要添加另一个域，你需要为每个域执行**验证域**和**设置域 A 记录**部分中所述的操作。

[ben-lobaugh]: http://ben.lobaugh.net
[ms-open-tech]: http://msopentech.com
[wordpress-codex-create-a-network]: http://codex.wordpress.org/Create_A_Network
[website-w-mysql-and-ftp-ftp-setup]: /documentation/articles/web-sites-php-mysql-deploy-use-ftp/#header-0
[website-w-mysql-and-git-git-setup]: /documentation/articles/web-sites-php-mysql-deploy-use-git/#header-1
[wordpress-network-setup]: ./media/web-sites-php-convert-wordpress-multisite/wordpress-network-setup.png
[wordpress-codex-types-of-networks]: http://codex.wordpress.org/Before_You_Create_A_Network#Types_of_multisite_network
[wordpress-plugin-wordpress-mu-domain-mapping]: http://wordpress.org/extend/plugins/wordpress-mu-domain-mapping/

[wordpress-manage-domains]: ./media/web-sites-php-convert-wordpress-multisite/wordpress-manage-domains.png

 

<!---HONumber=82-->