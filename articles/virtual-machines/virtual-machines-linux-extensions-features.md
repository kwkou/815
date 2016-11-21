<properties
 pageTitle="虚拟机扩展和功能 | Azure"
 description="了解可为 Azure 虚拟机提供的扩展，这些虚拟机扩展按它们提供或改进的功能进行分组。"
 services="virtual-machines-linux"
 documentationCenter=""
 authors="neilpeterson"
 manager="timlt"
 editor=""
 tags="azure-service-management,azure-resource-manager"/>  


<tags
 ms.service="virtual-machines-linux"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-linux"
 ms.workload="infrastructure-services"
 ms.date="09/22/2016"
 wacn.date="11/21/2016"
 ms.author="nepeters"/>

# 关于虚拟机扩展和功能

## Azure VM 扩展

Azure 虚拟机扩展是小型应用程序，可在Azure 虚拟机上提供部署后配置和自动化任务。例如，如果虚拟机要求安装软件、防病毒保护或 Docker 配置，便可以使用 VM 扩展来完成这些任务。可以使用 Azure CLI、PowerShell、Resource Manager 模板和 Azure 门户预览运行 Azure VM 扩展。扩展可与新虚拟机部署捆绑在一起，或者针对任何现有系统运行。

本文档提供 Azure 虚拟机扩展的先决条件，以及有关如何检测可用 VM 扩展的指导。

## Azure VM 代理

Azure VM 代理可管理 Azure 虚拟机与 Azure 结构控制器之间的交互。VM 代理负责部署和管理 Azure 虚拟机的许多功能层面，包括运行 VM 扩展。Azure VM 代理预先安装在 Azure 库映像上，并可安装在支持的操作系统上。

有关支持的操作系统和安装说明，请参阅 [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)。

## 发现 VM 扩展

有许多不同的 VM 扩展可与 Azure 虚拟机配合使用。若要查看完整列表，请使用 Azure CLI 运行以下命令，并将命令中的位置替换为所选位置。

    azure vm extension-image list chinanorth

<br />  


## 常见的 VM 扩展

|扩展名称 |说明 |更多信息 |
|---|---|---|
|适用于 Linux 的自定义脚本扩展 | 针对 Azure 虚拟机运行脚本 |[适用于 Linux 的自定义脚本扩展](/documentation/articles/virtual-machines-linux-extensions-customscript/) |
|VM 访问扩展 | 重新获取对 Azure 虚拟机的访问权限 |[VM 访问扩展](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) |
|Azure 诊断扩展 |管理 Azure 诊断 |[Azure 诊断扩展](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |

<!---HONumber=Mooncake_1114_2016-->