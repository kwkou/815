<properties
	pageTitle="通过 Azure CLI 重置 Linux VM 密码并添加用户 | Azure"
	description="如何从 Azure 管理门户或 CLI 使用 VMAccess 扩展重置 Linux VM 密码、SSH 密钥和 SSH 配置，添加或删除用户帐户，以及检查磁盘一致性。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="12/15/2015"
	wacn.date="03/21/2016"/>

# 如何使用 Linux 版 Azure VMAccess 扩展来重置访问权限、管理用户和检查磁盘#

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]


如果因为忘记密码、安全外壳 (SSH) 密钥不正确或 SSH 配置出现问题而不能连接到 Azure 上的 Linux 虚拟机，请使用 Azure 管理门户或 VMAccessForLinux 扩展通过 Azure CLI 重置密码或 SSH 密钥、修复 SSH 配置以及检查磁盘一致性。


## Azure CLI 和 PowerShell

你将需要以下各项：

- Azure Linux 代理 2.0.5 版或更高版本。虚拟机库中的大多数 Linux 映像都包含版本 2.0.5。若要了解所安装的版本，请运行 **waagent -version**。若要更新代理，请按照 [Azure Linux 代理用户指南]中的说明操作。
- Azure 命令行界面 (CLI)。有关设置 Azure CLI 的详细信息，请参阅[安装和配置 Azure 命令行界面](/documentation/articles/xplat-cli-install)。
- Azure PowerShell。你将使用 Set-AzureVMExtension cmdlet 中的命令自动加载和配置 VMAccessForLinux 扩展。有关设置 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell]。
- 一个新密码或一组新 SSH 密钥（如果想要重置任一项）。如果想要重置 SSH 配置，则不需要这些。

### 无需安装

VMAccess 扩展无需安装就可使用它。只要在虚拟机上安装了 Linux 代理，就会在你运行使用 **Set-AzureVMExtension** cmdlet 的 Azure PowerShell 命令时自动加载该扩展。

## 使用 Azure CLI

借助 Azure CLI，你将可以从命令行界面（Bash、终端、命令提示符）使用 **azure** 命令访问命令。运行 azure vm extension set -help 了解详细的扩展使用情况。

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

若要创建具有密码访问权限的新 sudo 用户，请使用[“重置密码”](#pwresetcli)中的脚本并指定新用户名。

若要创建具有 SSH 密钥访问权限的新 sudo 用户，请使用[“重置 SSH 密钥”](#sshkeyresetcli)中的脚本并指定新用户名。

你还可以使用[“重置密码和 SSH 密钥”](#resetbothcli)来创建同时具有密码和 SSH 密钥访问权限的新用户。

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

## 使用 Azure PowerShell

你将使用 **Set-AzureVMExtension** cmdlet 执行 VMAccess 允许执行的任何更改。在所有情况下，首先使用云服务名称和虚拟机名称来获取虚拟机对象并将其存储在变量中。

填写云服务和虚拟机名称，然后在管理员级别的 Azure PowerShell 命令提示符下运行以下命令。替换引号内的所有内容，包括 < and > 字符。

	$CSName = "<cloud service name>"
	$VMName = "<virtual machine name>"
	$vm = Get-AzureVM -ServiceName $CSName -Name $VMName

如果你不知道云服务和虚拟机名称，运行 **Get-AzureVM** 可显示当前订阅中所有 VM 的该信息。


> [AZURE.NOTE] 以 $ 开头的命令行用于设置 PowerShell 变量，这些变量随后用于 PowerShell 命令。

如果你使用 Azure 管理门户创建了虚拟机，请运行以下附加命令：

	$vm.GetInstance().ProvisionGuestAgent = $true

此命令可防止在以下节运行 Set-AzureVMExtension 命令时出现“在设置 IaaS VM Access 扩展前，必须对 VM 对象启用预配来宾代理”错误。

接下来，可以执行以下任务：

+ [重置密码](#password)
+ [重置 SSH 密钥](#SSHkey)
+ [重置密码和 SSH 密钥](#both)
+ [重置 SSH 配置](#config)
+ [删除用户](#delete)
+ [显示 VMAccess 扩展的状态](#status)
+ [检查所添加磁盘的一致性](#checkdisk)
+ [修复在 Linux VM 中添加的磁盘](#repairdisk)

### <a name="password"></a>重置密码

填写当前的 Linux 用户名和新密码，然后运行以下命令。

	$UserName = "<current Linux account name>"
	$Password = "<new password>"
	$PrivateConfig = '{"username":"' + $UserName + '", "password": "' +  $Password + '"}'
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version =  "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM $vm -Publisher $Publisher -Version $Version -PrivateConfiguration $PrivateConfig | Update-AzureVM

> [AZURE.NOTE] 如果想要重置现有用户帐户的密码或 SSH 密钥，务必键入准确的用户名。如果键入其他名称，VMAccess 扩展将创建一个新的用户帐户并将密码分配给该帐户。


### <a name="SSHkey"></a>重置 SSH 密钥

填写当前的 Linux 用户名和包含 SSH 密钥的证书的路径，然后运行以下命令。

	$UserName = "<current Linux user name>"
	$Cert = Get-Content "<certificate path>"
	$PrivateConfig = '{"username":"' + $UserName + '", "ssh_key":"' + $cert + '"}'
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version =  "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM  $vm -Publisher $Publisher -Version $Version -PrivateConfiguration $PrivateConfig | Update-AzureVM

### <a name="both"></a>重置密码和 SSH 密钥

填写当前的 Linux 用户名、新密码以及包含 SSH 密钥的证书的路径，然后运行以下命令。

	$UserName = "<current Linux user name>"
	$Password = "<new password>"
	$Cert = Get-Content "<certificate path>"
	$PrivateConfig = '{"username":"' + $UserName + '", "password": "' +  $Password + '", "ssh_key":"' + $cert + '"}'
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version =  "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM  $vm -Publisher $Publisher -Version $Version -PrivateConfiguration $PrivateConfig | Update-AzureVM

### <a name="config"></a>重置 SSH 配置

SSH 配置中的错误可能会导致你无法访问虚拟机。可以通过将 SSH 配置重置为其默认状态来解决该问题。此操作会删除配置中的所有新访问参数，比如用户名、密码和 SSH 密钥，但不会更改用户帐户的密码或 SSH 密钥。扩展将重新启动 SSH 服务器，打开虚拟机上的 SSH 端口，然后将 SSH 配置重置为默认值。

运行以下命令。

	$PrivateConfig = '{"reset_ssh": "True"}'
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version = "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM  $vm -Publisher $Publisher -Version $Version -PrivateConfiguration $PrivateConfig | Update-AzureVM

> [AZURE.NOTE] SSH 配置文件位于 /etc/ssh/sshd\_config 中。

### <a name="delete"></a> 删除用户

填写要删除的 Linux 用户名，然后运行以下命令。

	$UserName = "<Linux user name to delete>"
	$PrivateConfig = "{"remove_user": "' + $UserName + '"}"
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version = "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM $vm -Publisher $Publisher -Version $Version -PrivateConfiguration $PrivateConfig | Update-AzureVM


### <a name="status"></a>显示 VMAccess 扩展的状态

若要显示 VMAccess 扩展的状态，请运行以下命令。

	$vm.GuestAgentStatus

### <a name="checkdisk"></a>检查所添加磁盘的一致性

若要通过 fsck 实用程序检查磁盘的一致性，请运行以下命令。

	$PublicConfig = "{"check_disk": "true"}"
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version = "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM $vm -Publisher $Publisher -Version $Version -PublicConfiguration $PublicConfig | Update-AzureVM

### <a name="checkdisk"></a>修复在 Linux VM 中添加的磁盘

若要通过 fsck 实用程序修复磁盘，请运行以下命令。

	$PublicConfig = "{"repair_disk": "true", "disk_name": "my_disk"}"
	$ExtensionName = "VMAccessForLinux"
	$Publisher = "Microsoft.OSTCExtensions"
	$Version = "1.*"
	Set-AzureVMExtension -ExtensionName $ExtensionName -VM $vm -Publisher $Publisher -Version $Version -PublicConfiguration $PublicConfig | Update-AzureVM

## 其他资源

[Azure VM 扩展和功能][]

[使用 RDP 或 SSH 连接到 Azure 虚拟机][]


<!--Link references-->
[Azure Linux 代理用户指南]: /documentation/articles/virtual-machines-linux-agent-user-guide
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure
[Azure VM 扩展和功能]: /documentation/articles/virtual-machines-extensions-features
[使用 RDP 或 SSH 连接到 Azure 虚拟机]: /documentation/articles/virtual-machines-about

<!---HONumber=Mooncake_0314_2016-->