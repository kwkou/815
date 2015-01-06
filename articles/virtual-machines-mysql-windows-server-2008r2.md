<properties linkid="manage-windows-common-tasks-install-mysql" urlDisplayName="Install MySQL" pageTitle="Create a virtual machine running MySQL in Azure " metaKeywords="Azure virtual machines, Azure Windows Server, Azure installing MySQL, Azure configuring MySQL, Azure databases" description="Create an Azure virtual machine running Windows Server 2008 R2, and then install and configure a MySQL database on the virtual machine." metaCanonical="" services="virtual-machines" documentationCenter="" title="Install MySQL on a virtual machine running Windows Server 2008 R2 in Azure" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 在 Azure 中运行 Windows Server 2008 R2 的虚拟机上安装 MySQL

[MySQL](http://www.mysql.com) 是一种受欢迎的 SQL 开源数据库。使用 [Azure 管理门户][AzurePortal]，你可从映像库创建运行 Windows Server 2008 R2 的虚拟机。然后，你可以在虚拟机上安装和配置 MySQL 数据库。

在本教程中，你将了解如何：

-   使用管理门户创建一个运行 Windows Server 2008 R2 的虚拟机。

-   在虚拟机上安装和运行 MySQL Community Server。

## 创建运行 Windows Server 2008 R2 的虚拟机

[WACOM.INCLUDE [create-and-configure-windows-server-2008-vm-in-portal](../includes/create-and-configure-windows-server-2008-vm-in-portal.md)]

## 附加数据磁盘

[WACOM.INCLUDE [attach-data-disk-windows-server-2008-vm-in-portal](../includes/attach-data-disk-windows-server-2008-vm-in-portal.md)]

## 在虚拟机上安装和运行 MySQL Community Server

按照下列步骤操作可安装、配置和运行 MySQL Community Server：

1.  使用远程桌面连接到该虚拟机后，从“开始”菜单打开 Internet Explorer。

2.  选择右上角的“工具”按钮。在“Internet 选项”中，选择“安全”选项卡，然后选择“可信站点”图标，最后单击“站点”按钮。将 *http://\*.mysql.com* 添加到受信任站点列表中。

3.  转到[下载 MySQL 社区服务器][MySQLDownloads]。

4.  在“平台”下拉列表中选择 **Microsoft Windows**并单击“选择”。

5.  找到“Windows（x86，64 位），MSI 安装程序”并单击“下载”。

6.  选择“不，谢谢，请开始我的下载！”（或者，注册一个帐户）。如果出现提示，请选择镜像站点以下载 MySQL 安装程序并将该安装程序保存到桌面。

7.  双击桌面上的安装程序文件以开始安装。

8.  单击“下一步”。

	![MySQL Setup][MySQLInstall1]

9.  接受许可协议，然后单击**“下一步”**。

	![MySQL Setup][MySQLInstall2]

10. 单击“典型”以安装常见功能。

	![MySQL Setup][MySQLInstall3]

11. 单击**“安装”**。

	![MySQL Setup][MySQLInstall4]

12. 启动 MySQL 配置向导并单击“下一步”。

	![Configure MySQL][MySQLConfig1]

13. 选择“详细配置”，然后单击“下一步”。

	![Configure MySQL][MySQLConfig2]

14. 如果你计划让 MySQL 与服务器上的其他应用程序一起运行，或选择最适合你需要的选项，请选择“服务器计算机”。单击“下一步”。

	![Configure MySQL][MySQLConfig3]

15. 选择“多功能数据库”，或选择最适合你需要的选项。单击“下一步”。

	![Configure MySQL][MySQLConfig4]

16. 选择在上述步骤中附加的数据驱动器。

	![Configure MySQL][MySQLConfig5]

17. 选择“决策支持 (DSS)/OLAP”，或选择最适合你需要的选项。单击“下一步”。

	![Configure MySQL][MySQLConfig6]

18. 选择“启用 TCP/IP 网络”和“为此端口添加防火墙例外”（这样将在 Windows 防火墙中创建名为“MySQL 服务器”的入站规则）。

	![Configure MySQL][MySQLConfig7]

19. 如果你需要用多种不同语言存储文本，请选择“多语制的最佳支持”，或者选择最适合你需要的选项。单击“下一步”。

	![Configure MySQL][MySQLConfig8]

20. 选择“安装为 Windows 服务”和“自动启动 MySQL 服务器”。另请选择“包括 Windows 路径中的 Bin 目录”。单击“下一步”。

	![Configure MySQL][MySQLConfig9]

21. 输入根密码。不要选中“启用来自远程计算机的根访问”或“创建匿名帐户”。单击“下一步”。

	![Configure MySQL][MySQLConfig10]

22. 单击**“执行”**并等待配置完成。

	![Configure MySQL][MySQLConfig11]

23. 单击“完成”。

	![Configure MySQL][MySQLConfig12]

24. 单击“开始”并选择“MySQL 5.x 命令行客户端”以启动该命令行客户端。

25. 在提示符处输入你在上一步中设置的根密码，这将显示一条提示，告知你可从何处发出 SQL 语句以便与数据库进行交互。

26. 若要创建新的 MySQL 用户，请在 **mysql\>** 提示符处运行以下命令：

        mysql> CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';

    请注意，行尾的分号 (;) 对于结束命令很重要。

27. 若要创建数据库并授予`mysqluser` 用户访问它的权限，请发出以下命令：

        mysql> CREATE DATABASE testdatabase;
        mysql> GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';

    请注意，数据库用户名和密码仅由连接到数据库的脚本使用。数据库用户帐户名称不一定表示计算机上的实际用户帐户。

28. 若要从其他计算机登录，请执行以下命令：

        mysql> GRANT ALL ON testdatabase.* TO 'mysqluser'@'<ip-address>' IDENTIFIED BY 'password';

    其中`ip-address` 是你将从其中连接到 MySQL 的计算机的 IP 地址。

29. 若要退出 MySQL 命令行客户端，请发出以下命令：

        quit

30. 在安装 MySQL 后，你必须配置终结点才能远程访问 MySQL。登录到 [Azure 管理门户][AzurePortal]。在 Azure 门户中，依次单击“虚拟机”、你的新虚拟机的名称、“终结点”和“添加终结点”。

	![Endpoints][AddEndPoint]

31. 选择“添加终结点”，然后单击箭头以继续。

	![Endpoints][AddEndPoint2]

32. 添加一个名为“MySQL”的终结点，将其协议设置为“TCP”，并将“公用”和“专用”端口设置为“3306”。单击复选标记。这将允许对 MySQL 进行远程访问。

	![Endpoints][AddEndPoint3]

33. 可以远程连接到在 Azure 中的虚拟机上运行的 MySQL。从运行 MySQL 的本地计算机上，运行以下命令以登录为你在上面步骤中创建的 **mysqluser** 用户：

        mysql -u mysqluser -p -h <yourservicename>.chinacloudapp.cn

    例如，通过使用本教程中创建的虚拟机，该命令应为：

        mysql -u mysqluser -p -h testwinvm.chinacloudapp.cn

## 摘要

在本教程中，你学习了如何创建 Windows 2008 R2 虚拟机并远程连接到该虚拟机。你还了解了如何在虚拟机上安装和配置 MySQL 以及创建数据库和新的 MySQL 用户。有关 MySQL 的详细信息，请参阅 [MySQL 文档](http://dev.mysql.com/doc/)。

[AzurePortal]: http://manage.windowsazure.cn
[MySQLDownloads]: http://www.mysql.com/downloads/mysql/


[MySQLInstall1]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLInstall1.png
[MySQLInstall2]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLInstall2.png
[MySQLInstall3]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLInstall3.png
[MySQLInstall4]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLInstall4.png
[MySQLConfig1]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig1.png
[MySQLConfig2]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig2.png
[MySQLConfig3]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig3.png
[MySQLConfig4]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig4.png
[MySQLConfig5]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig5.png
[MySQLConfig6]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig6.png
[MySQLConfig7]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig7.png
[MySQLConfig8]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig8.png
[MySQLConfig9]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig9.png
[MySQLConfig10]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig10.png
[MySQLConfig11]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig11.png
[MySQLConfig12]: ./media/virtual-machines-mysql-windows-server-2008r2/MySQLConfig12.png
[AddEndPoint]: ./media/virtual-machines-mysql-windows-server-2008r2/WinVMAddEndpointMySQL0.png
[AddEndPoint2]: ./media/virtual-machines-mysql-windows-server-2008r2/WinVMAddEndpointMySQL1.png
[AddEndPoint3]: ./media/virtual-machines-mysql-windows-server-2008r2/WinVMAddEndpointMySQL.png
