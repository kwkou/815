<properties
	pageTitle="在 Azure 网站中配置 PHP"
	description="了解如何在 Azure 网站中为 Web Apps 配置默认 PHP 安装或添加自定义 PHP 安装。"
	services="app-service\web"
	documentationCenter="php"
	authors="tfitzmac"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="09/16/2015"
	wacn.date="11/02/2015"/>
#在 Azure 网站中配置 PHP

##目录

* [什么是 Azure 网站？](#WhatIs)
* [如何：更改默认 PHP 配置](#ChangeBuiltInPHP)
* [如何：启用内置 PHP 运行时中的扩展](#EnableExtDefaultPHP)
* [如何：使用自定义 PHP 运行时](#UseCustomPHP)
* [后续步骤](#NextSteps)

## 介绍

本指南将向你演示如何执行以下操作：在 Azure 网站中配置 Web Apps 的内置 PHP 运行时，提供自定义 PHP 运行时，以及启用扩展。若要使用 Azure 网站，请注册[免费试用版]。若要充分利用本指南，你应先在 Azure 网站中创建一个 PHP Web 应用。

## 如何：更改内置 PHP 版本
默认情况下，将安装 PHP 5.4 并且在创建 Azure 网站时立即可用。查看可用发行版、其默认配置以及已启用的扩展的最佳方式是部署调用 [phpinfo()] 函数的脚本。

PHP 5.5 和 PHP 5.6 也可用，但它们在默认情况下不启用。若要更新 PHP 版本，请使用下列方法之一：

1. 浏览到 Azure 门户中网站的仪表板，单击“配置”。

	![网站仪表板上的“配置”选项卡][configure]

1. 单击“PHP 5.5”。

	![选择 PHP 版本][select-php-version]

1. 单击页面底部的“保存”。

	![保存配置设置][save-button]

对于任一内置 PHP 运行时，可通过执下列步骤来更改任何不是仅在系统级别使用的指令的配置选项。（有关仅在系统级别使用的指令的信息，请参阅 [php.ini 指令的列表]。）

1. 将 [.user.ini] 文件添加到根目录。
2. 使用你将在 `php.ini` 文件中使用的语法，将配置设置添加到 `.user.ini` 文件。例如，如果你希望打开 `display_errors` 设置，并将 `upload_max_filesize` 设置设为 10 分钟，你的 `.user.ini` 文件将包含此文本：

		; Example Settings
		display_errors=On
		upload_max_filesize=10M

3. 部署你的 Web 应用。
4. 重新启动 Web 应用。（需要进行重新启动，因为 PHP 读取 `.user.ini` 文件的频率受 `user_ini.cache_ttl` 设置的约束，该设置是一个系统级别设置且默认值为 300 秒（5 分钟）。重新启动 Web 应用会强制 PHP 读取 `.user.ini` 文件中的新设置。）

作为使用 `.user.ini` 文件的替代方法，你可以使用脚本中的 [ini_set()] 函数来设置不是系统级别指令的配置选项。

### 更改 PHP_INI_SYSTEM 配置设置

1. 使用密钥 `PHP_INI_SCAN_DIR` 和值 `d:\home\site\ini` 将应用设置添加到你的 Web 应用
2. 使用 Kudu 控制台 (http://&lt;site-name&gt;.scm.azurewebsite.net) 在 `d:\home\site\ini` 目录中创建 `settings.ini` 文件。
3. 使用你将在 php.ini 文件中使用的语法，将配置设置添加到 `settings.ini` 文件。例如，如果你希望将 `curl.cainfo` 设置指向 `*.crt` 文件并将“wincache.maxfilesize”设置为 512K，则 `settings.ini` 文件将包含此文本：

		; Example Settings
		curl.cainfo="%ProgramFiles(x86)%\Git\bin\curl-ca-bundle.crt"
		wincache.maxfilesize=512
4. 重新启动 Web 应用以加载更改。

## 如何：在默认 PHP 运行时中启用扩展
如上一部分所述，查看默认 PHP 版本、其默认配置以及已启用的扩展的最佳方式是部署调用 [phpinfo()] 的脚本。若要启用其他扩展，请执行下列步骤：

### 通过 ini 设置进行配置

1. 将 `ext` 目录添加到 `d:\home\site` 目录。
2. 将 `.dll` 扩展文件置于 `ext` 目录中（例如 `php_mongo.dll` 和 `php_xdebug.dll`）。确保扩展与默认版本的 PHP（撰写本文时为 PHP 5.4）兼容，并且是 VC9 版本且与非线程安全 (nts) 兼容。
3. 使用密钥 `PHP_INI_SCAN_DIR` 和值 `d:\home\site\ini` 将应用设置添加到你的 Web 应用
4. 在 `d:\home\site\ini` 中创建名为 `extensions.ini` 的 `ini` 文件。
5. 使用你将在 php.ini 文件中使用的语法，将配置设置添加到 `extensions.ini` 文件。例如，如果你想要启用 MongoDB 和 XDebug 扩展，则 `extensions.ini` 文件将包含此文本：

		; Enable Extensions
		extension=d:\home\site\ext\php_mongo.dll
		zend_extension=d:\home\site\ext\php_xdebug.dll
6. 重新启动 Web 应用以加载更改。

### 通过应用设置进行配置
1. 将 `bin` 目录添加到根目录。
2. 将 `.dll` 扩展文件置于 `bin` 目录中（例如 `php_mongo.dll`）。确保扩展与默认版本的 PHP（撰写本文时为 PHP 5.4）兼容，并且是 VC9 版本且与非线程安全 (nts) 兼容。
3. 部署你的 Web 应用。
1. 导航到 Azure 门户中网站的仪表板，然后单击“配置”。

	![网站仪表板上的“配置”选项卡][configure]

1. 在“应用设置”部分，创建密钥 **PHP_EXTENSIONS** 和值 **bin\\your-ext-file**。若要启用多个扩展，请包括 `.dll` 文件的逗号分隔列表。

	![启用应用程序设置中的扩展][app-settings]

1. 单击页面底部的“保存”。

	![保存配置设置][save-button]



## 如何：使用自定义 PHP 运行时
Azure 网站可以使用提供的 PHP 运行时（而非默认 PHP 运行时）来执行 PHP 脚本。提供的运行时可由提供的 `php.ini` 文件配置。若要在 Azure 网站中使用自定义 PHP 运行时，请执行下列步骤。

1. 获取非线程安全、VC9 兼容版本的 PHP for Windows。可在此处找到 PHP for Windows 最新版本：[http://windows.php.net/download/]。可在此处的存档中找到旧版本：[http://windows.php.net/downloads/releases/archives/]。
1. 修改运行时的 `php.ini` 文件。请注意，Azure 网站将忽略作为任何仅在系统级别使用的指令的配置设置。（有关仅在系统级别使用的指令的信息，请参阅 [php.ini 指令的列表]。）
1. （可选）将扩展添加到 PHP 运行时并在 `php.ini` 文件中启用这些扩展。
1. 将 `bin` 目录添加到根目录，并将包含 PHP 运行时的目录置于根目录中（例如 `bin\php`）。
1. 部署应用程序。
1. 导航到 Azure 门户中网站的仪表板，然后单击“配置”。

	![网站仪表板上的“配置”选项卡][configure]

1. 在“处理程序映射”部分，将 `*.php` 添加到 EXTENSION，并添加指向 `php-cgi.exe` 可执行文件的路径。如果你将 PHP 运行时放在应用程序的根目录中的 `bin` 目录下，路径将为 `D:\home\site\wwwroot\bin\php\php-cgi.exe`。

	![指定处理程序映射中的处理程序][handler-mappings]

1. 单击页面底部的“保存”。

	![保存配置设置][save-button]


[免费试用版]: /zh-cn/pricing/1rmb-trial/
[PHP Developer Center Tutorials]: /develop/php/
[How to Configure  Websites]: /documentation/articles/web-sites-configure/
[phpinfo()]: http://php.net/manual/en/function.phpinfo.php
[select-php-version]: ./media/web-sites-php-configure/select-php-version.png
[php.ini 指令的列表]: http://www.php.net/manual/en/ini.list.php
[.user.ini]: http://www.php.net/manual/en/configuration.file.per-user.php
[ini\_set()]: http://www.php.net/manual/en/function.ini-set.php
[configure]: ./media/web-sites-php-configure/configure.png
[app-settings]: ./media/web-sites-php-configure/app-settings.png
[save-button]: ./media/web-sites-php-configure/save-button.png
[http://windows.php.net/download/]: http://windows.php.net/download/
[http://windows.php.net/downloads/releases/archives/]: http://windows.php.net/downloads/releases/archives/
[handler-mappings]: ./media/web-sites-php-configure/handler-mappings.png
[Configure, monitor, and scale your  Websites in Azure]: /zh-cn/documentation/services/web-sites
[Download the Azure SDK for PHP]: /zh-cn/downloads/?sdk=php

<!---HONumber=76-->