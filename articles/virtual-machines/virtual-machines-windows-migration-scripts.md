<properties
    pageTitle="社区工具 - 将经典资源迁移到 Azure Resource Manager | Azure"
    description="本文编录了社区提供的工具，这些工具可帮助将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="singhkays"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="228b697b-3950-49f5-84bb-283bb56621b1"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="1/23/2017"
    wacn.date="03/20/2017"
    ms.author="singhkay" />  


# 用于将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的社区工具
本文编录了社区提供的工具，这些工具可帮助将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型。

> [AZURE.NOTE]
Microsoft 支持服务并不正式支持这些工具。因此 Github 上提供了其开放源代码，我们很乐意接受有关修复或其他方案的 PR。若要报告问题，请使用 Github 问题功能。
> <p>
> 使用这些工具进行迁移会造成经典虚拟机停机。若要了解平台支持的迁移，请访问
> <p>
><p> * [Platform supported migration of IaaS resources from Classic to Azure Resource Manager stack（平台支持从经典部署模型迁移到 Azure Resource Manager 堆栈的 IaaS 资源）](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
><p> * [有关平台支持的从经典部署模型到 Azure Resource Manager 的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
><p> * [Migrate IaaS resources from Classic to Azure Resource Manager using Azure PowerShell（使用 Azure PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager）](/documentation/articles/virtual-machines-windows-ps-migration-classic-resource-manager/)
> 
> 

## ASM2ARM
这是一个 PowerShell 脚本模块，用于将**单个**虚拟机 (VM) 从经典部署模型迁移到 Azure Resource Manager 部署模型。

[工具文档链接](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/asm2arm)

## migAz
migAz 是另一选项，用于将整套经典 IaaS 资源迁移到 Azure Resource Manager IaaS 资源。可以在同一订阅中迁移，也可以在不同订阅和不同订阅类型（例如 CSP 订阅）中迁移。

[工具文档链接](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/migaz)

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: wording update-->