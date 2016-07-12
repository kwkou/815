<!-- Ibiza portal: tested -->

<properties
   pageTitle="VM 重新启动或大小调整问题 | Azure"
   description="排查在 Azure 中重新启动或调整现有 Windows 虚拟机时遇到的 Resource Manager 部署问题"
   services="virtual-machines-windows, azure-resource-manager"
   documentationCenter=""
   authors="Deland-Han"
   manager="felixwu"
   editor=""
   tags="top-support-issue"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/12/2016"
	wacn.date="06/20/2016"/>

# 排查在 Azure 中重新启动或调整现有 Windows 虚拟机时遇到的 Resource Manager 部署问题

> [AZURE.SELECTOR]
- [经典](/documentation/articles/virtual-machines-windows-classic-restart-resize-error-troubleshooting/)
- [资源管理器](/documentation/articles/virtual-machines-windows-restart-resize-error-troubleshooting/)

当你尝试启动已停止的 Azure 虚拟机 (VM)，或调整现有 Azure VM 的大小时，经常遇到的错误是分配失败。当群集或区域没有可用的资源或无法支持所请求的 VM 大小时，将发生此错误。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。

[AZURE.INCLUDE [support-disclaimer](../includes/support-disclaimer.md)]

## 收集审核日志

若要开始故障排除，请收集审核日志，以识别与问题相关的错误。以下链接包含有关过程的详细信息：

[使用资源管理器执行审核操作](/documentation/articles/resource-group-audit/)

## 问题：启动已停止的 VM 时发生错误

你尝试启动已停止的 VM，但出现分配失败。

### 原因

必须在托管云服务的原始群集上尝试发出启动已停止 VM 的请求。但是，群集没有足够的空间可完成该请求。

### 解决方法

*	停止可用性集中的所有 VM 并重新启动每个 VM。

  1. 单击“资源组”> 你的资源组 >“资源”> _你的可用性集_ >“虚拟机”> _你的虚拟机_ >“停止”。

  2. 所有 VM 停止后，选择每个已停止的 VM 并单击“启动”。

*	稍后重试重新启动请求。

## 问题：重新启动现有 VM 时发生错误

你尝试调整现有 VM 的大小，但出现分配失败。

### 原因

必须在托管云服务的原始群集上尝试发出调整 VM 大小的请求。但是，群集不支持请求的 VM 大小。

### 解决方法

* 以更小的 VM 大小重试请求。

* 如果无法更改请求的 VM 大小：

  1. 停止可用性集中的所有 VM。

    * 单击“资源组”> 你的资源组 >“资源”> _你的可用性集_ >“虚拟机”> _你的虚拟机_ >“停止”。

  2. 所有 VM 停止后，将所需的 VM 调整到更大的大小。
  3. 选择已调整大小的 VM，单击“启动”，然后启动每个已停止的 VM。

<!---HONumber=Mooncake_0613_2016-->