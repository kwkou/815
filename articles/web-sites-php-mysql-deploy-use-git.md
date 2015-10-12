<properties
	pageTitle="使用 Git 在 Azure 网站中创建和部署 PHP-MySQL Web 应用"
	description="本教程演示如何创建在 MySQL 中存储数据的 PHP Web 应用并使用 Git 部署到 Azure。"
	services="app-service\web"
	documentationCenter="php"
	authors="tfitzmac"
	manager="wpickett"
	editor="mollybos"
	tags="mysql"/>

<tags
	ms.service="app-service-web"
	ms.date="08/03/2015"
	wacn.date="10/03/2015"/>

#创建 PHP-MySQL Azure 网站并使用 Git 进行部署

本教程演示如何创建 PHP-MySQL Azure 网站以及如何使用 Git 部署该网站。你将使用计算机上已安装的 [PHP][install-php]、MySQL 命令行工具（[MySQL][install-mysql] 的一部分）、Web 服务器和 [Git][install-git]。本教程中的说明适用于任何操作系统，包括 Windows、Mac 和 Linux。完成本指南之后，您将拥有一个在 Azure 中运行的 PHP/MySQL 网站。
 
你将学习以下内容：

* 如何使用 Azure 管理门户创建 Azure 网站和 MySQL 数据库。由于在 Azure 网站中默认启用 PHP，因此运行 PHP 代码没有任何特殊要求。
* 如何使用 Git 将应用程序发布和重新发布到 Azure。
 
通过按照本教程中的说明进行操作，您将在 PHP 中构建简单的注册 Web 应用程序。将在 Azure 网站中托管应用程序。以下是已完成应用程序的屏幕快照：

![Azure PHP 网站][running-app]

> [AZURE.NOTE]若要完成本教程，你需要一个启用了 Azure 网站功能的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/1rmb-trial/?WT.mc_id=A74E0F923" target="_blank">Azure 免费试用</a>。


##设置开发环境

本教程假定你的计算机上已经安装了 [PHP][install-php]、MySQL 命令行工具（[MySQL][install-mysql] 的一部分）、Web 服务器和 [Git][install-git]。

> [AZURE.NOTE]如果你在 Windows 上执行本教程，则可通过安装 <a href="http://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/azurephpsdk.appids">Azure SDK for PHP</a> 为 PHP 设置计算机并自动配置 IIS（Windows 中的内置 Web 服务器）。

##<a id="create-web-site-and-set-up-git"></a>创建 Azure 网站并设置 Git 发布

按照以下步骤创建 Azure 网站和 MySQL 数据库：

1. 登录到 [Azure 管理门户][management-portal]。
2. 单击门户左下角的“新建”图标。

	![创建新的 Azure 网站][new-Website]

3. 单击“网站”，然后单击“自定义创建”。

	![自定义创建新的网站][custom-create]
	
	在“URL”中输入值，从“数据库”下拉列表中选择“新建 MySQL 数据库”，然后在“区域”下拉列表中选择网站的数据中心。单击对话框底部的箭头。

	![填写网站详细信息][Website-details]

4. 为数据库的“名称”输入一个值，在“区域”下拉列表中为数据库选择数据中心，并选中表明你同意法律条款的框。单击对话框底部的复选标记。

	![新建新的 MySQL 数据库][new-mysql-db]

	创建网站后，你将看到文本“创建网站 ‘[SITENAME]’ 成功完成”。现在，您可以启用 Git 发布。

6. 单击网站列表中显示的网站的名称以打开该网站的“快速启动”仪表板。

	![打开网站仪表板][go-to-dashboard]


7. 在“快速启动”页的底部，单击“设置 Git 发布”。

	![设置 Git 发布][setup-git-publishing]

8. 若要启用 Git 发布，必须提供用户名和密码。记下你创建的用户名和密码。（如果之前已设置 Git 存储库，则将跳过此步骤。）

	![创建发布凭据][credentials]

	设置存储库需要花费几秒钟的时间。

9. 在您的存储库已就绪后，将显示有关将应用程序文件推送到存储库的说明。记下这些说明 - 稍后您将使用它们。

	![Git 说明][git-instructions]

##获取远程 MySQL 连接信息

若要连接到正在 Azure 网站中运行的 MySQL 数据库，你将需要连接信息。若要获取 MySQL 连接信息，请按照以下步骤操作：

1. 从网站的仪表板中，单击页面右侧的“查看连接字符串”链接：

	![获取数据库连接信息][connection-string-info]
	
2. 记下 `Database`、`Data Source`、`User Id` 和 `Password` 的值。

##在本地生成并测试应用

现在你已创建 Azure 网站，可以本地开发应用程序，然后在测试后部署该应用程序。

注册应用程序是一个简单的 PHP 应用程序，它使您能够通过提供您的姓名和电子邮件地址来注册事件。有关以前的注册者的信息将显示在表中。注册信息将存储在 MySQL 数据库中。应用程序由一个文件组成（复制/粘贴以下可用代码）：

* **index.php**：将显示注册形式及包含注册者信息的表。

若要本地构建和运行应用程序，请执行下列步骤。请注意，这些步骤假定你已在本地计算机上设置 PHP、MySQL 命令行工具（MySQL 的一部分）和 Web 服务器，并且你已启用 [MySQL 的 PDO 扩展][pdo-mysql]。

1. 使用之前检索的 `Data Source`、`User Id`、`Password` 和 `Database` 的值，连接到远程 MySQL 服务器：

		mysql -h{Data Source] -u[User Id] -p[Password] -D[Database]

2. 此时将出现 MySQL 命令提示符：

		mysql>

3. 粘贴以下 `CREATE TABLE` 命令以在你的数据库中创建 `registration_tbl` 表：

		mysql> CREATE TABLE registration_tbl(id INT NOT NULL AUTO_INCREMENT, PRIMARY KEY(id), name VARCHAR(30), email VARCHAR(30), date DATE);

4. 在 Web 服务器的根目录中，创建一个名为 `registration` 的文件夹并在该文件夹中创建一个名为 `index.php` 的文件。

5. 在文本编辑器或 IDE 中打开 **index.php** 文件，添加以下代码，并完成用 `//TODO:` 注释标记的必需更改。


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

现在，你可以浏览到 ****http://localhost/registration/index.php** 以测试应用程序。


##发布应用

在本地测试你的应用之后，可以使用 Git 将其发布到 Web Apps。你将初始化本地 Git 存储库并发布该应用程序。

> [AZURE.NOTE]这些步骤与在门户中的“创建 Web 应用并设置 Git 发布”一节的结尾显示的步骤相同。

1. （可选）如果你忘记或误放了 Git 远程存储库 URL，请导航到门户上的 Web 应用属性。
	

1. 打开 GitBash（或终端，如果 Git 在 `PATH` 中），将目录更改为应用程序的根目录，并运行以下命令：

		git init
		git add .
		git commit -m "initial commit"
		git remote add azure [URL for remote repository]
		git push azure master

	系统将提示你输入之前创建的密码。

	![通过 Git 初始推送到 Azure][git-initial-push]

2. 浏览到 **http://[site name].chinacloudsites.cn/index.php** 以开始使用应用程序（此信息将存储在你的帐户仪表板上）：

	![Azure PHP 网站][running-app]

发布应用之后，你可以开始对其进行更改并使用 Git 发布所做的更改。

##发布对应用所做的更改

若要发布对应用所做的更改，请执行下列步骤：

1. 本地对应用进行更改。
2. 打开 GitBash（或终端，如果 Git 在 `PATH` 中），将目录更改为应用的根目录，并运行以下命令：

		git add .
		git commit -m "comment describing changes"
		git push azure master

	系统将提示你输入之前创建的密码。

	![通过 Git 将网站更改推送到 Azure][git-change-push]

3. 浏览到 **http://[site name].chinacloudsites.cn/index.php** 以查看你的应用程序及可能做出的任何更改：

	![Azure PHP 网站][running-app]

4. 你也可以在 Azure 管理门户中的“部署”选项卡上查看新的部署：

	![网站部署列表][deployments-list]

[install-php]: http://www.php.net/manual/en/install.php
[install-SQLExpress]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29062
[install-Drivers]: http://www.microsoft.com/zh-cn/download/details.aspx?id=20098
[install-git]: http://git-scm.com/

[pdo-mysql]: http://www.php.net/manual/en/ref.pdo-mysql.php
[running-app]: ./media/web-sites-php-mysql-deploy-use-git/running_app_2.png
[new- Website]: ./media/web-sites-php-mysql-deploy-use-git/new_Website.jpg
[custom-create]: ./media/web-sites-php-mysql-deploy-use-git/custom_create.png
[ Website-details]: ./media/web-sites-php-mysql-deploy-use-git/Website_details.jpg
[new-mysql-db]: ./media/web-sites-php-mysql-deploy-use-git/new_mysql_db.jpg
[go-to-dashboard]: ./media/web-sites-php-mysql-deploy-use-git/go_to_dashboard.png
[setup-git-publishing]: ./media/web-sites-php-mysql-deploy-use-git/setup_git_publishing.png
[credentials]: ./media/web-sites-php-mysql-deploy-use-git/git-deployment-credentials.png


[git-instructions]: ./media/web-sites-php-mysql-deploy-use-git/git-instructions.png
[git-change-push]: ./media/web-sites-php-mysql-deploy-use-git/php-git-change-push.png
[git-initial-push]: ./media/web-sites-php-mysql-deploy-use-git/php-git-initial-push.png
[deployments-list]: ./media/web-sites-php-mysql-deploy-use-git/php-deployments-list.png
[connection-string-info]: ./media/web-sites-php-mysql-deploy-use-git/connection_string_info.png
[management-portal]: https://manage.windowsazure.cn
[sql-database-editions]: http://msdn.microsoft.com/zh-cn/library/azure/ee621788.aspx

<!---HONumber=71-->