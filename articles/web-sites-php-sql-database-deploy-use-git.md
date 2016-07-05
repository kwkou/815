<properties 
	pageTitle="创建 PHP-SQL Web 应用并使用 Git 将其部署到 Azure Web 应用" 
	description="本教程演示如何创建在 Azure SQL 数据库中存储数据的 PHP Web 应用并使用 Git 部署到 Azure Web 应用。" 
	services="app-service\web, sql-database" 
	documentationCenter="php" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor="mollybos"/>

<tags
	ms.service="app-service-web"
	ms.date="04/08/2016"
	wacn.date="05/24/2016"/>

# 创建 PHP-SQL Web 应用并使用 Git 将其部署到 Azure Web 应用

本教程演示如何在 [Azure Web 应用](/documentation/services/web-sites/)中创建连接到 Azure SQL 数据库的 PHP Web 应用以及如何使用 Git 部署该 Web 应用。本教程假定你已在计算机上安装 [PHP][install-php]、[SQL Server Express][install-SQLExpress]、[Microsoft Drivers for SQL Server for PHP](http://www.microsoft.com/download/en/details.aspx?id=20098) 和 [Git][install-git]。完成本指南之后，你将拥有一个在 Azure 中运行的 PHP-SQL Web 应用。

> [AZURE.NOTE]
> 你可以使用 [Microsoft Web 平台安装程序](http://www.microsoft.com/web/downloads/platform.aspx)安装和配置 PHP、SQL Server Express 和 Microsoft Drivers for SQL Server for PHP。

你将学习以下内容：

* 如何使用 [Azure 经典管理门户](https://manage.windowsazure.cn/)创建 Azure Web 应用和 SQL 数据库。由于在 Azure Web 应用中默认启用 PHP，因此运行 PHP 代码没有任何特殊要求。
* 如何使用 Git 将应用程序发布和重新发布到 Azure。
 
通过按照本教程中的说明进行操作，你将使用 PHP 构建简单的注册 Web 应用。将在 Azure Web 应用中托管应用程序。以下是已完成应用程序的屏幕快照：

![Azure PHP Web 应用](./media/web-sites-php-sql-database-deploy-use-git/running_app_3.png)

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

##创建 Azure Web 应用并设置 Git 发布

按照以下步骤创建 Azure Web 应用和 SQL 数据库：

1. 登录到 [Azure 经典管理门户][management-portal]。
2. 单击该经典管理门户左下部的“新建”图标。

	![创建新的 Azure Web 应用][new- Website]

3. 单击“ Web 应用”，然后单击“自定义创建”。

	![自定义创建新的 Web 应用][custom-create]

	在“URL”中输入值，从“数据库”下拉列表中选择“新建 SQL 数据库”，然后选择“从源控制发布”。单击对话框底部的箭头。

	![填写 Web 应用详细信息][Website-details-sqlazure]

4. 输入数据库的“名称”值，选择“新建 SQL 数据库服务器”，提供登录凭据，然后选择一个区域。单击对话框底部的箭头。

	![填写 SQL 数据库设置][database-settings]

5. 为源代码选择“本地 Git 存储库”。

	![你的源代码在哪里][where-is-code]

	如果之前未设置 Git 存储库，则必须提供用户名和密码。

6. 创建了 Web 应用之后，打开 Web 应用的仪表板，然后选择“查看部署”。

	![ Web 应用仪表板][go-to-dashboard]

9. 您将看到有关将应用程序文件推送到存储库的说明。记下这些说明 — 稍后您将需要它们。

	![Git 说明][git-instructions]

##获取 SQL 数据库连接信息

若要连接到链接到 Web 应用的 SQL 数据库实例，你将需要在创建数据库时指定的连接信息。若要获取 SQL 数据库连接信息，请按照以下步骤操作：

1. 从 Azure 经典管理门户中，单击“链接的资源”，然后单击数据库名称。

	![链接的资源][linked-resources]

2. 单击“查看连接字符串”。

	![连接字符串][connection-string]
	
3. 从结果对话框的“PHP”部分，记下 `Server`、`SQL Database` 和 `User Name` 的值。稍后将 PHP Web 应用发布到 Azure Web 应用时，将使用这些值。

##本地构建和测试应用程序

注册应用程序是一个简单的 PHP 应用程序，它使您能够通过提供您的姓名和电子邮件地址来注册事件。有关以前的注册者的信息将显示在表中。注册信息将存储在 SQL 数据库实例中。应用程序由两个文件组成（复制/粘贴以下可用代码）：

* **index.php**：将显示注册形式及包含注册者信息的表。
* **createtable.php**：为应用程序创建 SQL 数据库表。该文件只能被使用一次。

若要本地运行应用程序，请执行下列步骤。请注意，这些步骤假定你已在本地计算机上设置 PHP 和 SQL Server Express，并且你已启用 [SQL Server 的 PDO 扩展][pdo-sqlsrv]。

1. 创建一个名为 `registration` 的 SQL Server 数据库。你可以通过 `sqlcmd` 命令提示符使用以下命令执行此操作：

		>sqlcmd -S localhost\sqlexpress -U <local user name> -P <local password>
		1> create database registration
		2> GO	


2. 在应用程序根目录中，创建两个文件 - 一个名为 `createtable.php`，另一个名为 `index.php`。

3. 在文本编辑器或 IDE 中打开 `createtable.php` 文件并添加以下代码。此代码将用于在 `registration` 数据库中创建 `registration_tbl` 表。

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

	请注意，你需要使用本地 SQL Server 用户名和密码更新 <code>$user</code> 和 <code>$pwd</code> 的值。

4. 在应用程序根目录的终端中，键入以下命令：

		php -S localhost:8000

4. 打开 Web 浏览器并浏览到 **http://localhost:8000/createtable.php**。这将在数据库中创建 `registration_tbl` 表。

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

    同样，需要使用本地 MySQL 用户名和密码更新 <code>$user</code> 和 <code>$pwd</code> 的值。

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

现在，你可以浏览到 **http://localhost:8000/index.php** 以测试应用程序。

##发布应用程序

在本地测试你的应用程序之后，你可以使用 Git 将其发布到 Azure Web 应用。但是，你首先需要更新应用程序中的数据库连接信息。使用之前获取的数据库连接信息（在“获取 SQL 数据库连接信息”部分中），使用适当的值在 `createdatabase.php` 和 `index.php` 文件中更新以下信息：

	// DB connection info
	$host = "tcp:<value of Server>";
	$user = "<value of User Name>";
	$pwd = "<your password>";
	$db = "<value of SQL Database>";

> [AZURE.NOTE]
> 在 <code>$host</code> 中，Server 的值的前面必须带有 <code>tcp:</code>。


现在，您已准备好设置 Git 发布并发布应用程序。

> [AZURE.NOTE]
> 这些步骤与在**创建 Azure Web 应用并设置 Git 发布**部分的结尾标明的步骤相同：


1. 打开 GitBash（或终端，如果 Git 在 `PATH` 中），将目录更改为应用程序的根目录（**registration** 目录），并运行以下命令：

		git init
		git add .
		git commit -m "initial commit"
		git remote add azure [URL for remote repository]
		git push azure master

	系统将提示你输入之前创建的密码。

2. 浏览到 **http://[web site name].chinacloudsites.cn/createtable.php** 以创建应用程序的 SQL 数据库表。
3. 浏览到 **http://[web site name].chinacloudsites.cn/index.php** 以开始使用应用程序。

发布应用程序之后，你可以开始对其进行更改并使用 Git 发布所做的更改。

##发布对应用程序所做的更改

若要发布对应用程序所做的更改，请执行下列步骤：

1. 本地对应用程序进行更改。
2. 打开 GitBash（或终端，如果 Git 在 `PATH` 中），将目录更改为应用程序的根目录，并运行以下命令：

		git add .
		git commit -m "comment describing changes"
		git push azure master

	系统将提示你输入之前创建的密码。

3. 浏览到 **http://[web app name].chinacloudsites.cn/index.php** 以查看所做的更改。



[running-app]: ./media/web-sites-php-sql-database-deploy-use-git/running_app_3.png
[new- Website]: ./media/web-sites-php-sql-database-deploy-use-git/new_Website.jpg
[custom-create]: ./media/web-sites-php-sql-database-deploy-use-git/custom_create.png
[website-details-sqlazure]: ./media/web-sites-php-sql-database-deploy-use-git/createphpgitsite.png
[database-settings]: ./media/web-sites-php-sql-database-deploy-use-git/setupdb.png
[create-server]: ./media/web-sites-php-sql-database-deploy-use-git/create_server.jpg
[go-to-dashboard]: ./media/web-sites-php-sql-database-deploy-use-git/viewdeploy.png
[setup-git-publishing]: ./media/web-sites-php-sql-database-deploy-use-git/setup_git_publishing.png
[credentials]: ./media/web-sites-php-sql-database-deploy-use-git/git-deployment-credentials.png
[git-instructions]: ./media/web-sites-php-sql-database-deploy-use-git/gitsettings.png
[linked-resources]: ./media/web-sites-php-sql-database-deploy-use-git/linked_resources.jpg
[connection-string]: ./media/web-sites-php-sql-database-deploy-use-git/connection_string.jpg
[management-portal]: https://manage.windowsazure.cn/
[sql-database-editions]: http://msdn.microsoft.com/zh-cn/library/azure/ee621788.aspx
[where-is-code]: ./media/web-sites-php-sql-database-deploy-use-git/setupgit.png

[install-php]: http://www.php.net/manual/en/install.php
[install-SQLExpress]: http://www.microsoft.com/download/details.aspx?id=29062
[install-Drivers]: http://www.microsoft.com/download/details.aspx?id=20098
[install-git]: http://git-scm.com/
[pdo-sqlsrv]: http://php.net/pdo_sqlsrv
 

<!---HONumber=Mooncake_0118_2016-->