<properties 
	pageTitle="使用 FTP 在 Azure 中创建和部署 PHP-MySQL Web 应用" 
	description="本教程演示如何创建在 MySQL 中存储数据的 PHP Web 应用并使用 FTP 部署到 Azure。" 
	services="app-service\web" 
	documentationCenter="php" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="04/08/2016"
	wacn.date="05/24/2016"/>


#使用 FTP 在 Azure 中创建和部署 PHP-MySQL Web 应用

本教程演示如何创建 PHP-MySQL Web 应用以及如何使用 FTP 部署该应用。本教程假定你已在计算机上安装 [PHP][install-php]、[MySQL][install-mysql]、Web 服务器和 FTP 客户端。本教程中的说明适用于任何操作系统，包括 Windows、Mac 和 Linux。完成本指南之后，你将拥有一个在 Azure 中运行的 PHP/MySQL Web 应用。
 
你将学习以下内容：

* 如何使用 Azure 经典管理门户创建 Web 应用和 MySQL 数据库。由于 Web 应用默认已启用 PHP，因此运行 PHP 代码没有任何特殊要求。
* 如何使用 FTP 将应用程序发布到 Azure。
 
通过按照本教程中的说明进行操作，你将在 PHP 中构建简单的注册 Web 应用。将在 Web 应用中托管应用程序。以下是已完成应用程序的屏幕快照：

![Azure PHP Web 应用][running-app]


##创建 Web 应用并设置 FTP 发布

按照以下步骤创建 Web 应用和 MySQL 数据库：

1. 登录到 [Azure 经典管理门户][management-portal]。
2. 单击该经典管理门户左下的“+ 新建”图标。

	![创建新的 Azure Web 应用][new-website]

3. 单击“ Web 应用”，然后单击“自定义创建”。
	
	在“URL”中输入值，从“数据库”下拉列表中选择“无数据库”，然后在“区域”下拉列表中选择 Web 应用的数据中心。点击确定新建 Web 应用

	![填写 Web 应用详细信息][Website-details]

4. 继续单击“新建” --> “数据服务” --> “MYSQL DATABASE ON AZURE” --> “快速创建”，为你的 Web 应用创建一个 MYSQL 数据库。

	![数据库][new-mysql-db]

	创建 Web 应用后，你将看到文本“创建 Web 应用 ‘[SITENAME]’ 成功完成”。现在，您可以启用 Git 发布。

5. 单击 Web 应用列表中显示的 Web 应用的名称以打开该 Web 应用的“快速启动”仪表板。

	![打开 Web 应用仪表板][go-to-dashboard]


6. 在“快速启动”页的底部，单击“重置部署凭据”。

	![重置部署凭据][reset-deployment-credentials]

7. 若要启用 FTP 发布，必须提供用户名和密码。记下你创建的用户名和密码。

	![创建发布凭据][portal-git-username-password]

##在本地生成并测试应用

注册应用程序是一个简单的 PHP 应用程序，它使您能够通过提供您的姓名和电子邮件地址来注册事件。有关以前的注册者的信息将显示在表中。注册信息将存储在 MySQL 数据库中。该应用由两个文件组成：

* **index.php**：将显示注册形式及包含注册者信息的表。
* **createtable.php**：创建用于应用程序的 MySQL 表。该文件只能被使用一次。

若要本地构建和运行应用，请执行下列步骤。请注意，这些步骤假定你已在本地计算机上设置 PHP、MySQL 和 Web 服务器，并且已启用 [MySQL 的 PDO 扩展][pdo-mysql]。

1. 创建名为 `registration` 的 MySQL 数据库。您可以在 MySQL 命令提示符中使用此命令执行此操作：

		mysql> create database registration;

2. 在 Web 服务器的根目录中，创建一个名为 `registration` 的文件夹，并在其中创建两个文件 - 一个名为 `createtable.php`，另一个名为 `index.php`。

3. 在文本编辑器或 IDE 中打开 `createtable.php` 文件并添加以下代码。此代码将用于在 `registration` 数据库中创建 `registration_tbl` 表。

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

	> [AZURE.NOTE]
	> 需要使用本地 MySQL 用户名和密码更新 <code>$user</code> 和 <code>$pwd</code> 的值。

4. 打开 Web 浏览器并浏览到 [http://localhost/registration/createtable.php][localhost-createtable]。这将在数据库中创建 `registration_tbl` 表。

5. 在文本编辑器或 IDE 中打开 **index.php** 文件，并为页面添加基本 HTML 和 CSS 代码（将在后续步骤中添加 PHP 代码）。

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

6. 在 PHP 标记中，添加用于连接到数据库的 PHP 代码。

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

	> [AZURE.NOTE]
	> 同样，需要使用本地 MySQL 用户名和密码更新 <code>$user</code> 和 <code>$pwd</code> 的值。

7. 在数据库连接代码后面添加用于将注册信息插入数据库的代码。

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

8. 最后，在上述代码后面添加从数据库中检索数据的代码。

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

你现在可以浏览到 [http://localhost/registration/index.php][localhost-index] 来测试应用。

##获取 MySQL 和 FTP 连接信息

若要连接到正在 Azure 中运行的 MySQL 数据库，你将需要连接信息。若要获取 MySQL 连接信息，请按照以下步骤操作：

1. 点击打开你的 MYSQL 服务器，进入“仪表板”页面，在“速览”下可以查看你的服务器地址以及端口。

	![获取数据库连接信息][connection-string-info]
	
2. 在“账户”页面可以查看各个账户名，也可以重置其密码。

3. 在“数据库”页面可以查看这个服务器下的数据库。

	于是 Data Source 为 tcp:<your MYSQL server name\>.database.chinacloudapi.cn,<port\>

##发布应用

在本地测试你的应用之后，你可以使用 FTP 将其发布到 Web 应用。但是，你首先需要更新应用程序中的数据库连接信息。使用之前获取的数据库连接信息（在“获取 MySQL 和 FTP 连接信息”部分中），使用适当的值在 `createdatabase.php` 和 `index.php` 文件中更新以下信息：

	// DB connection info
	$host = "value of Data Source";
	$user = "value of User Id";
	$pwd = "value of Password";
	$db = "value of Database";

现在你可使用 FTP 发布应用。

1. 打开选择的 FTP 客户端。

2. 将上文中记下的 `publishUrl` 属性中的*主机名部分*输入到 FTP 客户端。

3. 将上面记下的 `userName` 和 `userPWD` 属性按原样输入到 FTP 客户端。

4. 建立连接。

连接后，您将能够根据需要上载和下载文件。确保将文件上载到根目录 `/site/wwwroot`。

上载 `index.php` 和 `createtable.php` 之后，浏览到 **http://[site name].chinacloudsites.cn/createtable.php** 以创建用于应用程序的 MySQL 表，然后浏览到 **http://[site name].chinacloudsites.cn/index.php** 以开始使用应用程序。

[go-to-dashboard]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/go_to_dashboard.png
[reset-deployment-credentials]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/reset-deployment-credentials.png
[portal-git-username-password]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/git-deployment-credentials.png
[install-php]: http://www.php.net/manual/en/install.php
[install-mysql]: http://dev.mysql.com/doc/refman/5.6/en/installing.html
[pdo-mysql]: http://www.php.net/manual/en/ref.pdo-mysql.php
[localhost-createtable]: http://localhost/tasklist/createtable.php
[localhost-index]: http://localhost/tasklist/index.php
[running-app]: ./media/web-sites-php-mysql-deploy-use-ftp/running_app_2.png
[new-website]: ./media/web-sites-php-mysql-deploy-use-ftp/new_website2.jpg
[custom-create]: ./media/web-sites-php-mysql-deploy-use-ftp/create_web_mysql.png
[website-details]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/website_details.jpg
[new-mysql-db]: ./media/web-sites-php-mysql-deploy-use-ftp/new_mysql_db.jpg
[go-to-webapp]: ./media/web-sites-php-mysql-deploy-use-ftp/select_webapp.png
[set-deployment-credentials]: ./media/web-sites-php-mysql-deploy-use-ftp/set_credentials.png
[portal-ftp-username-password]: ./media/web-sites-php-mysql-deploy-use-ftp/save_credentials.png
[resource-group]: ./media/web-sites-php-mysql-deploy-use-ftp/set_group.png
[new-web-app]: ./media/web-sites-php-mysql-deploy-use-ftp/create_wa.png
[select-database]: ./media/web-sites-php-mysql-deploy-use-ftp/select_database.png
[select-resourcegroup]: ./media/web-sites-php-mysql-deploy-use-ftp/select_resourcegroup.png
[select-properties]: ./media/web-sites-php-mysql-deploy-use-ftp/select_properties.png
[note-properties]: ./media/web-sites-php-mysql-deploy-use-ftp/note-properties.png

[connection-string-info]: ./media/web-sites-php-web-site-mysql-deploy-use-ftp/connection_string_info.png
[management-portal]: https://manage.windowsazure.cn
[download-publish-profile]: ./media/web-sites-php-mysql-deploy-use-ftp/download_publish_profile_3.png
 

<!---HONumber=82-->