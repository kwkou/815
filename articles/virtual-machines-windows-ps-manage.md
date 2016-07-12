<!-- ARM: tested -->

<properties
	pageTitle="使用 Resource Manager 和 PowerShell 管理 VM | Azure"
	description="使用 Azure Resource Manager 与 PowerShell 来管理虚拟机。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/18/2016"
	wacn.date="06/20/2016"/>

# 使用 Resource Manager 与 PowerShell 来管理 Azure 虚拟机

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

## 安装 Azure PowerShell
 
有关如何安装最新版 Azure PowerShell 的信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。选择要使用的订阅，然后登录到你的 Azure 帐户。

## 设置变量

本文中的所有命令都需要虚拟机所在资源组的名称以及要管理的虚拟机的名称。将 **$rgName** 的值替换为包含虚拟机的资源组的名称。将 **$vmName** 的值替换为 VM 的名称。创建变量。

        $rgName = "resource group name"
        $vmName = "VM name"

## 显示有关虚拟机的信息

获取虚拟机信息。
  
        Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName

它会返回类似于下面的内容：

        ResourceGroupName        : rg1
        Id                       : /subscriptions/{subscription-id}/resourceGroups/
                                    rg1/providers/Microsoft.Compute/virtualMachines/vm1
        Name                     : vm1
        Type                     : Microsoft.Compute/virtualMachines
        Location                 : centralus
        Tags                     : {}
        AvailabilitySetReference : {
                                     "id": "/subscriptions/{subscription-id}/resourceGroups/
                                       rg1/providers/Microsoft.Compute/availabilitySets/av1"
                                   }
        Extensions               : []
        HardwareProfile          : {
                                     "vmSize": "Standard_A0"
                                   }
        InstanceView             : null
        NetworkProfile           : {
                                     "networkInterfaces": [
                                       {
                                         "properties.primary": null,
                                         "id": "/subscriptions/{subscription-id}/resourceGroups/
                                           rg1/providers/Microsoft.Network/networkInterfaces/nc1"
                                       }
                                     ]
                                   }
        OSProfile                : {
                                     "computerName": "vm1",
                                     "adminUsername": "myaccount1",
                                     "adminPassword": null,
                                     "customData": null,
                                     "windowsConfiguration": {
                                       "provisionVMAgent": true,
                                       "enableAutomaticUpdates": true,
                                       "timeZone": null,
                                       "additionalUnattendContents": [],
                                       "winRM": null
                                     },
                                     "linuxConfiguration": null,
                                     "secrets": []
                                   }
        Plan                     : null
        ProvisioningState        : Succeeded
        StorageProfile           : {
                                     "imageReference": {
                                       "publisher": "MicrosoftWindowsServer",
                                       "offer": "WindowsServer",
                                       "sku": "2012-R2-Datacenter",
                                       "version": "latest"
                                     },
                                     "osDisk": {
                                       "osType": "Windows",
                                       "encryptionSettings": null,
                                       "name": "osdisk",
                                       "vhd": {
                                         "Uri": "http://sa1.blob.core.chinacloudapi.cn/vhds/osdisk1.vhd"
                                       }
                                       "image": null,
                                       "caching": "ReadWrite",
                                       "createOption": "FromImage"
                                     }
                                     "dataDisks": [],
                                   }
        DataDiskNames            : {}
        NetworkInterfaceIDs      : {/subscriptions/{subscription-id}/resourceGroups/
                                     rg1/providers/Microsoft.Network/networkInterfaces/nc1}

## 启动虚拟机

启动虚拟机。

        Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName

几分钟后，将返回类似于下面的内容：

        RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
        ---------  -------------------  ----------  ------------
                                  True          OK  OK

## 停止虚拟机

停止虚拟机。

	    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName

系统会提示你进行确认：

        Virtual machine stopping operation
        This cmdlet will stop the specified virtual machine. Do you want to continue?
        [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
        
输入 **Y** 以停止虚拟机。

几分钟后，将返回类似于下面的内容：

        RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
        ---------  -------------------  ----------  ------------
                                  True          OK  OK

## 重新启动虚拟机

重启虚拟机。

        $rgName = "resource group name"
        $vmName = "VM name"
        Restart-AzureRmVM -ResourceGroupName $rgName -Name $vmName

它会返回类似于下面的内容：

        RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
        ---------  -------------------  ----------  ------------
                                  True          OK  OK

## 删除虚拟机

删除虚拟机。

        $rgName = "resource group name"
        $vmName = "VM name"
	    Remove-AzureRmVM -ResourceGroupName $rgName -Name $vmName

> [AZURE.NOTE] 可以使用 **-Force** 参数跳过确认提示。

如果你没有使用 -Force 参数，系统会提示你进行确认：

	    Virtual machine removal operation
	    This cmdlet will remove the specified virtual machine. Do you want to continue?
	    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

它会返回类似于下面的内容：

        RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
        ---------  -------------------  ----------  ------------
                                  True          OK  OK

## 更新虚拟机

本示例演示如何更新虚拟机的大小。
        
        $vmSize = "Standard_A1"
        $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
        $vm.HardwareProfile.vmSize = $vmSize
        Update-AzureRmVM -ResourceGroupName $rgName -VM $vm
    
    See [Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-windows-sizes/) for a list of available sizes for a virtual machine.

它会返回类似于下面的内容：

        RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
        ---------  -------------------  ----------  ------------
                                  True          OK  OK

<!---HONumber=Mooncake_0613_2016-->