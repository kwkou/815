<properties linkid="develop-php-website-with-mysql-and-ftp" urlDisplayName="Web w/ MySQL + FTP" pageTitle="PHP web site with MySQL and FTP - Azure tutorial" metaKeywords="" description="A tutorial that demonstrates how to create a PHP web site that stores data in MySQL and use FTP deployment to Azure." metaCanonical="" services="web-sites" documentationCenter="PHP" title="Create a PHP-MySQL Azure Web Site and Deploy Using FTP" authors="" solutions="" manager="" editor="" />

# 创建 PHP-MySQL Azure 网站并使用 FTP 进行部署

本教程演示如何创建 PHP-MySQL Azure 网站以及如何使用 FTP 部署该网站。本教程假定你已在计算机上安装 [PHP][PHP]、[MySQL][MySQL]、Web 服务器和 FTP 客户端。本教程中的说明适用于任何操作系统，包括 Windows、Mac 和 Linux。完成本指南之后，你将拥有一个在 Azure 中运行的 PHP/MySQL 网站。

你将了解到以下内容：

-   如何使用 Azure 管理门户创建 Azure 网站和 MySQL 数据库。由于默认情况下已在 Azure 网站中启用 PHP，因此运行 PHP 代码没有任何特殊要求。
-   如何使用 FTP 将应用程序发布到 Azure。

通过按照本教程中的说明进行操作，你将在 PHP 中构建简单的注册 Web 应用程序。应用程序将托管于 Azure 网站中。以下是已完成应用程序的屏幕快照：

![Azure PHP 网站][Azure PHP 网站]

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

## 创建 Azure 网站并设置 FTP 发布

按照以下步骤创建 Azure 网站和 MySQL 数据库：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  单击该门户左下角的“+ 新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  单击“网站”，然后单击“自定义创建”。

    ![自定义创建新的网站][自定义创建新的网站]

    在“URL”中输入值，从“数据库”下拉列表中选择“新建 MySQL 数据库”，然后在“区域”下拉列表中选择网站的数据中心。单击对话框底部的箭头。

    ![填写网站详细信息][填写网站详细信息]

4.  为数据库的“名称”输入一个值，在“区域”下拉列表中为数据库选择数据中心，并选中表明你同意法律条款的框。单击对话框底部的复选标记。

    ![新建 MySQL 数据库][新建 MySQL 数据库]

    创建网站后，你会看到文本“网站‘[SITENAME]’创建已成功完成”。现在，你可以启用 FTP 发布。

5.  单击网站列表中显示的网站的名称以打开该网站的“快速启动”仪表板。

    ![打开网站仪表板][打开网站仪表板]

6.  在“快速启动”页的底部，单击“重置部署凭据”。

    ![重置部署凭据][重置部署凭据]

7.  若要启用 FTP 发布，必须提供用户名和密码。记下你创建的用户名和密码。

    ![创建发布凭据][创建发布凭据]

## 本地构建和测试应用程序

注册应用程序是一个简单的 PHP 应用程序，它使你能够通过提供你的姓名和电子邮件地址来注册事件。有关以前的注册者的信息将显示在表中。注册信息将存储在 MySQL 数据库中。应用程序由两个文件组成：

-   **index.php**：将显示注册形式及包含注册者信息的表。
-   **createtable.php**：创建用于此应用程序的 MySQL 表。该文件只能被使用一次。

若要本地构建和运行应用程序，请执行下列步骤。请注意，这些步骤假定你已在本地计算机上设置 PHP、MySQL 和 Web 服务器，并且已启用 [MySQL 的 PDO 扩展][MySQL 的 PDO 扩展]。

1.  创建名为`registration` 的 MySQL 数据库。你可以在 MySQL 命令提示符中使用此命令执行此操作：

        mysql> create database registration;

2.  在 Web 服务器的根目录中，创建一个名为`registration` 的文件夹，并在该文件夹中创建两个文件：一个名为`createtable.php` ，另一个名为 `index.php`.

3.  在文本编辑器或 IDE 中打开`createtable.php` 文件并添加以下代码。此代码将用于创建`registration_tbl` 表（在`registration` 数据库中创建）。

        <?php
        // DB connection info
        $host = "localhost";
        $user = "user name";
        $pwd = "password";
        $db = "registration";
        try{
            $conn = new PDO( "mysql:host=$host;dbname=$db", $user, $pwd);
            $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );
            $sql = "CREATE TABLE registration_tbl(
                        id INT NOT NULL AUTO_INCREMENT, 
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

    > [WACOM.NOTE]
    > 你将需要使用本地 MySQL 用户名和密码更新`$user` 和 `$pwd` 的值。

4.  打开 Web 浏览器并浏览到 [][]<http://localhost/registration/createtable.php></a>。这样将在该数据库中创建`registration_tbl` 表。

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
        $host = "localhost";
        $user = "user name";
        $pwd = "password";
        $db = "registration";
        // Connect to database.
        try {
            $conn = new PDO( "mysql:host=$host;dbname=$db", $user, $pwd);
            $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );
        }
        catch(Exception $e){
            die(var_dump($e));
        }

    > [WACOM.NOTE]
    > 再次提示：你将需要使用本地 MySQL 用户名和密码更新`$user` 和 `$pwd` 的值。

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

你现在可以浏览到 [][1]<http://localhost/registration/index.php></a> 来测试此应用程序。

## 获取 MySQL 和 FTP 连接信息

若要连接到正在 Azure 网站中运行的 MySQL 数据库，你将需要连接信息。若要获取 MySQL 连接信息，请按照以下步骤操作：

1.  从网站的仪表板中，单击页面右侧的“查看连接字符串”链接：

    ![获取数据库连接信息][获取数据库连接信息]

2.  记下`Database`, `Data Source`, `User Id` 和 `Password`.

3.  从网站的仪表板中，单击页面右下角的“下载发布配置文件”链接：

    ![下载发布配置文件][下载发布配置文件]

4.  在 XML 编辑器中打开`.publishsettings` 文件。

5.  找到看起来与以下内容类似的`<publishProfile >` 元素（具有`publishMethod="FTP"` ）：

        <publishProfile publishMethod="FTP" publishUrl="ftp://cnws-prod-sha-001.ftp.chinacloudsites.chinacloudapi.cn/site/wwwroot" ftpPassiveMode="True" userName="[username]" userPWD="[password]" destinationAppUrl="http://[name].chinacloudsites.cn" 
            ...
        </publishProfile>

记下`publishUrl`、`userName` 和`userPWD` 属性。

## 发布应用程序

本地测试应用程序之后，你可以使用 FTP 将其发布到 Azure 网站。但是，你首先需要更新应用程序中的数据库连接信息。使用之前获取的数据库连接信息（在“获取 MySQL 和 FTP 连接信息”一节中），使用适当的值在`createdatabase.php` 和 `index.php` 文件中更新以下信息：

    // DB connection info
    $host = "value of Data Source";
    $user = "value of User Id";
    $pwd = "value of Password";
    $db = "value of Database";

现在你可使用 FTP 发布应用程序。

1.  打开选择的 FTP 客户端。

2.  将你在前面步骤中记下来的`publishUrl` 属性中的*主机名称部分*输入到你的 FTP 客户端中。

3.  将上文中记下的`userName` 和 `userPWD` 属性原样输入到你的 FTP 客户端中。

4.  建立连接。

连接后，你将能够根据需要上载和下载文件。确保将文件上载到根目录 `/site/wwwroot`。

在将`index.php` 和 `createtable.php`都上载后，浏览到 **http://[网站名].chinacloudsites.cn/createtable.php** 以创建用于应用程序的 MySQL 表，然后浏览到 **http://[网站名].chinacloudsites.cn/index.php** 以开始使用该应用程序。

  [PHP]: http://www.php.net/manual/en/install.php
  [MySQL]: http://dev.mysql.com/doc/refman/5.6/en/installing.html
  [Azure PHP 网站]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/running_app_2.png
  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [Azure 管理门户]: https://manage.windowsazure.cn
  [创建新的 Azure 网站]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/new_website.jpg
  [自定义创建新的网站]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/custom_create.png
  [填写网站详细信息]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/website_details.jpg
  [新建 MySQL 数据库]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/new_mysql_db.jpg
  [打开网站仪表板]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/go_to_dashboard.png
  [重置部署凭据]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/reset-deployment-credentials.png
  [创建发布凭据]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/git-deployment-credentials.png
  [MySQL 的 PDO 扩展]: http://www.php.net/manual/en/ref.pdo-mysql.php
  []: http://localhost/tasklist/createtable.php
  [1]: http://localhost/tasklist/index.php
  [获取数据库连接信息]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/connection_string_info.png
  [下载发布配置文件]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/download_publish_profile_2.png
  [http://[网站]: http://[site
