<properties linkid="manage-linux-common-tasks-lampstack" urlDisplayName="Install LAMP stack" pageTitle="Install the LAMP stack on a Linux virtual machine" metaKeywords="" description="Learn how to install the LAMP stack on a Linux virtual machine (VM) in Azure. You can install on Ubuntu or CentOS." metaCanonical="" services="virtual-machines" documentationCenter="" title="Install the LAMP Stack on a Linux virtual machine in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中的 Linux 虚拟机上安装 LAMP 堆栈

LAMP 堆栈包含以下不同元素：

-   **L**inux - 操作系统
-   **A**pache - Web 服务器
-   **M**ySQL - 数据库服务器
-   **P**HP - 编程语言

## 在 Ubuntu 上安装

你将需要安装以下程序包：

-   `apache2`
-   `mysql-server`
-   `php5`
-   `php5-mysql`
-   `libapache2-mod-auth-mysql`
-   `libapache2-mod-php5`
-   `php5-xsl`
-   `php5-gd`
-   `php-pear`

你可以将其作为单个`apt-get install` 命令运行：

    apt-get install apache2 mysql-server php5 php5-mysql libapache2-mod-auth-mysql libapache2-mod-php5 php5-xsl php5-gd php-pear

## 在 CentOS 上安装

你将需要安装以下程序包：

-   `httpd`
-   `mysql`
-   `mysql-server`
-   `php`
-   `php-mysql`

你可以将其作为单个`yum install` 命令运行：

    yum install httpd mysql mysql-server php-php-mysql

## 设置

1.  设置 **Apache**。

    1.  你将需要重新启动 Apache Web 服务器。运行以下命令：

            sudo /etc/init.d/apache2 restart

    2.  查看安装是否正在运行。使你的浏览器指向：[][]<http://localhost></a>。应该会显示“It works!”。

2.  设置 **MySQL**。

    1.  通过运行以下命令为 mysql 设置根密码

            mysqladmin -u root -p password yourpassword

    2.  使用`mysql` 或多种 MySQL 客户端登录到控制台。

3.  设置 **PHP**。

    1.  通过运行以下命令启用 Apache PHP 模块：

            sudo a2enmod php5

    2.  通过运行以下命令重新启动 Apache：

            sudo service apache2 restart

## 延伸阅读

有许多在 Ubuntu 上设置 LAMP 堆栈的资源。

-   [][1]<https://help.ubuntu.com/community/ApacheMySQLPHP></a>
-   [][2]<http://fedorasolved.org/server-solutions/lamp-stack></a>

  []: http://localhost
  [1]: https://help.ubuntu.com/community/ApacheMySQLPHP
  [2]: http://fedorasolved.org/server-solutions/lamp-stack
