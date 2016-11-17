<properties
	pageTitle="通过 SSH 连接到 Linux VM 被拒绝、失败或出现错误 | Azure"
	description="为运行 Linux 的 Azure 虚拟机排查并修复“SSH 连接失败”或“SSH 连接被拒绝”等 SSH 错误。"
	keywords="ssh 连接被拒绝, ssh 错误, azure ssh, SSH 连接失败"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="top-support-issue,azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/27/2016"
	wacn.date="08/15/2016"
	ms.author="iainfou"/>

# 对通过 SSH 连接到 Azure Linux VM 失败、出现错误或被拒绝进行故障排除

当你尝试连接到基于 Linux 的 Azure 虚拟机 (VM) 时，有多种原因可能会导致安全外壳 (SSH) 错误、SSH 连接失败或被拒绝。本文将帮助你找出原因并更正问题。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和 CSDN Azure](/support/forums/) 上的 Azure 专家。或者，你也可以提出 Azure 支持事件。请转到 [Azure 支持站点](/support/contact/)并选择“获取支持”。有关使用 Azure 支持的信息，请阅读 [Azure support FAQ](/support/faq/)（Azure 支持常见问题）。

## 使用 Resource Manager 部署模型创建的 VM

你可以直接使用 Azure CLI 命令或者使用 [Azure VMAccessForLinux 扩展](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess)重置凭据或 SSHD。在执行每个故障排除步骤之后，请尝试再次连接到 VM。

### Azure CLI 先决条件

如果尚未安装 Azure CLI，请[安装 Azure CLI 并连接到 Azure 订阅](/documentation/articles/xplat-cli-install/)。使用 `azure login -e AzureChinaCloud` 命令登录并确保处于 Resource Manager 模式 (`azure config mode arm`)。

确保已安装 [Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/) 2.0.5 版或更高版本。

### 重置 SSHD
SSHD 配置本身可能有误或服务遇到错误。你可以重置 SSHD 以确保 SSH 配置本身是有效的。

#### Azure CLI

	azure vm reset-access -g <resource group> -n <vm name> -r

#### VM 访问扩展
在 json 文件中读取的访问扩展用于定义要执行的操作，如重置 SSH、重置 SSH 密钥、添加新用户等。首先，创建包含以下内容的名为 PrivateConf.json 的文件：

	{  
		"reset_ssh":"True"
	}

然后手动运行 `VMAccessForLinux` 扩展以重置 SSHD 连接：

	azure vm extension set <resource group> <vm name> VMAccessForLinux Microsoft.OSTCExtensions "1.2" --private-config-path PrivateConf.json

### 重置用户的 SSH 凭据
如果 SSHD 显示运行正常，则可以重置指定用户的密码。

#### Azure CLI

	azure vm reset-access -g <resource group> <vm name> -u <username> -p <new password>

如果使用 SSH 密钥身份验证，则可以重置指定用户的 SSH 密钥：

	azure vm reset-access -g <resource group> -n <vm name> -u <usernamer> -M <~/.ssh/azure_id_rsa.pub>

#### VM 访问扩展
创建包含以下内容的名为 PrivateConf.json 的文件：

	{
		"username":"Username", "password":"NewPassword"
	}

或者如果要重置指定用户的 SSH 密钥，则创建具有以下内容的名为 PrivateConf.json 的文件：

	{
		"username":"Username", "ssh_key":"ContentsOfNewSSHKey"
	}

在上述任一步骤之后，你可以手动运行 `VMAccessForLinux` 扩展来重置 SSH 用户凭据：

	azure vm extension set <resource group> <vmname> VMAccessForLinux Microsoft.OSTCExtensions "1.2" --private-config-path PrivateConf.json

### 重新部署 VM
你可以将 VM 重新部署到 Azure 中的另一个节点，这可能可以更正任何潜在的网络问题。若要使用 Azure 门户预览重新部署 VM，请选择“浏览”>“虚拟机”> *你的 Linux 虚拟机* >“重新部署”。有关如何执行此操作的信息，请参阅 [Redeploy virtual machine to new Azure node](/documentation/articles/virtual-machines-windows-redeploy-to-new-node/)（将虚拟机重新部署到新的 Azure 节点）。当前不能使用 Azure CLI 重新部署 VM。

> [AZURE.NOTE] 请注意，完成此操作后，你会丢失临时磁盘数据，而系统则会更新与虚拟机关联的动态 IP 地址。


## 使用经典部署模型创建的 VM

若要解决使用经典部署模型创建的 VM 中最常见的 SSH 连接失败问题，请尝试以下步骤。在执行每个步骤之后，请尝试重新连接到 VM。

- 从 [Azure 门户预览](https://portal.azure.cn)重置远程访问。在 Azure 门户预览中，选择“浏览”>“虚拟机(经典)”> *你的 Linux 虚拟机* >“重置远程...”。

- 重启 VM。在 [Azure 门户预览](https://portal.azure.cn)中，选择“浏览”>“虚拟机(经典)”> *你的 Linux 虚拟机* >“重启”。

	- 或 -

	在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，选择“虚拟机”>“实例”>“重启”。

- 将 VM 重新部署到新的 Azure 节点。有关如何执行此操作的信息，请参阅 [Redeploy virtual machine to new Azure node](/documentation/articles/virtual-machines-windows-redeploy-to-new-node/)（将虚拟机重新部署到新的 Azure 节点）。

	请注意，完成此操作后，你会丢失临时磁盘数据，而系统则会更新与虚拟机关联的动态 IP 地址。

- 按照 [How to reset a password or SSH for Linux-based virtual machines](/documentation/articles/virtual-machines-linux-classic-reset-access/)（如何为基于 Linux 的虚拟机重置密码或 SSH）中的说明执行以下操作：
	- 重置密码或 SSH 密钥。
	- 创建新的 _sudo_ 用户帐户。
	- 重置 SSH 配置。

- 检查 VM 的资源运行状况以了解是否存在任何平台问题。<br> 选择“浏览”>“虚拟机(经典)”> *你的 Linux 虚拟机* >“设置”>“检查运行状况”。


## 其他资源

- 如果在执行后面的步骤之后仍不能通过 SSH 登录 VM，那么你可以执行[更详细的故障排除步骤](/documentation/articles/virtual-machines-linux-detailed-troubleshoot-ssh-connection/)来检查其他网络配置和步骤以解决你的问题。

- 有关对应用程序访问进行故障排除的详细信息，请参阅 [Troubleshoot access to an application running on an Azure virtual machine](/documentation/articles/virtual-machines-linux-troubleshoot-app-connection/)（对在 Azure 虚拟机上运行的应用程序的访问进行故障排除）

- 有关对使用经典部署模型创建的虚拟机进行故障排除的详细信息，请参阅 [How to reset a password or SSH for Linux-based virtual machines](/documentation/articles/virtual-machines-linux-classic-reset-access/)（如何为基于 Linux 的虚拟机重置密码或 SSH）。

<!---HONumber=Mooncake_0808_2016-->