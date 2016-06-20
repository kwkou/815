<properties 
  pageTitle="Php-网站 - Azure 微软云"
  metakeywords="" 
  description="" 
  services="" 
  documentationCenter="php" 
  authors="" 
  manager="Tiffena" 
  editor="EricChen"/>
<tags ms.service=""
    ms.date=""
    wacn.date="01/11/2016"
    />


<h1 id="menu-php-websites">网站</h1>
<h2 id="header-0">常规</h2>
<h3>指南：<a href="/documentation/articles/fundamentals-application-models/#WebSites">什么是 Azure 网站？</a></h3>
<p>Azure 为运行的应用程序提供三种计算模型：Azure 网站、Azure 虚拟机和 Azure 云服务。本主题概述了三种模型和信息，以帮助你确定适用于你的应用程序的模型。</p>
<h2 id="header-1">创建</h2>
<h3>功能指南：<a href="/documentation/articles/web-sites-php-create-web-sites/">创建 PHP 网站</a></h3>
<p>了解如何使用 Azure 门户、用于 Mac 和 Linux 的 Azure 命令行工具或 Azure PowerShell cmdlet 在 Azure 网站中创建 PHP 网站。</p>
<h3>教程：<a href="/documentation/articles/web-sites-php-mysql-deploy-use-git/">带 MySQL 的 PHP 网站（使用 Git）</a></h3>
<p>实现允许发送和检索存储在 MySQL 数据库中的注册信息的简单网站。使用所选的文本编辑器，然后使用 Git 上载应用程序。</p>
<h3>教程：<a href="/documentation/articles/web-sites-php-sql-database-deploy-use-git/">带 SQL 数据库 的 PHP 网站（使用 Git）</a></h3>
<p>实现允许发送和检索存储在 Azure SQL 数据库实例中的注册信息的简单网站。使用所选的文本编辑器，然后使用 Git 上载应用程序。</p>
<!--
<h3>教程：<a href="/documentation/articles/web-sites-php-web-site-gallery/">市场中的 WordPress 网站</a></h3>
<p>了解如何通过市场创建新网站并立即部署该网站。只需不到 5 分钟的时间，你即可启动并运行新的 WordPress 网站。</p>
<h3>教程：<a href="/documentation/articles/web-sites-php-mysql-use-webmatrix/">带 MySQL 的 PHP 网站（使用 WebMatrix）</a></h3>
<p>实现允许检索和创建存储在 MySQL 数据库中的待办事项的简单网站。使用 WebMatrix IDE 构建和部署应用程序。</p>-->
<h3>教程：<a href="/documentation/articles/web-sites-php-sql-database-use-webmatrix/">带 SQL 数据库 的 PHP 网站（使用 WebMatrix）</a></h3>
<p>实现允许检索和创建存储在 Azure SQL 数据库实例中的待办事项的简单网站。使用 WebMatrix IDE 构建和部署应用程序。</p>

<h3>教程：<a href="/documentation/articles/web-sites-php-mysql-deploy-use-ftp/">带 MySQL 的 PHP 网站（使用 FTP）</a></h3>
<p>实现允许检索和创建存储在 MySQL 数据库中的待办事项的简单网站。使用所选的文本编辑器，然后使用 FTP 上载应用程序。</p>
<!--<h3>教程：<a href="/documentation/articles/web-sites-php-storage/">使用 Azure 存储的 PHP 网站</a></h3>
<p>创建可存储和访问 Azure 表存储中的数据的网站。你将了解如何创建和使用 Azure 存储帐户，以及如何使用 PHP 客户端库创建、查询和删除表实体。</p>-->
<h3>如何：<a href="/documentation/articles/web-sites-create-web-jobs/">使用 Web 作业在 Azure 网站中运行后台任务。</a></h3>
<p>了解如何使用 Azure 管理门户部署 Web 作业。</p>
<h2 id="header-2">配置</h2>
<h3>如何：<a href="/documentation/articles/web-sites-php-configure/">在网站中配置 PHP</a></h3>
<p>默认情况下，Azure 网站已启用 PHP。本文说明如何更改默认 PHP 运行时的配置，提供自定义 PHP 运行时以及启用扩展。</p>
<!--<h3>如何：<a href="/documentation/articles/web-sites-transform-extend/">转换和扩展网站</a></h3>
<p>通过使用 XML 文档转换 (XDT) 声明，你可以转换 Azure 网站中的 ApplicationHost.config 文件。还可以使用 XDT 声明添加私有网站扩展，以实现自定义网站管理操作。本文中包含一个示例 PHP Manager 网站扩展，可实现通过 Web 界面管理 PHP 设置。</p>-->
<h3>如何：<a href="/documentation/articles/web-sites-custom-domain-name/">为 Azure 网站配置自定义域名</a></h3>
<p>当你创建网站时，Azure 会提供 azurewebsites.net 域的友好子域，以便你的用户可以使用类似 http://&lt;mysite&gt;.azurewebsites.net. 的 URL 访问你的网站。但是，如果你将网站配置为标准模式，则可将该网站映射到你自己的域名，例如 contoso.com。</p>
<h3>如何：<a href="/documentation/articles/web-sites-configure-ssl-certificate/">为 Azure 网站配置 SSL</a></h3>
<p>安全套接字层 (SSL) 加密是用于保护通过 Internet 发送的数据的最常见方法。此常见任务讨论了如何为网站指定 HTTPS 终结点以及如何上载 SSL 证书来保护你的应用程序。</p>

<h3>如何：<a href="/documentation/articles/web-sites-php-convert-wordpress-multisite/">将 WordPress 网站转换为 Multisite</a></h3>
<p>了解如何将现有 WordPress 网站转换到多站点安装，并利用 Azure 平台的可伸缩性来运行无限数量的自定义网站，每个网站均有自己的域。</p>
<!--
<h3>如何：<a href="/documentation/articles/web-sites-php-migrate-drupal/">将 Drupal 应用程序迁移到 Azure 网站</a></h3>
<p>了解如何将现有 Drupal 网站和 MySQL 数据库移至 Azure 网站。</p>-->
