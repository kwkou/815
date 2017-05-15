<properties
    pageTitle="Azure 虚拟机代理概述 | Azure"
    description="Azure 虚拟机代理概述"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid="0a1f212e-053e-4a39-9910-8d622959f594"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="11/17/2016"
    wacn.date="05/15/2017"
    ms.author="nepeters"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="e0833cf51733212389dfbe23a620f6b8336c461b"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="azure-virtual-machine-agent-overview"></a>Azure 虚拟机代理概述

Azure 虚拟机代理（AM 代理）是受保护的轻型过程，用于管理 VM 与 Azure 结构控制器的交互。 VM 代理有一个主要角色，目的是启用和执行 Azure 虚拟机扩展。 VM 扩展可用于对虚拟机进行部署后配置，例如安装和配置软件。 虚拟机扩展还可启用恢复功能，例如重置虚拟机的管理密码。 没有 Azure VM 代理，虚拟机扩展将无法运行。

本文档详细介绍如何安装、检测和删除 Azure 虚拟机代理。

## <a name="install-the-vm-agent"></a>安装 VM 代理

### <a name="azure-gallery-image"></a>Azure 库映像

Azure VM 代理默认安装在任何通过 Azure 库映像部署的 Windows 虚拟机上。 从门户、PowerShell、命令行接口或 Azure Resource Manager 模板部署 Azure 库映像时，也会安装 Azure VM 代理。 

### <a name="manual-installation"></a>手动安装

可以使用 Windows 安装程序包手动安装 Windows VM 代理。 创建将部署在 Azure 中的自定义虚拟机映像时，可能需要手动安装。 若要手动安装 Windows VM 代理，可从以下位置下载 VM 代理安装程序： [Microsoft Azure VM 代理下载](http://go.microsoft.com/fwlink/?LinkID=394789)。 

双击 Windows 安装程序文件即可安装 VM 代理。 若要以自动或无人参与方式安装 VM 代理，请运行以下命令。

    msiexec.exe /i WindowsAzureVmAgent.2.7.1198.778.rd_art_stable.160617-1120.fre /quiet

## <a name="detect-the-vm-agent"></a>检测 VM 代理

### <a name="powershell"></a>PowerShell

可以使用 Azure Resource Manager PowerShell 模块检索有关 Azure 虚拟机的信息。 运行 `Get-AzureRmVM` 会返回大量信息，包括 Azure VM 代理的预配状态。

    Get-AzureRmVM

下面仅仅是部分 `Get-AzureRmVM` 输出。 请注意嵌套在 `OSProfile` 中的 `ProvisionVMAgent` 属性，可以使用该属性来确定 VM 代理是否已部署到虚拟机。

    OSProfile                  :
      ComputerName             : myVM
      AdminUsername            : muUserName
      WindowsConfiguration     :
        ProvisionVMAgent       : True
        EnableAutomaticUpdates : True

可以使用以下脚本返回一个简明列表，其中包含虚拟机名称以及 VM 代理的状态。

    $vms = Get-AzureRmVM

    foreach ($vm in $vms) {
        $agent = $vm | Select -ExpandProperty OSProfile | Select -ExpandProperty Windowsconfiguration | Select ProvisionVMAgent
        Write-Host $vm.Name $agent.ProvisionVMAgent
    }

### <a name="manual-detection"></a>手动检测

登录到 Windows Azure VM 后，可以使用任务管理器检查正在运行的进程。 若要查看 Azure VM 代理，请打开任务管理器 > 单击详细信息选项卡，然后查找名为 `WindowsAzureGuestAgent.exe` 的进程。 存在该进程表示 VM 代理已安装。

## <a name="upgrade-the-vm-agent"></a>升级 VM 代理

适用于 Windows 的 Azure VM 代理会自动升级。 新虚拟机在部署到 Azure 后会获得最新 VM 代理。 应手动更新自定义 VM 映像，使之包括新的 VM 代理。