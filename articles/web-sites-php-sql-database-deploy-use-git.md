<properties linkid="develop-php-website-with-sql-database-and-git" urlDisplayName="Web w/ SQL + Git" pageTitle="PHP web site with SQL Database and Git - Azure tutorial" metaKeywords="" description="A tutorial that demonstrates how to create a PHP web site that stores data in SQL Database and use Git deployment to Azure." metaCanonical="" services="web-sites,sql-database" documentationCenter="PHP" title="Create a PHP web site with a SQL Database and deploy using Git" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

# 创建带 SQL Database 的 PHP 网站并使用 Git 进行部署

本教程演示如何创建带 Azure SQL Database 的 PHP Azure 网站以及如何使用 Git 部署该网站。本教程假定你已在计算机上安装 [PHP][PHP]、[SQL Server Express][SQL Server Express]、[Microsoft Drivers for SQL Server for PHP][Microsoft Drivers for SQL Server for PHP]、Web 服务器和 [Git][Git]。完成本指南之后，你将拥有一个在 Azure 中运行的 PHP-SQL Database 网站。

> [WACOM.NOTE]
> 你可以使用 [Microsoft Web 平台安装程序][Microsoft Web 平台安装程序]安装和配置 PHP、SQL Server Express、Microsoft Drivers for SQL Server for PHP 和 Internet Information Services (IIS)。

你将了解到以下内容：

-   如何使用管理门户创建 Azure 网站和 SQL Database。由于默认情况下已在 Azure 网站中启用 PHP，因此运行 PHP 代码没有任何特殊要求。
-   如何使用 Git 将应用程序发布和重新发布到 Azure。

通过按照本教程中的说明进行操作，你将在 PHP 中构建简单的注册 Web 应用程序。应用程序将托管于 Azure 网站中。以下是已完成应用程序的屏幕快照：

![Azure PHP 网站][Azure PHP 网站]

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

## 创建 Azure 网站并设置 Git 发布

按照以下步骤创建 Azure 网站和 SQL Database：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  单击门户左下角的“新建”图标。
    ![新建 Azure 网站][新建 Azure 网站]

3.  单击“网站”，然后单击“自定义创建”。

    ![自定义创建新的网站][自定义创建新的网站]

    输入“URL”的值，从“数据库”下拉列表中选择“新建 SQL Database”，并在“区域”下拉列表中选择网站的数据中心。单击对话框底部的箭头。

    ![填写网站详细信息][填写网站详细信息]

4.  输入数据库的“名称”的值，选择“版本”[（Web 版或企业版）][（Web 版或企业版）]，再依次选择数据库的“最大大小”、“排序规则”和“新建 SQL Database 服务器”。单击对话框底部的箭头。

    ![填写 SQL Database 设置][填写 SQL Database 设置]

5.  输入管理员名称和密码（并确认密码），选择你将在其中创建新的 SQL Database 服务器的区域，并选中“`Allow Azure Services to access the server` 框。

    ![新建 SQL Database 服务器][新建 SQL Database 服务器]

    创建网站后，你会看到文本“网站‘[SITENAME]’创建已成功完成”。现在，你可以启用 Git 发布。

6.  单击网站列表中显示的网站的名称以打开该网站的“快速启动”仪表板。

    ![打开网站仪表板][打开网站仪表板]

7.  在“快速启动”页的底部，单击“从源代码管理设置部署”。

    ![设置 Git 发布][设置 Git 发布]

8.  在系统询问“你的源代码在哪里?”时，选择“本地 Git 存储库”，然后单击箭头。

    ![你的源代码在哪里][你的源代码在哪里]

9.  若要启用 Git 发布，你必须提供用户名和密码。记下你创建的用户名和密码。（如果你之前已设置 Git 存储库，则将跳过此步骤。）

    ![创建发布凭据][创建发布凭据]

    设置存储库需要花费几秒钟的时间。

10. 在你的存储库已就绪后，将显示有关将应用程序文件推送到存储库的说明。记下这些说明 - 稍后你将使用它们。

    ![Git 说明][Git 说明]

## 获取 SQL Database 连接信息

若要连接到正在 Azure 网站中运行的 SQL Database 实例，你将需要连接信息。若要获取 SQL Database 连接信息，请按照以下步骤操作：

1.  从 Azure 管理门户中，单击“链接的资源”，然后单击数据库名称。

    ![链接的资源][链接的资源]

2.  单击“查看连接字符串”。

    ![连接字符串][连接字符串]

3.  从结果对话框的 **PHP** 部分，记下`SERVER`, `DATABASE` 和 `USERNAME`.

## 本地构建和测试应用程序

注册应用程序是一个简单的 PHP 应用程序，它使你能够通过提供你的姓名和电子邮件地址来注册事件。有关以前的注册者的信息将显示在表中。注册信息将存储在 SQL Database 实例中。应用程序由两个文件组成（复制/粘贴以下可用代码）：

-   **index.php**：将显示注册形式及包含注册者信息的表。
-   **createtable.php**：为应用程序创建 SQL Database 表。该文件只能被使用一次。

若要本地运行应用程序，请执行下列步骤。请注意，这些步骤假定你已在本地计算机上设置 PHP、SQL Server Express 和 Web 服务器，并且你已启用 [SQL Server 的 PDO 扩展][SQL Server 的 PDO 扩展]。

1.  创建一个 SQL Server 数据库，名为`registration`。你可以从`sqlcmd` 命令提示符使用以下命令执行此操作：

        >sqlcmd -S localhost\sqlexpress -U <local user name> -P <local password>
        1> create database registration
        2> GO   

2.  在 Web 服务器的根目录中，创建一个名为`registration` 的文件夹，并在该文件夹中创建两个文件：一个名为`createtable.php` ，另一个名为 `index.php`。

3.  在文本编辑器或 IDE 中打开`createtable.php` 文件并添加以下代码。此代码将用于创建`registration_tbl` 表（在`registration` 数据库中创建）。

        <?php
        // DB connection info
        $host = "localhost\sqlexpress";
        $user = "user name";
        $pwd = "password";
        $db = "registration";
        try{
            $conn = new PDO( "sqlsrv:Server= $host ; Database = $db ", $user, $pwd);
            $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );
            $sql = "CREATE TABLE registration_tbl(
            id INT NOT NULL IDENTITY(1,1) 
            PRIMARY KEY(id),
            name VARCHAR(30),
            email VARCHAR(30),
            date DATE)";
            $conn->query($sql);
        }
        catch(Exception $e){
            die(print_r($e));
        }
        echo "<h3>Table created.</h3>";
        ?>

    请注意，你将需要使用本地 SQL Server 用户名和密码更新`$user` 和 `$pwd` 的值。

4.  打开 Web 浏览器并浏览到 **<http://localhost/registration/createtable.php>**。这样将在该数据库中创建`registration_tbl` 表。

5.  在文本编辑器或 IDE 中打开 **index.php** 文件，并为页面添加基本 HTML 和 CSS 代码（将在后续步骤中添加 PHP 代码）。

        <html>
        <head>
        <Title>Registration Form</Title>
        <style type="text/css">
            body { background-color: #fff; border-top: solid 10px #000;
                color: #333; font-size: .85em; margin: 20; padding: 20;
                font-family: "Segoe UI", Verdana, Helvetica, Sans-Serif;
            }
            h1, h2, h3,{ color: #000; margin-bottom: 0; padding-bottom: 0; }
            h1 { font-size: 2em; }
            h2 { font-size: 1.75em; }
            h3 { font-size: 1.2em; }
            table { margin-top: 0.75em; }
            th { font-size: 1.2em; text-align: left; border: none; padding-left: 0; }
            td { padding: 0.25em 2em 0.25em 0em; border: 0 none; }
        </style>
        </head>
        <body>
        <h1>Register here!</h1>
        <p>Fill in your name and email address, then click <strong>Submit</strong> to register.</p>
        <form method="post" action="index.php" enctype="multipart/form-data" >
              Name  <input type="text" name="name" id="name"/></br>
              Email <input type="text" name="email" id="email"/></br>
              <input type="submit" name="submit" value="Submit" />
        </form>
        <?php

        ?>
        </body>
        </html>

6.  在 PHP 标记中，添加用于连接到数据库的 PHP 代码。

        // DB connection info
        $host = "localhost\sqlexpress";
        $user = "user name";
        $pwd = "password";
        $db = "registration";
        // Connect to database.
        try {
            $conn = new PDO( "sqlsrv:Server= $host ; Database = $db ", $user, $pwd);
            $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );
        }
        catch(Exception $e){
            die(var_dump($e));
        }

    再次提示：你将需要使用本地 MySQL 用户名和密码更新`$user` 和 `$pwd` 的值。

7.  在数据库连接代码后面添加用于将注册信息插入数据库的代码。

        if(!empty($_POST)) {
        try {
            $name = $_POST['name'];
            $email = $_POST['email'];
            $date = date("Y-m-d");
            // Insert data
            $sql_insert = "INSERT INTO registration_tbl (name, email, date) 
                           VALUES (?,?,?)";
            $stmt = $conn->prepare($sql_insert);
            $stmt->bindValue(1, $name);
            $stmt->bindValue(2, $email);
            $stmt->bindValue(3, $date);
            $stmt->execute();
        }
        catch(Exception $e) {
            die(var_dump($e));
        }
        echo "<h3>Your're registered!</h3>";
        }

8.  最后，在上述代码后面添加从数据库中检索数据的代码。

        $sql_select = "SELECT * FROM registration_tbl";
        $stmt = $conn->query($sql_select);
        $registrants = $stmt->fetchAll(); 
        if(count($registrants) > 0) {
            echo "<h2>People who are registered:</h2>";
            echo "<table>";
            echo "<tr><th>Name</th>";
            echo "<th>Email</th>";
            echo "<th>Date</th></tr>";
            foreach($registrants as $registrant) {
                echo "<tr><td>".$registrant['name']."</td>";
                echo "<td>".$registrant['email']."</td>";
                echo "<td>".$registrant['date']."</td></tr>";
            }
            echo "</table>";
        } else {
            echo "<h3>No one is currently registered.</h3>";
        }

你现在可以浏览到 **<http://localhost/registration/index.php>** 来测试此应用程序。

## 发布应用程序

在本地测试你的应用程序之后，你可以使用 Git 将其发布到 Azure 网站。但是，你首先需要更新应用程序中的数据库连接信息。使用之前获取的数据库连接信息（在“获取 SQL Database 连接信息”一节中），使用适当的值在`createdatabase.php` 和 `index.php` 文件中更新以下信息：

    // DB connection info
    $host = "tcp:<value of SERVER>";
    $user = "<value of USERNAME>@<server ID>";
    $pwd = "<your password>";
    $db = "<value of DATABASE>";

> [WACOM.NOTE]
> 在`$host`中，SERVER 的值的前面必须带有`tcp:`，并且`$user` 的值由 USERNAME、<'@'> 和服务器 ID 串联而成。你的服务器 ID 为 SERVER 的值的前 10 个字符。

现在，你已准备好设置 Git 发布并发布应用程序。

> [WACOM.NOTE]
> 这些步骤与上面的“创建 Azure 网站并设置 Git 发布”一节的结尾标明的步骤相同。

1.  打开 GitBash（或终端，如果 Git 在你的`PATH`中），将目录更改为应用程序的根目录，并运行以下命令：

        git init
        git add .
        git commit -m "initial commit"
        git remote add azure [URL for remote repository]
        git push azure master

    系统将提示你输入之前创建的密码。

2.  浏览到 **http://[网站名].chinacloudsites.cn/createtable.php** 以创建用于应用程序的 MySQL 表。
3.  浏览到 **http://[网站名].chinacloudsites.cn/index.php** 以开始使用应用程序。

发布应用程序之后，你可以开始对其进行更改并使用 Git 发布所做的更改。

## 发布对应用程序所做的更改

若要发布对应用程序所做的更改，请执行下列步骤：

1.  本地对应用程序进行更改。
2.  打开 GitBash（或终端，如果 Git 在你的`PATH`中），将目录更改为应用程序的根目录，并运行以下命令：

        git add .
        git commit -m "comment describing changes"
        git push azure master

    系统将提示你输入之前创建的密码。

3.  浏览到 **http://[网站名].chinacloudsites.cn/index.php** 以查看你的更改。

  [PHP]: http://www.php.net/manual/en/install.php
  [SQL Server Express]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29062
  [Microsoft Drivers for SQL Server for PHP]: http://www.microsoft.com/en-us/download/details.aspx?id=20098
  [Git]: http://git-scm.com/
  [Microsoft Web 平台安装程序]: http://www.microsoft.com/web/downloads/platform.aspx
  [Azure PHP 网站]: ./media/web-sites-php-sql-database-deploy-use-git/running_app_3.png
  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [新建 Azure 网站]: ./media/web-sites-php-sql-database-deploy-use-git/new_website.jpg
  [自定义创建新的网站]: ./media/web-sites-php-sql-database-deploy-use-git/custom_create.png
  [填写网站详细信息]: ./media/web-sites-php-sql-database-deploy-use-git/website_details_sqlazure.jpg
  [（Web 版或企业版）]: http://msdn.microsoft.com/zh-cn/library/azure/ee621788.aspx
  [填写 SQL Database 设置]: ./media/web-sites-php-sql-database-deploy-use-git/database_settings.jpg
  [新建 SQL Database 服务器]: ./media/web-sites-php-sql-database-deploy-use-git/create_server.jpg
  [打开网站仪表板]: ./media/web-sites-php-sql-database-deploy-use-git/go_to_dashboard.png
  [设置 Git 发布]: ./media/web-sites-php-sql-database-deploy-use-git/setup_git_publishing.png
  [你的源代码在哪里]: ./media/web-sites-php-sql-database-deploy-use-git/where_is_code.png
  [创建发布凭据]: ./media/web-sites-php-sql-database-deploy-use-git/git-deployment-credentials.png
  [Git 说明]: ./media/web-sites-php-sql-database-deploy-use-git/git-instructions.png
  [链接的资源]: ./media/web-sites-php-sql-database-deploy-use-git/linked_resources.jpg
  [连接字符串]: ./media/web-sites-php-sql-database-deploy-use-git/connection_string.jpg
  [SQL Server 的 PDO 扩展]: http://php.net/pdo_sqlsrv
  [http://[网站]: http://[site
