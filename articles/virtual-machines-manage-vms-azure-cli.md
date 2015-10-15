<properties
   pageTitle="使用适用于 Mac、Linux 和 Windows 的 Azure CLI 管理 Azure VM | Windows Azure"
   description="介绍如何使用适用于 Mac、Linux 和 Windows 的 Azure CLI 创建、管理和删除 Azure VM。"
   services="virtual-machines"
   documentationCenter="virtual-machines"
   authors="dlepow"
   manager="timlt"
   editor=""/>

   <tags
	   ms.service="virtual-machines"
	   ms.date="06/09/2015"
	   wacn.date="09/18/2015"/>

# 使用适用于 Mac、Linux 和 Windows 的 Azure CLI 管理虚拟机

你每天执行的管理 VM 的许多任务都可以使用 Azure CLI 自动执行。本文提供较简单任务的示例命令，并提供演示更复杂任务的命令的文章链接。

>[AZURE.NOTE]如果你尚未安装和配置 Azure CLI，则可以在[此处](/documentation/articles/xplat-cli-install)获取相关说明。如果你想要对 PowerShell 中的相同任务快速入门，请参阅[使用 Azure PowerShell 管理 VM](/documentation/articles/virtual-machines-manage-vms-powershell)。

## 如何使用示例命令
你需要将命令中的一些文本替换为适合你的环境的文本。< and > 符号指示需要替换的文本。替换文本时，请删除符号，但将引号保留在原处。

> [AZURE.NOTE]如果你想要以编程方式存储和操作控制台命令的输出，可以使用 JSON 分析工具（例如 **[jq](https://github.com/stedolan/jq)**、**[jsawk](https://github.com/micha/jsawk)**）或适合任务的语言库。

## 显示有关 VM 的信息

这是你将经常使用的基本任务。使用它可获取有关 VM 的信息、对 VM 执行任务或获取输出以存储在变量中。

若要获取有关 VM 的信息，请运行此命令，并替换引号内的所有内容（包括 < and > 字符）：

     azure vm show -g <group name> -n <virtual machine name>

若要将输出存储在 $vm 变量中作为 JSON 文档，请运行：

    vmInfo=$(azure vm show -g <group name> -n <virtual machine name> --json)

或者，你可以通过管道将 stdout 传递给文件。

## 登录到基于 Linux 的虚拟机

通常，Linux 计算机是通过 SSH 连接的。有关详细信息，请参阅[如何在 Azure 中将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-use-ssh-key)。Azure 资源管理器概述
## 停止 VM

运行以下命令：

    azure vm stop <group name> <virtual machine name>

>[AZURE.IMPORTANT]如果该 VM 是云服务中的最后一个 VM，则使用此参数可以保留该云服务的虚拟 IP (VIP)。<br><br> 如果你使用 StayProvisioned 参数，则仍要支付 VM 的费用。

## 启动 VM

运行此命令：azure vm start <group name> <virtual machine name>

## 附加数据磁盘

你还需要确定是要附加新的磁盘还是附加包含数据的磁盘。对于新磁盘，此命令将创建 .vhd 文件，然后将它附加在同一个命令中。

若要附加新磁盘，请运行以下命令：

     azure vm disk attach-new <resource-group> <vm-name> <size-in-gb>

若要附加现有数据磁盘，请运行以下命令：

    azure vm disk attach <resource-group> <vm-name> [vhd-url]

## 创建 Linux VM

若要创建新的基于 Linux 的 VM，你将需要准备好多个值，包括资源组名称、位置、映像名称、VM 名称以及用于存储后备 .vhd 映像的存储帐户。你准备好要使用的信息后，Azure CLI 可以创建交互式会话以提示你通过键入提供这些值：

    azure vm create

当然，如果你已准备好这些值，则可以通过键入 `azure help vm create` 找到相应开关来直接传递这些值。

## 后续步骤

有关 Azure CLI 用法和 **arm** 模式的更多示例，请参阅<!--[-->将适用于 Mac、Linux 和 Windows 的 Windows Azure CLI 用于 Azure 资源管理<!--](/documentation/articles/xplat-cli-resource-manager)-->。若要了解有关 Azure 资源及其概念的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。

<!---HONumber=70-->