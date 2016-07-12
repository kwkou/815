<!-- ARM: tested -->

<properties 
   pageTitle="Linux 和 Mac 的基本 Azure CLI 命令 | Azure"
   description="用于在 Linux 和 Mac 上开始管理 Azure Resource Manager 模式的 VM 的基本 Azure CLI 命令"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="RicksterCDN" 
   manager="timlt" 
   editor="tysonn" 
   tags="azure-resource-manager"/>
   
<tags
	ms.service="virtual-machines-linux"
	ms.date="04/11/2016"
	wacn.date="06/27/2016"/>

# Linux 和 Mac 上的常用 Azure CLI 命令

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

通过 Resource Manager 命令和模板使用 Azure CLI 以利用资源组部署 Azure 资源和工作负荷之前，你将需要一个 Azure 帐户。如果没有帐户，你可以[在此处获取 Azure 试用帐户](/pricing/1rmb-trial/)。

如果你尚未安装 Azure CLI 并连接到订阅，请参阅 [Install the Azure CLI（安装 Azure CLI）](/documentation/articles/xplat-cli-install/)以使用 `azure config mode arm` 将模式设置为 `arm`，并使用 `azure login -e AzureChinaCloud` 命令连接到 Azure。

## Azure CLI 中的基本 Azure Resource Manager 命令

本文章介绍配合 Azure CLI 来管理 Azure 订阅中的 ARM 资源（主要是 VM）并与其交互的基本命令。有关特定命令行开关和选项的详细帮助，可以通过键入 `azure <command> <subcommand> --help` 或 `azure help <command> <subcommand>` 来使用联机命令帮助和选项。

> [AZURE.NOTE] 这些示例不包括通常为资源管理器中的 VM 部署建议的基于模板的操作。有关信息，请参阅 [Deploy and manage virtual machines by using Azure Resource Manager templates and the Azure CLI（使用 Azure Resource Manager 模板和 Azure CLI 部署和管理虚拟机）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)。

任务 | 资源管理器
-------------- | ----------- | -------------------------
创建最基本的 VM | `azure vm quick-create [options] <resource-group> <name> <location> <os-type> <image-urn> <admin-username> <admin-password>`<br/><br/>（从 `azure vm image list` 命令获取 `image-urn`。有关示例，请参阅[此文](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。）
创建 Linux VM | `azure  vm create [options] <resource-group> <name> <location> -y "Linux"`
创建 Windows VM | `azure  vm create [options] <resource-group> <name> <location> -y "Windows"`
列出 VM | `azure  vm list [options]`
获取有关 VM 的信息 | `azure  vm show [options] <resource_group> <name>`
启动 VM | `azure vm start [options] <resource_group> <name>`
停止 VM | `azure vm stop [options] <resource_group> <name>`
释放 VM | `azure vm deallocate [options] <resource-group> <name>`
重新启动 VM | `azure vm restart [options] <resource_group> <name>`
删除 VM | `azure vm delete [options] <resource_group> <name>`
捕获 VM | `azure vm capture [options] <resource_group> <name>`
从用户映像创建 VM | `azure  vm create [options] -q <image-name> <resource-group> <name> <location> <os-type>`
从专用磁盘创建 VM | `azue  vm create [options] -d <os-disk-vhd> <resource-group> <name> <location> <os-type>`
将数据磁盘添加到 VM | `azure  vm disk attach-new [options] <resource-group> <vm-name> <size-in-gb> [vhd-name]`
从 VM 中删除数据磁盘 | `azure  vm disk detach [options] <resource-group> <vm-name> <lun>`
将泛型扩展添加到 VM |`azure  vm extension set [options] <resource-group> <vm-name> <name> <publisher-name> <version>`
将 VM 访问扩展添加到 VM | `azure vm reset-access [options] <resource-group> <name>`
将 Docker 扩展添加到 VM | `azure  vm docker create [options] <resource-group> <name> <location> <os-type>`
删除 VM 扩展 | `azure  vm extension set [options] -u <resource-group> <vm-name> <name> <publisher-name> <version>`
获取 VM 资源的使用情况 | `azure vm list-usage [options] <location>`
获取所有可用 VM 大小 | `azure vm sizes [options]`


## 后续步骤

* 有关超越基本 VM 管理的其他 CLI 命令示例，请参阅 [Using the Azure CLI with Azure Resource Manager（将 Azure CLI 与 Azure Resource Manager 配合使用）](/documentation/articles/azure-cli-arm-commands/)。

<!---HONumber=Mooncake_0620_2016-->