<properties 
	pageTitle="重新部署 Linux 虚拟机 | Azure" 
	description="介绍如何通过重新部署 Linux 虚拟机来解决 SSH 连接问题。" 
	services="virtual-machines-linux" 
	documentationCenter="virtual-machines" 
	authors="iainfoulds" 
	manager="timlt"
	tags="azure-resource-manager,top-support-issue" 
/>
	

<tags 
	ms.service="virtual-machines-linux" 
	ms.devlang="na" 
	ms.topic="support-article" 
	ms.tgt_pltfrm="vm-linux"
	ms.workload="infrastructure" 
	ms.date="09/19/2016" 
	wacn.date="12/26/2016" 
	ms.author="iainfou" 
/>

# 将虚拟机重新部署到新的 Azure 节点

如果在对 SSH 或应用程序对 Azure 虚拟机 (VM) 的访问进行故障排除时遇到困难，重新部署 VM 可能会有帮助。重新部署 VM 时，将 VM 移到 Azure 基础结构中的新节点，然后重新打开它，同时保留所有配置选项和关联的资源。本文介绍如何使用 Azure CLI 或 Azure 门户预览重新部署 VM。

> [AZURE.NOTE] 重新部署 VM 后，临时磁盘将丢失，与虚拟网络接口关联的动态 IP 地址将更新。


## 使用 Azure CLI

请确保在计算机上[已安装最新的 Azure CLI](/documentation/articles/xplat-cli-install/)，并且处于 Resource Manager 模式 (`azure config mode arm`)。

使用以下 Azure CLI 命令可重新部署虚拟机：

	azure vm redeploy --resourcegroup <resourcegroup> --vm-name <vmname> 

在重新部署 VM 的过程时，可以看到 VM 的状态发生更改。在将 VM 重新部署到新主机的过程中，VM 的 `PowerState` 将从“正在运行”变为“正在更新”，然后变为“正在启动”，最后变为“正在运行”。使用以下命令检查资源组中的 VM 的状态：

	azure vm list -g <resourcegroup>

[AZURE.INCLUDE [virtual-machines-common-redeploy-to-new-node](../../includes/virtual-machines-common-redeploy-to-new-node.md)]


## 后续步骤
如果在连接 VM 时遇到问题，可以在[SSH 连接故障排除](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)或[详细的 SSH 故障排除步骤](/documentation/articles/virtual-machines-linux-detailed-troubleshoot-ssh-connection/)中找到具体的帮助。如果无法访问在 VM 上运行的应用程序，还可以阅读 [应用程序故障排除问题](/documentation/articles/virtual-machines-linux-troubleshoot-app-connection/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->