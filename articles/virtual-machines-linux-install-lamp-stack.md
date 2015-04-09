<properties linkid="manage-linux-common-tasks-lampstack" urlDisplayName="Install LAMP stack" pageTitle="在 Linux 虚拟机上安装 LAMP 堆栈" metaKeywords="" description="了解如何在 Azure 中的 Linux 虚拟机 (VM) 上安装 LAMP 堆栈。您可以在 Ubuntu 或 CentOS 上进行安装。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Install the LAMP Stack on a Linux virtual machine in Azure" authors="" solutions="" manager="" editor="" />




#在 Azure 中的 Linux 虚拟机上安装 LAMP 堆栈

LAMP 堆栈包含以下不同元素：

- Linux - 操作系统
- Apache - Web 服务器
- MySQL - 数据库服务器
- PHP - 编程语言


##在 Ubuntu 上安装

您将需要安装以下程序包：

- `apache2`
- `mysql-server`
- `php5`
- `php5-mysql`

运行 `apt-get update` 以更新程序包的本地列表后，您随后可以使用单个 `apt-get install` 命令安装这些程序包：

	# sudo apt-get update
	# sudo apt-get install apache2 mysql-server php5 php5-mysql

运行上述命令后，将提示您安装这些程序包和一些其他依赖项。依次按"y"和"Enter"以继续，并按照任何其他提示为 MySQL 设置管理密码。

这将安装对 MySQL 使用 PHP 所需的最小 PHP 扩展。运行以下命令以查看作为程序包提供的其他 PHP 扩展：

	# apt-cache search php5


##在 CentOS 上安装

您将需要安装以下程序包：

- `httpd`
- `mysql`
- `mysql-server`
- `php`
- `php-mysql`

您可以使用单个 `yum install` 命令安装这些程序包：

	# sudo yum install httpd mysql mysql-server php php-mysql

运行上述命令后，将提示您安装这些程序包和一些其他依赖项。依次按"y"和"Enter"以继续。

这将安装对 MySQL 使用 PHP 所需的最小 PHP 扩展。运行以下命令以查看作为程序包提供的其他 PHP 扩展：

	# yum search php


## 在 SUSE Linux Enterprise Server 上安装

您将需要安装以下程序包：

- apache2
- mysql
- apache2-mod_php53
- php53-mysql

您可以使用单个 `zypper install` 命令安装这些程序包：

	# sudo zypper install apache2 mysql apache2-mod_php53 php53-mysql

运行上述命令后，将提示您安装这些程序包和一些其他依赖项。依次按"y"和"Enter"以继续。

这将安装对 MySQL 使用 PHP 所需的最小 PHP 扩展。运行以下命令以查看作为程序包提供的其他 PHP 扩展：

	# zypper search php


设置
----------

1. 设置 **Apache**

	- 运行以下命令以确保启动 Apache Web 服务器：

		- Ubuntu & SLES: `sudo service apache2 restart`

		- CentOS & Oracle: `sudo service httpd restart`

	- 默认情况下，Apache 在端口 80 上进行侦听。您可能需要打开终结点以远程访问您的 Apache 服务器。请参阅有关[配置终结点](/zh-cn/documentation/articles/virtual-machines-set-up-endpoints/)的文档 以获取更多详细说明。

	- 您现在可以查看 Apache 是否正在运行以及是否提供内容。将您的浏览器指向 `http://[MYSERVICE].cloudapp.net`，其中 **[MYSERVICE]** 是您的虚拟机所在的云服务的名称。在某些分发上，只声明"它有用！"的默认网页可能是欢迎网页。在其他分发上，您可能会看到一个更完整的网页，该网页带有指向其他文档和用于配置 Apache 服务器的内容的链接。

2. 设置 **MySQL**

	- 请注意，此步骤在 Ubuntu 上不是必需的，它会在安装 mysql-server 程序包时提示您输入 MySQL `root` 密码。

	- 在其他分发上，通过运行以下命令来为 MySQL 设置根密码：

			# mysqladmin -u root -p password yourpassword

	- 然后，您可以使用 `mysql` 或 `mysqladmin` 实用程序管理 MySQL。


##延伸阅读

有许多在 Ubuntu 上设置 LAMP 堆栈的资源。

- [https://help.ubuntu.com/community/ApacheMySQLPHP](https://help.ubuntu.com/community/ApacheMySQLPHP)
