<properties
	pageTitle="管理虚拟机规模集中的 VM | Azure"
	description="使用 Azure PowerShell 在虚拟机规模集中管理虚拟机。"
	services="virtual-machine-scale-sets"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machine-scale-sets"
	ms.date="07/14/2016"
	wacn.date="08/29/2016"/>

# 在虚拟机规模集中管理虚拟机

使用本文中的任务在虚拟机规模集中管理虚拟机。

执行所有涉及到在规模集中管理虚拟机的任务时，要求您知道要管理的虚拟机的实例 ID。

有关如何安装最新版 Azure PowerShell、选择要使用的订阅和登录到 Azure 帐户的信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

## 显示有关虚拟机规模集的信息

可获取有关规模集的常规信息，也称为实例视图。或者，可以获取更多特定信息，如规模集中资源的信息。

在此命令中，将*资源组名称*替换为包含该虚拟机规模集的资源组名称，将*规模集名称*替换为该虚拟机规模集的名称，然后运行该命令：

    Get-AzureRmVmss -ResourceGroupName "resource group name" -VMScaleSetName "scale set name"

它会返回类似于下面的内容：

    Id                                          : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Compute/virtualMachineScaleSets/myvmss1
    Name                                        : myvmss1
    Type                                        : Microsoft.Compute/virtualMachineScaleSets
    Location                                    : chinaeast
    Sku                                         :
      Name                                      : Standard_A0
      Tier                                      : Standard
      Capacity                                  : 3
    UpgradePolicy                               :
      Mode                                      : Manual
    VirtualMachineProfile                       :
      OsProfile                                 :
        ComputerNamePrefix                      : vmss1
        AdminUsername                           : admin1
        WindowsConfiguration                    :
          ProvisionVMAgent                      : True
          EnableAutomaticUpdates                : True
    StorageProfile                              :
      ImageReference                            :
        Publisher                               : MicrosoftWindowsServer
        Offer                                   : WindowsServer
        Sku                                     : 2012-R2-Datacenter
        Version                                 : latest
      OsDisk                                    :
        Name                                    : vmssosdisk
        Caching                                 : ReadOnly
        CreateOption                            : FromImage
        VhdContainers[0]                        : https://astore.blob.core.chinacloudapi.cn/vmss
        VhdContainers[1]                        : https://gstore.blob.core.chinacloudapi.cn/vmss
        VhdContainers[2]                        : https://mstore.blob.core.chinacloudapi.cn/vmss
        VhdContainers[3]                        : https://sstore.blob.core.chinacloudapi.cn/vmss
        VhdContainers[4]                        : https://ystore.blob.core.chinacloudapi.cn/vmss
    NetworkProfile                              :
      NetworkInterfaceConfigurations[0]         :
        Name                                    : mync1
        Primary                                 : True
        IpConfigurations[0]                     :
          Name                                  : ip1
          Subnet                                :
            Id                                  : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Network/virtualNetworks/myvn1/subnets/mysn1
          LoadBalancerBackendAddressPools[0]    :
            Id                                  : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Network/loadBalancers/mylb1/backendAddressPools/bepool1
        LoadBalancerInboundNatPools[0]          :
            Id                                  : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Network/loadBalancers/mylb1/inboundNatPools/natpool1
    ExtensionProfile                            :
      Extensions[0]                             :
        Name                                    : Microsoft.Insights.VMDiagnosticsSettings
        Publisher                               : Microsoft.Azure.Diagnostics
        Type                                    : IaaSDiagnostics
        TypeHandlerVersion                      : 1.5
        AutoUpgradeMinorVersion                 : True
        Settings                                : {"xmlCfg":"...","storageAccount":"astore"}
    ProvisioningState                           : Succeeded
    
在此命令中，将*资源组名称*替换为包含该虚拟机规模集的资源组名称，将*规模集名称*替换为该虚拟机规模集名称，将 *#* 替换为要获取其信息的虚拟机的实例标识符，然后运行该命令：

    Get-AzureRmVmssVM -ResourceGroupName "resource group name" -VMScaleSetName "scale set name" -InstanceId #
        
它会返回类似于下面的内容：

    Id                            : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Compute/
                                    virtualMachineScaleSets/myvmss1/virtualMachines/0
    Name                          : myvmss1_0
    Type                          : Microsoft.Compute/virtualMachineScaleSets/virtualMachines
    Location                      : chinaeast
    InstanceId                    : 0
    Sku                           :
      Name                        : Standard_A0
      Tier                        : Standard
    LatestModelApplied            : True
    StorageProfile                :
      ImageReference              :
        Publisher                 : MicrosoftWindowsServer
        Offer                     : WindowsServer
        Sku                       : 2012-R2-Datacenter
        Version                   : 4.0.20160617
      OsDisk                      :
        OsType                    : Windows
        Name                      : vmssosdisk-os-0-e11cad52959b4b76a8d9f26c5190c4f8
        Vhd                       :
          Uri                     : https://astore.blob.core.chinacloudapi.cn/vmss/vmssosdisk-os-0-e11cad52959b4b76a8d9f26c5190c4f8.vhd
        Caching                   : ReadOnly
        CreateOption              : FromImage
    OsProfile                     :
      ComputerName                : myvmss1-0
      AdminUsername               : admin1
      WindowsConfiguration        :
        ProvisionVMAgent          : True
        EnableAutomaticUpdates    : True
    NetworkProfile                :
      NetworkInterfaces[0]        :
        Id                        : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Compute/virtualMachineScaleSets/
                                    myvmss1/virtualMachines/0/networkInterfaces/mync1
    ProvisioningState             : Succeeded
    Resources[0]                  :
      Id                          : /subscriptions/{sub-id}/resourceGroups/myrg1/providers/Microsoft.Compute/virtualMachines/
                                    myvmss1_0/extensions/Microsoft.Insights.VMDiagnosticsSettings
      Name                        : Microsoft.Insights.VMDiagnosticsSettings
      Type                        : Microsoft.Compute/virtualMachines/extensions
      Location                    : chinaeast
      Publisher                   : Microsoft.Azure.Diagnostics
      VirtualMachineExtensionType : IaaSDiagnostics
      TypeHandlerVersion          : 1.5
      AutoUpgradeMinorVersion     : True
      Settings                    : {"xmlCfg":"...","storageAccount":"astore"}
      ProvisioningState           : Succeeded
        
## 启动规模集中的虚拟机

在此命令中，将*资源组名称*替换为包含该虚拟机规模集的资源组名称，将*规模集名称*替换为该规模集名称，将 *#* 替换为要启动的虚拟机的标识符，然后运行该命令：

    Start-AzureRmVmss -ResourceGroupName "resource group name" -VMScaleSetName "scale set name" -InstanceId #

在资源浏览器中，我们可以看到该实例的状态是**正在运行**：

    "statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "level": "Info",
        "displayStatus": "Provisioning succeeded",
        "time": "2016-03-15T02:10:08.0730839+00:00"
      },
      {
        "code": "PowerState/running",
        "level": "Info",
        "displayStatus": "VM running"
      }
    ]

不使用 -InstanceId 参数即可启动规模集中的所有虚拟机。
    
## 停止规模集中的虚拟机

在此命令中，将*资源组名称*替换为包含该虚拟机规模集的资源组名称，将*规模集名称*替换为该规模集名称，将 *#* 替换为要停止的虚拟机的标识符，然后运行该命令：

	Stop-AzureRmVmss -ResourceGroupName "resource group name" -VMScaleSetName "scale set name" -InstanceId #

在资源浏览器中，我们可以看到该实例的状态是“已解除分配”：

	"statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "level": "Info",
        "displayStatus": "Provisioning succeeded",
        "time": "2016-03-15T01:25:17.8792929+00:00"
      },
      {
        "code": "PowerState/deallocated",
        "level": "Info",
        "displayStatus": "VM deallocated"
      }
    ]
    
若要停止但不解除分配虚拟机，请使用 -StayProvisioned 参数。不使用 -InstanceId 参数即可停止规模集中的所有虚拟机。
    
## 重启规模集中的虚拟机

在此命令中，将*资源组名称*替换为包含该虚拟机规模集的资源组名称，将*规模集名称*替换为该规模集名称，将 *#* 替换为要重启的虚拟机的标识符，然后运行该命令：

	Restart-AzureRmVmss -ResourceGroupName "resource group name" -VMScaleSetName "scale set name" -InstanceId #
    
不使用 -InstanceId 参数即可重启规模集中的所有虚拟机。

## 从规模集中删除虚拟机

在此命令中，将*资源组名称*替换为包含该虚拟机规模集的资源组名称，将*规模集名称*替换为该规模集名称，将 *#* 替换为要从规模集中删除的虚拟机的标识符，然后运行该命令：

	Remove-AzureRmVmss -ResourceGroupName "resource group name" -VMScaleSetName "scale set name" -InstanceId #

不使用 -InstanceId 参数即可一次性删除整个虚拟机规模集。

<!---HONumber=Mooncake_0822_2016-->