<properties linkid="develop-php-website-with-sql-database-and-webmatrix" urlDisplayName="Web w/ SQL + WebMatrix" pageTitle="PHP web site with SQL Database and WebMatrix - Azure" metaKeywords="" description="A tutorial that demonstrates how to use the free WebMatrix IDE to create and deploy a PHP web site that stores data in SQL Database." metaCanonical="" services="" documentationCenter="" title="Create and Deploy a PHP Web Site and SQL Database using WebMatrix" authors="" solutions="" manager="" editor="mollybos" />

# 使用 WebMatrix 创建和部署 PHP 网站与 SQL Database

本教程向你演示如何使用 WebMatrix 开发使用 Azure SQL Database 的 PHP 应用程序并将该应用程序部署到 Azure 网站。WebMatrix 是 Microsoft 提供的一类免费的 Web 开发工具，此工具包含进行网站开发所需的一切。WebMatrix 支持 PHP 并包含用于 PHP 开发的智能感知。

本教程假定你已在计算机上安装 [SQL Server Express][SQL Server Express]，以便能本地测试应用程序。不过，你可以在不安装 SQL Server Express 的情况下完成本教程。相反，你可以将应用程序直接部署到 Azure 网站。

完成本指南之后，你将拥有一个在 Azure 中运行的 PHP-SQL Database 网站。

你将了解到以下内容：

-   如何使用管理门户创建 Azure 网站和 SQL Database。由于默认情况下已在 Azure 网站中启用 PHP，因此运行 PHP 代码没有任何特殊要求。
-   如何使用 WebMatrix 开发 PHP 应用程序。
-   如何使用 WebMatrix 将应用程序发布和重新发布到 Azure。

通过按照本教程中的说明进行操作，你将在 PHP 中构建简单的 Tasklist Web 应用程序。应用程序将托管于 Azure 网站中。以下是正在运行的应用程序的屏幕快照：

![Azure PHP 网站][Azure PHP 网站]

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

## 先决条件

1.  [下载][下载] Tasklist 应用程序文件。Tasklist 应用程序是一个简单的 PHP 应用程序，可利用该应用程序在任务列表中添加和删除项目以及将项目标记为完成。任务列表项目存储在 SQL Database（用于本地测试的 SQL Server Express）中。此应用程序包含以下文件：

-   **index.php**：显示任务并提供用于向列表添加项目的窗体。
-   **additem.php**：向列表中添加一个项。
-   **getitems.php**：获取数据库中的所有项目。
-   **markitemcomplete.php**：将项目的状态更改为已完成。
-   **deleteitem.php**：删除项目。
-   **taskmodel.php**：包含用于在数据库中添加、获取、更新和删除项目的函数。
-   **createtable.php**：为应用程序创建 SQL Database 表。该文件只能被调用一次。

1.  创建一个 SQL Server 数据库，名为`tasklist`。你可以从`sqlcmd` 命令提示符使用以下命令执行此操作：

        >sqlcmd -S <server name>\sqlexpress -U <user name> -P <password>
        1> create database tasklist
        2> GO

    此步骤仅在你需要本地测试应用程序时是必需的。

## 创建网站和 SQL Database

1.  登录到 [管理门户][预览门户]。
2.  单击该门户左下角的“+ 新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  单击“网站”，然后单击“自定义创建”。

    ![自定义创建新的网站][自定义创建新的网站]

    输入“URL”的值，从“数据库”下拉列表中选择“新建 SQL Database”，并在“区域”下拉列表中选择网站的数据中心。单击对话框底部的箭头。

    ![填写网站详细信息][填写网站详细信息]

4.  输入你的数据库的“名称”的值并且选择“新建 SQL Database 服务器”。输入服务器登录名和密码（并确认密码）。选择你将在其中创建新的 SQL Database 服务器的区域。

    ![填写 SQL Database 设置][填写 SQL Database 设置]

    创建网站后，你会看到文本“创建网站‘[SITENAME]’已成功”。接下来，你将获取数据库连接信息。

5.  单击“链接的资源”，然后选择数据库的名称。

    ![链接的资源][链接的资源]

6.  单击“查看连接字符串”。

    ![连接字符串][连接字符串]

从结果对话框的 **PHP** 部分，记下`UID`、`PWD`、`Database` 和`$serverName` 的值。稍后你将使用此信息。

## 安装 WebMatrix

你可以从 [管理门户][预览门户] 安装 WebMatrix。

1.  登录后，导航到网站的“快速启动”页面，并单击该页底部的“WebMatrix”图标：

    ![安装 WebMatrix][安装 WebMatrix]

    按照提示操作以安装 WebMatrix。

2.  安装 WebMatrix 之后，它会尝试将你的网站作为 WebMatrix 项目打开。你可以选择直接编辑你的实时网站或者下载一个本地副本。对于本教程，选择“编辑本地副本”。

3.  当提示下载网站时，请选择“是，从模板库安装”。

    ![下载网站][下载网站]

4.  从可用模板中选择 **PHP**。

    ![模板中的网站][模板中的网站]

5.  选择“空网站”模板。提供网站的名称并单击“下一步”。

    ![提供网站的名称][提供网站的名称]

将在 WebMatrix 上打开你的网站，其中包含一些默认文件。

## 开发应用程序

在下面的几个步骤中，你将通过添加之前下载的文件并进行一些修改来开发 Tasklist 应用程序。但是，你可以添加你自己的现有文件或创建新的文件。

1.  在 WebMatrix 中打开你的网站，通过单击“添加现有”来添加应用程序文件：

    ![WebMatrix - 添加现有文件][WebMatrix - 添加现有文件]

    在生成的对话框中，导航到之前下载的文件，选择所有这些文件并单击“打开”。系统提示时，请选择替换该`index.php` 文件。

2.  接下来，你需要将本地 SQL Server 数据库连接信息添加到`taskmodel.php` 文件。双击`taskmodel.php` 文件以打开该文件，并更新`connect` 函数中的数据库连接信息。（**注意**：跳转到[发布应用程序][发布应用程序]（如果你不需要本地测试应用程序，而是希望直接发布到 Azure 网站。）

        // DB connection info
        $host = "localhost\sqlexpress";
        $user = "your user name";
        $pwd = "your password";
        $db = "tasklist";

    保存`taskmodel.php` 文件。

3.  若要运行应用程序，需要创建`items` 表。右键单击`createtable.php` 文件并选择“在浏览器中启动”。这将在浏览器中启动`createtable.php` ，并执行创建`items` 表的代码（在`tasklist` 数据库中创建）。

    ![WebMatrix - 在浏览器中启动 createtable.php][WebMatrix - 在浏览器中启动 createtable.php]

4.  现在，你可以本地测试应用程序。右键单击`index.php` 文件并选择“在浏览器中启动”。通过添加项目、将项目标记为已完成并删除项目来测试应用程序。

## <span id="Publish"></span></a>发布应用程序

在将应用程序发布到 Azure 网站之前，需要将`taskmodel.php` 中的数据库连接信息与前面获取的连接信息（在[创建 Azure 网站和 SQL Database][创建 Azure 网站和 SQL Database] 一节中）一起更新。

1.  双击`taskmodel.php` 文件以打开该文件，并更新`connect` 函数中的数据库连接信息。

        // DB connection info
        $host = "value of $serverName";
        $user = "value of UID";
        $pwd = "the SQL password you created when creating the web site";
        $db = "value of Database";

    保存`taskmodel.php` 文件。

2.  在 WebMatrix 中单击“发布”，然后在“发布预览”对话框中单击“继续”。

    ![WebMatrix - 发布][WebMatrix - 发布]

3.  导航到 [http://[你的][http://[你的]网站名].chinacloudsites.cn/createtable.php 以创建`items` 表。

4.  最后，导航到 [http://[你的][http://[你的]网站名].chinacloudsites.cn/index.php 以便启动该应用程序。

## 修改并重新发布应用程序

你可以通过编辑之前下载的网站的本地副本轻松地对你的应用程序进行编辑并重新发布，或者可以直接在远程模式下进行编辑。在此，你将简单更改`index.php` 文件中的标题，然后将其直接保存到实时网站。

1.  在 WebMatrix 中单击你的网站的“远程”选项卡，然后选择“打开远程视图”。这将打开你的远程网站以便直接进行编辑。
    ![WebMatrix - 打开远程视图][WebMatrix - 打开远程视图]

2.  双击`index.php` 文件将其打开。
    ![WebMatrix - 打开索引文件][WebMatrix - 打开索引文件]

3.  在 **title** 和 **h1** 标记中将 **My ToDo List** 更改为 **My Task List** 并保存文件。

4.  在完成保存后，单击“运行”按钮以便查看实时网站上的更改。
    ![WebMatrix - 远程启动网站][WebMatrix - 远程启动网站]

## 后续步骤

你已了解如何创建网站并将网站从 WebMatrix 部署到 Azure。若要了解有关 WebMatrix 的详细信息，请参阅下列资源：

-   [WebMatrix for Azure][WebMatrix for Azure]

-   [WebMatrix 网站][WebMatrix 网站]

  [SQL Server Express]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29062
  [Azure PHP 网站]: ./media/web-sites-php-sql-database-use-webmatrix/tasklist_app_windows.png
  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [下载]: http://go.microsoft.com/fwlink/?LinkId=252504
  [创建新的 Azure 网站]: ./media/web-sites-php-sql-database-use-webmatrix/NewWebSite1.jpg
  [自定义创建新的网站]: ./media/web-sites-php-sql-database-use-webmatrix/NewWebSite2.png
  [填写网站详细信息]: ./media/web-sites-php-sql-database-use-webmatrix/NewWebSite3_SQL.png
  [填写 SQL Database 设置]: ./media/web-sites-php-sql-database-use-webmatrix/NewWebSite4_SQL.png
  [链接的资源]: ./media/web-sites-php-sql-database-use-webmatrix/NewWebSite6_SQL.png
  [连接字符串]: ./media/web-sites-php-sql-database-use-webmatrix/NewWebSite7.png
  [安装 WebMatrix]: ./media/web-sites-php-sql-database-use-webmatrix/InstallWebMatrix.png
  [下载网站]: ./media/web-sites-php-sql-database-use-webmatrix/download-site-1.png
  [模板中的网站]: ./media/web-sites-php-sql-database-use-webmatrix/site-from-template.png
  [提供网站的名称]: ./media/web-sites-php-sql-database-use-webmatrix/site-from-template-2.png
  [WebMatrix - 添加现有文件]: ./media/web-sites-php-sql-database-use-webmatrix/edit_addexisting.png
  [发布应用程序]: #Publish
  [WebMatrix - 在浏览器中启动 createtable.php]: ./media/web-sites-php-sql-database-use-webmatrix/edit_run.png
  [创建 Azure 网站和 SQL Database]: #CreateWebsite
  [WebMatrix - 发布]: ./media/web-sites-php-sql-database-use-webmatrix/edit_publish.png
  [http://[你的]: http://[your
  [WebMatrix - 打开远程视图]: ./media/web-sites-php-sql-database-use-webmatrix/OpenRemoteView.png
  [WebMatrix - 打开索引文件]: ./media/web-sites-php-sql-database-use-webmatrix/Remote_editIndex.png
  [WebMatrix - 远程启动网站]: ./media/web-sites-php-sql-database-use-webmatrix/Remote_run.png
  [WebMatrix for Azure]: http://go.microsoft.com/fwlink/?LinkID=253622&clcid=0x409
  [WebMatrix 网站]: http://www.microsoft.com/click/services/Redirect2.ashx?CR_CC=200106398
