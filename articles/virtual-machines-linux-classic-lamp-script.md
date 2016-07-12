<properties
	pageTitle="使用 Azure CustomScript Extension 部署 Linux 应用程序"
	description="了解如何使用 Azure CustomScript 扩展在 Linux 虚拟机上部署应用程序"
	editor="tysonn"
	manager="timlt"
	documentationCenter=""
	services="virtual-machines-linux"
	authors="gbowerman"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/23/2015"
	wacn.date="04/07/2016"/>

#使用适用于 Linux 的 Azure CustomScript 扩展部署 LAMP 应用程序#

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

适用于 Linux 的 Azure CustomScript 扩展提供了一种方式来通过运行以 VM 支持的任何脚本语言（例如，Python、Bash 等）编写的任意代码来自定义你的虚拟机 (VM)。这提供了一种非常灵活的方式来在多台计算机上自动执行应用程序部署。

你可以使用 Azure 经典管理门户、PowerShell 或 Azure 命令行界面 (Azure CLI) 来部署 CustomScript 扩展。

在本例中，我们将演练使用 Azure CLI 将一个简单的 LAMP 应用程序部署到 Ubuntu。

## 先决条件

对于此演练，请创建两个运行 Ubuntu 14.04 的 Azure VM。此处，我将它们称作 *script-vm* 和 *lamp-vm*。在尝试此操作时请使用唯一的名称。一个将用于运行 CLI 命令，另一个用来部署 LAMP 应用程序。

你还可能需要 Azure 存储帐户和密钥（可以从经典管理门户来获取此信息）来访问它。

虽然特定的安装命令将采用 Ubuntu，但你可以针对任何受支持的发行版改编一般步骤。

*script-vm* VM 需要使用与 Azure 之间的有效链接安装 Azure CLI。有关这方面的帮助，请参阅[安装和配置 Azure 命令行界面](/documentation/articles/xplat-cli-install/)。

## 上载脚本

在本例中，你应当在远程 VM 上执行脚本来安装 LAMP 包并配置一个 PHP 包。为了能够从任何位置访问该脚本，我们将其上载为 Azure blob。

**脚本**

此脚本将 LAMP 堆栈安装到 Ubuntu（包括设置 MySQL 的无提示安装类）、编写简单的 PHP 文件并启动 Apache：

	#!/bin/bash
	# set up a silent install of MySQL
	dbpass="mySQLPassw0rd"

	export DEBIAN_FRONTEND=noninteractive
	echo mysql-server-5.6 mysql-server/root_password password $dbpass | debconf-set-selections
	echo mysql-server-5.6 mysql-server/root_password_again password $dbpass | debconf-set-selections

	# install the LAMP stack
	apt-get -y install apache2 mysql-server php5 php5-mysql  

	# write some PHP
	echo <center><h1>My Demo App</h1><br/></center> > /var/www/html/phpinfo.php
	echo <\?php phpinfo()\; \?> >> /var/www/html/phpinfo.php

	# restart Apache
	apachectl restart

**上载**

将脚本另存为文本文件，例如 *lamp\_install.sh*，然后将其上载到 Azure 存储空间。你可以使用 Azure CLI 轻松执行此操作。以下示例将文件上载到名为“scripts”的存储容器。注意：如果该容器不存在，你需要先创建它。

    azure storage blob upload -a <yourStorageAccountName> -k <yourStorageKey> --container scripts ./lamp_install.sh

还要创建一个描述如何从 Azure 存储下载脚本的 JSON 文件。将该文件另存为 *public\_config.json*（使用你的存储帐户的名称替换“mystorage”）：

    {"fileUris":["https://mystorage.blob.core.chinacloudapi.cn/scripts/lamp_install.sh"], "commandToExecute":"sudo sh install_lamp.sh" }


## 部署扩展

现在，我们已准备好使用 Azure CLI 将 Linux CustomScript 扩展部署到远程 VM：

    azure vm extension set -c "./public_config.json" lamp-vm CustomScriptForLinux Microsoft.OSTCExtensions 1.*

这将在名为 *lamp-vm* 的 VM 上下载并执行 *lamp\_install.sh* 脚本。

因为应用程序包括一个 web 服务器，请记得在远程 VM 上打开 HTTP 侦听端口：

    azure vm endpoint create -n Apache -o tcp lamp-vm 80 80

## 监视和故障排除

你可以通过查看远程 VM 上的日志文件来检查自定义脚本的执行进度。对 *lamp-vm* 使用 SSH 并对日志文件执行 tail 命令：

    cd /var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.3.0.0/
    tail -f extension.log

当 CustomScript 扩展完成执行后，你可以浏览找到你创建的 PHP 页面，在本例中将是：*http://lamp-vm.chinacloudapp.cn/phpinfo.php*。

## 其他资源

你可以使用相同的基本步骤来部署较复杂的应用程序。在本例中，安装脚本已保存为 Azure 存储中的公共 blob。比较安全的选择是使用[安全访问签名](https://msdn.microsoft.com/zh-cn/library/azure/ee395415.aspx) (SAS) 将安装脚本存储为安全 blob。

下面是针对 Azure CLI、Linux 和 CustomScript 扩展的一些其他资源：

[使用 CustomScript 扩展自动执行 Linux VM 自定义任务](http://azure.microsoft.com/blog/2014/08/20/automate-linux-vm-customization-tasks-using-customscript-extension/)

[Azure Linux 扩展 (GitHub)](https://github.com/Azure/azure-linux-extensions)

[Azure 上的 Linux 和开源计算](/documentation/articles/virtual-machines-linux-opensource-links/)

<!---HONumber=70-->