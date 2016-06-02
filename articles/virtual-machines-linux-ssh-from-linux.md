<properties 
	pageTitle="在 Linux 和 Mac 上使用 SSH | Azure" 
	description="在 Linux 和 Mac 上为 Azure 上的资源管理器和经典部署模型生成和使用 SSH 密钥。" 
	services="virtual-machines-linux" 
	documentationCenter="" 
	authors="squillace" 
	manager="timlt" 
	editor=""
	tags="azure-service-management,azure-resource-manager" />

<tags 
	ms.service="virtual-machines-linux"
	ms.date="12/15/2015" 
	wacn.date="05/24/2016"/>

#如何在 Azure 上将 SSH 用于 Linux 和 Mac

> [AZURE.SELECTOR]
- [Windows](/documentation/articles/virtual-machines-linux-ssh-from-windows)
- [Linux/Mac](/documentation/articles/virtual-machines-linux-ssh-from-linux)

本主题介绍如何在 Linux 和 Mac 上使用 **ssh-keygen** 和 **openssl**，创建和使用 **ssh-rsa** 格式和 **.pem** 格式文件来基于 Linux 保护与 Azure VM 的通信。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

想要为使用 Windows 计算机安全地链接 Azure 中的 Linux VM 创建这类型的文件，请参阅[在 Windows 使用 SSH](virtual-machines-linux-ssh-from-windows)。

## 你需要哪些文件？

Azure 的基本 SSH 设置包括 2048 位的 **ssh-rsa** 公钥和私钥对（默认情况下，**ssh-keygen** 会将这些文件存储为 **~/.ssh/id\_rsa** 和 **~/.ssh/id-rsa.pub**，除非更改默认值）以及从 **id\_rsa** 私钥文件生成的 `.pem` 文件，以供与管理门户的经典部署模型一起使用。

以下是部署方案，你在每个方案中使用的文件类型为：

1. 无论是哪种部署模型，使用[预览门户](https://manage.windowsazure.cn)的所有部署都必需使用 **ssh-rsa** 密钥。
2. 使用[管理门户](https://manage.windowsazure.cn)创建 VM 时必需使用 .pem 文件。使用 [Azure CLI](/documentation/articles/xplat-cli-install) 的经典部署也支持 .pem 文件。 

## 创建密钥以用于 SSH

如果你已有SSH 密钥，在创建 Azure VM 时，请传递公钥文件。

如果你需要创建这些文件：

1. 请确保 **ssh-keygen** 和 **openssl** 实现是最新的。这因平台而异。 

	- 对于 Mac，请务必访问 [Apple 产品安全性网站](https://support.apple.com/HT201222)，并选择适当的更新（如有必要）。
	- 对于基于 Debian 的 Linux 分发（如 Ubuntu、Debian、Mint 等）：

			sudo apt-get install --upgrade-only openssl

	- 对于基于 RPM 的 Linux 分发（如 CentOS 和 Oracle Linux）：

			sudo yum update openssl

	- 对于 SLES 和 OpenSUSE

			sudo zypper update openssl

2. 使用 **ssh-keygen** 创建 2048 位的 RSA 公钥和私钥文件，除非你为文件设置了特定位置或特定名称，否则接受默认位置和名称（即，`~/.ssh/id_rsa`）。基本命令是：

		ssh-keygen -t rsa -b 2048 

	通常情况下，**ssh-keygen** 实现将添加注释，通常为计算机的用户名和主机名。你可以使用 `-C` 选项指定特定注释。

3. 从 `~/.ssh/id_rsa` 文件创建 .pem 文件，使你能够使用Azure 门户。使用 **openssl**，如下所示：

		openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myCert.pem

	如果要从不同的私钥文件创建 .pem 文件，请修改 `-key` 参数。

> [AZURE.NOTE]如果你计划管理使用经典部署模型部署的服务，则可能还要创建 **.cer** 格式的文件来上载到门户，尽管这不涉及 **ssh** 或连接到 Linux VM，但这是本文的主题。若要在 Linux 或 Mac 上创建这些文件，请键入：<br /> openssl  x509 -outform der -in myCert.pem -out myCert.cer

将 .pem 文件转换为 DER 编码的 X509 证书文件。

## 使用已有的 SSH 密钥

如果你需要使用管理门户，可能需要基于密钥创建 `.pem` 文件。

## 使用公钥文件创建 VM

创建所需的文件后，有许多方式可创建一个 VM，你可以使用公钥/私钥交换安全地连接到它。几乎在所有情况下，当系统提示输入 ssh 密钥文件路径或字符串形式的文件内容时，可传递 .pub 文件。

### 示例：使用 .pem 文件创建 VM

然后可以在管理门户或经典部署模式和 `azure vm create` 中使用 .pem 文件，如以下示例所示：

	azure vm create \
	-l "China North" -n testpemasm \
	-P -t myCert.pem -e 22 \
	testpemasm \
	b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20160406-en-us-30GB \
	ops
	info:    Executing command vm create
	warn:    --vm-size has not been specified. Defaulting to "Small".
	+ Looking up image b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20160406-en-us-30GB
	+ Looking up cloud service
	info:    cloud service testpemasm not found.
	+ Creating cloud service
	+ Retrieving storage accounts
	+ Configuring certificate
	+ Creating VM
	info:    vm create command OK



## 连接到 VM

**ssh** 命令使用用户名、计算机的网络地址、连接到该地址的端口以及许多其他特殊变体来登录。（有关 **ssh** 的详细信息，可参阅这篇[有关 Secure Shell 的文章](https://en.wikipedia.org/wiki/Secure_Shell)）

如果你要连接到经典部署云服务，则要使用的地址可能如下所示：

	ssh user@subdomain.chinacloudapp.cn -p 22

由于地址形式可能会更改（你始终可以使用 IP 地址或者也许你已分配自定义域名），你需要发现 Azure VM 的地址。

### 使用经典部署发现 Azure VM SSH 地址

你可以通过使用包含 VM 名称的 `azure vm show` 命令来发现要用于 VM 和经典部署模型的地址：

	azure vm show testpemasm
	info:    Executing command vm show
	+ Getting virtual machines
	data:    DNSName "testpemasm.chinacloudapp.cn"
	data:    Location "China East"
	data:    VMName "testpemasm"
	data:    IPAddress "100.116.160.154"
	data:    InstanceStatus "ReadyRole"
	data:    InstanceSize "Small"
	data:    Image "b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20150908-en-us-30GB"
	data:    OSDisk hostCaching "ReadWrite"
	data:    OSDisk name "testpemasm-testpemasm-0-201510102050230517"
	data:    OSDisk mediaLink "https://portalvhds4blttsxgjj1rf.blob.core.chinacloudapi.cn/vhd-store/testpemasm-2747c9c432b043ff.vhd"
	data:    OSDisk sourceImageName "b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20150908-en-us-30GB"
	data:    OSDisk operatingSystem "Linux"
	data:    OSDisk iOType "Standard"
	data:    ReservedIPName ""
	data:    VirtualIPAddresses 0 address "40.83.178.221"
	data:    VirtualIPAddresses 0 name "testpemasmContractContract"
	data:    VirtualIPAddresses 0 isDnsProgrammed true
	data:    Network Endpoints 0 localPort 22
	data:    Network Endpoints 0 name "ssh"
	data:    Network Endpoints 0 port 22
	data:    Network Endpoints 0 protocol "tcp"
	data:    Network Endpoints 0 virtualIPAddress "40.83.178.221"
	data:    Network Endpoints 0 enableDirectServerReturn false
	info:    vm show command OK

### 示例：使用 .pem 密钥和经典部署的 SSH 会话的输出

如果你已使用从 `~/.ssh/id_rsa` 文件创建的 .pem 文件创建 VM，则可以直接通过 ssh 登录到该 VM。请注意，当你执行该操作时，证书握手将使用 `~/.ssh/id_rsa` 处的私钥。它可能如下例所示：

	ssh ops@testpemasm.chinacloudapp.cn -p 22
	The authenticity of host 'testpemasm.chinacloudapp.cn (40.83.178.221)' can't be established.
	RSA key fingerprint is dc:bb:e4:cc:59:db:b9:49:dc:71:a3:c8:37:36:fd:62.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'testpemasm.chinacloudapp.cn,40.83.178.221' (RSA) to the list of known hosts.
	
    Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-49-generic x86_64)
	
	* Documentation:  https://help.ubuntu.com/

    System information as of Fri Apr 15 18:51:42 UTC 2016

    System load: 0.31              Memory usage: 2%   Processes:       213
    Usage of /:  42.1% of 1.94GB   Swap usage:   0%   Users logged in: 0

    Graph this data and manage this system at:
    https://landscape.canonical.com/

    Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

    0 packages can be updated.
	0 updates are security updates.
	
	The programs included with the Ubuntu system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.
	
	Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
	applicable law.

## 如果你无法连接

可以阅读 [SSH 连接疑难解答](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection)中的建议以了解这些建议是否可以帮助解决该情况。

## 后续步骤
 
现在，你已连接到 VM，请确保在继续使用所选分发之前对其进行更新。

<!---HONumber=Mooncake_0118_2016-->