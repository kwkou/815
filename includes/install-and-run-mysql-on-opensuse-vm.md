
1. 若要提升特权，请键入：

		sudo -s

	输入您的密码。

2. 若要安装 MySQL Community Server 版本，请键入：

		zypper install mysql-community-server

	下载和安装 MySQL 时，请等待。

3. 若要将 MySQL 设置为在系统引导时启动，请键入：

		insserv mysql

4. 使用以下命令手动启动 MySQL 守护程序 (mysqld)：

		rcmysql start

	若要检查 MySQL 守护程序的状态，请键入：

		rcmysql status

	若要停止 MySQL 守护程序，请键入：

		rcmysql stop

	> [AZURE.IMPORTANT] 在安装后，MySQL 根密码默认为空。建议你运行 **mysql\_secure\_installation**，这是一个可帮助保护 MySQL 的脚本。该脚本将提示你更改 MySQL 根密码、删除匿名用户帐户、禁用远程根登录、删除测试数据库以及重新加载特权表。建议你对所有这些选项回答“是”并更改根密码。

5. 键入以下内容来运行脚本 MySQL 安装脚本：

		mysql_secure_installation

6. 登录到 MySQL：

		mysql -u root -p

	输入在上一步中所更改的 MySQL 根密码，将显示一条可发出 SQL 语句以与数据库进行交互的位置提示。

7. 若要创建新的 MySQL 用户，请在 **mysql>** 提示符处运行以下命令：

		CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	请注意，行尾的分号 (;) 对于结束命令很重要。

8. 若要创建数据库并向其授予 `mysqluser` 用户权限，请发出以下命令：

		CREATE DATABASE testdatabase;
		GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	请注意，数据库用户名和密码仅由连接到数据库的脚本使用。数据库用户帐户名称不一定表示系统上的实际用户帐户。

9. 若要从另一台计算机登录，请键入：

		GRANT ALL ON testdatabase.* TO 'mysqluser'@'<ip-address>' IDENTIFIED BY 'password';

	其中，`ip-address` 是你将从其中连接到 MySQL 的计算机的 IP 地址。

10. 若要退出 MySQL 数据库管理实用程序，请键入：

		quit
		
## 添加终结点

1. 安装 MySQL 后，你必须配置终结点，以便远程访问 MySQL。登录到 [Azure 经典管理门户][AzurePortal]。依次单击“虚拟机”、你的新虚拟机的名称和“终结点”。

2. 单击页面底部的“添加”。


3. 添加名为“MySQL”的终结点，协议为“TCP”，并将“公用”和“专用”端口均设置为“3306”。

4. 若要从你的计算机远程连接到虚拟机，请键入：

		mysql -u mysqluser -p -h <yourservicename>.chinacloudapp.cn

	例如，使用我们在本教程中创建的虚拟机时，键入以下命令：

		mysql -u mysqluser -p -h testlinuxvm.chinacloudapp.cn

[MySQLDocs]: http://dev.mysql.com/doc/
[AzurePortal]: http://manage.windowsazure.cn

[Image9]: ./media/install-and-run-mysql-on-opensuse-vm/LinuxVmAddEndpointMySQL.png

<!---HONumber=Mooncake_0314_2016-->