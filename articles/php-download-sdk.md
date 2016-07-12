<properties
	pageTitle="下载 Azure SDK for PHP"
	description="了解如何下载和安装 Azure SDK for PHP。"
	documentationCenter="php"
	services="app-service\web"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="04/08/2016"
	wacn.date="05/05/2016"/>

#下载 Azure SDK for PHP

## 概述

Azure SDK for PHP 包括允许你针对 Azure 开发、部署和管理 PHP 应用程序的组件。具体而言，Azure SDK for PHP 包括以下组件：

* **Azure 的 PHP 客户端库**。这些类库提供用于访问 Azure 功能（例如数据管理服务和云服务）的接口。  
* **适用于 Mac、Linux 和 Windows (Azure CLI) 的 Azure 命令行界面**。这是一组用于部署和管理 Azure 服务（例如 Azure 网站和 Azure 虚拟机）的命令。Azure CLI 可在任何平台（包括 Mac、Linux 和 Windows）上使用。
* **Azure PowerShell（仅限 Windows）**。这是一组用于部署和管理 Azure 服务（例如云服务和虚拟机）的 PowerShell cmdlet。
* **Azure 模拟器（仅限 Windows）**。计算和存储模拟器是一系列云服务和数据管理服务的本地模拟器，允许你在本地测试应用程序。Azure 模拟器仅在 Windows 上运行。

以下各节将介绍如何下载和安装上述组件。

本主题中的说明假定您已安装 [PHP][install-php]。

> [AZURE.NOTE] 若要使用 Azure 的 PHP 客户端库，您必须安装 PHP 5.3 或更高版本。

##Azure 的 PHP 客户端库

Azure 的 PHP 客户端库提供了一个用于从任何操作系统访问 Azure 功能（例如，数据管理服务和云服务）的接口。可通过 Composer 或 PEAR 包管理器安装或手动安装这些库。

有关如何使用 Azure 的 PHP 客户端库的信息，请参阅[如何使用 Blob 服务][blob-service]、[如何使用表服务][table-service]以及[如何使用队列服务][queue-service]。

###通过 Composer 安装

1. [安装 Git][install-git]。


	> [AZURE.NOTE] 在 Windows 上，您还需要向您的 PATH 环境变量添加 Git 可执行文件。

2. 在你的项目的根目录中创建一个名为 **composer.json** 的文件并向其添加以下代码：

        {
            "repositories": [
                {
                    "type": "pear",
                    "url": "http://pear.php.net"
                }
            ],
            "require": {
                "pear-pear.php.net/mail_mime" : "*",
                "pear-pear.php.net/http_request2" : "*",
                "pear-pear.php.net/mail_mimedecode" : "*",
                "microsoft/windowsazure": "*"
            }
        }

3. 将 **[composer.phar][composer-phar]** 下载到您的项目根目录中。

4. 打开命令提示符并在项目根目录中执行该文件

		php composer.phar install

###作为 PEAR 包安装

若要将 Azure 的 PHP 客户端库作为 PEAR 包安装，请执行下列步骤：

1. [安装 PEAR][install-pear]。
2. 设置 Azure PEAR 通道：

		pear channel-discover pear.windowsazure.cn
3. 安装 PEAR 包：

		pear install pear.windowsazure.cn/WindowsAzure-0.4.1

安装完成后，你可以从应用程序中引用类库。

###手动安装

若要手动下载并安装用于 Azure 的 PHP 客户端库，请执行以下步骤：

1. 下载包含 [GitHub][php-sdk-github] 中的库的 .zip 存档。或者，复制现有存储库并将其克隆到您的本地计算机。（后一种选择需要一个 GitHub 帐户并要求已在本地安装 Git。）

	> [AZURE.NOTE] 用于 Azure 的 PHP 客户端库依赖于 [HTTP\_Request2](http://pear.php.net/package/HTTP_Request2)、[Mail\_mime](http://pear.php.net/package/Mail_mime) 和 [Mail\_mimeDecode](http://pear.php.net/package/Mail_mimeDecode) PEAR 包。若要处理这些依赖关系，建议使用 [PEAR 包管理器](http://pear.php.net/manual/en/installation.php)安装这些包

2. 将已下载的存档的 `WindowsAzure` 目录复制到应用程序目录结构中并从应用程序引用类。

##Azure PowerShell 和 Azure 模拟器

Azure PowerShell 是一组用于部署和管理 Azure 服务（例如，云服务和虚拟机）的 PowerShell cmdlet。Azure 模拟器是一系列云服务和数据管理服务的模拟器，允许你在本地测试应用程序。这些组件仅受 Windows 支持。

安装 Azure PowerShell 和 Azure 模拟器的建议方法是使用 [Microsoft Web 平台安装程序][download-wpi]。请注意，你也可以选择安装其他开发组件，如 PHP、SQL Server、Microsoft Drivers for SQL Server for PHP 和 WebMatrix。

有关如何使用 Azure PowerShell 的信息，请参阅[如何使用 Azure PowerShell][powershell-tools]。

##Azure CLI

Azure CLI 是一组用于部署和管理 Azure 服务（例如 Azure 网站和 Azure 虚拟机）的命令。有关安装 Azure CLI 的信息，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。

## 后续步骤

有关详细信息，请参阅 [PHP 开发中心](/develop/php/)。


[install-php]: http://www.php.net/manual/en/install.php
[composer-github]: https://github.com/composer/composer
[composer-phar]: http://getcomposer.org/composer.phar
[pear-net]: http://pear.php.net/
[http-request2-package]: http://pear.php.net/package/HTTP_Request2
[mail-mimedecode-package]: http://pear.php.net/package/Mail_mimeDecode
[mail-mime-package]: http://pear.php.net/package/Mail_mime
[install-pear]: http://pear.php.net/manual/en/installation.getting.php
[nodejs-org]: http://nodejs.org/
[install-node-linux]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager
[download-wpi]: http://go.microsoft.com/fwlink/?LinkId=253447
[mac-installer]: http://go.microsoft.com/fwlink/?LinkId=252249
[blob-service]: /documentation/articles/storage-php-how-to-use-blobs/
[table-service]: /documentation/articles/storage-php-how-to-use-table-storage/
[queue-service]: /documentation/articles/storage-php-how-to-use-queues/
[azure cli]: /documentation/articles/xplat-cli-install/
[powershell-tools]: /documentation/articles/powershell-install-configure/
[php-sdk-github]: https://github.com/Azure/azure-sdk-for-php
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git

<!---HONumber=Mooncake_0425_2016-->