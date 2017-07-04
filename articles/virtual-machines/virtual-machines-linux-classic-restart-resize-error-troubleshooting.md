<properties
   pageTitle="VM 重新启动或大小调整问题 | Azure"
   description="排查在 Azure 中重新启动现有 Linux 虚拟机或调整其大小时遇到的经典部署问题"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="Deland-Han"
   manager="felixwu"
   editor=""
   tags="top-support-issue"/>

<tags
   ms.service="virtual-machines-linux"
   ms.topic="support-article"
   ms.tgt_pltfrm="vm-linux"
   ms.workload="required"
   ms.date="01/10/2017"
   wacn.date="03/01/2017"
   ms.devlang="na"
   ms.author="delhan"/>

# 排查在 Azure 中重新启动现有 Linux 虚拟机或调整其大小时遇到的经典部署问题

> [AZURE.SELECTOR]
- [经典](/documentation/articles/virtual-machines-linux-classic-restart-resize-error-troubleshooting/)
- [资源管理器](/documentation/articles/virtual-machines-linux-restart-resize-error-troubleshooting/)

在尝试启动已停止的 Azure 虚拟机 (VM)，或调整现有 Azure VM 的大小时，经常遇到的错误是分配失败。当群集或区域没有可用的资源或无法支持所请求的 VM 大小时，就会发生此错误。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)]

更多关于资源管理器模型，请点击[这里](/documentation/articles/virtual-machines-linux-restart-resize-error-troubleshooting/)。

[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## 收集审核日志

若要开始故障排除，请收集审核日志，以识别与问题相关的错误。

在 Azure 门户中，单击“浏览”>“虚拟机”> _你的 Linux 虚拟机_ >“设置”>“审核日志”。

## 问题：启动已停止的 VM 时发生错误

尝试启动已停止的 VM，但出现分配失败错误。

### 原因

必须在托管云服务的原始群集上尝试发出启动已停止 VM 的请求。但是，群集没有可用的空间来完成该请求。

### 解决方法

* 创建新的云服务，并将其与区域或基于区域的虚拟网络（而不是地缘组）关联。

* 删除已停止的 VM。

* 使用磁盘在新的云服务中重新创建 VM。

* 启动重新创建的 VM。

如果在尝试创建新的云服务时收到错误，请稍后再试一次，或更改云服务的区域。

> [AZURE.IMPORTANT] 新的云服务将有新的名称和 VIP，因此需要针对现有云服务中依赖于此信息的所有项目更改该信息。

## 问题：调整现有 VM 的大小时发生错误

尝试调整现有 VM 的大小，但出现分配失败错误。

### 原因

必须在托管云服务的原始群集上尝试发出调整 VM 大小的请求。但是，群集不支持请求的 VM 大小。

### 解决方法

减小请求的 VM 大小，然后重试调整大小请求。

* 单击“浏览全部”>“虚拟机(经典)”> _你的虚拟机_ >“设置”>“大小”。有关详细步骤，请参阅[调整虚拟机的大小](https://msdn.microsoft.com/zh-cn/library/dn168976.aspx)。

如果无法减小 VM 大小，请遵循以下步骤：

  * 创建新的云服务，确保它不链接到地缘组，并且未与链接到地缘组的虚拟网络相关联。

  * 在其中创建更大的新 VM。

可以在同一个云服务中合并所有 VM。如果现有的云服务与基于区域的虚拟网络相关联，则可以将新的云服务连接到现有虚拟网络。

如果现有的云服务未与基于区域的虚拟网络相关联，则必须删除现有云服务中的 VM，并在新云服务中从其磁盘重新创建 VM。然而，请务必记住，新的云服务将有新的名称和 VIP，因此需要为所有目前将此信息用于现有云服务的依赖项更新该信息。

## 后续步骤

如果你在 Azure 中创建新的 Linux VM 时遇到问题，请参阅[排查在 Azure 中新建 Linux 虚拟机时遇到的部署问题](/documentation/articles/virtual-machines-linux-troubleshoot-deployment-new-vm/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->