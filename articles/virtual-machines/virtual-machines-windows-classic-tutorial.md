<properties
    pageTitle="在 Azure 门户预览中创建 VM | Azure"
    description="在 Azure 门户预览中创建 Windows 虚拟机。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="1871f823-ebd7-4eff-9a22-8e2411555595"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/27/2017"
    wacn.date="04/24/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="baa96694fb5d9b274e1af9ec81dcd113babd87a3"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-virtual-machine-running-windows-in-the-azure-portal-preview"></a>在 Azure 门户预览中创建运行 Windows 的虚拟机
> [AZURE.SELECTOR]
- [Azure 经典管理门户](/documentation/articles/virtual-machines-windows-classic-tutorial/)
- [PowerShell：经典部署](/documentation/articles/virtual-machines-windows-classic-create-powershell/)

<br>

> [AZURE.IMPORTANT]
> Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 和经典模型](/documentation/articles/resource-manager-deployment-model/)。 本文介绍如何使用经典部署模型。 Azure 建议大多数新部署使用 Resource Manager 模型。 了解如何通过 **Azure 门户预览**[使用 Resource Manager 部署模型执行这些步骤](/documentation/articles/virtual-machines-windows-hero-tutorial/)。

本教程演示如何在 Azure 门户预览中创建运行 Windows 的 Azure 虚拟机 (VM)。 我们将使用 Windows Server 映像作为示例，但这只是 Azure 提供的众多映像的其中一个。

本部分演示如何使用 Azure 门户预览中的“仪表板”选择并创建虚拟机。

你也可以使用[自己的映像](/documentation/articles/virtual-machines-windows-classic-createupload-vhd/)创建 VM。 若要了解此方法和其他方法，请参阅[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-creation-choices/)。

<!-- 02/27/2017 Video removed as it was based on the Classic Management Portal. -->

## <a id="createvirtualmachine"> </a>创建虚拟机
[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../../includes/virtual-machines-create-windowsvm.md)]

## <a name="next-steps"></a>后续步骤
* 了解如何在 Azure 门户预览中[使用 Resource Manager 部署模型创建 VM](/documentation/articles/virtual-machines-windows-hero-tutorial/)。
* 登录到虚拟机。 有关说明，请参阅[登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-windows-classic-connect-logon/)。
* 附加磁盘以存储数据。 你可以附加空磁盘和包含数据的磁盘。 有关说明，请参阅[将数据磁盘附加到使用经典部署模型创建的 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)。
<!--Update_Description: change to the new portal-->