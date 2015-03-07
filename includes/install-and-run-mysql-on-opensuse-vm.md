
1. 若要提升权限，请运行：

		sudo -s
	
	输入您的密码。

2. 运行以下命令以安装 MySQL Community Server 版本：

		# zypper install mysql-community-server

	下载和安装 MySQL 时，请等待。
3. 若要将 MySQL 设置为在系统启动时启动，请执行以下命令：

		# insserv mysql
4. 现在，您可以通过以下命令手动启动 MySQL daemon (mysqld)：

		# rcmysql start

	若要检查 MySQL daemon 的状态，请运行：

		# rcmysql status

	若要停止 MySQL daemon，请运行：

		# rcmysql stop

5. 警告！在安装后，MySQL 根密码默认为空。建议您运行 **mysql\_secure\_installation**，这是一个可帮助保护 MySQL 的脚本。运行 **mysql\_secure\_installation** 时，系统将提示您更改 MySQL 根密码、删除匿名用户帐户、禁用远程根登录名、删除测试数据库以及重新加载权限表。建议您对所有这些选项回答"是"并更改根密码。运行以下命令以执行脚本：

		$ mysql_secure_installation

6. 在运行后，您可登录到 MySQL：

		$ mysql -u root -p

	输入在上一步中所更改的 MySQL 根密码，将显示一条可发出 SQL 语句以与数据库进行交互的位置提示。

7. 若要创建新的 MySQL 用户，请在 **mysql>** 提示符处运行以下命令：

		mysql> CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	请注意，行尾的分号 (;) 对于结束命令很重要。

8. 若要创建数据库并向其授予  `mysqluser` 用户权限，请发出以下命令：

		mysql> CREATE DATABASE testdatabase;
		mysql> GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	请注意，数据库用户名和密码仅由连接到数据库的脚本使用。数据库用户帐户名称不一定表示系统上的实际用户帐户。

9. 若要从其他计算机登录，请执行以下命令：

		mysql> GRANT ALL ON testdatabase.* TO 'mysqluser'@'<ip-address>' IDENTIFIED BY 'password';

	其中， `ip-address` 是您将从其中连接到 MySQL 的计算机的 IP 地址。
	
10. 若要退出 MySQL 数据库管理实用程序，请发出以下命令：

		quit

11. 在安装 MySQL 后，您必须配置一个终结点，这样才能远程访问 MySQL。登录到 [Azure 管理门户][AzurePreviewPortal]。在 Azure 门户中，依次单击"虚拟机"、您的新 VM 的名称和"终结点"。

	![终结点][Image7]

12. 单击页面底部的"添加终结点"。
	![终结点][Image8]

13. 添加名为"MySQL"的终结点、协议 **TCP**，并将"公用"和"专用"端口均设置为"3306"。这将允许对 MySQL 进行远程访问。
	![终结点][Image9]

14. 若要在 Azure 中远程连接到在您的 OpenSUSE 虚拟机上运行的 MySQL，请在您的本地计算机上运行以下命令：

		mysql -u mysqluser -p -h <yourservicename>.cloudapp.net

	例如，使用我们在本教程中创建的虚拟机时该命令应为：

		mysql -u mysqluser -p -h testlinuxvm.cloudapp.net

15. 您已成功配置 MySQL、创建数据库和新用户。有关 MySQL 的详细信息，请参阅 [MySQL 文档][MySQLDocs]。	

[MySQLDocs]: http://dev.mysql.com/doc/
[AzurePreviewPortal]: http://manage.windowsazure.com
[Image7]: ./media/install-and-run-mysql-on-opensuse-vm/LinuxVmAddEndpoint.png
[Image8]: ./media/install-and-run-mysql-on-opensuse-vm/LinuxVmAddEndpoint2.png
[Image9]: ./media/install-and-run-mysql-on-opensuse-vm/LinuxVmAddEndpointMySQL.png
<!--HONumber=41-->
