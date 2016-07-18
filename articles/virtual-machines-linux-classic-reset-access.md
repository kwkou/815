<properties
	pageTitle="从 CLI 重置 Linux VM 密码和 SSH 密钥 | Azure"
	description="如何使用 VMAccess 扩展从 Azure 命令行接口 (CLI) 重置 Linux VM 密码或 SSH 密钥、修复 SSH 配置，以及检查磁盘一致性"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/20/2016"
	wacn.date="06/13/2016"/>

# 如何使用 Linux 版 Azure VMAccess 扩展来重置访问权限、管理用户和检查磁盘


如果因为忘记密码、安全外壳 (SSH) 密钥不正确或 SSH 配置出现问题而不能连接到 Azure 上的 Linux 虚拟机，请使用 VMAccessForLinux 扩展通过 Azure CLI 重置密码或 SSH 密钥、修复 SSH 配置以及检查磁盘一致性。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理模型](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess)。




## Azure CLI

你将需要以下各项：

- Azure 命令行界面 (CLI)。你需要[安装 Azure CLI](/documentation/articles/xplat-cli-install/)，并[连接到你的订阅](/documentation/articles/xplat-cli-connect/)以使用与你帐户关联的 Azure 资源。
- 一个新密码或一组新 SSH 密钥（如果想要重置任一项）。如果想要重置 SSH 配置，则不需要这些。


## 使用 azure vm extension set 命令

借助 Azure CLI，你可以从命令行接口（Bash、终端、命令提示符）使用 **azure vm extension set** 命令访问命令。运行 **azure help vm extension set** 可了解扩展的详细用法。

借助 Azure CLI，你可以执行以下任务：

+ [重置密码](#pwresetcli)
+ [重置 SSH 密钥](#sshkeyresetcli)
+ [重置密码和 SSH 密钥](#resetbothcli)
+ [创建新的 sudo 用户帐户](#createnewsudocli)
+ [重置 SSH 配置](#sshconfigresetcli)
+ [删除用户](#deletecli)
+ [显示 VMAccess 扩展的状态](#statuscli)
+ [检查所添加磁盘的一致性](#checkdisk)
+ [修复在 Linux VM 中添加的磁盘](#repairdisk)

### <a name="pwresetcli"></a>重置密码

步骤 1：使用以下内容创建名为 PrivateConf.json 的文件，并替换占位符值。

	{
	"username":"currentusername",
	"password":"newpassword",
	"expiration":"2016-01-01"
	}

步骤 2：运行以下命令，并将虚拟机的名称替换为“vmname”。

	azure vm extension set vmname VMAccessForLinux Microsoft.OSTCExtensions 1.* --private-config-path PrivateConf.json

### <a name="sshkeyresetcli"></a>重置 SSH 密钥

步骤 1：使用以下内容创建名为 PrivateConf.json 的文件，并替换占位符值。

	{
	"username":"currentusername",
	"ssh_key":"contentofsshkey"
	}

步骤 2：运行以下命令，并将虚拟机的名称替换为“vmname”。

	azure vm extension set vmname VMAccessForLinux Microsoft.OSTCExtensions 1.* --private-config-path PrivateConf.json

### <a name="resetbothcli"></a>重置密码和 SSH 密钥

步骤 1：使用以下内容创建名为 PrivateConf.json 的文件，并替换占位符值。

	{
	"username":"currentusername",
	"ssh_key":"contentofsshkey",
	"password":"newpassword"
	}

步骤 2：运行以下命令，并将虚拟机的名称替换为“vmname”。

	azure vm extension set vmname VMAccessForLinux Microsoft.OSTCExtensions 1.* --private-config-path PrivateConf.json

### <a name="createnewsudocli"></a>创建新的 sudo 用户帐户

如果忘记了用户名，可以使用 VMAccess 创建一个具有 sudo 授权的新用户帐户。在这种情况下，不会修改现有的用户名和密码。

若要创建具有密码访问权限的新 sudo 用户，请使用“[重置密码](#pwresetcli)”中的脚本并指定新用户名。

若要创建具有 SSH 密钥访问权限的新 sudo 用户，请使用“[重置 SSH 密钥](#sshkeyresetcli)”中的脚本并指定新用户名。

你还可以使用“[重置密码和 SSH 密钥](#resetbothcli)”来创建同时具有密码和 SSH 密钥访问权限的新用户。

### <a name="sshconfigresetcli"></a>重置 SSH 配置

如果 SSH 配置处于某种意外状态，你可能会丢失对 VM 的访问权限。可以使用 VMAccess 扩展将配置重置为其默认状态。为此，只需将“reset\_ssh”键设置为“True”。该扩展将重新启动 SSH 服务器，打开 VM 上的 SSH 端口，然后将 SSH 配置重置为默认值。将不会更改用户帐户（名称、密码或 SSH 密钥）。

> [AZURE.NOTE] 重置后的 SSH 配置文件位于 /etc/ssh/sshd\_config 中。

步骤 1：使用以下内容创建一个名为 PrivateConf.json 的文件。

	{
	"reset_ssh":"True"
	}

步骤 2：运行以下命令，并将虚拟机的名称替换为“vmname”。

	azure vm extension set vmname VMAccessForLinux Microsoft.OSTCExtensions 1.* --private-config-path PrivateConf.json

### <a name="deletecli"></a>删除用户

如果想要不登录 VM 就直接删除用户帐户，可以使用此脚本。

步骤 1：使用以下内容创建名为 PrivateConf.json 的文件，并替换占位符值。

	{
	"remove_user":"usernametoremove"
	}

步骤 2：运行以下命令，并将虚拟机的名称替换为“vmname”。

	azure vm extension set vmname VMAccessForLinux Microsoft.OSTCExtensions 1.* --private-config-path PrivateConf.json

### <a name="statuscli"></a>显示 VMAccess 扩展的状态

若要显示 VMAccess 扩展的状态，请运行以下命令。

	azure vm extension get

### <a name='checkdisk'></a>检查所添加磁盘的一致性

若要在 Linux 虚拟机的所有磁盘上运行 fsck，需执行以下操作：

步骤 1：使用以下内容创建一个名为 PublicConf.json 的文件。Check Disk 采用的布尔值表示是否检查附加到虚拟机的磁盘。

    {   
    "check_disk": "true"
    }

步骤 2：运行需要执行的以下命令，替换占位符值。

   azure vm extension set vm-name VMAccessForLinux Microsoft.OSTCExtensions 1.* --public-config-path PublicConf.json

### <a name='repairdisk'></a>修复在 Linux 虚拟机中添加的磁盘

若要修复无法装入或存在装入配置错误的磁盘，请使用 VMAccess 扩展重置 Linux 虚拟机上的装入配置。

步骤 1：使用以下内容创建一个名为 PublicConf.json 的文件。

    {
    "repair_disk":"true",
    "disk_name":"yourdisk"
    }

步骤 2：运行需要执行的以下命令，替换占位符值。

    azure vm extension set vm-name VMAccessForLinux Microsoft.OSTCExtensions 1.* --public-config-path PublicConf.json



## 其他资源

* 如果你要使用 Azure PowerShell cmdlet 或 Azure Resource Manager 模板来重置密码或 SSH 密钥、修复 SSH 配置和检查磁盘一致性，请参阅 [GitHub 上的 VMAccess 扩展文档](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess)。 

* 你也可以使用 [Azure 门户预览](https://portal.azure.cn)来重置部署在经典部署模型中的 Linux VM 的密码或 SSH 密钥。目前你无法使用门户预览来针对部署在 Resource Manager 部署模型中的 Linux VM 执行上述操作。

* 有关使用适用于 Azure 虚拟机的 VM 扩展的详细信息，请参阅 [About virtual machine extensions and features（关于虚拟机扩展和功能）](/documentation/articles/virtual-machines-linux-extensions-features/)。

<!---HONumber=Mooncake_0606_2016-->