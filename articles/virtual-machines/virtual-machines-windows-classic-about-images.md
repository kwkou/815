<properties
    pageTitle="关于 Windows 虚拟机的映像 | Azure"
    description="了解如何对 Azure 中的 Windows 虚拟机使用映像。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor="tysonn"
    tags="azure-service-management" />
<tags
    ms.assetid="66ff3fab-8e7f-4dff-b8da-ab1c9c9c9af8"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="05/15/2017"
    ms.author="cynthn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="15343f21fced95891d692da6bcb7899e31585041"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="about-images-for-windows-virtual-machines"></a>关于 Windows 虚拟机的映像
> [AZURE.IMPORTANT]
> Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 和经典模型](/documentation/articles/resource-manager-deployment-model/)。 本文介绍如何使用经典部署模型。 Azure 建议大多数新部署使用 Resource Manager 模型。 有关在 Resource Manager 模型中查找和使用映像的信息，请参阅[此处](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)。

[AZURE.INCLUDE [virtual-machines-common-classic-about-images](../../includes/virtual-machines-common-classic-about-images.md)]

## <a name="working-with-images"></a>使用映像

可以通过 Azure PowerShell 模块和 Azure 门户预览管理可供 Azure 订阅使用的映像。 Azure PowerShell 模块提供更多命令选项，以便可以明确确定要查看或执行的操作。 Azure 门户预览为许多日常管理任务提供了 GUI。

下面是一些使用 Azure PowerShell 模块的示例。

* **获取所有映像**：`Get-AzureVMImage`返回当前订阅中可用的所有映像的列表：你的映像以及 Azure 或合作伙伴提供的映像。 生成的列表可能较大。 下一个示例演示如何获取较短的列表。
* **获取映像系列**：`Get-AzureVMImage | select ImageFamily` 通过显示 **ImageFamily** 字符串属性获取映像系列的列表。
* **获取特定系列中的所有映像**：`Get-AzureVMImage | Where-Object {$_.ImageFamily -eq $family}`
* **查找 VM 映像**：`Get-AzureVMImage | where {(gm -InputObject $_ -Name DataDiskConfigurations) -ne $null} | Select -Property Label, ImageName`此 cmdlet 通过筛选 DataDiskConfiguration 属性来完成，仅适用于 VM 映像。 此示例还仅根据标签和映像名称来筛选输出。
* **保存通用映像**：`Save-AzureVMImage -ServiceName "myServiceName" -Name "MyVMtoCapture" -OSState "Generalized" -ImageName "MyVmImage" -ImageLabel "This is my generalized image"`
* **保存专用映像**：`Save-AzureVMImage -ServiceName "mySvc2" -Name "MyVMToCapture2" -ImageName "myFirstVMImageSP" -OSState "Specialized" -Verbose`

    > [AZURE.TIP]
    > 若要创建包含操作系统磁盘和附加数据磁盘的 VM 映像，OSState 参数是必需的。 如果未使用此参数，该 cmdlet 将创建 OS 映像。 该参数的值根据操作系统磁盘是否已准备好重用来指示映像是通用的还是专用的。

* **删除映像**：`Remove-AzureVMImage -ImageName "MyOldVmImage"`

## <a name="next-steps"></a>后续步骤
还可以[使用 Azure 门户预览创建 Windows 计算机](/documentation/articles/virtual-machines-windows-classic-tutorial/)。

<!--Update_Description: wording update-->