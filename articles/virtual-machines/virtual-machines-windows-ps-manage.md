<properties
    pageTitle="使用 PowerShell 管理 Azure VM | Azure"
    description="使用 PowerShell 管理 Azure 虚拟机。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="48930854-7888-4e4c-9efb-7d1971d4cc14"
    ms.service="virtual-machines-windows"
    ms.workload="na"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/07/2017"
    wacn.date="04/17/2017"
    ms.author="davidmu"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="62dedcf88d5b30feec4970f3beaac1f9185777ed"
    ms.lasthandoff="04/06/2017" />

# <a name="manage-azure-virtual-machines-using-powershell"></a>使用 PowerShell 管理 Azure 虚拟机

本文介绍常见的管理任务，用户可以在 Azure 虚拟机上执行这些任务。

有关安装最新版 Azure PowerShell、选择订阅和登录到帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

> [AZURE.NOTE]
> 可能需要重新安装 Azure PowerShell 才能使用本文中的功能。
> 
> 

创建这些变量是为了方便命令的运行，请根据自己的环境更改这些值：

    $myResourceGroup = "myResourceGroup"
    $myVM = "myVM"
    $location = "chinaeast"

## <a name="display-information-about-a-virtual-machine"></a>显示有关虚拟机的信息

获取有关虚拟机的信息。

    Get-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM -DisplayHint Expand

它会返回类似于此示例的内容：

    ResourceGroupName             : myResourceGroup
    Id                            : /subscriptions/{subscription-id}/resourceGroups/
                                    myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM
    VmId                          : #########-####-####-####-############
    Name                          : myVM
    Type                          : Microsoft.Compute/virtualMachines
    Location                      : chinaeast
    Tags                          : {}
    DiagnosticsProfile            :
      BootDiagnostics             :
      Enabled                     : True
      StorageUri                  : https://mystorage1.blob.core.chinacloudapi.cn/
    AvailabilitySetReference      : 
      Id                          : /subscriptions/{subscription-id}/resourceGroups/
                                    myResourceGroup/providers/Microsoft.Compute/availabilitySets/myAV1
    Extensions[0]                 :
      Id                          : /subscriptions/{subscription-id}/resourceGroups/
                                    myResourceGroup/providers/Microsoft.Compute/
                                    virtualMachines/myVM/extensions/BGInfo
      Name                        : BGInfo
      Type                        : Microsoft.Compute/virtualMachines/extensions
      Location                    : chinaeast
      Publisher                   : Microsoft.Compute
      VirtualMachineExtensionType : BGInfo
      TypeHandlerVersion          : 2.1
      AutoUpgradeMinorVersion     : True
      ProvisioningState           : Succeeded
    HardwareProfile               : 
      VmSize                      : Standard_DS1_v2
    NetworkProfile          
      NetworkInterfaces[0]        :
        Id                        : /subscriptions/{subscription-id}/resourceGroups/
                                    myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNIC
    OSProfile                     : 
      ComputerName                : myVM
      AdminUsername               : myaccount1
      WindowsConfiguration        :
        ProvisionVMAgent          : True
         EnableAutomaticUpdates   : True
    ProvisioningState             : Succeeded
    StorageProfile                :
      ImageReference              :
        Publisher                 : MicrosoftWindowsServer
        Offer                     : WindowsServer
        Sku                       : 2012-R2-Datacenter
        Version                   : latest   
      OsDisk                      :
        OsType                    : Windows
        Name                      : myosdisk
        Vhd                       : 
          Uri                     : http://mystore1.blob.core.chinacloudapi.cn/vhds/myosdisk.vhd
        Caching                   : ReadWrite
        CreateOption              : FromImage
    NetworkInterfaceIDs[0]        : /subscriptions/{subscription-id}/resourceGroups/
                                    myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNIC

## <a name="stop-a-virtual-machine"></a>停止虚拟机

可以使用两种方法停止虚拟机。 可停止虚拟机并保留其所有设置，但需继续付费；还可停止虚拟机并解除分配。 解除分配某个虚拟机也会解除分配与其关联的所有资源，并停止该虚拟机的计费。

### <a name="stop-and-deallocate"></a>停止并解除分配

    Stop-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM

系统会提示进行确认：

    Virtual machine stopping operation
    This cmdlet will stop the specified virtual machine. Do you want to continue?
    [Y] Yes [N] No [S] Suspend [?] Help (default is "Y"):

输入 **Y** 以停止虚拟机。

几分钟后，将返回类似于下面示例的内容：

    OperationId :
    Status      : Succeeded
    StartTime   : 9/13/2016 12:11:57 PM
    EndTime     : 9/13/2016 12:14:40 PM
    Error       :

### <a name="stop-and-stay-provisioned"></a>停止并保持预配

    Stop-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM -StayProvisioned

系统会提示你进行确认：

    Virtual machine stopping operation
    This cmdlet will stop the specified virtual machine. Do you want to continue?
    [Y] Yes [N] No [S] Suspend [?] Help (default is "Y"):

输入 **Y** 以停止虚拟机。

几分钟后，将返回类似于下面示例的内容：

    OperationId :
    Status      : Succeeded
    StartTime   : 9/13/2016 12:11:57 PM
    EndTime     : 9/13/2016 12:14:40 PM
    Error       :

## <a name="start-a-virtual-machine"></a>启动虚拟机

启动虚拟机（如果已停止）。

    Start-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM

几分钟后，将返回类似于以下示例的内容：

    OperationId :
    Status      : Succeeded
    StartTime   : 9/13/2016 12:32:55 PM
    EndTime     : 9/13/2016 12:35:09 PM
    Error       :

如果想要重新启动已在运行虚拟机，请使用下文所述的 **Restart-AzureRmVM** 。

## <a name="restart-a-virtual-machine"></a>重新启动虚拟机

重新启动正在运行的虚拟机。

    Restart-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM

它会返回类似于此示例的内容：

    OperationId :
    Status      : Succeeded
    StartTime   : 9/13/2016 12:54:40 PM
    EndTime     : 9/13/2016 12:55:54 PM
    Error       :    

## <a name="add-a-data-disk-to-a-virtual-machine"></a>将数据磁盘添加到虚拟机

### <a name="managed-data-disk"></a>托管数据磁盘

Azure 中国目前暂时不支持托管磁盘

### <a name="unmanaged-data-disk"></a>非托管数据磁盘

    $vm = Get-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM
    Add-AzureRmVMDataDisk -VM $vm -Name "myDataDisk1" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/myDataDisk1.vhd" -LUN 0 -Caching ReadWrite -DiskSizeinGB 1 -CreateOption Empty
    Update-AzureRmVM -ResourceGroupName $myResourceGroup -VM $vm

运行 Add-AzureRmVMDataDisk 以后，所看到的结果应如以下示例所示：

    ResourceGroupName        : myResourceGroup
    Id                       : /subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/
                               Microsoft.Compute/virtualMachines/myVM
    VmId                     : ########-####-####-####-############
    Name                     : myVM
    Type                     : Microsoft.Compute/virtualMachines
    Location                 : chinaeast
    Tags                     : {}
    AvailabilitySetReference : {Id}
    DiagnosticsProfile       : {BootDiagnostics}
    HardwareProfile          : {VmSize}
    NetworkProfile           : {NetworkInterfaces}
    OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
    ProvisioningState        : Succeeded
    StorageProfile           : {ImageReference, OsDisk, DataDisks}
    DataDiskNames            : {myDataDisk1}
    NetworkInterfaceIDs      : {/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/
                               Microsoft.Network/networkInterfaces/myNIC}

运行 Update-AzureRmVM 以后，所看到的结果应如以下示例所示：

    RequestId IsSuccessStatusCode StatusCode ReasonPhrase
    --------- ------------------- ---------- ------------
                             True         OK OK

## <a name="update-a-virtual-machine"></a>更新虚拟机

本示例演示如何更新虚拟机的大小。

    $vm = Get-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM
    $vm.HardwareProfile.vmSize = "Standard_DS2_v2"
    Update-AzureRmVM -ResourceGroupName $myResourceGroup -VM $vm

它会返回与此示例类似的内容：

    RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
    ---------  -------------------  ----------  ------------
                              True          OK  OK

有关虚拟机的可用大小列表，请参阅 [Azure 中的虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/) 。

## <a name="delete-a-virtual-machine"></a>删除虚拟机

删除虚拟机。  

    Remove-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM

> [AZURE.NOTE]
> 可以使用 **-Force** 参数跳过确认提示。
> 
> 

如果没有使用 -Force 参数，系统会提示进行确认：

    Virtual machine removal operation
    This cmdlet will remove the specified virtual machine. Do you want to continue?
    [Y] Yes [N] No [S] Suspend [?] Help (default is "Y"):

它会返回类似于此示例的内容：

    RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
    ---------  -------------------  ----------  ------------
                              True          OK  OK

## <a name="next-steps"></a>后续步骤

- 如果部署出现问题，可以参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)
- 阅读[使用 Resource Manager 和 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/)，了解如何使用 Azure PowerShell 创建虚拟机
- 参考 [使用 Resource Manager 模板创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template/)

<!--Update_Description: wording update-->