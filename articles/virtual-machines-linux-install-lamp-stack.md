<properties
	pageTitle="在 Linux 虚拟机上安装 LAMP 堆栈 | Azure"
	description="了解如何在 Azure 中的 Linux 虚拟机 (VM) 上安装 LAMP 堆栈。"
	services="virtual-machines"
	documentationCenter=""
	authors="szarkos"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="07/29/2015"
	wacn.date="11/12/2015"/>



#在 Azure 中的 Linux 虚拟机上安装 LAMP 堆栈

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]

LAMP 堆栈包含以下不同元素：

- **L**inux - 操作系统
- **A**pache - Web 服务器
- **M**ySQL - 数据库服务器
- **P**HP - 编程语言


##在 Ubuntu 上安装

你将需要安装以下程序包：

- `apache2`
- **mysql-server**
- **php5**
- **php5-mysql**

运行 `apt-get update` 以更新程序包的本地列表以后，即可使用单个 `apt-get install` 命令安装这些程序包：

	# sudo apt-get update
	# sudo apt-get install apache2 mysql-server php5 php5-mysql

运行以上命令以后，系统会提示你安装这些程序包以及多个其他的依赖项。按“y”再按“Enter”即可继续，然后再遵循任何其他的提示为 MySQL 设置一个管理密码。

此时会安装最低要求的 PHP 扩展，这些扩展是通过 MySQL 使用 PHP 所需的。运行以下命令即可查看以程序包形式提供的其他 PHP 扩展：

	# apt-cache search php5


##在 CentOS 和 Oracle Linux 上安装

你将需要安装以下程序包：

- `httpd`
- **mysql**
- **mysql-server**
- **php**
- **php-mysql**

你可以使用单个 `yum install` 命令安装这些程序包：

	# sudo yum install httpd mysql mysql-server php php-mysql

运行以上命令以后，系统会提示你安装这些程序包以及多个其他的依赖项。按“y”再按“Enter”即可继续。

此时会安装最低要求的 PHP 扩展，这些扩展是通过 MySQL 使用 PHP 所需的。运行以下命令即可查看以程序包形式提供的其他 PHP 扩展：

	# yum search php


## 在 SUSE Linux Enterprise Server 上进行安装

你将需要安装以下程序包：

- apache2
- mysql
- apache2-mod_php53
- php53-mysql

你可以使用单个 `zypper install` 命令安装这些程序包：

	# sudo zypper install apache2 mysql apache2-mod_php53 php53-mysql

运行以上命令以后，系统会提示你安装这些程序包以及多个其他的依赖项。按“y”再按“Enter”即可继续。

此时会安装最低要求的 PHP 扩展，这些扩展是通过 MySQL 使用 PHP 所需的。运行以下命令即可查看以程序包形式提供的其他 PHP 扩展：

	# zypper search php


设置
----------

1. 设置 **Apache**

	- 运行以下命令以确保启动 Apache Web 服务器：

		- Ubuntu 和 SLES：`sudo service apache2 restart`

		- CentOS 和 Oracle：`sudo service httpd restart`

	- 默认情况下，Apache 在端口 80 上进行侦听。你可能需要打开一个终结点才能远程访问 Apache 服务器。请参阅[配置终结点](/documentation/articles/virtual-machines-linux-classic-setup-endpoints/)上的文档以获取详细说明。

	- 你现在可以查看 Apache 是否正在运行并提供内容服务。将浏览器指向 `http://[MYSERVICE].chinacloudapp.cn`，其中 **[MYSERVICE]** 是虚拟机所在的云服务的名称。在某些分发中，你可能会看到一个默认的 Web 页面，其内容只是简单的一句话：“It works!”。在其他分发中，你可能会看到更完整的 Web 页面，其中包含的链接指向更多配置 Apache 服务器所需的文档和内容。

2. 设置 **MySQL**

	- 请注意，此步骤在 Ubuntu 中是不必要的，它会在 mysql-server 程序包安装完以后提示你输入 MySQL `root` 密码。

	- 在其他分发中，可通过运行以下命令为 MySQL 设置根密码：

			# mysqladmin -u root -p password yourpassword

	- 然后，你可以使用 **mysql** 或 **mysqladmin** 实用程序管理 MySQL。


##延伸阅读

假设一下，你是否需要将这些步骤自动化，以便将应用程序部署到远程 Linux 虚拟机？ 你可以使用 Linux CustomScript 扩展来这样做。请参阅[使用适用于 Linux 的 Azure CustomScript 扩展部署 LAMP 应用程序](/documentation/articles/virtual-machines-linux-classic-lamp-script/)。

有许多在 Ubuntu 上设置 LAMP 堆栈的其他资源。

- [https://help.ubuntu.com/community/ApacheMySQLPHP](https://help.ubuntu.com/community/ApacheMySQLPHP)

<!---HONumber=79-->