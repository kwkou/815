<properties
	pageTitle="使用 Git 在 Azure App Service 中创建和部署 PHP-MySQL Web 应用"
	description="本教程演示如何创建在 MySQL 中存储数据的 PHP Web 应用并使用 Git 部署到 Azure。"
	services="app-service\web"
	documentationCenter="php"
	authors="rmcmurray"
	manager="wpickett"
	editor=""
	tags="mysql"/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="PHP"
	ms.topic="article"
	ms.date="08/11/2016"
	wacn.date="12/12/2016"
	ms.author="robmcm"/>

# 使用 Git 在 Azure App Service 中创建和部署 PHP-MySQL Web 应用

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本教程演示如何创建 PHP-MySQL Web 应用以及如何使用 Git 将该应用部署到[应用服务](/documentation/articles/app-service-changes-existing-services/)。需要使用计算机上已安装的 [PHP][install-php]、MySQL 命令行工具（[MySQL][install-mysql] 的一部分）和 [Git][install-git]。本教程中的说明适用于任何操作系统，包括 Windows、Mac 和 Linux。完成本指南之后，你将拥有一个在 Azure 中运行的 PHP/MySQL Web 应用。

你将学习以下内容：

* 如何使用 [Azure 门户预览][management-portal]创建 Web 应用和 MySQL 数据库。由于[应用服务 Web 应用](/documentation/articles/app-service-changes-existing-services/)已默认启用 PHP，因此运行 PHP 代码没有任何特殊要求。
* 如何使用 Git 将应用程序发布和重新发布到 Azure。
* 如何启用该编辑器扩展才能在每个 `git push` 自动执行编辑器任务。

通过按照本教程中的说明进行操作，你将在 PHP 中构建简单的注册 Web 应用。将在 Web 应用中托管应用程序。以下是已完成应用程序的屏幕快照：

![Azure PHP 网站][running-app]

## 设置开发环境

本教程假定计算机上已安装 [PHP][install-php]、MySQL 命令行工具（[MySQL][install-mysql] 的一部分）和 [Git][install-git]。


##<a id="create-web-site-and-set-up-git"></a>创建 Web 应用并设置 Git 发布

按照以下步骤创建 Web 应用和 MySQL 数据库：

1. 登录到 [Azure 门户预览][management-portal]。
2. 单击“新建”图标。

3. 单击“应用商店”旁边的“查看全部”。

4. 单击“Web + 移动”，然后单击“Web 应用”。然后单击**创建**。

4. 为资源组输入有效的名称。

5. 为新的 Web 应用输入值。

6. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)，创建一个 MYSQL，然后再“仪表板”为你的 Web 应用找到 MYSQL 的连接字符串。

7. 创建 Web 应用程序后，用户将看到新的 Web 应用边栏选项卡。

9. 若要启用 Git 发布，必须提供用户名和密码。记下你创建的用户名和密码。（如果之前已设置 Git 存储库，则会跳过此步骤。）

	![创建发布凭据][credentials]

11. 使用以下 PowerShell 命令行设置“本地 Git 存储库”。

		$a = Get-AzureRmResource -ResourceId /subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Web/sites/<web app name>/Config/web -ApiVersion 2015-08-01

		$a.Properties.scmType = "LocalGit"

		Set-AzureRmResource -PropertyObject $a.Properties -ResourceId /subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Web/sites/<web app name>/Config/web -ApiVersion 2015-08-01


## 获取远程 MySQL 连接信息

若要连接到正在 Web Apps 中运行的 MySQL 数据库，你将需要连接信息。若要获取 MySQL 连接信息，请按照以下步骤操作：

1. 在 Azure 经典管理门户中，单击“AZURE 上的 MYSQL 数据库”，并打开 MYSQL 数据库服务器。在“仪表板”页上的“速览”下，可以获取主机和端口。

2. 在“帐户”页中，可以获取所有帐户名称，并可以重置密码。

3. 在“数据库”页中，可以获取此 MYSQL 数据库服务器下的所有数据库。

	数据源将为 `tcp:<your MYSQL server name>.database.chinacloudapi.cn,<port>`

## 在本地生成并测试应用

现在已创建 Web 应用，可以在本地开发应用程序，然后在测试后部署该应用程序。

注册应用程序是一个简单的 PHP 应用程序，在该应用程序中提供姓名和电子邮件地址即可注册事件。以前的注册者的信息将显示在表中。注册信息存储在 MySQL 数据库中。应用程序由一个文件组成（复制/粘贴以下可用代码）：

* **index.php**：显示注册形式及包含注册者信息的表。

若要本地构建和运行应用程序，请执行下列步骤。请注意，这些步骤假定已在本地计算机上设置 PHP 和 MySQL 命令行工具（MySQL 的一部分），并且已启用 [MySQL 的 PDO 扩展][pdo-mysql]。

1. 使用之前检索的 `Data Source`、`User Id`、`Password` 和 `Database` 的值，连接到远程 MySQL 服务器：

		mysql -h{Data Source] -u[User Id] -p[Password] -D[Database]

2. 此时将出现 MySQL 命令提示符：

		mysql>

3. 粘贴以下 `CREATE TABLE` 命令以在你的数据库中创建 `registration_tbl` 表：

		CREATE TABLE registration_tbl(id INT NOT NULL AUTO_INCREMENT, PRIMARY KEY(id), name VARCHAR(30), email VARCHAR(30), date DATE);

4. 在本地应用程序文件夹的根目录中创建 **index.php** 文件。

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
			//using the values you retrieved earlier from the Azure Portal.
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

4.  在终端中转到应用程序文件夹，并键入以下命令：

		php -S localhost:8000

现在，你可以浏览到 **http://localhost:8000/** 以测试应用程序。


## 发布应用

在本地测试你的应用之后，可以使用 Git 将其发布到 Web Apps。你将初始化本地 Git 存储库并发布该应用程序。

> [AZURE.NOTE]
这些步骤与 Azure 门户预览中的“创建 Web 应用并设置 Git 发布”一节的结尾显示的步骤相同。

1. （可选）如果你忘记或误放了 Git 远程存储库 URL，请导航到 Azure 门户预览上的 Web 应用属性。

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

## 发布对应用所做的更改

若要发布对应用所做的更改，请执行下列步骤：

1. 本地对应用进行更改。
2. 打开 GitBash（或终端，如果 Git 在 `PATH` 中），将目录更改为应用的根目录，并运行以下命令：

		git add .
		git commit -m "comment describing changes"
		git push azure master

	系统将提示你输入之前创建的密码。

	![通过 Git 将网站更改推送到 Azure][git-change-push]

3. 浏览到 **http://[site name].chinacloudsites.cn/index.php** 以查看应用及可能做出的任何更改：

	![Azure PHP 网站][running-app]
	
## <a name="composer"></a> 使用编辑器扩展启用编辑器自动化

默认情况下，如果 PHP 项目中有 composer.json，则应用服务中的 git 部署过程与其不相关。`git push` 期间可以通过启用编辑器扩展启用 composer.json 处理。

1. 在 [Azure 门户预览][management-portal]中的 PHP Web 应用的边栏选项卡中，请单击“工具”>“扩展”。

    ![设置编辑器扩展插件][composer-extension-settings]

2. 单击“添加”，然后单击“编辑器”。

    ![添加编辑器扩展插件][composer-extension-add]
    
3. 单击“确定”接受法律条款。再次单击“确定”以添加该扩展。

    **已安装扩展**边栏选项卡将不会显示编辑器扩展。

	![查看编辑器扩展插件][composer-extension-view]
    
4. 现在，如上一节所示，执行 `git add`、`git commit` 和 `git push`。现在将看到编辑器正在安装在 composer.json 中定义的依赖项。

    ![编辑器扩展插件成功][composer-extension-success]

## 后续步骤

有关详细信息，请参阅 [PHP 开发中心](/develop/php/)。

<!-- URL List -->

[install-php]: http://www.php.net/manual/en/install.php
[install-SQLExpress]: http://www.microsoft.com/download/details.aspx?id=29062
[install-Drivers]: http://www.microsoft.com/download/details.aspx?id=20098
[install-git]: http://git-scm.com/
[install-mysql]: http://dev.mysql.com/downloads/mysql/
[pdo-mysql]: http://www.php.net/manual/en/ref.pdo-mysql.php
[management-portal]: https://portal.azure.cn
[sql-database-editions]: http://msdn.microsoft.com/zh-cn/library/azure/ee621788.aspx

<!-- IMG List -->

[running-app]: ./media/web-sites-php-mysql-deploy-use-git/running_app_2.png
[new-website]: ./media/web-sites-php-mysql-deploy-use-git/new_website2.png
[custom-create]: ./media/web-sites-php-mysql-deploy-use-git/create_web_mysql.png
[new-mysql-db]: ./media/web-sites-php-mysql-deploy-use-git/create_db.png
[go-to-webapp]: ./media/web-sites-php-mysql-deploy-use-git/select_webapp.png
[setup-git-publishing]: ./media/web-sites-php-mysql-deploy-use-git/setup_git_publishing.png
[credentials]: ./media/web-sites-php-mysql-deploy-use-git/save_credentials.png
[resource-group]: ./media/web-sites-php-mysql-deploy-use-git/set_group.png
[new-web-app]: ./media/web-sites-php-mysql-deploy-use-git/create_wa.png
[setup-publishing]: ./media/web-sites-php-mysql-deploy-use-git/setup_deploy.png
[setup-repository]: ./media/web-sites-php-mysql-deploy-use-git/select_local_git.png
[select-database]: ./media/web-sites-php-mysql-deploy-use-git/select_database.png
[select-properties]: ./media/web-sites-php-mysql-deploy-use-git/select_properties.png
[note-properties]: ./media/web-sites-php-mysql-deploy-use-git/note-properties.png

[git-instructions]: ./media/web-sites-php-mysql-deploy-use-git/git-instructions.png
[git-change-push]: ./media/web-sites-php-mysql-deploy-use-git/php-git-change-push.png
[git-initial-push]: ./media/web-sites-php-mysql-deploy-use-git/php-git-initial-push.png

[composer-extension-settings]: ./media/web-sites-php-mysql-deploy-use-git/composer-extension-settings.png
[composer-extension-add]: ./media/web-sites-php-mysql-deploy-use-git/composer-extension-add.png
[composer-extension-view]: ./media/web-sites-php-mysql-deploy-use-git/composer-extension-view.png
[composer-extension-success]: ./media/web-sites-php-mysql-deploy-use-git/composer-extension-success.png

<!---HONumber=Mooncake_Quality_Review_1118_2016-->