<properties title="How to Configure PHP in Azure Web Sites" pageTitle="How to Configure PHP in Azure Web Sites" metaKeywords="Azure, Azure Web Sites, configuration, PHP" description="Learn how to configure the default PHP installation or add a custom PHP installation in Azure Web Sites." services="Web Sites" documentationCenter="PHP" authors="" />

# 如何在 Azure 网站中配置 PHP

本指南将向你演示如何执行以下操作：在 Azure 网站中配置内置 PHP 运行时，提供自定义 PHP 运行时以及在 Azure 网站中启用扩展。若要使用 Azure 网站，请注册以[免费试用][免费试用]。若要充分利用本指南，你应先在 Azure 网站中创建一个 PHP 站点（请参阅 [PHP 开发中心教程][PHP 开发中心教程]）。有关在 Azure 网站中配置站点的常规信息，请参阅[如何配置网站][如何配置网站]。

## 目录

-   [什么是 Azure 网站？][什么是 Azure 网站？]
-   [如何：更改默认 PHP 配置][如何：更改默认 PHP 配置]
-   [如何：启用内置 PHP 运行时中的扩展][如何：启用内置 PHP 运行时中的扩展]
-   [如何：使用自定义 PHP 运行时][如何：使用自定义 PHP 运行时]
-   [后续步骤][后续步骤]

## <a name="WhatIs"></a>什么是 Azure 网站？

利用 Azure 网站，你可以在 Azure 上构建可高度缩放的网站。可以快速而轻松地将网站部署到可高度缩放的云环境，这样你便能首先构建小型网站，然后随着流量的增加来扩展网站。Azure 网站使用你所选的语言和开放源应用程序，并支持使用 Git、FTP 和 TFS 进行的部署。可以轻松集成其他服务，如 MySQL、SQL Database、Caching、CDN 和存储。

## <a name="ChangeBuiltInPHP"></a>如何：更改内置 PHP 配置

默认情况下，将安装 PHP 5.4 并且在创建 Azure 网站时立即可用。查看可用发行版、其默认配置以及已启用的扩展的最佳方式是部署调用 [phpinfo()][phpinfo()] 的脚本。

PHP 5.5 也可用，但它在默认情况下不启用。若要启用它，请执行下列步骤：

1.  浏览到 Azure 门户中的网站的仪表板，单击“配置”。

    ![网站仪表板上的“配置”选项卡][网站仪表板上的“配置”选项卡]

2.  单击“PHP 5.5”。

    ![选择 PHP 版本][选择 PHP 版本]

3.  单击页面底部的“保存”。

    ![保存配置设置][保存配置设置]

对于任一内置 PHP 运行时，可通过执下列步骤来更改任何不是仅在系统级别使用的指令的配置选项。（有关仅在系统级别使用的指令的信息，请参阅 [php.ini 指令的列表][php.ini 指令的列表]。）

1.  将 [.user.ini][.user.ini] 文件添加到根目录。
2.  将配置设置添加到`.user.ini` 文件（使用你将在`php.ini` 文件中使用的语法）。例如，如果你想打开`display_errors` 设置并将`upload_max_filesize` 设置设为 10M，那么你的`.user.ini` 文件就会包含这样的文本：

        ; Example Settings
        display_errors=On
        upload_max_filesize=10M

3.  部署应用程序。
4.  重新启动网站。（需要进行重新启动，因为 PHP 读取`.user.ini` 文件的频率受`user_ini.cache_ttl` 设置的约束，该设置是一个系统级别设置且默认值为 300 秒（5 分钟）。重新启动站点会强制 PHP 读取`.user.ini` 文件中的新设置。）

作为使用`.user.ini` 文件的替代方法，你可以使用脚本中的 [ini\_set()][ini\_set()] 函数来设置不是系统级别指令的配置选项。

## <a name="EnableExtDefaultPHP"></a>如何：启用默认 PHP 运行时中的扩展

如上一节所述，查看默认 PHP 版本、其默认配置以及已启用的扩展的最佳方式是部署调用 [phpinfo()][phpinfo()] 的脚本。若要启用其他扩展，请执行下列步骤。

1.  将一个`bin` 目录添加到你的根目录。
2.  将`.dll` 扩展文件置于`bin` 目录中（例如，`php_mongo.dll`）。确保扩展与默认版本的 PHP（撰写本文时为 PHP 5.4）兼容，并且是 VC9 版本且与非线程安全 (nts) 兼容。
3.  部署应用程序。
4.  导航到 Azure 门户中的网站的仪表板，并单击“配置”。

    ![网站仪表板上的“配置”选项卡][网站仪表板上的“配置”选项卡]

5.  在“应用程序设置”部分，创建密钥 **PHP\_EXTENSIONS** 和值 **bin\\your-ext-file**。若要启用多个扩展，请包括`.dll` 文件的逗号分隔的列表。

    ![启用应用程序设置中的扩展][启用应用程序设置中的扩展]

6.  单击页面底部的“保存”。

    ![保存配置设置][保存配置设置]

## <a name="UseCustomPHP"></a>如何：使用自定义 PHP 运行时

Azure 网站可以使用提供的 PHP 运行时（而非默认 PHP 运行时）来执行 PHP 脚本。你提供的运行时可由同样是你提供的`php.ini` 文件来配置。若要在 Azure 网站中使用自定义 PHP 运行时，请执行下列步骤。

1.  获取非线程安全、VC9 兼容版本的 PHP for Windows。可在此处找到 PHP for Windows 最新版本：[][]<http://windows.php.net/download/></a>。可在此处的存档中找到旧版本：[][1]<http://windows.php.net/downloads/releases/archives/></a>。
2.  修改用于你的运行时的`php.ini` 文件。请注意，Azure 网站将忽略作为任何仅在系统级别使用的指令的配置设置。（有关仅在系统级别使用的指令的信息，请参阅 [php.ini 指令的列表][php.ini 指令的列表]）。
3.  （可选）将扩展添加到 PHP 运行时并在`php.ini` 文件中启用这些扩展。
4.  将`bin` 目录添加到根目录，并将包含 PHP 运行时的目录置于根目录中（例如， `bin\php`）。
5.  部署应用程序。
6.  导航到 Azure 门户中的网站的仪表板，并单击“配置”。

    ![网站仪表板上的“配置”选项卡][网站仪表板上的“配置”选项卡]

7.  在“处理程序映射”部分中，将`*.php` 添加到 EXTENSION，并将路径添加到`php-cgi.exe` 可执行文件。如果你将 PHP 运行时置于你应用程序根目录的`bin` 目录中，该路径将会是 `D:\home\site\wwwroot\bin\php\php-cgi.exe`。

    ![指定处理程序映射中的处理程序][指定处理程序映射中的处理程序]

8.  单击页面底部的“保存”。

    ![保存配置设置][保存配置设置]

## <a name="NextSteps"></a>后续步骤

现在，你已了解如何在 Azure 网站中配置 PHP，访问下列链接可了解如何执行更多操作。

-   [在 Azure 中配置、监视和缩放网站][在 Azure 中配置、监视和缩放网站]
-   [下载 Azure SDK for PHP][下载 Azure SDK for PHP]

  [免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [PHP 开发中心教程]: http://azure.microsoft.com/zh-cn/develop/php/
  [如何配置网站]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-configure/
  [什么是 Azure 网站？]: #WhatIs
  [如何：更改默认 PHP 配置]: #ChangeBuiltInPHP
  [如何：启用内置 PHP 运行时中的扩展]: #EnableExtDefaultPHP
  [如何：使用自定义 PHP 运行时]: #UseCustomPHP
  [后续步骤]: #NextSteps
  [phpinfo()]: http://php.net/manual/en/function.phpinfo.php
  [网站仪表板上的“配置”选项卡]: ./media/web-sites-php-configure/configure.png
  [选择 PHP 版本]: ./media/web-sites-php-configure/select-php-version.png
  [保存配置设置]: ./media/web-sites-php-configure/save-button.png
  [php.ini 指令的列表]: http://www.php.net/manual/en/ini.list.php
  [.user.ini]: http://www.php.net/manual/en/configuration.file.per-user.php
  [ini\_set()]: http://www.php.net/manual/en/function.ini-set.php
  [启用应用程序设置中的扩展]: ./media/web-sites-php-configure/app-settings.png
  []: http://windows.php.net/download/
  [1]: http://windows.php.net/downloads/releases/archives/
  [指定处理程序映射中的处理程序]: ./media/web-sites-php-configure/handler-mappings.png
  [在 Azure 中配置、监视和缩放网站]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/
  [下载 Azure SDK for PHP]: http://www.windowsazure.cn/zh-cn/downloads/?sdk=php
