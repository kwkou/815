<properties 
	pageTitle="在 Azure 中使用 Linux 虚拟机的根权限" 
	description="了解如何在 Azure 中使用 Linux 虚拟机的根权限。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="szarkos" 
	manager="timlt" 
	editor=""
	tags="azure-service-management,azure-resource-manager" />

<tags 
	ms.service="virtual-machines-linux" 
	ms.date="03/25/2016" 
	wacn.date="05/24/2016"/>


# 在 Azure 中使用 Linux 虚拟机的根权限

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

默认情况下，禁止在 Azure 中的 Linux 虚拟机上使用 `root` 用户。用户可以通过 `sudo` 命令使用提升的权限来运行各种命令。但是，体验可能因系统设置方式而异。

1. **SSH 密钥和密钥或者仅密码** - 对虚拟机进行预配时，使用的是证书（`.CER` 文件）或者 SSH 密钥和密码，或只是用户名和密码。在本示例中，`sudo` 在执行命令前将提示用户输入密码。

2. **仅 SSH 密钥** - 虚拟机预配时使用的是证书（`.cer`、`.pem` 或 `.pub` 文件）或 SSH 密钥，但不是密码。在本示例中，在执行命令前 `sudo` **将不**提示用户输入密码。


## SSH 密钥和密码，或仅密码

使用 SSH 密钥或密码身份验证登录到 Linux 虚拟机中，然后使用 `sudo` 运行命令，例如：

	# sudo <command>
	[sudo] password for azureuser:

在这种情况下，将会提示用户输入密码。在输入密码后，`sudo` 将使用 `root` 权限运行该命令。

你还可以通过编辑 `/etc/sudoers.d/waagent` 文件来启用不需要密码的 sudo，例如：

	#/etc/sudoers.d/waagent
	azureuser ALL = (ALL) NOPASSWD: ALL

对于用户“azureuser”，此更改将启用不需要密码的 sudo。

## 仅 SSH 密钥

使用 SSH 密钥身份验证登录到 Linux 虚拟机中，然后使用 `sudo` 运行命令，例如：

	# sudo <command>

在这种情况下，将**不会**提示用户输入密码。按 `<enter>` 后，`sudo` 将使用 `root` 权限运行此命令。

 

<!---HONumber=Mooncake_0118_2016-->