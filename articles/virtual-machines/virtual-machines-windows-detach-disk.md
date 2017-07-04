<properties
    pageTitle="从 Windows VM 分离数据磁盘 - Azure | Azure"
    description="了解如何从使用资源管理器部署模型的 Azure 中的虚拟机分离磁盘。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="13180343-ac49-4a3a-85d8-0ead95e2028c"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/21/2017"
    wacn.date="05/15/2017"
    ms.author="cynthn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="e0600d9b5ed10b5e1c50b3fe358e8fefd6078f4e"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="how-to-detach-a-data-disk-from-a-windows-virtual-machine"></a>如何从 Windows 虚拟机分离数据磁盘
当不再需要附加到虚拟机的数据磁盘时，可以轻松地分离它。 这将从虚拟机中删除该磁盘，但不会从存储中删除它。 

> [AZURE.WARNING]
> 如果用户分离磁盘，它将不会自动删除。 如果用户订阅了高级存储，则将继续承担该磁盘的存储费用。 有关详细信息，请参阅[使用高级存储时的定价和计费方式](/documentation/articles/storage-premium-storage/#pricing-and-billing)。 
> 
> 

如果希望再次使用该磁盘上的现有数据，可以将其重新附加到相同的虚拟机或另一个虚拟机。  

## <a name="detach-a-data-disk-using-the-portal"></a>使用门户分离数据磁盘
1. 在门户中心中，选择“虚拟机”。
2. 选择具有要分离的数据磁盘的虚拟机，然后单击“停止”以解除分配 VM。
3. 在虚拟机边栏选项卡中，选择“磁盘”。
4. 在“磁盘”边栏选项卡的顶部，选择“编辑”。
5. 在“磁盘”边栏选项卡中，转到要分离的数据磁盘最右侧，然后单击![分离按钮图像](./media/virtual-machines-windows-detach-disk/detach.png)分离按钮。
5. 删除磁盘后，单击边栏选项卡顶部的“保存”。
6. 在虚拟机边栏选项卡中，单击“概述”，然后单击边栏选项卡顶部的“开始”按钮重启 VM。

磁盘保留在存储中，但不再附加到虚拟机。

## <a name="detach-a-data-disk-using-powershell"></a>使用 PowerShell 分离数据磁盘
在此示例中，第一个命令将获取使用 Get-AzureRmVM cmdlet 的 **RG11** 资源组中名为 **MyVM07** 的虚拟机。 该命令在 **$VirtualMachine** 变量中存储虚拟机。 

第二个命令将从该虚拟机中删除名为 DataDisk3 的数据磁盘。 

最后一个命令将更新虚拟机的状态以完成数据磁盘删除过程。

    $VirtualMachine = Get-AzureRmVM -ResourceGroupName "RG11" -Name "MyVM07" 
    Remove-AzureRmVMDataDisk -VM $VirtualMachine -Name "DataDisk3"
    Update-AzureRmVM -ResourceGroupName "RG11" -Name "MyVM07" -VM $VirtualMachine

有关详细信息，请参阅 [Remove-AzureRmVMDataDisk](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.compute/remove-azurermvmdatadisk?view=azurermps-4.0.0)。

## <a name="next-steps"></a>后续步骤
如果想要重新使用数据磁盘，只需将它[附加到另一个 VM](/documentation/articles/virtual-machines-windows-attach-disk-portal/)

<!--Update_Description: wording update-->