<!-- Ibiza portal: tested -->

<properties
   pageTitle="VM 重新启动或大小调整问题 | Azure"
   description="排查在 Azure 中重新启动或调整现有 Linux 虚拟机时遇到的 Resource Manager 部署问题"
   services="virtual-machines-linux, azure-resource-manager"
   documentationCenter=""
   authors="Deland-Han"
   manager="felixwu"
   editor=""
   tags="top-support-issue"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="09/09/2016"
	wacn.date="10/25/2016"/>

# 排查在 Azure 中重新启动或调整现有 Linux 虚拟机时遇到的 Resource Manager 部署问题

> [AZURE.SELECTOR]
- [经典](/documentation/articles/virtual-machines-linux-classic-restart-resize-error-troubleshooting/)
- [资源管理器](/documentation/articles/virtual-machines-linux-restart-resize-error-troubleshooting/)

当你尝试启动已停止的 Azure 虚拟机 (VM)，或调整现有 Azure VM 的大小时，经常遇到的错误是分配失败。当群集或区域没有可用的资源或无法支持所请求的 VM 大小时，将发生此错误。

[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## 收集审核日志

若要开始故障排除，请收集审核日志，以识别与问题相关的错误。以下链接包含有关过程的详细信息：

[使用 Azure 门户预览对资源组部署进行故障排除](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)

[使用资源管理器执行审核操作](/documentation/articles/resource-group-audit/)

## 问题：启动已停止的 VM 时发生错误

你尝试启动已停止的 VM，但出现分配失败。

### 原因

必须在托管云服务的原始群集上尝试发出启动已停止 VM 的请求。但是，群集没有足够的空间可完成该请求。

### 解决方法

*	停止可用性集中的所有 VM 并重新启动每个 VM。

  1. 单击“资源组”> “你的资源组” >“资源”> “你的可用性集” >“虚拟机”> “你的虚拟机” >“停止”。

  2. 所有 VM 停止后，选择每个已停止的 VM 并单击“启动”。

*	稍后重试重新启动请求。

## 问题：调整现有 VM 的大小时发生错误

你尝试调整现有 VM 的大小，但出现分配失败。

### 原因

必须在托管云服务的原始群集上尝试发出调整 VM 大小的请求。但是，群集不支持请求的 VM 大小。

### 解决方法

* 以更小的 VM 大小重试请求。

* 如果无法更改请求的 VM 大小：

  1. 停止可用性集中的所有 VM。

    * 单击“资源组”> “你的资源组” >“资源”> “你的可用性集” >“虚拟机”> “你的虚拟机” >“停止”。

  2. 所有 VM 停止后，将所需的 VM 调整到更大的大小。
  3. 选择已调整大小的 VM，单击“启动”，然后启动每个已停止的 VM。

## 后续步骤

如果你在 Azure 中创建新的 Linux VM 时遇到问题，请参阅[排查在 Azure 中新建 Linux 虚拟机时遇到的部署问题](/documentation/articles/virtual-machines-linux-troubleshoot-deployment-new-vm/)。

<!---HONumber=Mooncake_0718_2016-->