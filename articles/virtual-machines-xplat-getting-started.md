<properties
   pageTitle="如何使用 Azure CLI 创建 Azure 虚拟机 | Windows Azure"
   description="本主题介绍如何在任何平台上安装 Azure CLI、如何使用它连接到你的 Azure 帐户，以及如何从 Azure CLI 创建 VM。"
   services="virtual-machines"
   documentationCenter="virtual-machines"
   authors="dlepow"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="virtual-machines"
   ms.date="06/09/2015"
   wacn.date="09/18/2015"/>

# 使用 Azure 命令行界面 (Azure CLI) 创建 VM
Azure CLI 是从任何平台管理 Azure 基础结构的好办法。

仅安装 Azure CLI 并拥有 Azure 订阅将阻止你立即创建 VM，因此让我们注意这些步骤。如果你没有 Azure 帐户，请[获取免费试用版](http://azure.microsoft.com/pricing/free-trial/)。

## 安装 Azure CLI

按照说明[安装 Azure CLI](/documentation/articles/xplat-cli#install)。

## 使用 Azure CLI 连接到 Azure

你可以将 Azure CLI 安装与个人 Azure 帐户或者工作或学校 Azure 帐户关联。若要了解差别并进行选择，请参阅[如何连接到 Azure 订阅](/documentation/articles/xplat-cli#configure)。

## 在 Azure 中创建 VM 并连接到该 VM

创建 VM 从选择（或上载）映像开始，并使用 `azure vm create` 命令创建。

1. 若要从命令行中选择映像，可以使用 `azure vm image list` 命令列出可用的 VM 映像。由于有这么多映像，你将需要使用 `more` 对结果进行分页，或者使用 `grep` (Linux) 或 `findstr` (Windows) 进行筛选。例如，如果你要使用如下命令在 Linux 上查找 Ubuntu 映像：

        azure vm image list | grep Ubuntu

    这仍会生成很长的映像列表，你可以使用版本进一步缩小范围：

        azure vm image list | grep Ubuntu-14_10

    在此处你可以选择一个映像并使用 `show` 命令更详细地查看其属性：

        azure vm image show b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB

2. 选择一个 VM 映像后，使用 `vm create` 命令可创建该映像。此命令有很多选项，你可以使用 `help` 命令列出这些选项：

        vm create --help

    除了步骤 1 创建的映像外，创建 VM 所需的关键参数是位置、DNS 名称和用户名。

    对于身份验证，你可以选择指定密码（在命令行上或通过交互方式），或者使用证书进行身份验证。如果你选择密码，该密码必须至少为 8 个字符、包含大写和小写字母，并包含特殊字符（例如 !@#$%^&+= 之一）。如果要在命令行上传递密码，最好将它放入引号和转义特殊字符中。

    若要选择位置，可以使用 `vm location list` 命令来选取你附近的区域。

  你选择的 DNS 名称必须是唯一的（它将映射到 `dnsname.cloudapp.net`），并且将与计算机名称相同（如果你未在命令行上单独指定计算机名称）。

    The Linux example below creates a VM in China North, opens the default SSH port 22 (the -e argument), and creates a user called `myadminuser`:

        azure vm create -e -l "China North"  my-new-cli-vm b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB "myadminuser" "myAdm1n@passwd"

## 后续步骤

让我们来使用虚拟机做些事情。

由于上面的示例打开了默认 SSH 端口，在 VM 已启动且在运行后连接到它是很简单的。从 Linux 命令行：

    ssh myadminuser@my-new-cli-vm.chinacloudapp.cn

在 [Azure CLI 命令参考页](/documentation/articles/virtual-machines-command-line-tools)上，可以查看有关使用 Azure CLI 管理 Azure 基础结构的更多示例。

<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png

<!---HONumber=70-->