<properties linkid="migrating-drupal-to-azure-websites" urlDisplayName="Migrating Drupal to Azure Web Sites" pageTitle="Migrating Drupal to Azure Web Sites" metaKeywords="Drupal, PHP, Web Sites" description="Migrate a Drupal PHP site to Azure Web Sites." metaCanonical="" services="web-sites" documentationCenter="PHP" title="Migrating Drupal to Azure Web Sites" authors="jroth" solutions="" manager="paulettm" editor="mollybos" />

# 将 Drupal 迁移到 Azure 网站

因为 Azure 网站支持 PHP 和 MySQL，所以，将 Drupal 网站迁移到 Azure 网站相对比较简单。此外，因为 Drupal 和 PHP 可在任何平台上运行，所以，该过程应该不论你的当前平台如何，都适用于将 Drupal 迁移到 Azure 网站。此外，Drupal 安装可能会变化很大，因此，在以下材料中可能未涵盖一些独有的迁移步骤。请注意不要使用 Drush 工具，因为在 Azure 网站上不支持该工具。

> [WACOM.NOTE]
> 如果你在迁移复杂的大型 Drupal 应用程序，还可以考虑使用 Azure 云服务。有关网站和云服务之间的差异的详细信息，请参阅 [Azure 网站、云服务和虚拟机：何时使用何种产品？][Azure 网站、云服务和虚拟机：何时使用何种产品？]。有关将 Drupal 迁移到云服务的帮助，请参阅[将 Drupal 网站从 LAMP 迁移到 Azure][将 Drupal 网站从 LAMP 迁移到 Azure]。

## 目录

-   [创建 Azure 网站][创建 Azure 网站]
-   [复制数据库][复制数据库]
-   [修改 Settings.php][修改 Settings.php]
-   [部署 Drupal 代码][部署 Drupal 代码]
-   [相关信息][相关信息]

## <a name="create-siteanddb"></a><span class="short-header">创建 Azure 网站和 MySQL 数据库</span>1. 创建 Azure 网站和 MySQL 数据库

首先，完成以下分步教程以便了解如何使用 MySQL 创建新网站：[创建 PHP-MySQL Azure 网站并使用 Git 进行部署][创建 PHP-MySQL Azure 网站并使用 Git 进行部署]。如果想要使用 Git 发布你的 Drupal 网站，则按照该教程中的步骤执行，这些部署说明了如何配置 Git 存储库。请确保按照**获取远程 MySQL 连接信息**部分中的说明执行，因为你在后面将需要这些信息。出于部署你的 Drupal 网站的目的，你可以忽略该教程的其余部分，但如果你还不熟悉 Azure 网站（以及 Git），继续阅读该教程可能会对你有所裨益。

在你设置了具有 MySQL 数据库的新网站后，现在将具有你的 MySQL 数据库连接信息和（可选）的 Git 存储库。下一步是将你的数据库复制到 Azure 网站中的 MySQL。

## <a name="copy-database"></a><span class="short-header">将数据库复制到 Azure 网站中的 MySQL</span>2. 将数据库复制到 Azure 网站中的 MySQL

有许多方法可以将数据库迁移到 Azure。适合于 MySQL 数据库的一个方法是使用 [MySqlDump][MySqlDump] 工具。下面的命令提供一个示例，演示如何从本地计算机复制到 Azure 网站：

    mysqldump -u local_username --password=local_password  drupal | mysql -h remote_host -u remote_username --password=remote_password remote_db_name

当然，你需要为你的现有 Drupal 数据库提供用户名和密码。你还需要提供你在第一步中为 MySQL 数据库创建的主机名、用户名、密码和数据库名称。这些信息位于你之前收集的连接字符串信息中。该连接字符串应具有与以下字符串相似的格式：

    Database=remote_db_name;Data Source=remote_host;User Id=remote_username;Password=remote_password

根据你的数据库的大小，该复制过程可能要用几分钟。

现在，你的 Drupal 数据库已处于 Azure 网站中了。在部署 Drupal 代码前，你需要修改该数据库，以便它可以连接到新数据库。

## <a name="modify-settingsphp"></a><span class="short-header">在 settings.php 中修改数据库连接信息</span>3. 在 settings.php 中修改数据库连接信息

在这里，你需要创建新的数据库连接信息。在文本编辑器中打开 **/drupal/sites/default/setting.php** 文件，用针对新数据库的正确值替代 **$databases** 数组中的“database”、“username”、“password”和“host”的值。完成后，你应该具有如下代码：

    $databases = array (
       'default' => 
       array (
         'default' => 
         array (
           'database' => 'remote_db_name',
           'username' => 'remote_username',
           'password' => 'remote_password',
           'host' => 'remote_host',
           'port' => '',
           'driver' => 'mysql',
           'prefix' => '',
         ),
       ),
     );

保存 **settings.php** 文件。现在可开始部署了。

## <a name="deploy-drupalcode"></a><span class="short-header">使用 Git 或 FTP 部署 Drupal 代码</span>4. 使用 Git 或 FTP 部署 Drupal 代码

最后一步是使用 Git 或 FTP 将你的代码部署到 Azure 网站。

如果你在使用 FTP，则从你的网站的仪表板获取 FTP 主机名和用户名。然后，使用任意 FTP 客户端将 Drupal 文件上载到远程站点的 **/site/wwwroot** 文件夹。

如果你在使用 Git，则应已在前面的步骤中设置了 Git 存储库。你必须在本地计算机上安装 Git。然后，在创建了存储库后按照提供的说明操作。

> [WACOM.NOTE]
> 根据你的 Git 设置，可能需要编辑 .gitignore 文件（一个隐藏的文件，与你在执行了 git commit 后在你的本地根目录中创建的 .git 文件夹同级）。这指定你的 Drupal 应用程序中可忽略的文件。如果这包含应部署的文件，请删除这些条目，以便不忽略这些文件。

在将 Drupal 部署到 Azure 网站后，你可以继续通过 Git 或 FTP 部署更新。

## <a name="related-information"></a><span class="short-header">相关信息</span>相关信息

有关详细信息，请参阅下列博客文章和主题：

-   [Azure 网站，PHP 视角][Azure 网站，PHP 视角]
-   [Azure 网站、云服务和虚拟机：何时使用何种产品？][Azure 网站、云服务和虚拟机：何时使用何种产品？]
-   [使用 .user.ini 文件在 Azure 网站中配置 PHP][使用 .user.ini 文件在 Azure 网站中配置 PHP]
-   [Azure 集成模块][Azure 集成模块]
-   [Azure Blob 存储模块][Azure Blob 存储模块]

  [Azure 网站、云服务和虚拟机：何时使用何种产品？]: http://azure.microsoft.com/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/
  [将 Drupal 网站从 LAMP 迁移到 Azure]: http://blogs.msdn.com/b/brian_swan/archive/2012/03/19/azure-real-world-migrating-drupal-from-lamp-to-windows-azure.aspx
  [创建 Azure 网站]: #create-siteanddb
  [复制数据库]: #copy-database
  [修改 Settings.php]: #modify-settingsphp
  [部署 Drupal 代码]: #deploy-drupalcode
  [相关信息]: #related-information
  [创建 PHP-MySQL Azure 网站并使用 Git 进行部署]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-php-mysql-deploy-use-git/
  [Azure 网站，PHP 视角]: http://blogs.msdn.com/b/silverlining/archive/2012/06/12/windows-azure-websites-a-php-perspective.aspx
  [使用 .user.ini 文件在 Azure 网站中配置 PHP]: http://blogs.msdn.com/b/silverlining/archive/2012/07/10/configuring-php-in-windows-azure-websites-with-user-ini-files.aspx
  [Azure 集成模块]: https://drupal.org/project/azure_auth
  [Azure Blob 存储模块]: https://drupal.org/project/azure_blob
