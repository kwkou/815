<properties
    pageTitle="在 Azure PowerShell 创建 SQL Server 虚拟机（经典）| Azure"
    description="提供用于创建具有 SQL Server 虚拟机库映像的 Azure VM的步骤和 PowerShell 脚本。本主题使用经典部署模式。"
    services="virtual-machines-windows"
    documentationcenter="na"
    author="rothja"
    manager="jhubbard"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="b73be387-9323-4e08-be53-6e5928e3786e"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="02/02/2017"
    wacn.date="03/28/2017"
    ms.author="jroth" />  


# 使用 Azure PowerShell 预配 SQL Server 虚拟机（经典）
## 概述
本文提供了使用 PowerShell cmdlet 在 Azure 中创建 SQL Server 虚拟机的步骤。

> [AZURE.IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

有关此主题中的 Resource Manager 版本，请参阅[使用 Azure PowerShell Resource Manager 预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-ps-sql-create/)。

### 安装和配置 PowerShell：
1. 如果你没有 Azure 帐户，请访问 [Azure 试用](/pricing/1rmb-trial/)。
2. [下载和安装最新 Azure PowerShell 命令](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。
3. 启动 Windows PowerShell，并通过 **Add-AzureAccount** 命令将其连接到 Azure 订阅。
   
        Add-AzureAccount -Environment AzureChinaCloud

## 确定目标 Azure 区域
将在位于特定 Azure 区域的云服务中托管你的 SQL Server 虚拟机。以下步骤帮助你确定将用于本教程其余部分的区域、存储帐户和云服务。

1. 确定要用于托管 SQL Server VM 的数据中心。以下 PowerShell 命令将显示可用区域的详细信息，末尾提供摘要列表。
   
        Get-AzureLocation
        (Get-AzureLocation).Name
2. 确定首选位置后，为该区域设置一个名为 **$dcLocation** 的变量。
   
        $dcLocation = "<region name>"

## 设置订阅和存储帐户
1. 确定将用于新虚拟机的 Azure 订阅。
   
        (Get-AzureSubscription).SubscriptionName
2. 将目标 Azure 订阅分配到 **$subscr** 变量。然后将此变量设置为当前 Azure 订阅。
   
        $subscr="<subscription name>"
        Select-AzureSubscription -SubscriptionName $subscr -Current
3. 然后检查现有存储帐户。以下脚本显示所选区域中存在的所有存储帐户：
   
        (Get-AzureStorageAccount | where { $_.GeoPrimaryLocation -eq $dcLocation }).StorageAccountName
   
    > [AZURE.NOTE]
    如果需要新的存储帐户，请先使用 New-AzureStorageAccoun 命令创建全部小写的存储帐户名称，如下例中所示：**New-AzureStorageAccount -StorageAccountName "\<storage account name\>" -Location $dcLocation**
    > 
    > 
4. 将目标存储帐户名称分配给 **$staccount**。然后使用 **Set-azuresubscription** 设置订阅和当前存储帐户。
   
        $staccount="<storage account name>"
        Set-AzureSubscription -SubscriptionName $subscr -CurrentStorageAccountName $staccount

## 选择 SQL Server 虚拟机映像
1. 从库中找出可用 SQL Server 虚拟机映像的列表。这些映像的 **ImageFamily** 属性都以“SQL”开头。下面的查询将显示已经预安装 SQL Server 的可用映像系列。
   
        Get-AzureVMImage | where { $_.ImageFamily -like "SQL*" } | select ImageFamily -Unique | Sort-Object -Property ImageFamily
2. 当发现虚拟机映像系列时，该系列中可能有多个已发布的映像。使用以下脚本查你所选映像系列所发布的最新虚拟机映像名称，例如 **Windows Server 2012 R2 上的 SQL Server 2016 RTM Enterprise**：
   
        $family="<ImageFamily value>"
        $image=Get-AzureVMImage | where { $_.ImageFamily -eq $family } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
   
        echo "Selected SQL Server image name:"
        echo "   $image"

## 创建虚拟机
最后，使用 PowerShell 创建虚拟机：

1. 创建一个云服务以托管新 VM。请注意，还可以改用现有云服务。使用云服务的短名称创建新变量 **$svcname**。
   
        $svcname = "<cloud service name>"
        New-AzureService -ServiceName $svcname -Label $svcname -Location $dcLocation
2. 指定虚拟机名称和大小。有关虚拟机大小的更多信息，请参阅[适用于 Azure 的虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。
   
        $vmname="<machine name>"
        $vmsize="<Specify one: Large, ExtraLarge, A5, A6, A7, or see the link to the other VM sizes>"
        $vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image
3. 指定本地管理员帐户和密码。
   
        $cred=Get-Credential -Message "Type the name and password of the local administrator account."
        $vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password
4. 运行以下脚本来创建虚拟机。
   
        New-AzureVM -ServiceName $svcname -VMs $vm1

> [AZURE.NOTE]
有关更多说明和配置选项，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-classic-create-powershell/)中的**构建你的命令集**部分。
> 
> 

## PowerShell 脚本示例
以下脚本提供了一个完整脚本的示例，该脚本在 **Windows Server 2012 R2 虚拟机上创建 SQL Server 2016 RTM Enterprise**。如果使用此脚本，则必须根据本主题中先前的步骤自定义初始变量。

    # Customize these variables based on your settings and requirements:
    $dcLocation = "China East"
    $subscr="mysubscription"
    $staccount="mystorageaccount"
    $family="SQL Server 2016 RTM Enterprise on Windows Server 2012 R2"
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
2. 在文档目录中启动 RDP 文件。使用之前提供的管理员用户名和密码进行连接（例如，如果用户名是 VMAdmin，则指定“\\VMAdmin”作为用户并提供密码）。
   
        cd $documentspath
        .\vm1.rdp

## 完成 SQL Server 计算机的配置以进行远程访问
在通过远程桌面登录计算机之后，根据[在 Azure VM 中配置 SQL Server 连接的步骤](/documentation/articles/virtual-machines-windows-classic-sql-connect/#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)中的说明配置 SQL Server。

## 后续步骤
你可以在[虚拟机文档](/documentation/articles/virtual-machines-windows-classic-create-powershell/)中找到使用 PowerShell 设置虚拟机的其他说明。有关与 SQL Server 和高级存储相关的其他脚本，请参阅[将 Azure 高级存储用于虚拟机上的 SQL Server](/documentation/articles/virtual-machines-windows-classic-sql-server-premium-storage/)。

在许多情况下，下一步是将数据库迁移到此新的 SQL Server VM。有关数据库迁移指南，请参阅[将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-windows-migrate-sql/)。

如果还希望使用 Azure 门户预览创建 SQL 虚拟机，请参阅[在 Azure 上预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)。请注意，本教程将引导你在门户中使用推荐的 Resource Manager 模型创建 VM（而非使用此 PowerShell 主题中所用的经典模型）。

除了上述资源，还建议查看[与在 Azure 虚拟机中运行 SQL Server 相关的其他主题](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->