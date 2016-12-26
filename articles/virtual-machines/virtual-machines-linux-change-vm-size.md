<properties
   pageTitle="如何调整 Linux VM 的大小 | Azure"
   description="如何通过更改 VM 大小来增加或减少 Linux 虚拟机。"
   services="virtual-machines-linux"
   documentationCenter="na"
   authors="mikewasson"
   manager="roshar"
   editor=""
   tags=""/>

<tags
   ms.service="virtual-machines-linux"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="05/16/2016"
   wacn.date="12/26/2016"
   ms.author="mikewasson"/>


# 如何调整 Linux VM 的大小

## 概述 

预配虚拟机 (VM) 后，可以通过更改 [VM 大小][vm-sizes]来扩展或缩减 VM。在某些情况下，必须先解除分配 VM。如果新大小在托管 VM 的硬件群集上不可用，则可能会出现这种情况。

本文介绍了如何使用 [Azure CLI][azure-cli] 来调整 Linux VM 的大小。

> [AZURE.NOTE] Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。


## 调整 Linux VM 的大小 

若要调整 VM 的大小，请执行以下步骤。

1. 运行以下 CLI 命令。此命令将列出托管 VM 的硬件群集上的可用 VM 大小。

    	azure vm sizes -g <resource-group> --vm-name <vm-name>

2. 如果列出了所需大小，请运行以下命令来调整 VM 的大小。

	    azure vm set -g <resource-group> --vm-size <new-vm-size> -n <vm-name>  
	        --enable-boot-diagnostics --boot-diagnostics-storage-uri
	        https://<storage-account-name>.blob.core.chinacloudapi.cn/ 

    在此过程中，VM 将重新启动。重新启动后，现有 OS 和数据磁盘将重新映射。临时磁盘上的所有内容将会丢失。

    使用 `--enable-boot-diagnostics` 选项启用[启动诊断][boot-diagnostics]，以记录所有与启动相关的错误。

3. 如果未列出所需大小，请运行以下命令来解除分配 VM、调整其大小，然后将它重新启动。

	    azure vm deallocate -g <resource-group> <vm-name>
	    azure vm set -g <resource-group> --vm-size <new-vm-size> -n <vm-name>  
	        --enable-boot-diagnostics --boot-diagnostics-storage-uri
	        https://<storage-account-name>.blob.core.chinacloudapi.cn/ 
	    azure vm start -g <resource-group> <vm-name>

   > [AZURE.WARNING] 解除分配 VM 也会释放分配给该 VM 的所有动态 IP 地址。OS 和数据磁盘不受影响。

<!-- links -->
   
[azure-cli]: /documentation/articles/xplat-cli-install/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[vm-sizes]: /documentation/articles/virtual-machines-linux-sizes/

<!---HONumber=Mooncake_Quality_Review_1215_2016-->