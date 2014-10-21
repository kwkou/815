<properties linkid="develop-php-website-with-mysql-and-git" urlDisplayName="Web w/ MySQL + Git" pageTitle="PHP web site with MySQL and Git - Azure tutorial" metaKeywords="" description="A tutorial that demonstrates how to create a PHP web site that stores data in MySQL and use Git deployment to Azure." metaCanonical="" services="web-sites" documentationCenter="PHP" title="Create a PHP-MySQL Azure web site and deploy using Git" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

# 创建 PHP-MySQL Azure 网站并使用 Git 进行部署

本教程演示如何创建 PHP-MySQL Azure 网站以及如何使用 Git 部署该网站。你将使用计算机上已安装的 [PHP][PHP]、MySQL 命令行工具（[MySQL][install-mysql] 的一部分）、Web 服务器和 [Git][Git]。本教程中的说明适用于任何操作系统，包括 Windows、Mac 和 Linux。完成本指南之后，你将拥有一个在 Azure 中运行的 PHP/MySQL 网站。

你将了解到以下内容：

-   如何使用 Azure 管理门户创建 Azure 网站和 MySQL 数据库。由于默认情况下已在 Azure 网站中启用 PHP，因此运行 PHP 代码没有任何特殊要求。
-   如何使用 Git 将应用程序发布和重新发布到 Azure。

通过按照本教程中的说明进行操作，你将在 PHP 中构建简单的注册 Web 应用程序。应用程序将托管于 Azure 网站中。以下是已完成应用程序的屏幕快照：

![Azure PHP 网站][Azure PHP 网站]

<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，你需要一个启用了 Azure 网站功能的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/">Azure 免费试用</a>。</p> </div>

## 设置开发环境

本教程假定你的计算机上已经安装了 [PHP][PHP]、MySQL 命令行工具（[MySQL][install-mysql] 的一部分）、Web 服务器和 [Git][Git]。

> [WACOM.NOTE]
> 如果你在 Windows 上执行本教程，则可通过安装 [Azure SDK for PHP][Azure SDK for PHP] 为 PHP 设置计算机并自动配置 IIS（Windows 中的内置 Web 服务器）。

## <span id="create-web-site-and-set-up-git"></span></a>创建 Azure 网站并设置 Git 发布

按照以下步骤创建 Azure 网站和 MySQL 数据库：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  单击该门户左下角的“新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  单击“网站”，然后单击“自定义创建”。

    ![自定义创建新的网站][自定义创建新的网站]

    在“URL”中输入值，从“数据库”下拉列表中选择“新建 MySQL 数据库”，然后在“区域”下拉列表中选择网站的数据中心。单击对话框底部的箭头。

    ![填写网站详细信息][填写网站详细信息]

4.  为数据库的“名称”输入一个值，在“区域”下拉列表中为数据库选择数据中心，并选中表明你同意法律条款的框。单击对话框底部的复选标记。

    ![新建 MySQL 数据库][新建 MySQL 数据库]

    创建网站后，你会看到文本“网站‘[SITENAME]’创建已成功完成”。现在，你可以启用 Git 发布。

5.  单击网站列表中显示的网站的名称以打开该网站的“快速启动”仪表板。

    ![打开网站仪表板][打开网站仪表板]

6.  在“快速启动”页的底部，单击“设置 Git 发布”。

    ![设置 Git 发布][设置 Git 发布]

7.  若要启用 Git 发布，你必须提供用户名和密码。记下你创建的用户名和密码。（如果你之前已设置 Git 存储库，则将跳过此步骤。）

    ![创建发布凭据][创建发布凭据]

    设置存储库需要花费几秒钟的时间。

8.  在你的存储库已就绪后，将显示有关将应用程序文件推送到存储库的说明。记下这些说明 - 稍后你将使用它们。

    ![Git 说明][Git 说明]

## 获取远程 MySQL 连接信息

若要连接到正在 Azure 网站中运行的 MySQL 数据库，你将需要连接信息。若要获取 MySQL 连接信息，请按照以下步骤操作：

1.  从网站的仪表板中，单击页面右侧的“查看连接字符串”链接：

    ![获取数据库连接信息][获取数据库连接信息]

2.  记下`Database`、`Data Source`、`User Id` 和`Password` 的值。

## 本地构建和测试应用程序

现在你已创建 Azure 网站，可以本地开发应用程序，然后在测试后部署该应用程序。

注册应用程序是一个简单的 PHP 应用程序，它使你能够通过提供你的姓名和电子邮件地址来注册事件。有关以前的注册者的信息将显示在表中。注册信息将存储在 MySQL 数据库中。应用程序由一个文件组成（复制/粘贴以下可用代码）：

-   **index.php**：将显示注册形式及包含注册者信息的表。

若要本地构建和运行应用程序，请执行下列步骤。请注意，这些步骤假定你已在本地计算机上设置 PHP、MySQL 命令行工具（MySQL 的一部分）和 Web 服务器，并且你已启用 [MySQL 的 PDO 扩展][MySQL 的 PDO 扩展]。

1.  使用你之前检索到的`Data Source`, `User Id`, `Password` 和`Database` 的值连接到远程 MySQL 服务器：

        mysql -h{Data Source] -u[User Id] -p[Password] -D[Database]

2.  MySQL 命令提示符将出现：

        mysql>

3.  粘贴在以下`CREATE TABLE` 命令中可在数据库中创建`registration_tbl` 表：

        mysql> CREATE TABLE registration_tbl(id INT NOT NULL AUTO_INCREMENT, PRIMARY KEY(id), name VARCHAR(30), email VARCHAR(30), date DATE);

4.  在 Web 服务器的根目录中，创建一个名为`registration` 的文件夹并在其中创建一个名为`index.php`的文件。

5.  在文本编辑器或 IDE 中打开 **index.php** 文件，添加以下代码，并完成用`//TODO:` 注释标记的必需更改。

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
            // DB connection info
            //TODO: Update the values for $host, $user, $pwd, and $db
            //using the values you retrieved earlier from the portal.
            $host = "value of Data Source";
            $user = "value of User Id";
            $pwd = "value of Password";
            $db = "value of Database";
            // Connect to database.
            try {
                $conn = new PDO( "mysql:host=$host;dbname=$db", $user, $pwd);
                $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );
            }
            catch(Exception $e){
                die(var_dump($e));
            }
            // Insert registration info
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
            // Retrieve data
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
        ?>
        </body>
        </html>

你现在可以浏览到 **<http://localhost/registration/index.php>** 来测试此应用程序。

## 发布应用程序

在本地测试你的应用程序之后，你可以使用 Git 将其发布到 Azure 网站。你将初始化本地 Git 存储库并发布该应用程序。

> [WACOM.NOTE]
> 这些步骤与在门户中的“创建 Azure 网站并设置 Git 发布”一节的结尾显示的步骤相同。

1.  （可选）如果你忘记或误放了 Git 远程存储库 URL，请导航到门户上的“部署”选项卡。

    ![获取 Git URL][Git 说明]

2.  打开 GitBash（或终端，如果 Git 在你的`PATH`中），将目录更改为应用程序的根目录，并运行以下命令：

        git init
        git add .
        git commit -m "initial commit"
        git remote add azure [URL for remote repository]
        git push azure master

    系统将提示你输入之前创建的密码。

    ![通过 Git 初始推送到 Azure][通过 Git 初始推送到 Azure]

3.  浏览到 **http://[网站名].chinacloudsites.cn/index.php** 以开始使用应用程序（该信息将存储在帐户仪表板上）：

    ![Azure PHP 网站][Azure PHP 网站]

发布应用程序之后，你可以开始对其进行更改并使用 Git 发布所做的更改。

## 发布对应用程序所做的更改

若要发布对应用程序所做的更改，请执行下列步骤：

1.  本地对应用程序进行更改。
2.  打开 GitBash（或终端，如果 Git 在你的`PATH`中），将目录更改为应用程序的根目录，并运行以下命令：

        git add .
        git commit -m "comment describing changes"
        git push azure master

    系统将提示你输入之前创建的密码。

    ![通过 Git 将网站更改推送到 Azure][通过 Git 将网站更改推送到 Azure]

3.  浏览到 **http://[网站名].chinacloudsites.cn/index.php** 可查看你的应用程序和可能已做的任何更改：

    ![Azure PHP 网站][Azure PHP 网站]

4.  你也可以在 Azure 管理门户中的“部署”选项卡上查看新的部署：

    ![网站部署列表][网站部署列表]

  [PHP]: http://www.php.net/manual/en/install.php
  [Git]: http://git-scm.com/
  [Azure PHP 网站]: ./media/web-sites-php-mysql-deploy-use-git/running_app_2.png
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [Azure SDK for PHP]: http://www.windowsazure.cn/zh-cn/downloads/?sdk=php
  [Azure 管理门户]: https://manage.windowsazure.cn
  [创建新的 Azure 网站]: ./media/web-sites-php-mysql-deploy-use-git/new_website.jpg
  [自定义创建新的网站]: ./media/web-sites-php-mysql-deploy-use-git/custom_create.png
  [填写网站详细信息]: ./media/web-sites-php-mysql-deploy-use-git/website_details.jpg
  [新建 MySQL 数据库]: ./media/web-sites-php-mysql-deploy-use-git/new_mysql_db.jpg
  [打开网站仪表板]: ./media/web-sites-php-mysql-deploy-use-git/go_to_dashboard.png
  [设置 Git 发布]: ./media/web-sites-php-mysql-deploy-use-git/setup_git_publishing.png
  [创建发布凭据]: ./media/web-sites-php-mysql-deploy-use-git/git-deployment-credentials.png
  [Git 说明]: ./media/web-sites-php-mysql-deploy-use-git/git-instructions.png
  [获取数据库连接信息]: ./media/web-sites-php-mysql-deploy-use-git/connection_string_info.png
  [MySQL 的 PDO 扩展]: http://www.php.net/manual/en/ref.pdo-mysql.php
  [通过 Git 初始推送到 Azure]: ./media/web-sites-php-mysql-deploy-use-git/php-git-initial-push.png
  [http://[网站]: http://[site
  [通过 Git 将网站更改推送到 Azure]: ./media/web-sites-php-mysql-deploy-use-git/php-git-change-push.png
  [网站部署列表]: ./media/web-sites-php-mysql-deploy-use-git/php-deployments-list.png
