<!-- Ibiza Portal -->

<properties
	pageTitle="使用 Azure 创建 LAMP 堆栈 | Azure"
	description="了解如何在 Azure 中使用运行 Linux 的 Azure 虚拟机 (VM) 创建 LAMP 堆栈。"
	services="virtual-machines"
	documentationCenter=""
	authors="NingKuang"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="12/15/2015"
	wacn.date="01/29/2016"/>

#如何使用 Azure 创建 LAMP 堆栈

“LAMP”堆栈是一组开源软件，通常一起安装，使服务器可以托管动态网站和 Web 应用程序。此术语实际上首字母缩写词，表示带 Apache Web 服务器的 Linux 操作系统。站点数据将存储在 MySQL 数据库中，而动态内容将由 PHP 进行处理。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


在本指南中，我们将获取 Linux 映像上安装的 LAMP 堆栈，并将其部署在 Azure 上。

你将学习以下内容：

-	如何创建 Azure 虚拟机。
-	如何准备用于 LAMP 堆栈的虚拟机。
-	如何安装虚拟机上的 LAMP 服务器所需的软件。

假定读者已拥有 Azure 订阅。如果没有，你可以在 [https://www.azure.cn/](/pricing/1rmb-trial/) 中注册试用版。

如果你已有虚拟机，并且只想查找在不同 linux 分发上安装 LAMP 堆栈的基础知识，则除了本主题外，还可以参阅[在 Azure 中的 Linux 虚拟机上安装 LAMP 堆栈](/documentation/articles/virtual-machines-linux-install-lamp-stack/)。


##阶段 1：创建映像
在此阶段中，你将在 Azure 中使用 Linux 映像创建虚拟机。

###步骤 1：生成 SSH 身份验证密钥
SSH 是面向系统管理员的重要工具。但是，依赖于人工确定的密码来确保安全并不总是明智的做法。使用强 SSH 密钥，可让远程访问保持打开状态，而无需担心密码。此方法包括使用非对称加密进行身份验证。用户的私钥是授予身份验证的密钥。你甚至可以锁定用户的帐户，以完全禁止密码身份验证。

按照下列步骤进行操作可生成 SSH 身份验证密钥。

-	从 [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 下载并安装 puttygen
-	运行 puttygen.exe。
-	单击**“生成”**按钮以生成密钥。在此过程中，可以通过将鼠标放在窗口中的空白区域上来增加随机性。  
![][1]
-	在生成过程结束后，puttygen.exe 将显示生成的密钥。例如：  
![][2]
-	在**“密钥”**中选择并复制公钥，并将它保存在一个名为 **publicKey.pem** 的文件中。不要单击**“保存公钥”**，因为保存的公钥的文件格式不同于我们所需的公钥。
-	单击**“保存私钥”**，并将其保存到名为 **privateKey.ppk** 的文件中。

###步骤 2：在 Azure 经典管理门户中创建映像。
在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中，单击任务栏中的**“新建”**，按照这些说明创建映像，并根据你的需要选择 Linux 映像。此示例使用 Ubuntu 14.04 映像。

![][3]

对于**“主机名”**，指定你的 Internet 客户端将用于访问此虚拟机的 URL 的名称。定义 DNS 名称的最后部分（例如 LAMPDemo），Azure 将生成 Lampdemo.chinacloudapp.cn 作为 URL。

对于**“用户名”**，选取稍后将用于登录到虚拟机的名称。

对于**“SSH 身份验证密钥”**，从 **publicKey.pem** 文件中复制密钥值，其中包含由 puttygen 生成的公钥。

![][4]

根据需要配置其他设置，然后单击**“创建”**。

##阶段 2：准备用于 LAMP 堆栈的虚拟机
在此阶段中，将为 Web 流量配置终结点，然后连接到新的虚拟机。

###步骤 1：打开 HTTP 端口，以允许 Web 访问
Azure 中的终结点由协议（TCP 或 UDP）以及公用和专用端口组成。专用端口是服务侦听虚拟机的端口。公用端口是 Azure 云服务从外部侦听基于 Internet 的通信的端口。在某些情况下，这是同一端口号。

TCP 端口 80 是 Apache 侦听的默认端口号。使用 Azure 终结点打开此端口将允许你和其他 Internet 客户端访问 Apache Web 服务器。

在 Azure 经典管理门户中，单击“虚拟机”，然后单击你创建的虚拟机。

![][5]

若要将终结点添加到虚拟机，请单击**“终结点”**。

![][6]

单击**“添加”**。在设置新的虚拟机时，可以根据需要启用或禁用终结点。

配置终结点：

1.	在**“终结点”**中键入终结点的名称。
2.	在**“公用端口”**中键入 80。如果你更改了 Apache 的默认侦听端口，则应将专用端口更新为与 Apache 侦听端口相同。
3.	在**“公用端口”**中键入 80。默认情况下，HTTP 通信使用端口 80。如果将其设置为 80，则无需在 URL 中包括端口号即可允许你访问 Apache Web 服务。例如，http://lampdemo.chinacloudapp.cn。如果将 Apache 侦听端口设为另一个值（例如 81），则需要将端口号添加到 URL 中才能访问 Apache Web 服务。例如，http://lampdemo.chinacloudapp.cn:81/。

![][7]

单击**“确定”**将该终结点添加到你的虚拟机。




###步骤 2：连接到你创建的映像
你可以选择用于连接到新虚拟机的任何 SSH 工具。在此示例中，我们使用 Putty。

首先，从 Azure 经典管理门户获取你的虚拟机的 DNS 名称。依次单击**“虚拟机”->**、你的虚拟机的名称**->“仪表板”**。

从 **SSH Details** 字段获取 SSH 连接的端口号。下面是一个示例。

![][8]

从[此处](http://www.putty.org/)下载 Putty。

下载后，单击可执行文件 PUTTY.EXE。使用从虚拟机的属性获取的主机名和端口号配置基本选项。下面是一个示例：

![][9]

在左窗格中，单击**“连接”-> SSH->“身份验证”**，然后单击**“浏览”**以指定 **privateKey.ppk** 文件（其中包含在“第 1 阶段：创建映像”中 puttygen 生成的私钥）的位置。下面是一个示例：

![][10]

单击**“打开”**。此时可能会通过一个消息框提醒你。如果你已正确配置 DNS 名称和端口号，请单击**“是”**。

![][11]


你应该会看到以下内容。

![][12]

输入在“第 1 阶段：创建映像”中创建虚拟机时指定的用户名。你将看到如下内容：

![][13]

##第 3 阶段：安装 LAMP 堆栈

根据用于创建虚拟机的 Linux 分发，有不同方法可安装 LAMP 堆栈。以下部分包含针对一些常见 Linux 操作系统的典型步骤。

###Red Hat、CentOS base

####安装 Apache
若要安装 Apache，请打开终端并执行此命令：

	sudo yum install httpd

安装后，使用此命令启动 Apache：

	sudo service httpd start

####测试 Apache
若要检查 Apache 是否已成功安装，请浏览到 Apache 服务器的 DNS 名称（对于本文中的示例 URL，为 http://lampdemo.chinacloudapp.cn/)。 该页应显示单词“It works!”
![][14]

####故障排除
如果 Apache 正在运行，但你看不到上面的 Apache 默认页，则需要检查以下项：

-	Apache Web 服务侦听地址/端口
	-	检查 Azure 虚拟机的终结点设置。确保终结点的配置适用。查看本文中“第 1 阶段：创建映像”中的说明。
	-	打开 /etc/httpd/conf/httpd.conf，然后搜索字符串“Listen”。确保 Apache 侦听端口与你为终结点配置的专用端口相同。Apache 的默认端口为 80。下面是一个示例。  

			……
			......
				# prevent Apache from glomming onto all bound IP addresses (0.0.0.0)
				#
				#Listen 12.34.56.78:80
				Listen 80
				#
				# Dynamic Shared Object (DSO) Support
			……
			……  

-	防火墙 iptables 配置，  
如果你可以在本地主机上看到 Apache 默认页，则问题可能出在 Apache 所侦听的端口被防火墙阻止。可以使用 w3m 工具来浏览 Apache 网页。以下命令安装 w3m 并浏览到 Apache 默认页：

		sudo yum  install w3m w3m-img  
		w3m http://localhost

	如果此问题是由防火墙或 iptables 导致的，请将以下行添加到 /etc/sysconfig/iptables：

		-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
		-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT

	请注意，只有 https 通信才需要第二行。

	请确保这些行在全局限制访问权限的所有行上方，如下所示：

		-A INPUT -j REJECT --reject-with icmp-host-prohibited  

	要使新设置生效，请使用以下命令：

		service iptables restart

####安装 MySQL
若要安装 MySQL，请打开终端并运行这些命令：

	sudo yum install mysql-server
	sudo service mysqld start

在安装过程中，MySQL 将两次要求你提供权限。当你确认这两个查询后，将安装 MySQL。

####配置 MySQL
在安装完成后，可以使用以下项设置 MySQL 根密码：

	sudo /usr/bin/mysql_secure_installation  

此提示将要求你提供当前根密码。

由于你刚安装 MySQL，很可能没有该密码，因此可通过按 ENTER 将其留空。

	Enter current password for root (enter for none):
	OK, successfully used password, moving on...  

系统将提示你设置根密码。继续操作并选择 Y，然后按照说明进行操作。

CentOS 将自动执行设置 MySQL 的过程，询问你一系列 yes 或 no 问题。这些问题如下所示。为你的配置选择 Y 或 N。在结束时，MySQL 将重新加载并实现新更改。

	By default, a MySQL installation has an anonymous user, allowing anyone
	to log into MySQL without having to have a user account created for
	them.  This is intended only for testing, and to make the installation
	go a bit smoother.  You should remove them before moving into a
	production environment.

	Remove anonymous users? [Y/n] y
	 ... Success!

	Normally, root should only be allowed to connect from 'localhost'.  This
	ensures that someone cannot guess at the root password from the network.

	Disallow root login remotely? [Y/n] y
	... Success!

	By default, MySQL comes with a database named 'test' that anyone can
	access.  This is also intended only for testing, and should be removed
	before moving into a production environment.

	Remove test database and access to it? [Y/n] y
	 - Dropping test database...
	 ... Success!
	 - Removing privileges on test database...
	 ... Success!

	Reloading the privilege tables will ensure that all changes made so far
	will take effect immediately.

	Reload privilege tables now? [Y/n] y
	 ... Success!

	Cleaning up...

	All done!  If you've completed all of the above steps, your MySQL
	installation should now be secure.

	Thanks for using MySQL!  

####安装 PHP
PHP 是广泛用于生成动态网页的开源 Web 脚本语言。

若要在虚拟机上安装 PHP，请打开终端并运行此命令：

	sudo yum install php php-mysql  

回答“y”以下载软件程序包。然后，回答“y”以导入 GPG 密钥 0xE8562897 "CentOS-5 密钥（CentOS 5 正式签名密钥）。此时将安装 PHP。

	warning: rpmts_HdrFromFdno: Header V3 DSA signature: NOKEY, key ID e8562897
	updates/gpgkey                                                                                                                                                                       | 1.5 kB     00:00
	Importing GPG key 0xE8562897 "CentOS-5 Key (CentOS 5 Official Signing Key) <centos-5-key@centos.org>" from /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
	Is this ok [y/N]: y  

###Debian、Ubuntu base
这已在 Ubuntu 14.04 上进行了测试。

Ubuntu 基于 Debian。可以用安装 Red Hat 系列的相同方式来安装 LAMP 堆栈。若要简化步骤，请使用 Tasksel 工具。

Tasksel 是一个 Debian/Ubuntu 工具，它将多个相关包作为协调任务安装到你的系统中。有关详细信息，请参阅 [Tasksel - 社区帮助 Wiki](https://help.ubuntu.com/community/Tasksel)。

使用 Tasksel 安装 LAMP 堆栈所需的软件。

- 若要从存储库中下载程序包列表并更新它们以获取有关最新版本的程序包及其依赖项的信息：  

		sudo apt-get update
-	若要使用 Tasksel 安装 Ubuntu LAMP 堆栈，请执行以下操作：  

		sudo apt-get install tasksel
		sudo tasksel install lamp-server

接下来，完成该向导并选择你的 **MySQL 根密码**。

![][15]


##在你的服务器上上测试 LAMP
可以通过创建 php 快速信息页来测试 LAMP 系统。

首先，创建一个新文件：

	sudo nano /var/www/html/info.php  

添加以下行：

	<?php
	phpinfo();
	?>  

然后，保存并退出。

重启 Apache，以便所有更改都在你的计算机上生效。如果你的虚拟机的操作系统是 CentOS，则使用以下命令重启 Apache：

	sudo service httpd restart

如果你的虚拟机的操作系统是 Ubuntu，则使用以下命令重启 Apache：

	sudo service apache2 restart  

通过浏览到 php 信息页（对于本主题中的示例 Web 服务器，URL 将为 http://lampdemo.chinacloudapp.cn/info.php)）来完成此过程。

你的浏览器应如下所示：

![][16]

##附加步骤

常规做法是，你将更改某些默认设置以准备进行 Web 应用程序部署。

###允许远程访问 MySQL
如果你随 MySQL 一起安装了多个虚拟机，并且这些虚拟机需要交换数据，则应启用 MySQL 远程访问并授予相应权限。

**命令参考格式：**

	grant [authority] on [databaseName].[tableName] to [username]@[login host] identified by "[passwd]"  

**示例：**

	grant select,insert,update,delete on studentManage.student to user1@"%" Identified by "abc";

你还应更改 /etc/mysql/my.cnf 配置文件。如果你有如下代码行：

	skip-networking
	bind-address = 127.0.0.1  

应将它们注释掉（在这些行开头添加 #），然后重启 MySQL。

若要添加终结点以允许远程访问，请参阅“第 1 阶段：创建映像”中的说明创建新终结点。MySQL 的默认远程访问 TCP 端口号为 3306。下面是一个示例：

![][17]

###将 Web 应用程序部署到 Apache 服务器
成功安装 LAMP 堆栈后，便可以将现有 Web 应用程序部署到 Apache Web 服务器（你的虚拟机）了。这些步骤与在物理 Web 服务器上部署现有 Web 应用程序的步骤相同。

-	Web 服务器的根位于 **/var/www/html**。你应向需要将文件上载到此文件夹的用户授予特权。以下示例演示如何将写权限添加到名为 lampappgroup 的组并在此组中放置 azureuser 用户名：  

		sudo groupadd lampappgroup                      # Create a group
		sudo gpasswd -a azureuser lampappgroup    # Add azureuser to lampappgroup
		sudo chgrp lampappgroup /var/www/html/  # Change the ownership to group lampappgroup
		sudo chmod g+w /var/www/html/                 # grant write permission to group lampappgroup

	>[AZURE.NOTE]你可能需要重新登录才能修改 /var/www/html/ 中的文件。
-	使用任何 SFTP 客户端（例如 FileZilla）连接到你的虚拟机（例如，lampdemo.chinacloudapp.cn）的 DNS 名称，并导航到 /**var/www/html** 以发布站点。  
![][18]



##常见问题和故障排除

###无法通过 Internet 使用 Apache 和 Moodle 访问虚拟机

-	**症状** Apache 正在运行，但你使用浏览器看不到 Apache 默认页。
-	**可能的根本原因**
	1.	Apache 侦听端口与用于 Web 通信的虚拟机终结点的专用端口不同。</br>
	检查你的公用端口和专用端口终结点设置，并确保专用端口与 Apache 侦听端口相同。有关如何为你的虚拟机配置终结点的说明，请参阅“第 1 阶段：创建映像”。</br>
	若要确定 Apache 侦听端口，请打开 /etc/httpd/conf/httpd.conf （Red Hat 发行版）或 /etc/apache2/ports.conf（Debian 发行版），搜索字符串“Listen”。默认端口为 80。

	2.	防火墙已禁用 Apache 侦听端口。</br>  
	如果你可以在本地主机上看到 Apache 默认页，则问题很可能出在 Apache 所侦听的端口被防火墙阻止。可以使用 w3m 工具来浏览网页。以下命令安装 w3m 并浏览到 Apache 默认页：  

			sudo yum  install w3m w3m-img
			w3m http://localhost

-	**解决方案**

	1.	如果 Apache 侦听端口与虚拟机上用于 Web 通信的终结点专用端口不同，则需要将终结点专用端口更改为与 Apache 侦听端口相同。
	2.	如果此问题是由防火墙/iptables 导致的，请将以下行添加到 /etc/sysconfig/iptables：  

			-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
			-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT

		请注意，只有 https 通信才需要第二行。

		请确保该行在全局限制访问权限的所有行上方，如下所示：

			-A INPUT -j REJECT --reject-with icmp-host-prohibited  

		若要重新加载 iptables，请运行以下命令：

			service iptables restart  

		这已在 CentOS 6.3 上进行了测试。

###将项目文件上载到 /var/www/html/ 时，权限被拒绝  

-	**症状**  
当你使用任何 SFTP 客户端（例如 FileZilla）连接到虚拟机并导航到 /var/www/html 来发布站点时，你收到如下错误消息：  

		status:	Listing directory /var/www/html
		Command:	put "C:\Users\liang\Desktop\info.php" "info.php"
		Error:	/var/www/html/info.php: open for write: permission denied
		Error:	File transfer failed

-	**可能的根本原因**
你无权访问 /var/www/html 文件夹。
-	**解决方案**  
你需要获得根帐户权限。你可以将该文件夹的所有权从 root 更改为在设置计算机时使用的用户名。下面是使用 azureuser 帐户名称的示例：  

		sudo chown azureuser -R /var/www/html  

	也使用 -R 选项对目录内的所有文件应用权限。

	请注意，此命令也适用于目录。-R 选项可更改目录内的所有文件和目录的权限。下面是一个示例：

		sudo chown -R username:group directory  

	此命令将更改目录内的所有文件和目录以及目录本身的所有权（用户和组）。

	以下命令只会更改文件夹目录的权限，但不会更改目录内的文件和文件夹的权限。

		sudo chown username:group directory  

###无法可靠地确定服务器的完全限定域名

-	**症状**  
使用以下命令之一重启 Apache 服务器时：  

		sudo /etc/init.d/apache2 restart  # Debian release  

	或

		sudo /etc/init.d/httpd restart   # Red Hat release  

	如果你收到以下错误：

		Restarting web server apache2
		apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName
		... waiting apache2:
		Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName  

-	**可能的根本原因**
	你尚未设置 Apache 的服务器名称。

-	**解决方案**  
	在 httpd.conf（Red Hat 发行版）或 apache2.conf（Debian 发行版）的 /etc/apache2 中插入“ServerName localhost”行，并重启 Apache。注意将消失。



[1]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-01.png
[2]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-02.png
[3]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-03.png
[4]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-04.png
[5]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-05.png
[6]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-06.png
[7]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-07.png
[8]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-08.png
[9]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-09.png
[10]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-10.png
[11]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-11.png
[12]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-12.png
[13]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-13.png
[14]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-14.png
[15]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-15.png
[16]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-16.png
[17]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-17.png
[18]: ./media/virtual-machines-linux-create-lamp-stack/virtual-machines-linux-create-lamp-stack-18.jpg

<!---HONumber=Mooncake_0118_2016-->