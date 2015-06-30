<properties
   pageTitle="manage-vms-azure-powershell"
   description="使用 Azure PowerShell 管理 VM"
   services="virtual-machines"
   documentationCenter="windows"
   authors="singhkay"
   manager="timlt"
   editor=""/>
<tags ms.service="virtual-machines"
    ms.date="03/09/2015"
    wacn.date="04/15/2015"
    />

# 使用 Azure PowerShell 管理虚拟机

在开始之前，你需要确保你已安装了 Azure PowerShell。为此，请访问[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

## 获取映像

在创建 VM 之前，你需要决定**要使用的映像**。你可以使用以下 cmdlet 获取映像列表

      Get-AzureVMImage

此 cmdlet 将返回 Azure 中可用的所有映像的列表。这是一个非常长的列表，可能很难发现你要使用的确切映像。在下面的示例中，我将使用其他 PowerShell cmdlet 来将返回的映像列表减小为仅包含基于 **Windows Server 2012 R2 Datacenter** 的那些容器。此外，我还通过针对返回的映像数组指定 [-1] 来仅获取最新的映像

    $img = (Get-AzureVMImage | Select -Property ImageName, Label | where {$_.Label -like '*Windows Server 2012 R2 Datacenter*'})[-1]

## 创建 VM

VM 的创建通过 **New-AzureVMConfig** cmdlet 开始。在此，你将指定你的 VM 的**名称**、你的 VM 的 **大小**以及要用于该 VM 的**映像**。此 cmdlet 创建一个本地 VM 对象 **$myVM**，在本指南中稍后将使用其他 Azure PowerShell cmdlet 修改该对象。

      $myVM = New-AzureVMConfig -Name "testvm" -InstanceSize "Small" -ImageName $img.ImageName

接下来，你需要为你的 VM 选择**用户名**和**密码**。你可以使用 **Add-AzureProvisioningConfig** cmdlet 完成该操作。你在此处告诉 Azure 你的 VM 的 OS 是什么。注意，你还将对本地 **$myVM** 对象进行更改。

    $user = "azureuser"
    $pass = "&Azure1^Pass@"
    $myVM = Add-AzureProvisioningConfig -Windows -AdminUsername $user -Password $pass

最后，你将准备在 Azure 上注册你的 VM。若要执行此操作，你需要使用 **New-AzureVM** cmdlet

> [AZURE.NOTE] 在创建 VM 之前，你需要配置云服务。可通过两种方式来执行此操作。
* 使用 New-AzureService cmdlet 创建云服务。如果你选择了此方法，则需要确保在下面的 New-AzureVM cmdlet 中指定的位置与你的云服务的位置匹配，否则，New-AzureVM cmdlet 执行将失败。
* 请让 New-AzureVM cmdlet 替你完成此工作。你需要确保服务的名称是唯一的，否则，New-AzureVM cmdlet 执行将失败。

    New-AzureVM -ServiceName "mytestserv" -VMs $myVM -Location "China North"

**可选**

你可以使用其他 cmdlet（例如 **Add-AzureDataDisk**、**Add-AzureEndpoint**）来为你的 VM 配置其他选项

## 获取 VM
现在，你已经在 Azure 上创建了一个 VM，你肯定希望看看它在如何运行。你可以使用 **Get-AzureVM** cmdlet 执行此操作，如下所示。

    Get-AzureVM -ServiceName "mytestserv" -Name "testvm"


## 后续步骤
[使用 RDP 或 SSH 连接到 Azure 虚拟机](https://msdn.microsoft.com/zh-cn/library/azure/dn535788.aspx)<br>
[Azure 虚拟机常见问题解答](https://msdn.microsoft.com/zh-cn/library/azure/dn683781.aspx)

<!--HONumber=50-->