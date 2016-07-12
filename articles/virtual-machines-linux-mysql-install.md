<properties
	pageTitle="在 Linux VM 上设置 MySQL | Azure"
	description="了解如何在 Azure 中的 Linux 虚拟机（Ubuntu 或 RedHat 系列 OS）上安装 MySQL 堆栈"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/01/2016"
	wacn.date="03/28/2016"/>


#如何在 Azure 上安装 MySQL


在本文中，你将了解如何在运行 Linux 的 Azure 虚拟机上安装和配置 MySQL。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


##在虚拟机上安装 MySQL

> [AZURE.NOTE]你必须已经有一个运行 Linux 的 Azure 虚拟机，才能完成本教程。在继续操作前，请参阅 [Azure Linux VM 教程](/documentation/articles/virtual-machines-linux-quick-create-cli/)创建并设置一个 Linux VM，其中 `mysqlnode` 为 VM 名称，`azureuser` 为用户。

在此情况下，请使用 3306 端口作为 MySQL 端口。

通过 putty 连接到你创建的 Linux VM。如果这是首次使用 Azure Linux VM，请参阅[此处](/documentation/articles/virtual-machines-linux-ssh-from-linux/)了解如何使用 putty 连接到 Linux VM。

我们将使用存储库包来安装 MySQL5.6，作为本文中的示例。实际上，MySQL5.6 在性能上相对于 MySQL5.5 而言有更大的改进。[此处](http://www.mysqlperformanceblog.com/2013/02/18/is-mysql-5-6-slower-than-mysql-5-5/)提供更多信息。


###如何在 Ubuntu 上安装 MySQL5.6
我们在这里会将 Linux VM 与 Azure 中的 Ubuntu 一起使用。

- 步骤 1：为 `root` 用户安装 MySQL Server 5.6 开关：

            #[azureuser@mysqlnode:~]sudo su -

    安装 mysql-server 5.6：

            #[root@mysqlnode ~]# apt-get update
            #[root@mysqlnode ~]# apt-get -y install mysql-server-5.6

    在安装过程中，你会看到如下所示的对话窗口弹出，要求你设置 MySQL 根密码。你需要在此处设置该密码。

    ![图像](./media/virtual-machines-linux-mysql-install/virtual-machines-linux-install-mysql-p01.png)


    再次输入密码进行确认。

    ![图像](./media/virtual-machines-linux-mysql-install/virtual-machines-linux-install-mysql-p02.png)

- 步骤 2：登录 MySQL Server

    在 MySQL Server 安装完成后，将自动启动 MySQL 服务。你可以使用 `root` 用户登录 MySQL Server。使用以下命令登录并输入密码。

             #[root@mysqlnode ~]# mysql -uroot -p

- 步骤 3：管理正在运行的 MySQL 服务

    (a) 获取 MySQL 服务的状态

             #[root@mysqlnode ~]# service mysql status

    (b) 启动 MySQL 服务

             #[root@mysqlnode ~]# service mysql start

    (c) 停止 MySQL 服务

             #[root@mysqlnode ~]# service mysql stop

    (d) 重新启动 MySQL 服务

             #[root@mysqlnode ~]# service mysql restart


###如何在 Red Hat OS 系列（例如 CentOS、Oracle Linux）上安装 MySQL
在这里，我们会将 Linux VM 用于 CentOS 或 Oracle Linux。

- 步骤 1：将 MySQL Yum 存储库开关添加到 `root` 用户：

            #[azureuser@mysqlnode:~]sudo su -

    下载并安装 MySQL 发行包：

            #[root@mysqlnode ~]# wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
            #[root@mysqlnode ~]# yum localinstall -y mysql-community-release-el6-5.noarch.rpm

- 步骤 2：编辑以下文件，以便允许 MySQL 存储库下载 MySQL5.6 程序包。

            #[root@mysqlnode ~]# vim /etc/yum.repos.d/mysql-community.repo

    将此文件的每个值更新为下面的值：

        # *Enable to use MySQL 5.6*

        [mysql56-community]
        name=MySQL 5.6 Community Server

        baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/

        enabled=1

        gpgcheck=1

        gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

- 第 3 步：从 MySQL 存储库的“安装 MySQL”安装 MySQL：

           #[root@mysqlnode ~]#yum install mysql-community-server

    将安装 MySQL RPM 程序包和所有相关的程序包。

- 步骤 4：管理正在运行的 MySQL 服务

    (a) 检查 MySQL Server 的服务状态：

           #[root@mysqlnode ~]#service mysqld status

    (b) 检查 MySQL Server 的默认端口是否正在运行：

           #[root@mysqlnode ~]#netstat  –tunlp|grep 3306


    (c) 启动 MySQL Server：

           #[root@mysqlnode ~]#service mysqld start

    (d) 停止 MySQL Server：

           #[root@mysqlnode ~]#service mysqld stop

    (e) 将 MySQL 设置为在系统启动时启动：

           #[root@mysqlnode ~]#chkconfig mysqld on


###如何在 SUSE Linux 上安装 MySQL
我们在这里会通过 OpenSUSE 来使用 Linux VM。

- 步骤 1：下载并安装 MySQL Server

    通过以下命令切换到 `root` 用户：

           #sudo su -

    下载并安装 MySQL 程序包：

           #[root@mysqlnode ~]# zypper update

           #[root@mysqlnode ~]# zypper install mysql-server mysql-devel mysql

- 步骤 2：管理正在运行的 MySQL 服务

    (a) 检查 MySQL Server 的状态：

           #[root@mysqlnode ~]# rcmysql status

    (b) 检查是否为 MySQL Server 的默认端口：

           #[root@mysqlnode ~]# netstat  –tunlp|grep 3306


    (c) 启动 MySQL Server：

           #[root@mysqlnode ~]# rcmysql start

    (d) 停止 MySQL Server：

           #[root@mysqlnode ~]# rcmysql stop

    (e) 将 MySQL 设置为在系统启动时启动：

           #[root@mysqlnode ~]# insserv mysql

###后续步骤
在[此处](https://www.mysql.com/)查找更多有关 MySQL 的使用方法和信息。

<!---HONumber=Mooncake_1207_2015-->