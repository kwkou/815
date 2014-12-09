<properties title="Download the Azure SDK for PHP" pageTitle="Download the Azure SDK for PHP" metaKeywords="" description="Learn how to download and install the Azure SDK for PHP." documentationCenter="PHP" services="" solutions="web" authors="" />

# 下载 Azure SDK for PHP

Azure SDK for PHP 包括允许你针对 Azure 开发、部署和管理 PHP 应用程序的组件。具体而言，Azure SDK for PHP 包括以下组件：

-   **Azure 的 PHP 客户端库**。这些类库提供用于访问 Azure 功能（例如数据管理服务和云服务）的接口。
-   **适用于 Mac 和 Linux 的 Azure 命令行工具**。这是一组用于部署和管理 Azure 服务（例如 Azure 网站和 Azure 虚拟机）的命令行工具。这些工具可在任何平台（包括 Mac、Linux 和 Windows）上使用。
-   **Azure PowerShell（仅限 Windows）**。这是一组用于部署和管理 Azure 服务（例如云服务和虚拟机）的 PowerShell cmdlet。
-   **Azure 模拟器（仅限 Windows）**。计算和存储模拟器是一系列云服务和数据管理服务的本地模拟器，允许你在本地测试应用程序。Azure 模拟器仅在 Windows 上运行。

以下各节将介绍如何下载和安装上述组件。

本主题中的说明假定你已安装 [PHP][PHP]。

> [WACOM.NOTE]
> 若要使用 Azure 的 PHP 客户端库，你必须安装 PHP 5.3 或更高版本。

## Azure 的 PHP 客户端库

Azure 的 PHP 客户端库提供了一个用于从任何操作系统访问 Azure 功能（例如，数据管理服务和云服务）的接口。可通过 Composer 或 PEAR 包管理器安装或手动安装这些库。

有关如何使用 Azure 的 PHP 客户端库的信息，请参阅[如何使用 Blob 服务][如何使用 Blob 服务]、[如何使用表服务][如何使用表服务]以及[如何使用队列服务][如何使用队列服务]。

### 通过 Composer 安装

1.  [安装 Git][安装 Git]。

    > [WACOM.NOTE]
    > 在 Windows 上，你还需要向你的 PATH 环境变量中添加 Git 可执行文件。

2.  在你的项目的根目录中创建一个名为 **composer.json** 的文件并向其中添加以下代码：

        {
            "require": {
                "microsoft/windowsazure": "*"
            },          
            "repositories": [
                {
                    "type": "pear",
                    "url": "http://pear.php.net"
                }
            ],
            "minimum-stability": "dev"
        }

3.  将 **[composer.phar][composer.phar]** 下载到项目根目录中。

4.  打开命令提示符并在项目根目录中执行该文件

        php composer.phar install

### 作为 PEAR 包安装

若要将 Azure 的 PHP 客户端库作为 PEAR 包安装，请执行下列步骤：

1.  [安装 PEAR][安装 PEAR]。
2.  设置 Azure PEAR 通道：

        pear channel-discover pear.windowsazure.com

3.  安装 PEAR 包：

        pear install pear.windowsazure.com/WindowsAzure-0.3.1

安装完成后，你可以从应用程序中引用类库。

### 手动安装

若要手动下载并安装用于 Azure 的 PHP 客户端库，请执行以下步骤：

1.  下载包含 [GitHub][GitHub] 中的库的 .zip 存档。或者，复制现有存储库并将其克隆到你的本地计算机。（后一种选择需要一个 GitHub 帐户并要求已在本地安装 Git。）

    > [WACOM.NOTE]
    > Azure 的 PHP 客户端库依赖于 [HTTP\_Request2][HTTP\_Request2]、[Mail\_mime][Mail\_mime] 和 [Mail\_mimeDecode][Mail\_mimeDecode] PEAR 包。若要处理这些依赖关系，建议使用 [PEAR 包管理器][PEAR 包管理器]安装这些包

2.  将已下载的存档的 `WindowsAzure` 目录复制到应用程序目录结构中并从应用程序引用类。

## Azure PowerShell 和 Azure 模拟器

Azure PowerShell 是一组用于部署和管理 Azure 服务（例如，云服务和虚拟机）的 PowerShell cmdlet。Azure 模拟器是一系列云服务和数据管理服务的模拟器，允许你在本地测试应用程序。这些组件仅受 Windows 支持。

安装 Azure PowerShell 和 Azure 模拟器的建议方法是使用 [Microsoft Web 平台安装程序][Microsoft Web 平台安装程序]。请注意，你也可以选择安装其他开发组件，如 PHP、SQL Server、Microsoft Drivers for SQL Server for PHP 和 WebMatrix。

有关如何使用 Azure PowerShell 的信息，请参阅[如何使用 Azure PowerShell][如何使用 Azure PowerShell]。

## 适用于 Mac 和 Linux 的 Azure 命令行工具

适用于 Mac 和 Linux 的 Azure 命令行工具是一组用于部署和管理 Azure 服务（例如，Azure 网站和 Azure 虚拟机）的命令行工具。以下列表说明如何根据你的操作系统安装工具：

-   **Mac**：在此处下载 Azure SDK 安装程序：[][]<http://go.microsoft.com/fwlink/?LinkId=252249></a>。打开已下载的 .pkg 文件并按照系统提示完成安装步骤。

-   **Linux**：安装最新版本的 [Node.js][Node.js]（请参阅[通过程序包管理器安装 Node.js][通过程序包管理器安装 Node.js]），然后运行以下命令：

        npm install azure-cli -g

    > [WACOM.NOTE]
    > 你可能需要使用提升的权限才能运行此命令： `sudo npm install azure-cli -g`

有关如何使用适用于 Mac 和 Linux 的 Azure 命令行工具的信息，请参阅[如何使用适用于 Mac 和 Linux 的命令行工具][如何使用适用于 Mac 和 Linux 的命令行工具]。

  [PHP]: http://www.php.net/manual/en/install.php
  [如何使用 Blob 服务]: http://go.microsoft.com/fwlink/?LinkId=252714
  [如何使用表服务]: http://go.microsoft.com/fwlink/?LinkId=252715
  [如何使用队列服务]: http://go.microsoft.com/fwlink/?LinkId=252716
  [安装 Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
  [composer.phar]: http://getcomposer.org/composer.phar
  [安装 PEAR]: http://pear.php.net/manual/en/installation.getting.php
  [GitHub]: http://go.microsoft.com/fwlink/?LinkId=252719
  [HTTP\_Request2]: http://pear.php.net/package/HTTP_Request2
  [Mail\_mime]: http://pear.php.net/package/Mail_mime
  [Mail\_mimeDecode]: http://pear.php.net/package/Mail_mimeDecode
  [PEAR 包管理器]: http://pear.php.net/manual/en/installation.php
  [Microsoft Web 平台安装程序]: http://go.microsoft.com/fwlink/?LinkId=253447
  [如何使用 Azure PowerShell]: http://go.microsoft.com/fwlink/?LinkId=252718
  []: http://go.microsoft.com/fwlink/?LinkId=252249
  [Node.js]: http://nodejs.org/
  [通过程序包管理器安装 Node.js]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager
  [如何使用适用于 Mac 和 Linux 的命令行工具]: http://go.microsoft.com/fwlink/?LinkId=252717
