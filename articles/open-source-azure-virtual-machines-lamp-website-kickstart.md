<properties
	pageTitle="在 Azure Linux 虚拟机上快速搭建单机版 LAMP 网站"
	description="了解如何在 Azure Linux 虚拟机上快速搭建单机版 LAMP 网站"
	services="open-source"
	documentationCenter=""
	authors=""
	manager=""
	editor=""/>

<tags
	ms.service="open-source-website"
	ms.date=""
	wacn.date="05/26/2017"/>

# 在 Azure Linux 虚拟机上快速搭建单机版 LAMP 网站

##目录
- [安装 LAMP](#instal-lamp)  
	- [Azure PowerShell 方式](#powershell)  
	- [Azure CLI 方式](#azure-cli)
	- [使用 SHELL 脚本方式](#shell)  
- [访问网站](#visit-site)

LAMP 通常表示 Linux + Apache + MySQL/MariaDB + Perl/PHP/Python，LAMP 的各个组件不是一成不变的，并不局限于它最初的选择。作为一个解决方案套件，LAMP 非常适合构建动态网站和网站应用程序。另外使用类似 Zabbix 这样的组件做监控也是网站必不可少的。

##<a id="instal-lamp"></a>安装 LAMP
本文档的 LAMP 代指 Linux + Apache2 + MySQL + php5 且各示例步骤基于 Azure 环境下的 LINUX 虚拟机。本文档介绍三种方式来安装LAMP：1. 使用 Azure PowerShell 脚本； 2. 使用 Azure CLI； 3. 使用 SHELL 脚本。 安装 LAMP 过程中会在虚拟机上自动安装 Zabbix agent。

说明：  
目前 Azure PowerShell 脚本和 Azure CLI 方式仅支持 CentOS(6.5, 6.6, 6.7, 7.0, 7.1, 7.2)。而 SHELL 脚本则有 CentOS, Ubuntu以及 SLES 三个版本。  

参数使用注意事项：  
DNSNamePrefix：必须小写，需保证唯一性，该参数将作为 DNS 前缀。  
ZabbixServerIPAddress：可选项，指定 Zabbix 服务器地址。 

###<a id="powershell"></a>Azure PowerShell 方式
PowerShell 脚本运行注意事项：  
需要以管理员权限运行 PowerShell，使用之前需运行如下命令：  

	Set-ExecutionPolicy -ExecutionPolicy Unrestricted

如果您选择 Azure PowerShell 方式安装 LAMP，那么请按[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 中的说明在本地计算机上安装 Azure PowerShell。然后打开 Azure PowerShell 命令提示符，通过运行以下命令并遵循提示进行 Azure 帐户的交互式登录体验，来使用[工作或学校 ID 登录](/documentation/articles/xplat-cli-connect/#use-the-log-in-method/)：

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

然后您需要创建一个 Azure 资源组 (Resource Group)，创建 Azure 虚拟机和安装LAMP都在该资源组中进行，运行以下命令创建 Azure 资源组：

	New-AzureRmResourceGroup -Name "YOUR-RESOURCE-GROUP-NAME" -Location "China East"

您需要下载 PowerShell 脚本 [single-lamp-deploy.ps1](http://msmirrors.blob.core.chinacloudapi.cn/single-lamp/single-lamp-deploy.ps1)，按照以下示例运行 single-lamp-deploy.ps1 脚本，即可在资源组 rg1 中生成一台 CentOS 虚拟机，接着会在该虚机上安装 LAMP。其中 rg1 是在之前步骤中创建的资源组名字。

	PS C:\> .\single-lamp-deploy.ps1 -ResourceGroupName rg1 -CentOSVersion 7.0 -AdminUserName azureuser -AdminPassword “YOUR-PASSWORD”  -MySqlPassword “YOUR-MYSQL-PASSWORD” -DNSNamePrefix “YOUR-DNS-PREFIX” 

创建过程大概需要 20 分钟，运行成功后会出现如下提示, 这里我们直接去到下面访问网站的步骤去进行验证。

	Deploy LAMP Server successfully.
	To veriy the lamp server deployment, following below steps:
	Open the URL  http://<YOUR-DNS-PREFIX>.chinaeast.cloudapp.chinacloudapi.cn/mysql.php to check if php can connect to MySQL, if can do some insert operation, and finally it will return the result on the web page. 
	If you refresh the webpage, will insert another record into mysql db table.
	We strongly recommend you to delete /var/www/html/mysql.php after you access the URL and see the successful result because mysql.php stores your mysql root password.
	You can delete the inserted data by executing below commands:
	mysql -uroot -p
	drop database testdb;

###<a id="azure-cli"></a>Azure CLI 方式

如果您选择 Azure CLI 方式安装 LAMP，那么请[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。然后请确保您是处于[资源管理器模式](/documentation/articles/resource-manager-deployment-model/)下，可通过运行以下命令来验证：

	azure config mode arm

现在，通过运行以下命令并遵循提示进行 Azure 帐户的交互式登录体验，来使用[工作或学校 ID 登录](/documentation/articles/xplat-cli-connect/)：

	azure login -e AzureChinaCloud -u <your account>

然后您需要创建一个 Azure 资源组 (Resource Group)，创建 Azure 虚拟机和安装 LAMP 都在该资源组中进行，运行以下命令创建 Azure 资源组：

	azure group create "YOUR-RESOURCE-GROUP-NAME" "China East"

您需要在安装好 Azure CLI 的机器上，运行如下命令下载 azuredeploy.parameters.json 参数配置文件：

	wget http://msmirrors.blob.core.chinacloudapi.cn/single-lamp/azuredeploy.parameters.json -O azuredeploy.parameters.json

接着修改 azuredeploy.parameters.json 参数配置文件:

	vi azuredeploy.parameters.json

只需修改 "adminPassword", "dnsNamePrefix" 以及 "mySqlPassword" 的值即可，其他参数值可以保持默认不变。然后运行如下命令即可安装 CentOS 虚拟机和 LAMP，创建过程大概需要 20 分钟，其中 rg1 是之前步骤中创建的资源组名字：

	$TemplateUri="http://msmirrors.blob.core.chinacloudapi.cn/single-lamp/azuredeploy.json"
	azure group deployment create rg1 DeployLAMP --template-uri $TemplateUri -e azuredeploy.parameters.json

创建完毕会有提示信息，这里我们直接去到下面访问网站的步骤去进行验证。

###<a id="shell"></a>使用 SHELL 脚本方式

如果您已经建好了 LINUX 虚拟机，就可以直接运行 SHELL 脚本来安装 LAMP.  

如果您还没有 Azure 下的 LINUX 虚拟机，请参考 [Azure Linux VM tutorial](/documentation/articles/virtual-machines-linux-quick-create-portal/). 创建 LINUX 虚拟机。  

连接到您的 LINUX 虚拟机。如果这是您第一次使用 Azure 的 LINUX 虚拟机，请参考 [Azure Linux VM tutorial](/documentation/articles/virtual-machines-linux-quick-create-portal/) 连接到虚拟机。

不同的 LINUX 发行版在安装 LAMP 时有少许的不同。请根据您的 LINUX 版本选择对应的步骤。

**Redhat base Linux**: (以 CentOS 7.0, 64-bit system, MySQL Server 5.6, apache 2.4, php5 为例)

下载SHELL脚本：

	$sudo wget http://msmirrors.blob.core.chinacloudapi.cn/single-lamp/install_single_lamp.sh

然后执行下面命令。注意：其中的 mySqlPassword 指的是您的 MySQL root 密码，请根据您的具体情况设置；insertValue 指的是您要往 MySQL 测试表中写入的值，这个值在访问 http://yourwebsite/mysql.php 会显示出来。  

比如您运行 **sudo bash install_single_lamp.sh s3cret jack** 那么 s3cret 就是您的 MySQL root 密码，jack 就是要写入 MySQL 测试表中的值。

	$sudo bash install_single_lamp.sh mySqlPassword insertValue


**Ubuntu Linux**: (以 Ubuntu 14.04, 64-bit system, MySQL 5.5, apache 2.4, php5 为例)

下载SHELL脚本：

	$sudo wget http://msmirrors.blob.core.chinacloudapi.cn/single-lamp/install_single_lamp_Ubuntu.sh

然后执行下面命令。注意：其中的 mySqlPassword 指的是您的 MySQL root 密码，请根据您的具体情况设置；insertValue 指的是您要往 MySQL 测试表中写入的值，这个值在访问 http://yourwebsite/mysql.php 会显示出来。
比如您运行 **sudo bash install_single_lamp_Ubuntu.sh s3cret jack**  那么 s3cret 就是您的 MySQL root 密码，jack 就是要写入 MySQL 测试表中的值。

	$sudo bash install_single_lamp_Ubuntu.sh mySqlPassword insertValue




**SUSE Linux**: (以 SLES 12, 64-bit system, MySQL Server 5.6, apache 2.4, php5 为例)

下载 SHELL 脚本：  

	$sudo wget http://msmirrors.blob.core.chinacloudapi.cn/single-lamp/install_single_lamp_SLES.sh

然后执行下面命令。注意：其中的 mySqlPassword 指的是您的 MySQL root 密码，请根据您的具体情况设置；insertValue 指的是您要往 MySQL测试表中写入的值，这个值在访问 http://yourwebsite/mysql.php 会显示出来。 

比如您运行 **sudo bash install_single_lamp_SLES.sh s3cret jack**  那么 s3cret 就是您的 MySQL root 密码，jack 就是要写入MySQL 测试表中的值。  

	$sudo bash install_single_lamp_SLES.sh mySqlPassword insertValue


##<a id="visit-site"></a>访问网站

不管是使用哪种方式部署的 LAMP，其验证方式大同小异，区别在于 URL 地址的不同。  

如果是 Azure PowerShell 或者 Azure CLI 方式部署 LAMP 的话，URL 地址为： 

	http://<YOUR-DNS-PREFIX>.<RESOURCE-LOCATION>.cloudapp.chinacloudapi.cn/info.php
	http://<YOUR-DNS-PREFIX>.<RESOURCE-LOCATION>.cloudapp.chinacloudapi.cn/mysql.php

其中 <YOUR-DNS-PREFIX> 是您在之前用 Azure PowerShell 或者 Azure CLI 时定义的 DNSNamePrefix 的值， 而 <RESOURCE-LOCATION> 是您在之前创建资源组时指定的 location 值。  

比如您用 Azure PowerShell 命令或者 Azure CLI 命令创建了一个位于中国东部的资源组 rg1:  

	New-AzureRmResourceGroup -Name "rg1" -Location "China East"
	azure group create "rg1" "China East"

然后您定义的 DNSNamePrefix 为 mylamptest , 则 URL 为 

	http://mylamptest.chinaeast.cloudapp.chinacloudapi.cn/info.php
	http://mylamptest.chinaeast.cloudapp.chinacloudapi.cn/mysql.php



如果是使用 SHELL 脚本的方式安装 LAMP 的话，请参考如下方式：  

打开虚拟机 80 端口。请参考[创建终结点](/documentation/articles/virtual-machines-windows-nsg-quickstart-portal/)打开端口 

URL 地址为：  

	http://yourcloudservice.chinacloudapp.cn/info.php 
	http://yourcloudservice.chinacloudapp.cn/mysql.php

其中 ‘yourcloudservice’ 是您的云服务地址，在创建虚拟机时由您定义的。也可以输入 **http://虚拟机公网IP/info.php** 访问。访问结果类似下图:

![](./media/open-source-azure-virtual-machines-lamp-website-kickstart/visit.png)
 
此图表明 PHP 已经正常安装，APACHE 软件能与 PHP 正常工作。

网址 http://yourcloudservice.chinacloudapp.cn/mysql.php 用来检测 MySQL 是否正常安装，PHP 能否正常操作数据库，对数据库的读写是否正常。结果类似下图:  

![](./media/open-source-azure-virtual-machines-lamp-website-kickstart/visit2.png)
 
此图表明数据库读写操作均正常。从输出中我们可以看到创建了数据库名为 testdb, 创建了表名为 test01,插入了一条数据，值为 jack. 这个jack 就是在执行命令 **bash install_single_lamp.sh mySqlPassword insertValue** 时的 insertValue, 这里 insertValue 被 jack 代替。如果刷新浏览器，会再次插入 jack 的值到 test01 表。

如果测试完毕想要删除这些测试数据库和表，只需在 LINUX 虚拟机上执行以下命令:  

	#mysql -uroot -p  

输入 MySQL root 密码后执行  

	drop database testdb;

即可。

> [AZURE.NOTE] 注意：我们强烈建议您在测试完毕无误后把 mysql.php 删除掉，因为它里面包含了 MySQL root 的明文密码。  

如果是 Redhat, Ubuntu Linux, mysql.php 在 /var/www/html/ 路径下，如果是 SUSE LINUX, mysql.php 在 /srv/www/htdocs/ 路径下。

如果您想使用您自己的域名作为网站的 URL，而不是用虚拟机的云服务名字的话，请参考[使用自定义域名](/documentation/articles/cloud-services-custom-domain-name/)。






