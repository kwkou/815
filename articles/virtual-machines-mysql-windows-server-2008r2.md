<properties linkid="manage-windows-common-tasks-install-mysql" urlDisplayName="Install MySQL" pageTitle="创建在 Azure 中运行 MySQL 的虚拟机 " metaKeywords="Azure virtual machines, Azure Windows Server, Azure installing MySQL, Azure configuring MySQL, Azure databases" description="创建 Azure 虚拟机运行 Windows Server 2008 R2，然后安装并在虚拟机上配置 MySQL 数据库。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Install MySQL on a virtual machine running Windows Server 2008 R2 in Azure" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />





#在 Azure 中运行 Windows Server 2008 R2 的虚拟机上安装 MySQL

[MySQL](http://www.mysql.com)是一种流行的开放源、SQL 数据库。使用[Azure 管理门户](http://www.windowsazure.cn)，可以创建从映像库中运行 Windows Server 2008 R2 的虚拟机。然后，可以安装，并在虚拟机上配置 MySQL 数据库。

本教程演示如何为：

- 使用管理门户创建一个运行 Windows Server 2008 R2 的虚拟机。

- 在虚拟机上安装和运行 MySQL Community Server。

##创建运行 Windows Server 的虚拟机

[WACOM.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-WindowsVM.md)]

##附加数据磁盘

创建虚拟机后，将附加数据磁盘。此磁盘提供了安装 MySQL 时需要的数据存储。请参阅[如何将数据磁盘附加到 Windows 虚拟机](/zh-cn/documentation/articles/storage-windows-attach-disk/) 然后按照用于附加空磁盘的说明进行操作。

##登录到虚拟机
接下来，您将登录到虚拟机以便可以安装 MySQL。

[WACOM.INCLUDE [virtual-machines-log-on-win-server](../includes/virtual-machines-log-on-win-server.md)]

##在虚拟机上安装和运行 MySQL Community Server
请按照以下步骤来安装、配置和运行 MySQL Community Server 操作：

1. 您已连接到使用远程桌面的虚拟机后，打开**Internet Explorer**从**启动**菜单。 

2. 选择**工具**右上角的按钮。在**Internet 选项**，选择**安全**选项卡上，然后选择**受信任的站点**图标，最后单击**站点**按钮。添加*http://\*.mysql.com*到受信任的站点的列表。

3. 转到[下载 MySQL Community Server][MySQLDownloads]。

4. 选择**Microsoft Windows**中**平台**下拉列表菜单，然后单击**选择**。

5. 查找最新版本的**Windows （x86，64 位）、 MSI 安装程序**单击**下载**。 

6. 选择**不，谢谢，仅开始我的下载 ！**（或者，注册帐户）。如果系统提示，请选择一个镜像站点以下载 MySQL 安装程序并将该安装程序保存到桌面。

7. 双击桌面上的安装程序文件以开始安装。

8. 单击**下一步**。

9. 接受许可协议，然后单击**下一步**。

10. 单击**典型**以安装常用功能。

11. 单击**安装**。

12. 启动 MySQL 配置向导，然后单击**下一步**。

13. 选择**详细配置**和单击下一步。

14. 选择**Server 机**如果你打算在服务器上，与其他应用程序一起运行 MySQL 或选择选项最适合您的需要。单击**下一步**。

15. 选择**多功能数据库**，或选择最适合您需求的选项。单击**下一步**。

16. 选择您在上一节中附加的数据驱动器。

	![Configure MySQL][MySQLConfig5]

17. 选择**决策支持 (DSS) / OLAP**，或选择最适合您需求的选项。单击**下一步**。

18. 选择**启用 TCP/IP 网络**和**添加此端口的防火墙例外**（这将在名为 Windows 防火墙中创建一个入站的规则**MySQL Server**)。

	![Configure MySQL][MySQLConfig7]

19. 如果您需要将文本存储在许多不同的语言，则选择**最佳支持的多语言的**。否则，请选择其他选项之一。单击**下一步**。

	![Configure MySQL][MySQLConfig8]

20. 选择**作为 Windows 服务安装**和**自动启动 MySQL Server**。此外选择**Windows 路径中包括 Bin 目录**。单击**下一步**。

	![Configure MySQL][MySQLConfig9]

21. 输入根密码。不会检查**启用从远程计算机的根访问**或**创建匿名帐户**。单击**下一步**。

	![Configure MySQL][MySQLConfig10]

22. 单击**Execute**并等待配置完成。

23. 单击**完成**。

24. 单击**启动**，然后选择**MySQL 5.x 命令行客户端**启动命令行客户端。

25.  输入在提示符下 （这在上一步中设置了） 的根密码，然后您将显示一条提示您可以在其中发出 SQL 语句以与数据库交互。

26. 若要创建新的 MySQL 用户，运行以下命令在**mysql >**提示：

		mysql> CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	用分号 (;) 结尾的行所需结束命令。

27. 若要创建数据库以及授予`mysqluser`用户权限，请运行以下命令：

		mysql> CREATE DATABASE testdatabase;
		mysql> GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';

	请注意，数据库用户名和密码仅由连接到数据库的脚本使用。数据库用户帐户名称不一定表示计算机上的实际用户帐户。

28. 若要从另一台计算机记录中，请运行以下命令：

		mysql> GRANT ALL ON testdatabase.* TO 'mysqluser'@'<ip-address>' IDENTIFIED BY 'password';

	其中，'ip-address' 是您從中連接到 MySQL 的電腦 IP 位址。
	
29. 若要退出 MySQL 命令行客户端，请运行以下命令：

		quit

30. 安装 MySQL 后，配置终结点，以便可以远程访问 MySQL。登录到[Azure 管理门户][AzurePreviewPortal]。在 Azure 门户中，单击**虚拟机**，然后是名称的新虚拟机，然后**终结点**，然后**添加终结点**。

31. 选择**添加终结点**，然后单击箭头以继续。
	

32. 添加名为"MySQL"，协议的终结点**TCP**，并且两个**公共**和**私有**端口设置为"3306"。单击复选标记。这允许您远程访问 MySQL。
	

33. 测试你的远程连接到 MySQL。从运行 MySQL 的本地计算机，运行类似于以下命令以登录**mysqluser**用户：

		mysql -u mysqluser -p -h <yourservicename>.chinacloudapp.cn

	例如，如果名为虚拟机"testwinvm"时，运行此命令：

		mysql -u mysqluser -p -h testwinvm.chinacloudapp.cn

##资源
有关 MySQL 的信息，请参阅[MySQL 文档](http://dev.mysql.com/doc/)。

[AzurePortal]: http://manage.windowsazure.cn
[MySQLDownloads]: http://www.mysql.com/downloads/mysql/


[MySQLConfig5]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig5.png
[MySQLConfig7]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig7.png
[MySQLConfig8]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig8.png
[MySQLConfig9]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig9.png
[MySQLConfig10]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig10.png

