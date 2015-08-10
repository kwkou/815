<properties 
   pageTitle="web-sites-connect-to-redis-using-memcache-protocol" 
   description="使用 Memcached 协议将 Web 应用程序连接到 Redis Cache" 
   services="websites" 
   documentationCenter="php" 
   authors="syntaxc4" 
   manager="yochayk" 
   editor="riande"/>
   
<tags ms.service="websites" ms.date="03/19/2015" wacn.date="04/11/2015"/>


# 通过 Memcache 协议将 Web 应用连接到 Redis Cache

Azure Websites 是一个开放的平台，提供对各种编程语言的支持，其中包括开源编程语言，例如 PHP、Python 和 Node.js。构建高缩放应用程序（尤其是那些在多个服务器中分发的应用程序）时，常见的做法是引入一个集中式的内存中缓存机制，以抵消对外部依赖关系（例如第三方服务或某个应用程序的数据层）的时间密集型调用。在许多新方案中，[Redis][12]（一种常用的开源数据结构服务器）可加强应用程序的内存中缓存角色。Azure Redis Cache Service 是在 Windows Azure 中建议的第一方缓存机制。在许多应用程序和应用程序框架中，内存中缓存的角色可通过 [memcache][13] 增强，后者是另一个常用的可缩放缓存系统。

为了满足大多数应用程序（尤其是那些使用开源语言的应用程序）的需求，Azure Websites 现在公开了一个可通过代理方式将缓存调用转到 Azure Redis Cache Service 的本地 memcache 终结点。这样一来，任何使用 memcached 协议进行通信的应用程序都可以将数据缓存到某个集中式的缓存机制，从而降低应用程序的事务开销。本文将概述如何针对 WordPress 站点将 Azure Websites memcache 填充程序配置为集中式的内存中缓存并进行验证。必须指出，此 memcache 填充程序在协议级别工作，因此只要它使用 memcached 协议通信，就不会针对任何特定的应用程序或应用程序框架。


## 先决条件

Memcache 填充程序可以与任何应用程序一起使用，前提是使用 memcached 协议通信。就此特定示例来说，引用应用程序是一个可缩放的 WordPress 站点，它可以从 Azure Marketplace 设置。 

请按照以下文章中所述的步骤操作：

* [在 Azure 中部署可缩放的 WordPress 站点][0]
* [设置 Azure Redis Cache Service 的实例][1]

部署可缩放的 WordPress 站点并设置 Redis Cache 实例后，你随时可以启用 Azure Websites 中的 memcache 填充程序。

## 启用 Azure Websites memcache 填充程序

若要配置 Azure Websites 填充程序，你必须创建三项应用设置。这可以使用多种方法来完成，其中包括[全功能门户][2]、[Azure PowerShell Cmdlet][4] 或 [Azure 跨平台命令行工具][5]。在本文中，我将使用预览门户来设置应用程序设置。从"Redis Cache Service 设置"边栏选项卡中，可以检索以下值。

![Azure Redis Cache Settings Blade](./media/web-sites-connect-to-redis-using-memcache-protocol/1-azure-redis-cache-settings.png)

### 添加 REDIS_HOST 应用设置

我们将创建的第一项应用设置是 **REDIS\_HOST** 应用设置，该设置将设置一个目标，以便缓存信息通过填充程序后即将其发送到该目标。REDIS_HOST 应用设置所需的值可以从 Redis Cache Service 的"属性"边栏选项卡检索。

![Azure Redis Cache Host Name](./media/web-sites-connect-to-redis-using-memcache-protocol/2-azure-redis-cache-hostname.png)

将应用设置的项设置为 **REDIS\_HOST**，将应用设置的值设置为 Redis Cache Service 的**主机名**。

![Azure Website AppSetting REDIS_HOST](./media/web-sites-connect-to-redis-using-memcache-protocol/3-azure-website-appsettings-redis-host.png)

### 添加 REDIS_KEY 应用设置

我们将创建的第二项应用设置是 **REDIS\_KEY** 应用设置，该设置将提供安全地访问 Redis Cache Service 所需的身份验证令牌。REDIS_KEY 应用设置所需的值可以从 Redis Cache Service 的"访问密钥"边栏选项卡检索。

![Azure Redis Cache Primary Key](./media/web-sites-connect-to-redis-using-memcache-protocol/4-azure-redis-cache-primarykey.png)

将应用设置的项设置为 **REDIS\_KEY**，将应用设置的值设置为 Redis Cache Service 的**主键**。

![Azure Website AppSetting REDIS_KEY](./media/web-sites-connect-to-redis-using-memcache-protocol/5-azure-website-appsettings-redis-primarykey.png)

### 添加 MEMCACHESHIM_REDIS_ENABLE 应用设置

将通过最后一个应用设置来启用 Azure Websites 中的 Memcache 填充程序，以便使用 REDIS_HOST 和 REDIS_KEY 来连接和代理对 Azure Redis Cache Service 的缓存调用。将应用设置的项设置为 **MEMCACHESHIM\_REDIS\_ENABLE**，将其值设置为 **true**。

![Azure Website AppSetting MEMCACHESHIM_REDIS_ENABLE](./media/web-sites-connect-to-redis-using-memcache-protocol/6-azure-website-appsettings-enable-shim.png)

添加完这三 (3) 项应用设置以后，单击**"保存"**。

## 启用针对 PHP 的 Memcache 扩展

若要让应用程序使用 memcached 协议进行沟通，必须将 memcache 扩展安装到 PHP 编程语言中。

### 下载 php_memcache 扩展

浏览到 [PECL][6]，在缓存类别下单击["memcache"][7]。在下载列中，单击 DLL 链接。

![PHP PECL Website](./media/web-sites-connect-to-redis-using-memcache-protocol/7-php-pecl-website.png)

下载 Websites 中启用的 PHP 版本的非线性安全 (NTS) x86 链接中的内容。（默认为 PHP 5.4）

![PHP PECL Website Memcache Package](./media/web-sites-connect-to-redis-using-memcache-protocol/8-php-pecl-memcache-package.png)

### 启用 php_memcache 扩展

下载文件之后，将 **php\_memcache.dll** 解压缩并上载到 **d:\\home\\site\\wwwroot\\bin\\ext\\** 目录。php_memcache.dll 上载到网站以后，需要启用扩展，使之成为 PHP Runtime 的扩展。若要启用 memcache 扩展，请打开网站的"应用程序设置"边栏选项卡，然后添加一个新的应用设置，其项为 **PHP\_EXTENSIONS**，其值为 **bin\\ext\\php_memcache.dll**。


> 如果该站点要求加载多个 PHP 扩展，则 PHP_EXTENSIONS 的值应该是一个由逗号分隔的 dll 文件相对路径的列表。

![Azure Website AppSetting PHP_EXTENSIONS](./media/web-sites-connect-to-redis-using-memcache-protocol/9-azure-website-appsettings-php-extensions.png)

完成后，单击**"保存"**。

## 安装 Memcache WordPress 插件

在"WordPress 插件"页上，单击**"添加新项"**按钮。

![WordPress Plugin Page](./media/web-sites-connect-to-redis-using-memcache-protocol/10-wordpress-plugin.png)

在搜索框中，键入**"memcached"**然后按 Enter 键。

![WordPress Add New Plugin](./media/web-sites-connect-to-redis-using-memcache-protocol/11-wordpress-add-new-plugin.png)

在列表中查找**"Memcached 对象缓存"**，然后单击**"立即安装"**按钮。

![WordPress Install Memcache Plugin](./media/web-sites-connect-to-redis-using-memcache-protocol/12-wordpress-install-memcache-plugin.png)

### 启用 memcache WordPress 插件

> 按照此博客中关于[如何启用 Azure Websites 中的站点扩展][6]的说明，安装 Visual Studio Online"Monaco"。


```php
$memcached_servers = array(
	'default' => array('localhost:' . getenv("MEMCACHESHIM_PORT"))
);
```

将 **object-cache.php** 从 **wp-content/memcached** 文件夹拖放到 **wp-content** 文件夹，以便启用 Memcache 对象缓存功能。

![Locate the memcache object-cache.php plugin](./media/web-sites-connect-to-redis-using-memcache-protocol/13-locate-memcache-object-cache-plugin.png)

现在，**object-cache.php** 文件位于 **wp-content** 文件夹中，Memcached 对象缓存已启用。

![Enable the memcache object-cache.php plugin](./media/web-sites-connect-to-redis-using-memcache-protocol/14-enable-memcache-object-cache-plugin.png)

## 验证 Memcache 对象缓存插件正常运行

启用 Azure Websites Memcache 填充程序的所有步骤现在均已完成，但仍有一个重要步骤有待完成，即验证数据是否填充到 Redis Cache Service 中。

### 启用 Azure Redis Cache Service 中的非 SSL 端口支持

> 撰写本文档时，Redis CLI 尚不支持 SSL 连接，因此以下步骤是必需的。

浏览到已为本站点创建的 Redis Cache Service。打开缓存边栏选项卡后，单击"设置"图标。

![Azure Redis Cache Settings Button](./media/web-sites-connect-to-redis-using-memcache-protocol/15-azure-redis-cache-settings-button.png)

从列表中选择**"访问端口"**。

![Azure Redis Cache Access Port](./media/web-sites-connect-to-redis-using-memcache-protocol/16-azure-redis-cache-access-port.png)

当系统询问你是否**"仅允许通过 SSL 访问"**时，单击**"否"**。

![Azure Redis Cache Access Port SSL Only](./media/web-sites-connect-to-redis-using-memcache-protocol/17-azure-redis-cache-access-port-ssl-only.png)

你将看到非 SSL 端口现在已设置。单击"保存"。

![Azure Redis Cache Redis Access Portal Non-SSL](./media/web-sites-connect-to-redis-using-memcache-protocol/18-azure-redis-cache-access-port-non-ssl.png)

### 从 redis-cli 连接到 Azure Redis Cache

> 此步骤假定 redis 已通过本地方式安装在你的开发计算机上。[按以下说明通过本地方式安装 Redis][9]。

打开终端或你的所选控制台。键入以下命令：

```shell
redis-cli -h <hostname-for-redis-cache> -a <primary-key-for-redis-cache> -p 6379
```

将 **<hostname-for-redis-cache>** 替换为实际的 xxxxx.redis.cache.windows.net 主机名，将 **<primary-key-for-redis-cache>** 替换为缓存的访问密钥，然后按 Enter。一旦 CLI 连接到 Redis Cache Service 并发出 Redis 命令，即可选择列出相关密钥。

![Connect to Azure Redis Cache from Redis CLI in Terminal](./media/web-sites-connect-to-redis-using-memcache-protocol/19-redis-cli-terminal.png)

通过调用来列出密钥时，该调用会返回一个值，如果没有返回结果列表，则可尝试导航到站点，然后再试一次。

## 结束语

恭喜！应用程序现在有了一个集中式的内存中缓存，这将有助于吞吐量的提高。请记住，Azure Websites Memcache 填充程序可以用于任何 memcache 客户端，不用考虑编程语言或应用程序框架。若要提供反馈或者提问有关 Azure Websites memcache 填充程序的问题，请在 [MSDN 论坛][10]或 [Stackoverflow][11] 上发布相关帖子。

[0]: https://msdn.microsoft.com/zh-CN/library/dn690516.aspx
[1]: http://azure.microsoft.com/blog/2014/09/15/how-to-host-a-scalable-and-optimized-wordpress-for-azure-in-minutes/
[3]: http://manage.windowsazure.cn
[5]: /downloads
[6]: http://pecl.php.net
[7]: http://pecl.php.net/package/memcache
[8]: http://blog.syntaxc4.net/post/2015/02/05/how-to-enable-a-site-extension-in-azure-websites.aspx
[9]: http://redis.io/download#installation
[10]: https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurewebsitespreview
[11]: http://stackoverflow.com/questions/tagged/azure-web-sites
[12]: http://redis.io
[13]: http://memcached.org


<!--HONumber=51-->