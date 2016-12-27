<!-- not suitable for Mooncake -->


<properties
	pageTitle="适用于从 Azure 服务管理到 Azure Resource Manager 的迁移的社区工具"
	description="本文编录了社区提供的工具，这些工具适用于将 IaaS 资源从 Azure 服务管理迁移到 Azure Resource Manager 堆栈。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="singhkays"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/29/2016"
	wacn.date=""
	ms.author="singhkay"/>  


# 适用于从 Azure 服务管理到 Azure Resource Manager 的迁移的社区工具

本文编录了社区提供的工具，这些工具适用于将 IaaS 资源从 Azure 服务管理迁移到 Azure Resource Manager 堆栈。

>[AZURE.NOTE]Microsoft 支持服务并不正式支持这些工具。因此 Github 上提供了其开放源代码，我们很乐意接受有关修复或其他方案的 PR。若要报告问题，请使用 Github 问题功能。
>
> 使用这些工具进行迁移会造成经典虚拟机停机。若要了解平台支持的迁移，请访问
>
>- [Platform supported migration of IaaS resources from Classic to Azure Resource Manager stack（平台支持从经典部署模型迁移到 Azure Resource Manager 堆栈的 IaaS 资源）](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
>- [有关平台支持的从经典部署模型到 Azure Resource Manager 的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
>- [Migrate IaaS resources from Classic to Azure Resource Manager using Azure PowerShell（使用 Azure PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager）](/documentation/articles/virtual-machines-windows-ps-migration-classic-resource-manager/)

## ASM2ARM

这是一个 PowerShell 脚本模块，用于将**单个**虚拟机 (VM) 从 Azure 服务管理堆栈迁移到 Azure Resource Manager 堆栈。

[工具文档链接](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/asm2arm)

## migAz

migAz 是另一选项，用于将整套 Azure 服务管理 IaaS 资源迁移到 Azure Resource Manager IaaS 资源。可以在同一订阅中迁移，也可以在不同订阅和不同订阅类型（例如 CSP 订阅）中迁移。

[工具文档链接](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/migaz)

<!---HONumber=Mooncake_1017_2016-->