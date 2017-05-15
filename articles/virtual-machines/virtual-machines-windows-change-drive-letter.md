<properties
    pageTitle="将 VM 的 D: 盘设为数据磁盘 | Azure"
    description="介绍如何更改 Windows VM 的驱动器号，以将 D: 盘用作数据驱动器。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager,azure-service-management" />
<tags
    ms.assetid="0867a931-0055-4e31-8403-9b38a3eeb904"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/27/2016"
    wacn.date="05/15/2017"
    ms.author="cynthn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="06080a449443800921883312b1c6222192a53e33"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="use-the-d-drive-as-a-data-drive-on-a-windows-vm"></a>使用 D: 盘作为 Windows VM 上的数据驱动器
如果应用程序需要使用 D 盘存储数据，请按照以下说明使用其他驱动器号作为临时磁盘。 切勿使用临时磁盘来存储需要保存的数据。

如果调整虚拟机大小或**停止（解除分配）**虚拟机，这可能会触发将虚拟机放置于新虚拟机监控程序的操作。 计划中或计划外的维护事件也可能触发此放置操作。 在这种情况下，临时磁盘将重新分配给第一个可用的驱动器号。 如果应用程序明确需要 D: 盘，则需要遵循以下步骤暂时移动 pagefile.sys，并连接新的数据磁盘并为其分配驱动器号 D，然后将 pagefile.sys 移回到临时驱动器。 完成后，如果 VM 移到不同的虚拟机监控程序，Azure 将不收回 D:。

有关 Azure 如何使用临时磁盘的详细信息，请参阅 [了解 Azure 虚拟机上的临时驱动器](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

## <a name="attach-the-data-disk"></a>附加数据磁盘
首先，需要将数据磁盘附加到虚拟机。 

* 若要使用门户，请参阅[如何在 Azure 门户预览中附加数据磁盘](/documentation/articles/virtual-machines-windows-attach-disk-portal/)
* 若要使用经典管理门户，请参阅[如何将数据磁盘附加到 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)。 

## <a name="temporarily-move-pagefilesys-to-c-drive"></a>将 pagefile.sys 暂时移到 C 盘
1. 连接到虚拟机。 
2. 右键单击“开始”菜单，然后选择“系统”。
3. 在左侧菜单中，选择“高级系统设置”。
4. 在“性能”部分中，选择“设置”。
5. 选择“高级”选项卡。
6. 在“虚拟内存”部分中，选择“更改”。
7. 选择 **C** 盘，然后依次单击“系统管理的大小”、“设置”。
8. 选择 **D** 盘，然后依次单击“无分页文件”、“设置”。
9. 单击“应用”。 你将收到警告，指出计算机需要重新启动才能使更改生效。
10. 重启虚拟机。

## <a name="change-the-drive-letters"></a>更改驱动器号
1. VM 重新启动后，重新登录到 VM。
2. 单击“开始”菜单，键入 **diskmgmt.msc**，然后按 Enter。 此时将启动“磁盘管理”。
3. 右键单击 **D**（临时存储驱动器），然后选择“更改驱动器号和路径”。
4. 在“驱动器号”下，选择一个新驱动器，如 **T**，然后单击“确定”。 
5. 右键单击数据磁盘，并选择“更改驱动器号和路径”。
6. 在“驱动器号”下，选择驱动器 **D**，然后单击“确定”。 

## <a name="move-pagefilesys-back-to-the-temporary-storage-drive"></a>将 pagefile.sys 移回临时存储驱动器
1. 右键单击“开始”菜单，然后选择“系统”。
2. 在左侧菜单中，选择“高级系统设置”。
3. 在“性能”部分中，选择“设置”。
4. 选择“高级”选项卡。
5. 在“虚拟内存”部分中，选择“更改”。
6. 选择 OS 驱动器 **C**，然后依次单击“无分页文件”、“设置”。
7. 选择临时存储驱动器 **T**，然后依次单击“系统管理的大小”、“设置”。
8. 单击“应用” 。 你将收到警告，指出计算机需要重新启动才能使更改生效。
9. 重启虚拟机。

## <a name="next-steps"></a>后续步骤
* 可以通过[附加更多数据磁盘](/documentation/articles/virtual-machines-windows-attach-disk-portal/)来增加虚拟机的可用存储空间。