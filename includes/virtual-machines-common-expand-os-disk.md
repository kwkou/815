## 概述
当你在资源组中通过从 Azure 库部署映像来创建新的虚拟机 (VM) 时，默认的 OS 驱动器空间为 127 GB。尽管你可以将数据磁盘添加到 VM（数量取决于所选择的 SKU），并且我们建议将应用程序和需要大量 CPU 的工作负荷安装在这些附加的磁盘上，但客户有时候还是必须扩展 OS 驱动器以支持特定的方案，例如：

1.  支持将组件安装在 OS 驱动器上的传统应用程序。
2.  从本地迁移具有较大 OS 驱动器的物理电脑或虚拟机。

>[AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：Resource Manager 和经典。本文介绍如何使用 Resource Manager 模型。Azure 建议大多数新部署使用资源管理器模型。

## 调整 OS 驱动器的大小
在本文中，我们将使用 [Azure Powershell](/documentation/articles/powershell-install-configure/) 的 Resource Manager 模块，完成调整 OS 驱动器大小的任务。在管理模式下打开 Powershell ISE 或 Powershell 窗口，并遵循以下步骤：

1.  在资源管理模式下登录你的 Microsoft Azure 帐户，然后选择你的订阅，如下所示：

	    Login-AzureRmAccount
	    Select-AzureRmSubscription -SubscriptionName 'my-subscription-name'

2.  设置资源组名称和 VM 名称，如下所示：

	    $rgName = 'my-resource-group-name'
	    $vmName = 'my-vm-name'

3.  获取对 VM 的引用，如下所示：

    	$vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName

4. 在调整磁盘大小之前停止 VM，如下所示：

    	Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName

5.  接下来就是我们期待已久的时刻！ 将 OS 磁盘的大小设置为所需值，并更新 VM，如下所示：

	    $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
	    Update-AzureRmVM -ResourceGroupName $rgName -VM $vm

    >[AZURE.WARNING]新大小应该大于现有磁盘大小。允许的最大值为 1023 GB。

6.  更新 VM 可能需要几秒钟时间。命令完成执行后，请重新启动 VM，如下所示：

    	Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName

大功告成！ 现在，请通过 RDP 访问 VM，打开“计算机管理”（或“磁盘管理”），然后使用刚刚分配的空间扩展驱动器。

## 摘要
在本文中，我们已使用 Powershell 的 Azure Resource Manager 模块扩展 IaaS 虚拟机的 OS 驱动器。以下重现了完整的脚本供你参考：

	Login-AzureRmAccount
	Select-AzureRmSubscription -SubscriptionName 'my-subscription-name'
	$rgName = 'my-resource-group-name'
	$vmName = 'my-vm-name'
	$vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
	Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName
	$vm.StorageProfile.OSDisk.DiskSizeGB = 1023
	Update-AzureRmVM -ResourceGroupName $rgName -VM $vm
	Start-AzureRmVM -ResourceGroupName $rgName -Name $vmName

## 后续步骤
在本文中，我们着重于扩展 VM 的 OS 磁盘，但是，开发的脚本也可用于通过更改一行代码，来扩展附加到 VM 的数据磁盘。例如，若要扩展附加到 VM 的第一个数据磁盘，请将 ```StorageProfile``` 的 ```OSDisk``` 对象替换为 ```DataDisks``` 数组，并使用数字索引获取对第一个附加的数据磁盘的引用，如下所示：

	$vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023

同样，你可以使用如上所示的索引，或如下所示的磁盘 ```Name``` 属性，引用附加到 VM 的其他数据磁盘：

	($vm.StorageProfile.DataDisks | Where {$_.Name -eq 'my-second-data-disk'})[0].DiskSizeGB = 1023

如果想要了解如何将磁盘附加到 Azure Resource Manager VM，请参阅[此文](/documentation/articles/virtual-machines-windows-attach-disk-portal/)。

<!---HONumber=Mooncake_0425_2016-->