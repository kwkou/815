<properties
	pageTitle="VM 任务的等效 Azure CLI 命令 | Microsoft Azure"
	description="用于在 Azure 资源管理器和 Azure 服务管理模式下创建和管理 Azure VM 的等效 Azure CLI 命令"
	services="virtual-machines"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="12/14/2015"
	wacn.date=""/>


# 适合使用 Azure 命令行界面执行 VM 任务的等效资源管理器和服务管理命令
本文介绍用于在 Azure 服务管理和 Azure 资源管理器中创建和管理 Azure VM 的等效 Microsoft Azure 命令行界面 (Azure CLI) 命令。在将脚本从一个命令模式迁移到其他命令模式时，请使用本文作为简易指南。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]



* 如果你尚未安装 Azure CLI 并连接到订阅，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install) 和[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect)。如果你想要使用资源管理器模式命令，请务必使用登录方法连接。

* 若要开始在 Azure CLI 中使用资源管理器模式，可能需要切换命令模式。默认情况下，CLI 在服务管理模式下启动。若要更改为资源管理器模式，请运行 `azure config mode arm`。若要回到服务管理模式，请运行 `azure config mode asm`。

* 如需联机命令帮助和选项，请键入 `azure <command> <subcommand> --help` 或 `azure help <command> <subcommand>`。

## VM 任务
下表比较了你可以在服务管理和资源管理器中使用 Azure CLI 命令执行的常见 VM 任务。使用许多资源管理器命令时，你需要传递现有资源组的名称。

> [AZURE.NOTE] 这些示例不包括通常为资源管理器中的 VM 部署建议的基于模板的操作。有关信息，请参阅[将 Azure CLI 与 Azure 资源管理器配合使用](/documentation/articles/xplat-cli-azure-resource-manager)和[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli)。

任务 | 服务管理 | 资源管理器
-------------- | ----------- | -------------------------
创建最基本的 VM | `azure vm create [options] <dns-name> <image> [userName] [password]` | `azure vm quick-create [options] <resource-group> <name> <location> <os-type> <image-urn> <admin-username> <admin-password>`<br/><br/>（从 `azure vm image list` 命令获取 `image-urn`。有关示例，请参阅[本文](/documentation/articles/resource-groups-vm-searching)。）
创建 Linux VM | `azure vm create [options] <dns-name> <image> [userName] [password]` | `azure  vm create [options] <resource-group> <name> <location> -y "Linux"`
创建 Windows VM | `azure vm create [options] <dns-name> <image> [userName] [password]` | `azure  vm create [options] <resource-group> <name> <location> -y "Windows"`
列出 VM | `azure  vm list [options]` | `azure  vm list [options]`
获取有关 VM 的信息 | `azure  vm show [options] <vm_name>` | `azure  vm show [options] <resource_group> <name>`
启动 VM | `azure vm start [options] <name>` | `azure vm start [options] <resource_group> <name>`
停止 VM | `azure vm shutdown [options] <name>` | `azure vm stop [options] <resource_group> <name>`
释放 VM | 不可用 | `azure vm deallocate [options] <resource-group> <name>`
重新启动 VM | `azure vm restart [options] <vname>` | `azure vm restart [options] <resource_group> <name>`
删除 VM | `azure vm delete [options] <name>` | `azure vm delete [options] <resource_group> <name>`
捕获 VM | `azure vm capture [options] <name>` | `azure vm capture [options] <resource_group> <name>`
从用户映像创建 VM | `azure  vm create [options] <dns-name> <image> [userName] [password]` | `azure  vm create [options] -q <image-name> <resource-group> <name> <location> <os-type>`
从专用磁盘创建 VM | `azure  vm create [options]-d <custom-data-file> <dns-name> [userName] [password]` | `azue  vm create [options] -d <os-disk-vhd> <resource-group> <name> <location> <os-type>`
将数据磁盘添加到 VM | `azure  vm disk attach [options] <vm-name> <disk-image-name>` -或- <br/> `vm disk attach-new [options] <vm-name> <size-in-gb> [blob-url]` | `azure  vm disk attach-new [options] <resource-group> <vm-name> <size-in-gb> [vhd-name]`
从 VM 中删除数据磁盘 | `azure  vm disk detach [options] <vm-name> <lun>` | `azure  vm disk detach [options] <resource-group> <vm-name> <lun>`
将泛型扩展添加到 VM | `azure  vm extension set [options] <vm-name> <extension-name> <publisher-name> <version>` | `azure  vm extension set [options] <resource-group> <vm-name> <name> <publisher-name> <version>`
将 VM 访问扩展添加到 VM | 不可用 | `azure vm reset-access [options] <resource-group> <name>`
将 Docker 扩展添加到 VM | `azure  vm docker create [options] <dns-name> <image> <user-name> [password]` | `azure  vm docker create [options] <resource-group> <name> <location> <os-type>`
将 Chef 扩展添加到 VM | `azure  vm extension get-chef [options] <vm-name>` | 不可用
禁用 VM 扩展 | `azure  vm extension set [options] -b <vm-name> <extension-name> <publisher-name> <version>` | 不可用
删除 VM 扩展 | `azure  vm extension set [options] -u <vm-name> <extension-name> <publisher-name> <version>` | `azure  vm extension set [options] -u <resource-group> <vm-name> <name> <publisher-name> <version>`
列出 VM 扩展 | `azure vm extension list [options]` | 不可用
显示 VM 映像 | `azure vm image show [options]` | 不可用
获取 VM 资源的使用情况 | 不可用 | `azure vm list-usage [options] <location>`
获取所有可用 VM 大小 | 不可用 | `azure vm sizes [options]`


## 后续步骤

* 如需 CLI 命令的其他示例，请参阅[将 Azure 命令行接口用于 Azure 服务管理](/documentation/articles/virtual-machines-command-line-tools)和[将 Azure CLI 用于 Azure 资源管理器](/documentation/articles/azure-cli-arm-commands)。

<!---HONumber=Mooncake_0215_2016-->