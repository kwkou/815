<!-- ARM: tested -->

<properties
	pageTitle="在资源管理器中捕获 Windows VM | Azure"
	description="了解如何捕获使用 Azure Resource Manager 部署模型创建的、基于 Windows 的 Azure 虚拟机 (VM) 的映像。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="01/29/2016"
	wacn.date=""/>


# 如何在 Resource Manager 部署模型中捕获 Windows 虚拟机

本文演示如何使用 Azure PowerShell 来捕获运行 Windows 的 Azure 虚拟机，以便用它来创建其他虚拟机。此映像包括操作系统磁盘和附加到虚拟机的数据磁盘。它不包括创建 Windows VM 所需的虚拟网络资源，因此你需要在创建另一个使用此映像的虚拟机之前，先设置好这些资源。还需对此映像进行准备，使之成为[通用化的 Windows 映像](https://technet.microsoft.com/zh-cn/library/hh824938.aspx)。


## 先决条件

这些步骤假定你已使用 Resource Manager 部署模型创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘和完成其他自定义事项（如安装应用程序）。如果你尚未完成此操作，请阅读[如何使用 Resource Manager 和 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-create-windows-powershell-resource-manager)。你可以同样轻松地使用 [Azure 门户](https://portal.azure.cn)创建 Windows 虚拟机。请阅读[如何在 Azure 门户中创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)。


## 准备要进行映像捕获的 VM

本部分说明如何使 Windows 虚拟机通用化。在通用化过程中，将删除你在各个位置的所有个人帐户信息。当你要使用此 VM 映像快速部署类似的虚拟机时，通常需要执行此操作。

1. 登录到 Windows 虚拟机。在 [Azure 门户](https://portal.azure.cn)中，导航至“浏览”>“虚拟机”> 你的 Windows 虚拟机 >“连接”。

2. 以管理员身份打开“命令提示符”窗口。

3. 将目录更改为 `%windir%\system32\sysprep`，然后运行 sysprep.exe。

4. 在“系统准备工具”对话框中，执行以下操作：

	- 在“系统清理操作”中，选择“进入系统全新体验(OOBE)”，并确保选中“一般化”。有关使用 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](http://technet.microsoft.com/zh-cn/library/bb457073.aspx)。

	- 在“关机选项”中选择“关机”。

	- 单击“确定”。

	![运行 Sysprep](./media/virtual-machines-windows-capture-image-resource-manager/SysprepGeneral.png)

5.	Sysprep 关闭虚拟机。它在 Azure 门户中的状态将变为“已停止”。


</br>
## 捕获 VM

你可以使用 Azure PowerShell 或新的 Azure Resource Manager (ARM) 资源管理器工具来捕获通用化的 Windows VM。本部分将介绍这两种方法的步骤。

### 使用 PowerShell

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

本文假定你已安装 Azure PowerShell 1.0.x 版。我们建议你使用此版本，因为新的 Resource Manager 功能将不会添加到旧的 PowerShell 版本中。如果你还没有安装 PowerShell，请参阅[如何安装配置 Azure PowerShell](/documentation/articles/powershell-install-configure)，查看安装步骤。

1. 打开 Azure PowerShell 并登录到你的 Azure 帐户。

		Login-AzureRmAccount

	此命令将打开一个弹出窗口，供你输入 Azure 凭据。

2. 如果默认选择的订阅 ID 不同于你想要使用的订阅 ID，请使用以下任一项设置正确的订阅。

		Set-AzureRmContext -SubscriptionId "xxxx-xxxx-xxxx-xxxx"

	或

		Select-AzureRmSubscription -SubscriptionId "xxxx-xxxx-xxxx-xxxx"

	你可以使用命令 `Get-AzureRmSubscription` 查找你的 Azure 帐户已拥有的订阅。

3. 现在，你需要解除分配此虚拟机使用的资源。

		Stop-AzureRmVM -ResourceGroupName YourResourceGroup -Name YourWindowsVM

	你会看到，Azure 门户中该 VM 的“状态”已从“已停止”变为“已停止(已解除分配)”。

	>[AZURE.TIP] 你还可以在 PowerShell 中查找虚拟机的状态，方法是使用：</br> `$vm = Get-AzureRmVM -ResourceGroupName YourResourceGroup -Name YourWindowsVM -status`</br> `$vm.Statuses`</br> “DisplayStatus”字段对应于 Azure 门户中显示的“状态”。

4. 接下来，你需要将虚拟机的状态设置为“通用化”。请注意，你需要执行此步骤，因为 Azure 无法理解上述通用化步骤 (`sysprep`) 的执行方式。

		Set-AzureRmVm -ResourceGroupName YourResourceGroup -Name YourWindowsVM -Generalized

	>[AZURE.NOTE] 上面设置的通用化状态不会显示在门户中。不过，你可以使用上述提示中显示的 Get-AzureRmVM 命令进行验证。

5. 使用以下命令将虚拟机映像捕获到目标存储容器。

		Save-AzureRmVMImage -ResourceGroupName YourResourceGroup -VMName YourWindowsVM -DestinationContainerName YourImagesContainer -VHDNamePrefix YourTemplatePrefix -Path Yourlocalfilepath\Filename.json

	`-Path` 变量是可选的，可以用于在本地保存 JSON 模板。`-DestinationContainerName` 变量是需在其中保存映像的容器的名称。存储的映像的 URL 将类似于 `https://YourStorageAccountName.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/YourImagesContainer/YourTemplatePrefix-osDisk.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`。该 URL 在创建时所使用的存储帐户与原始虚拟机的相同。

	>[AZURE.NOTE] 若要查找你的映像的位置，请打开本地 JSON 文件模板。转到“资源”>“storageProfile”>“osDisk”>“映像”>“URI”部分即可查找映像的完整路径。目前尚无简便方法可查找门户中的这些映像，因为存储帐户中的 system 容器处于隐藏状态。因此，虽然 `-Path` 变量为可选变量，但最好是使用它，一方面可以在本地保存模板，另一方面可以轻松查找映像 URL。你也可以在门户里确认这个 URI；它会被复制到一个名为 **system** 的 blob。


### 使用 Azure 资源浏览器（预览版）

[Azure 资源浏览器（预览版）](https://azure.microsoft.com/blog/azure-resource-explorer-a-new-tool-to-discover-the-azure-api/)是新开发的一种工具，用于管理在 Resource Manager 部署模型下创建的 Azure 资源。利用此工具，你可以轻松地：

- 发现 Azure 资源管理 API；
- 获取 API 文档；
- 直接在 Azure 订阅中进行 API 调用。

若要详细了解这个强大工具的所有功能，请观看 [David Ebbo 讲述 Azure Resource Manager 资源管理器](https://channel9.msdn.com/Shows/Azure-Friday/Azure-Resource-Manager-Explorer-with-David-Ebbo)中的视频。

你可以使用资源浏览器来捕获虚拟机，作为 PowerShell 方法的替代方法。

1. 打开[资源浏览器网站](https://resources.azure.com/)并登录到你的 Azure 帐户。

2. 在工具右上方选择“读/写”以启用 PUT 和 POST 操作。该选项默认设置为“只读”，这意味着在默认情况下，你只能执行 GET 操作。

	![资源浏览器读/写](./media/virtual-machines-windows-capture-image-resource-manager/ArmExplorerReadWrite.png)

3. 接下来，请找到你的 Windows 虚拟机。你可以在工具顶部的“搜索”框中键入名称，也可以按以下顺序浏览左侧的菜单：“订阅”> 你的 Azure 订阅 >“resourceGroups”> 你的资源组 >“提供程序”> Microsoft.Compute >“virtualMachines”> 你的 Windows 虚拟机。单击左侧导航中的虚拟机时，你会在工具右侧看到其模板。

4. 在模板页的右上方，你会看到适用于此虚拟机的各种操作的选项卡。单击“操作(POST/DELETE)”所对应的选项卡。

	![资源浏览器操作菜单](./media/virtual-machines-windows-capture-image-resource-manager/ArmExplorerActionMenu.png)

	- 你会看到可以在虚拟机上执行的所有操作的列表。

		![资源浏览器操作项](./media/virtual-machines-windows-capture-image-resource-manager/ArmExplorerActionItems.png)

6. 单击“解除分配”操作按钮，解除对虚拟机的分配。VM 的状态将从“已停止”变为“已停止(已解除分配)”。

7. 单击“通用化”操作按钮，将虚拟机标记为“已通用化”。验证状态变化时，你可以先单击左侧虚拟机名称下的“InstanceView”菜单，然后导航到右侧的“状态”部分。

8. 在“捕获”操作按钮下，你可以设置捕获映像所需的值。填充的值可以如下所示。

	![资源浏览器捕获](./media/virtual-machines-windows-capture-image-resource-manager/ArmExplorerCaptureAction.png)

	单击“捕获”操作按钮以捕获虚拟机的映像。此时会创建映像的全新 VHD 以及 JSON 模板文件。

9. 若要访问这个新的映像 VHD 和模板，请下载并安装管理存储资源所需的 Azure 工具，即 [Azure 存储资源管理器](http://storageexplorer.com/)。将在计算机本地安装 Azure 存储资源管理器。

	- 打开存储资源管理器，登录到你的 Azure 订阅。此时会显示适用于你的订阅的所有存储帐户。

	- 在左侧，你会看到我们在上述步骤中捕获的虚拟机的存储帐户。双击其下的“系统”菜单。你会看到右侧“system”文件夹的内容。

		![存储资源管理器系统](./media/virtual-machines-windows-capture-image-resource-manager/StorageExplorer1.png)

	- 双击“Microsoft.Compute”，然后双击“映像”以显示所有映像文件夹。双击你在从资源浏览器捕获映像时为 destinationContainerName 变量输入的文件夹名称。此时会显示 VHD 以及 JSON 模板文件。

	- 在这里，你可以找出该 URL，也可以通过直接单击的方式来下载 VHD/模板。

		![存储资源管理器模板](./media/virtual-machines-windows-capture-image-resource-manager/StorageExplorer2.png)


## 从捕获的映像部署新的 VM

现在，你可以使用已捕获的映像创建新的 Windows VM。以下步骤演示如何使用 Azure PowerShell 以及在上述步骤中捕获的 VM 映像在新的虚拟网络中创建 VM。

>[AZURE.NOTE] VM 映像应与将创建的实际虚拟机存在于同一存储帐户中。

### 创建网络资源

使用以下示例 PowerShell 脚本为新 VM 设置虚拟网络和 NIC。根据应用程序需要使用 $ 符号所表示的变量的值。

	$pip = New-AzureRmPublicIpAddress -Name $pipName -ResourceGroupName $rgName -Location $location -AllocationMethod Dynamic

	$subnetconfig = New-AzureRmVirtualNetworkSubnetConfig -Name $subnet1Name -AddressPrefix $vnetSubnetAddressPrefix

	$vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location -AddressPrefix $vnetAddressPrefix -Subnet $subnetconfig

	$nic = New-AzureRmNetworkInterface -Name $nicname -ResourceGroupName $rgName -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

### 创建新的 VM

以下 PowerShell 脚本演示如何设置虚拟机配置，以及如何将已捕获的 VM 映像用作新安装的源。</br>

	#Enter a new username and password in the pop-up for the following
	$cred = Get-Credential

	#Get the storage account where the captured image is stored
	$storageAcc = Get-AzureRmStorageAccount -ResourceGroupName $rgName -AccountName $storageAccName

	#Set the VM name and size
	$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A2"

	#Set the Windows operating system configuration and add the NIC
	$vm = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $computerName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate

	$vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

	#Create the OS disk URI
	$osDiskUri = '{0}vhds/{1}{2}.vhd' -f $storageAcc.PrimaryEndpoints.Blob.ToString(), $vmName.ToLower(), $osDiskName

	#Configure the OS disk to be created from image (-CreateOption fromImage) and give the URL of the captured image VHD for the -SourceImageUri parameter.
	#We found this URL in the local JSON template in the previous sections.
	$vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri $urlOfCapturedImageVhd -Windows

	#Create the new VM
	New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm

你应该在 [Azure 门户](https://portal.azure.cn)的“浏览”>“虚拟机”下查看新创建的 VM，或者使用以下 PowerShell 命令进行查看：

	$vmList = Get-AzureRmVM -ResourceGroupName $rgName
	$vmList.Name


## 后续步骤

若要使用 Azure PowerShell 管理新虚拟机，请阅读[使用 Azure Resource Manager 和 PowerShell 管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)。

<!---HONumber=Mooncake_0411_2016-->