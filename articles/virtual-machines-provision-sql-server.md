<properties
	pageTitle="在 Azure PowerShell 中创建 SQL Server 虚拟机（经典） | Azure"
	description="提供用于创建具有 SQL Server 虚拟机库映像的 Azure VM的步骤和 PowerShell 脚本。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"
	tags="azure-service-management" />
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/20/2016"
	wacn.date="06/29/2016"/>

# 在 Azure PowerShell 中创建 SQL Server 虚拟机（经典）

## 概述

本文提供了有关如何通过使用 PowerShell cmdlet 在 Azure 中创建 SQL Server 虚拟机的步骤。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-machines-windows-classic-extensions-customscript/)。

## 安装和配置 PowerShell

1. 如果你没有 Azure 帐户，请访问 [Azure 试用](/pricing/1rmb-trial/)。

2. [安装最新的 Azure PowerShell cmdlet](/documentation/articles/powershell-install-configure/)。

3. 安装完毕之后，打开 Windows PowerShell。

4. 使用 Add-AzureAccount 命令连到 PowerShell 到你的 Azure 订阅.

		Add-AzureAccount -Environment AzureChinaCloud

## 确定你的目标 Azure 区域

将在位于特定 Azure 区域的云服务中托管你的 SQL Server 虚拟机。以下步骤帮助你确定要为本教程其余部分使用的区域、存储帐户和云服务。

1. 确定你要用于托管 SQL Server VM 的数据中心。以下 PowerShell 命令将显示可用区域的详细信息，末尾提供摘要列表。

		Get-AzureLocation
		(Get-AzureLocation).Name

2.  确定首选位置后，为该区域设置一个名为 **$dcLocation** 的变量。

		$dcLocation = "<region name>"

## 设置订阅和存储帐户

1. 确定将用于新虚拟机的 Azure 订阅。

		(Get-AzureSubscription).SubscriptionName

1. 将你的目标 Azure 订阅分配到 **$subscr** 变量。然后将此变量设置为当前 Azure 订阅。

		$subscr="<subscription name>"
		Select-AzureSubscription -SubscriptionName $subscr -Current

1. 然后检查现有存储帐户。以下脚本显示所选区域中存在的所有存储帐户：

		(Get-AzureStorageAccount | where { $_.GeoPrimaryLocation -eq $dcLocation }).StorageAccountName

	>[AZURE.NOTE] 如果需要新存储帐户，请首先使用 New-AzureStorageAccoun 命令创建一个全部小写的存储帐户名称，如以下示例中所示：**New-AzureStorageAccount -StorageAccountName "<storage account name>" -Location $dcLocation**

1. 将目标存储帐户名称分配给 **$staccount**。然后使用 **Set-azuresubscription** 设置订阅和当前存储帐户。

		$staccount="<storage account name>"
		Set-AzureSubscription -SubscriptionName $subscr -CurrentStorageAccountName $staccount

## 选择 SQL Server 虚拟机映像

1. 从库中找出可用的 SQL Server 虚拟机映像的列表。这些映像都具有以“SQL”开头的 **ImageFamily** 属性。下面的查询显示你可用的预安装 SQL Server 的映像系列。

		Get-AzureVMImage | where { $_.ImageFamily -like "SQL*" } | select ImageFamily -Unique | Sort-Object -Property ImageFamily

1. 当发现虚拟机映像系列时，该系列中可能有多个已发布的映像。使用以下脚本查找你所选的映像系列的最新已发布虚拟机映像名称（如 **Windows Server 2012 R2 上的 SQL Server 2014 SP1 Enterprise**）：

		$family="<ImageFamily value>"
		$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

		echo "Selected SQL Server image name:"
		echo "   $image"

## 创建虚拟机

最后，使用 PowerShell 创建虚拟机：

1. 创建一个云服务托管新 VM。请注意，还可以改用现有云服务。使用云服务的短名称创建新变量 **$svcname**。

		$svcname = "<cloud service name>"
		New-AzureService -ServiceName $svcname -Label $svcname -Location $dcLocation

2. 指定虚拟机名称和大小。有关虚拟机大小的更多信息，请参见[适用于 Azure 的虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。

		$vmname="<machine name>"
		$vmsize="<Specify a valid machine size>" # see the link to virtual machine sizes
		$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

3. 制定本地管理员帐户和密码。

		$cred=Get-Credential -Message "Type the name and password of the local administrator account."
		$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

4. 运行以下脚本来创建虚拟机。

		New-AzureVM -ServiceName $svcname -VMs $vm1

>[AZURE.NOTE] 有关更多说明和配置选项，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-classic-create-powershell/)中的**构建你的命令集**部分。

## PowerShell 脚本示例

以下脚本提供了一个完整脚本的示例，该脚本在 **Windows Server 2012 R2 虚拟机上创建一个 SQL Server 2014 SP1 Enterprise**。如果你使用此脚本，你必须基于本主题中的之前步骤自定义初始变量。

	# Customize these variables based on your settings and requirements:
	$dcLocation = "China East"
	$subscr="mysubscription"
	$staccount="mystorageaccount"
	$family="SQL Server 2014 SP1 Enterprise on Windows Server 2012 R2"
	$svcname = "mycloudservice"
	$vmname="myvirtualmachine"
	$vmsize="A5"

	# Set the current subscription and storage account
	# Comment out the New-AzureStorageAccount line if the account already exists
	Select-AzureSubscription -SubscriptionName $subscr -Current
	New-AzureStorageAccount -StorageAccountName $staccount -Location $dcLocation
	Set-AzureSubscription -SubscriptionName $subscr -CurrentStorageAccountName $staccount

	# Select the most recent VM image in this image family:
	$image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

	# Create the new cloud service; comment out this line if cloud service exists already:
	New-AzureService -ServiceName $svcname -Label $svcname -Location $dcLocation

	# Create the VM config:
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image

	# Set administrator credentials:
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

	# Create the SQL Server VM:
	New-AzureVM -ServiceName $svcname -VMs $vm1


## 使用远程桌面进行连接

1. 在当前用户的文档文件夹中创建 .RDP 文件，以启动这些虚拟机完成安装：

		$documentspath = [environment]::getfolderpath("mydocuments")
		Get-AzureRemoteDesktopFile -ServiceName $svcname -Name $vmname -LocalPath "$documentspath\vm1.rdp"

1. 在文档目录中启动 RDP 文件。使用之前提供的管理员用户名和密码进行连接（例如，如果你的用户名称是 VMAdmin，请指定“\\VMAdmin”作为用户并提供密码）。

		.\vm1.rdp

## 为远程访问完成 SQL Server 计算机的配置

在通过远程桌面登录计算机之后，根据[在 Azure VM 中配置 SQL Server 连接的步骤](/documentation/articles/virtual-machines-windows-classic-sql-connect/#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)中的说明配置 SQL Server。

## 后续步骤

你可以在[虚拟机文档](/documentation/articles/virtual-machines-windows-classic-create-powershell/)中找到使用 PowerShell 设置虚拟机的其他说明。有关与 SQL Server 和高级存储相关的其他脚本，请参阅[将 Azure 高级存储用于虚拟机上的 SQL Server](/documentation/articles/virtual-machines-windows-classic-sql-server-premium-storage/)。

在许多情况下，下一步是将数据库迁移到这个新 SQL Server VM。有关数据库迁移指南，请参阅[将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-windows-migrate-sql/)。

除了这些资源外，我们还建议你查看[与在 Azure 虚拟机中运行 SQL Server 相关的其他主题](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0215_2016-->