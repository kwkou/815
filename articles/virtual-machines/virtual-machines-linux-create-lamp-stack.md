<!-- ARM: tested -->

<properties
	pageTitle="在 Linux 虚拟机上部署 LAMP | Azure"
	description="了解如何在 Linux VM 上安装 LAMP 堆栈"
	services="virtual-machines-linux"
	documentationCenter="virtual-machines"
	authors="jluk"
	manager="squillace"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="NA"
	ms.topic="article"
	ms.date="06/07/2016"
	wacn.date="07/18/2016"
	ms.author="jluk"/>

# 在 Azure 上部署 LAMP 堆栈
本文将指导你完成在 Azure 上部署 Apache web 服务器、MySQL 和 PHP（LAMP 堆栈）。你需要一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）和[连接到 Azure 帐户](/documentation/articles/xplat-cli-connect/)的 [Azure CLI](/documentation/articles/xplat-cli-install/)。

本文介绍了两种方法用于安装所涉及的 LAMP：

## 快速命令摘要

1) 在新的 VM 上部署 LAMP

	# One command to create a resource group holding a VM with LAMP already on it
	$ azure group create -n uniqueResourceGroup -l chinanorth --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/lamp-app/azuredeploy.json

2) 在现有 VM 上部署 LAMP

	# Two commands: one updates packages, the other installs Apache, MySQL, and PHP
	user@ubuntu$ sudo apt-get update
	user@ubuntu$ sudo apt-get install apache2 mysql-server php5 php5-mysql


## 在新的 VM 上部署 LAMP 的演练

首先创建一个新的包含 VM 的[资源组](/documentation/articles/resource-group-overview/)：

    $ azure group create uniqueResourceGroup chinanorth
    info:    Executing command group create
    info:    Getting resource group uniqueResourceGroup
    info:    Creating resource group uniqueResourceGroup
    info:    Created resource group uniqueResourceGroup
    data:    Id:                  /subscriptions/########-####-####-####-############/resourceGroups/uniqueResourceGroup
    data:    Name:                uniqueResourceGroup
    data:    Location:            chinanorth
    data:    Provisioning State:  Succeeded
    data:    Tags: null
    data:
    info:    group create command OK

要创建 VM，可以使用在[此处的 GitHub 上](https://github.com/Azure/azure-quickstart-templates/tree/master/lamp-app)找到的已写入的 Azure Resource Manager 模板。

    $ azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/lamp-app/azuredeploy.json uniqueResourceGroup uniqueLampName

你会看到提示输入更多内容的响应：

    info:    Executing command group deployment create
    info:    Supply values for the following parameters
    storageAccountNamePrefix: lampprefix
    location: chinanorth
    adminUsername: someUsername
    adminPassword: somePassword
    mySqlPassword: somePassword
    dnsLabelPrefix: azlamptest
    info:    Initializing template configurations and parameters
    info:    Creating a deployment
    info:    Created template deployment "uniqueLampName"
    info:    Waiting for deployment to complete
    data:    DeploymentName     : uniqueLampName
    data:    ResourceGroupName  : uniqueResourceGroup
    data:    ProvisioningState  : Succeeded
    data:    Timestamp          :
    data:    Mode               : Incremental
    data:    CorrelationId      : d51bbf3c-88f1-4cf3-a8b3-942c6925f381
    data:    TemplateLink       : https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/lamp-app/azuredeploy.json
    data:    ContentVersion     : 1.0.0.0
    data:    DeploymentParameters :
    data:    Name                      Type          Value
    data:    ------------------------  ------------  -----------
    data:    storageAccountNamePrefix  String        lampprefix
    data:    location                  String        chinanorth
    data:    adminUsername             String        someUsername
    data:    adminPassword             SecureString  undefined
    data:    mySqlPassword             SecureString  undefined
    data:    dnsLabelPrefix            String        azlamptest
    data:    ubuntuOSVersion           String        14.04.2-LTS
    info:    group deployment create command OK

现在你已创建已安装 LAMP 的 Linux VM。如果你想要，可以跳转到 [验证是否已成功安装 LAMP] 来验证此安装。

## 在现有 VM 上部署 LAMP 的演练

如果你需要有关创建 Linux VM 方面的帮助，可以转到[此处以了解如何创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。 
接下来，你需要通过 SSH 登录 Linux VM。如果你需要有关创建 SSH 密钥方面的帮助，可以转到[此处了解如何在 Linux/Mac 上创建 SSH 密钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。
如果你已经有 SSH 密钥，则继续并使用 `ssh username@uniqueDNS` 通过 SSH 登录 Linux VM。

既然你正在使用 Linux VM，我们将指导你如何在基于 Debian 的分发版上安装 LAMP 堆栈。对于其他 Linux 分发版，确切的命令可能会有所不同。

#### 在 Debian/Ubuntu 上安装

你将需要安装以下程序包：`apache2`、`mysql-server`、`php5` 和 `php5-mysql`。您可以通过直接获取这些程序包或使用 Tasksel 进行安装。下面列出了这两种方法的说明。
在安装之前，需要下载并更新包列表。

    user@ubuntu$ sudo apt-get update
    
##### 单个包
使用 apt-get：

	user@ubuntu$ sudo apt-get install apache2 mysql-server php5 php5-mysql

##### 使用 Tasksel
此外，你可以下载 Tasksel，它是一个 Debian/Ubuntu 工具，可将多个相关包作为协同“任务”安装到你的系统中。

    user@ubuntu$ sudo apt-get install tasksel
    user@ubuntu$ sudo tasksel install lamp-server

运行上述任一选项以后，系统会提示你安装这些程序包以及多个其他的依赖项。按“y”再按“Enter”即可继续，然后再遵循任何其他的提示为 MySQL 设置一个管理密码。此时会安装最低要求的 PHP 扩展，这些扩展是通过 MySQL 使用 PHP 所需的。

![][1]

运行以下命令即可查看以程序包形式提供的其他 PHP 扩展：

	user@ubuntu$ apt-cache search php5


#### 创建 info.php 文档

现在你可以通过在命令行中键入 `apache2 -v`、`mysql -v` 或 `php -v` 来检查 Apache、MySQL 和 PHP 的版本。

如果你想要进行进一步的检测，可以创建在浏览器中查看的快速 PHP 信息页。通过以下命令使用 Nano 文本编辑器创建一个新的文件：

    user@ubuntu$ sudo nano /var/www/html/info.php

在 GNU Nano 文本编辑器中，添加以下行：

    <?php
    phpinfo();
    ?>

然后保存并退出文本编辑器。

使用此命令重启 Apache 以便所有新安装都生效。

    user@ubuntu$ sudo service apache2 restart

## 验证是否已成功安装 LAMP

现在你可以转到 http://youruniqueDNS/info.php 检查你刚在浏览器中创建的 PHP 信息页，如下所示。

![][2]

你可以通过转到 http://youruniqueDNS/ 并查看 Apache2 Ubuntu 默认页来检查 Apache 安装。你应看到类似如下的内容。

![][3]

恭喜，你刚刚在 Azure VM 上安装了 LAMP 堆栈！

## 后续步骤

签出 LAMP 堆栈上的 Ubuntu 文档：

- [https://help.ubuntu.com/community/ApacheMySQLPHP](https://help.ubuntu.com/community/ApacheMySQLPHP)

[1]: ./media/virtual-machines-linux-deploy-lamp-stack/configmysqlpassword-small.png
[2]: ./media/virtual-machines-linux-deploy-lamp-stack/phpsuccesspage.png
[3]: ./media/virtual-machines-linux-deploy-lamp-stack/apachesuccesspage.png

<!---HONumber=Mooncake_0711_2016-->