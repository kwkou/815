<properties
	pageTitle="如何在 Azure 上安装 MySQL "
	description="了解如何在 Azure 中的 Linux 虚拟机 (VM) 上安装 MySQL 堆栈。可以在 Ubuntu 或 CentOS 上进行安装。"
	services="virtual-machines"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/12/2015"
    wacn.date="05/15/2015"
	ms.author="mingzhan"/>


# 如何在 Azure 上安装 MySQL


在本主题中，tt 假定读者已拥有 Azure 帐户。如果没有，建议通过访问 [Azure](http://www.windowsazure.cn) 进行注册。



## 在 Microsoft Azure 中创建 VM 映像。
这里我们将从 Microsoft Azure 管理门户创建新的 VM。
### 生成"SSH 身份验证密钥"
我们将需要 SSH 密钥才能访问 Azure 门户。 


- 从[此处](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)下载并安装 puttygen。 
- 运行 puttygen.exe。
- 单击"生成"按钮以生成密钥。


 ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p01.png)
 
- 生成密钥后，它将显示如下。 
 
 ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p02.png)

- 复制公钥，并将它保存到名为"publicKey.key"的文件中。不要只单击"保存公钥"按钮，因为保存的公钥的格式不同于我们想要的公钥。
- 单击"保存私钥"按钮，并将其保存为"privateKey.ppk"。 

### 登录 Azure 门户

转到   https://manage.windowsazure.cn 并登录。

### 创建 Linux VM

单击左下角的"新建"，按照下面的说明创建映像，根据你的需求选择 Linux 映像。在这里我选择 Ubuntu 14.04 作为示例。 

  ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p03.png)

### 设置主机名

对于"主机名"：这是可用于访问虚拟机的 URL。你只需指定 DNS 名称（例如"mysqlnode1"），Azure 将 URL 生成为"mysqlnode1.chinacloudapp.cn"；对于"SSH 身份验证密钥"：这是由 puttygen 生成的公钥，需要从文件"publicKey.key"的内容中复制它。

  ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p04.png)
  

## 打开 MySQL 默认端口
Microsoft Azure 中的终结点包含协议以及公用和专用端口。专用端口是服务侦听本地计算机的端口。公用端口是服务在外部侦听的端口。
端口 3306 是 MySQL 将侦听的默认端口号。
单击"浏览"⇒"虚拟机"，然后单击你创建的映像。
 
   ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p05.png)


## 连接到你创建的映像
你可以选择用于连接到虚拟机的任何 SSH 工具。在此处我们使用 Putty 作为示例。
 

- 首先下载 Putty，Putty 的下载 URL 在这里。
- 下载 Putty 后，单击可执行文件"PUTTY.EXE"。按如下所示设置。


     "主机名(或 IP 地址)"是创建映像时作为"DNS 名称"的 URL。
     
     "端口"我们可以选择 22。这是 SSH 服务的默认端口。

   ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p06.png)
 
- 在选择"打开"之前，请单击"连接"> SSH >"身份验证"选项卡以浏览你的文件，该文件由 Puttygen 生成并包含私钥。在下面的屏幕快照中查看要填充的字段。

   ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p07.png)
 
- 然后单击"打开"，一个消息框可能会向你发出警报，指出你连接的计算机可能不是你要连接的计算机。如果你已正确配置 DNS 名称和端口号，请单击"是"。
  
 ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p08.png)

- 然后，你将看到如下内容。 
 
 ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p09.png)


## 在虚拟机上安装 MySQL
MySQL 支持三种安装方法：二进制文件包、rpm 包和源包。
出于性能考虑，我们将使用 MySQL 5.6 的 rpm 包，作为本文的示例。相对 MySQL5.5 而言，MySQL5.6 性能已显著改进。更多信息可在[此处](http://www.mysqlperformanceblog.com/2013/02/18/is-mysql-5-6-slower-than-mysql-5-5/)找到。


### 如何在 Ubuntu 或 Debian 上安装 MySQL5.6
我们将使用 Ubuntu 14.04 LTS 作为本文的示例。 

- 步骤 1：安装 MySQL Server 5.6

    使用 apt-get 命令安装 mysql-server 5.6

              # azureuser@mysqlnode1:~$ sudo apt-get update
              # azureuser@mysqlnode1:~$ sudo apt-get -y install mysql-server-5.6

    在安装期间，你将看到一个弹出对话框，要求你设置 MySQL 根密码。你将需要指定新的 MySQL 用户根密码。
    下面是屏幕截图。

 ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p10.png)

    按照要求再次确认密码。

 ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p11.png)
 
- 步骤 2：登录到 MySQL Server

    在 MySQL Server 安装完成后，自动启动 MySQL 服务。你可以使用用户 root 登录 MySQL Server。
    若要登录 MySQL Server，请使用以下命令。它将要求输入在 MySQL Server 安装过程中设置的 mysql 根密码。

             # azureuser@mysqlnode1:~$ mysql -uroot -p

- 步骤 3：在 VM 上查看 MySQL 服务
    
    登录之后，请确保 MySQL 服务正在运行，你可以使用以下命令启动/重新启动该服务。

    (a) 获取 MySQL 服务的状态

             #sudo service mysql status

    (b) 启动 MySQL 服务

             #sudo service mysql start

    (c) 停止 MySQL 服务

             #sudo service mysql stop

    (d) 重新启动 MySQL 服务

             #sudo service mysql restart


### 如何在 Redhat OS 系列或 Oracle Linux 上安装 MySQL
- 步骤 1：添加 MySQL Yum 存储库
    若要获取根权限，请运行命令： 

            #sudo su -
            #[root@azureuser ~]# wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm 
            #[root@azureuser ~]# yum localinstall -y mysql-community-release-el6-5.noarch.rpm 

- 步骤 2：选择一个版本系列
 
            #[root@azureuser ~]# vim /etc/yum.repos.d/mysql-community.repo

    这是文件中版本系列的子存储库的典型条目：

        \# *Enable to use MySQL 5.6*

        [mysql56-community]
        name=MySQL 5.6 Community Server

        baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/

        enabled=1

        gpgcheck=1

        gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

- 步骤 3：使用 Yum 安装 MySQL
    使用以下命令安装 MySQL：

           #[root@azureuser ~]#yum install mysql-community-server 

    这将安装 MySQL Server 包，以及其他所需的包。

- 步骤 4：查看 MySQL 运行状态

    你可以使用以下命令查看 MySQL Server 的状态：
   
           #[root@azureuser ~]#service mysqld status

    你可以检查 MySQL Server 的默认端口是否正在运行：

           #[root@azureuser ~]#netstat  -tunlp|grep 3306

- 步骤 5：启动和停止 MySQL Server

    使用以下命令启动 MySQL Server：

           #[root@azureuser ~]#service mysqld start

    使用以下命令停止 MySQL Server：

           #[root@azureuser ~]#service mysqld stop

    若要将 MySQL 设置为在系统启动时启动，请执行以下命令：

           #[root@azureuser ~]#chkconfig mysqld on


### 如何在 Suse Linux 上安装 MySQL

- 步骤 1：安装 MySQL Server

    若要提升权限，请运行命令： 

           #sudo su -

    使用以下命令安装 MySQL：

           #mysql-test:~ # zypper update

           #mysql-test:~ # zypper install mysql-server mysql-devel mysql

- 步骤 2：查看 MySQL 运行状态

    你可以使用以下命令查看 MySQL Server 的状态：

           #mysql-test:~ # rcmysql status

    你可以检查 MySQL Server 的默认端口是否正在运行；

           #mysql-test:~ # netstat  -tunlp|grep 3306

- 步骤 3：启动和停止 MySQL Server

    使用以下命令启动 MySQL Server：

           #mysql-test:~ # rcmysql start

    使用以下命令停止 MySQL Server：

           #mysql-test:~ # rcmysql stop

    若要将 MySQL 设置为在系统启动时启动，请执行以下命令：

           #mysql-test:~ # insserv mysql

<!--HONumber=53-->