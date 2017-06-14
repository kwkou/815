<properties
    pageTitle="通过 PowerShell 迁移到 Resource Manager | Azure"
    description="本文介绍如何在支持的平台上使用 Azure PowerShell 命令将 IaaS 资源（例如虚拟机 (VM)、虚拟网络 (VNET) 和存储帐户）从经典部署模型迁移到 Azure Resource Manager (ARM) 部署模型"
    services="virtual-machines-windows"
    documentationcenter=""
    author="singhkays"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="2b3dff9b-2e99-4556-acc5-d75ef234af9c"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/30/2017"
    wacn.date="05/15/2017"
    ms.author="kasing"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="3c7158e6328581d8efb6e9cf1ad46c2553332ebb"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-azure-powershell"></a>使用 Azure PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager
以下步骤演示了如何使用 Azure PowerShell 命令将基础结构即服务 (IaaS) 资源从经典部署模型迁移到 Azure Resource Manager 部署模型。 

也可根据需要通过 [Azure 命令行接口 (Azure CLI)](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/) 迁移资源。

* 如需了解受支持的迁移方案的背景信息，请参阅[平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)。 
* 如需详细的指南和迁移演练，请参阅[从技术方面深入探讨如何在支持的平台上完成从经典部署模型到 Azure Resource Manager 部署模型的迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)。
* [查看最常见的迁移错误](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-errors/)

<br>
下面是一个流程图，用于确定在迁移过程中需要执行步骤的顺序

![Screenshot that shows the migration steps](./media/virtual-machines-windows-migration-classic-resource-manager/migration-flow.png)

## <a name="step-1-plan-for-migration"></a>步骤 1：做好迁移规划
下面是建议你在将 IaaS 资源从经典部署模型迁移到 Resource Manager 部署模型时遵循的一些最佳实践：

* 通读[受支持的和不受支持的功能和配置](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)。 如果虚拟机使用不受支持的配置或功能，建议你等到我们宣布支持该配置/功能时再进行迁移。 也可根据需要删除该功能或移出该配置，以利迁移进行。
* 如果你通过自动化脚本来部署目前的基础结构和应用程序，则可尝试使用这些脚本进行迁移，以便创建类似的测试性设置。 也可以使用 Azure 门户设置示例环境。

> [AZURE.IMPORTANT]
> 目前不支持通过应用程序网关从经典部署模型迁移到 Resource Manager 部署模型。 若要迁移带应用程序网关的经典虚拟网络，请先删除该网关，然后运行准备操作来移动网络。 完成迁移后，在 Azure Resource Manager 中重新连接该网关。 
><p>
>无法自动迁移其他订阅中连接到 ExpressRoute 线路的 ExpressRoute 网关。 此类情况下，请删除 ExpressRoute 网关、迁移虚拟网络并重新创建网关。 有关详细信息，请参阅[将 ExpressRoute 线路和关联的虚拟网络从经典部署模型迁移到 Resource Manager 部署模型](/documentation/articles/expressroute-migration-classic-resource-manager/)。
> 
> 

## <a name="step-2-install-the-latest-version-of-azure-powershell"></a>步骤2：安装最新版本的 Azure PowerShell
安装 Azure PowerShell 可以通过两个主要的选项：[PowerShell 库](https://www.powershellgallery.com/profiles/azure-sdk/)或 [Web 平台安装程序 (WebPI)](http://aka.ms/webpi-azps)。 WebPI 接收每月的更新。 PowerShell 库会持续接收更新。 本文基于 Azure PowerShell 2.1.0 版。

如需安装说明，请参阅 [How to install and configure Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell）。

<br>

## <a name="step-3-ensure-that-you-are-co-administrator-for-the-subscription-in-azure-classic-management-portal"></a>步骤 3：确保你是 Azure 经典管理门户中订阅的共同管理员
若要执行此迁移，必须在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中将你添加为订阅的共同管理员。 即使你已被添加为 [Azure 门户](https://portal.azure.cn)中的所有者，这仍是必需的操作。 尝试在 Azure 经典管理门户中添加订阅的共同管理员，确定你是否是订阅的共同管理员。 如果不能添加协同管理员，请联系订阅的服务管理员或协同管理员，以将自己添加为协同管理员。   

## <a name="step-4-set-your-subscription-and-sign-up-for-migration"></a>步骤 4：设置订阅并针对迁移进行注册
首先，请启动 PowerShell 提示符。 对于迁移，需要针对经典部署模型和 Resource Manager 部署模型设置环境。

登录到 Resource Manager 模型的帐户。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

使用以下命令获取可用订阅：

        Get-AzureRMSubscription | Sort SubscriptionName | Select SubscriptionName

设置当前会话的 Azure 订阅。 此示例将默认订阅名称设置为 **My Azure Subscription**。 使用自己的订阅名称替换示例名称。 

        Select-AzureRmSubscription -SubscriptionName "My Azure Subscription"

> [AZURE.NOTE]
> 注册是一次性步骤，但必须在尝试迁移之前完成。 如果不注册，则会出现以下错误消息： 
> <p>
> *错误请求：未针对迁移注册订阅。* 
> 
> 

使用以下命令向迁移资源提供程序注册：

        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

请等五分钟让注册完成。 可以使用以下命令来检查审批状态：

        Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

请确保在继续操作之前，RegistrationState 为 `Registered` 。 

现在，请登录到经典模型的帐户。

        Add-AzureAccount -Environment AzureChinaCloud

使用以下命令获取可用订阅：

        Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName

设置当前会话的 Azure 订阅。 此示例将默认订阅设置为 **My Azure Subscription**。 将示例订阅名称替换为你自己的名称。 

        Select-AzureSubscription -SubscriptionName "My Azure Subscription"

<br>

## <a name="step-5-make-sure-you-have-enough-azure-resource-manager-virtual-machine-cores-in-the-azure-region-of-your-current-deployment-or-vnet"></a>步骤 5：确保在当前部署或 VNET 的 Azure 区域中有足够的 Azure Resource Manager 虚拟机核心
可以使用以下 PowerShell 命令检查 Azure Resource Manager 中目前的核心数量。 若要了解有关核心配额的详细信息，请参阅[限制和 Azure Resource Manager](/documentation/articles/azure-subscription-service-limits/#limits-and-the-azure-resource-manager)。 

此示例检查 **中国北部** 区域的可用性。 使用自己的区域名称替换示例名称。 

    Get-AzureRmVMUsage -Location "China North"

## <a name="step-6-run-commands-to-migrate-your-iaas-resources"></a>步骤 6：运行迁移 IaaS 资源的命令
> [AZURE.NOTE]
> 此处描述的所有操作都是幂等的。 如果你遇到功能不受支持或配置错误以外的问题，建议你重试准备、中止或提交操作。 然后，平台会尝试再次操作。
> 
> 

## <a name="step-61-migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network"></a>步骤 6.1：迁移云服务中的虚拟机（不在虚拟网络中）
使用以下命令获取云服务列表，然后选取要迁移的云服务。 如果云服务中的 VM 在虚拟网络中或者具有 Web 角色或辅助角色，该命令会返回错误消息。

        Get-AzureService | ft Servicename

获取云服务的部署名称。 在此示例中，服务名称是 **My Service**。 使用自己的服务名称替换示例名称。 

        $serviceName = "My Service"
        $deployment = Get-AzureDeployment -ServiceName $serviceName
        $deploymentName = $deployment.DeploymentName

准备迁移云服务中的虚拟机。 可以从两个选项中进行选择。

* **选项 1.将 VM 迁移到平台所创建的虚拟网络上**

    首先，使用以下命令验证用户是否可以迁移云服务：

        $validate = Move-AzureService -Validate -ServiceName $serviceName `
            -DeploymentName $deploymentName -CreateNewVirtualNetwork
        $validate.ValidationMessages

    前一个命令会显示任何阻止迁移的警告和错误。 如果验证成功，可继续执行“准备”步骤：

        Move-AzureService -Prepare -ServiceName $serviceName `
            -DeploymentName $deploymentName -CreateNewVirtualNetwork

* **选项 2.迁移到 Resource Manager 部署模型中的现有虚拟网络**

    此示例将资源组名称设置为 **myResourceGroup**、将虚拟网络名称设置为 **myVirtualNetwork** 并将子网名称设置为 **mySubNet**。 将该示例中的名称替换为你自己的资源名称。

        $existingVnetRGName = "myResourceGroup"
        $vnetName = "myVirtualNetwork"
        $subnetName = "mySubNet"

    首先，请使用以下命令验证用户是否可以迁移虚拟网络：

        $validate = Move-AzureService -Validate -ServiceName $serviceName `
            -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
        $validate.ValidationMessages

    前一个命令会显示任何阻止迁移的警告和错误。 如果验证成功，则可继续执行以下准备步骤：

        Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
            -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
            -VirtualNetworkName $vnetName -SubnetName $subnetName

使用前述任一选项成功完成准备操作以后，即可查询 VM 的迁移状态。 确保 VM 处于“ `Prepared` ”状态。

此示例将 VM 名称设置为 **myVM**。 使用自己的 VM 名称替换示例名称。

        $vmName = "myVM"
        $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
        $vm.VM.MigrationState

使用 PowerShell 或 Azure 门户，检查准备就绪的资源的配置。 如果尚未做好迁移准备，因此想要回到旧的状态，请使用以下命令：

        Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName

如果准备好的配置看起来没问题，则可继续进行，使用以下命令提交资源：

        Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName

## <a name="step-62-migrate-virtual-machines-in-a-virtual-network"></a>步骤 6.2：迁移虚拟网络中的虚拟机
若要迁移虚拟网络中的虚拟机，可迁移虚拟网络。 虚拟机随虚拟网络自动迁移。 选取要迁移的虚拟网络。 

此示例将虚拟网络名称设置为 **myVnet**。 使用自己的虚拟网络名称替换示例名称。 

        $vnetName = "myVnet"

> [AZURE.NOTE]
> 如果虚拟网络包含的 Web 角色/辅助角色或 VM 的配置不受支持，则会出现验证错误消息。
> 
> 

首先，请使用以下命令验证用户是否可以迁移虚拟网络：

        Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName

前一个命令会显示任何阻止迁移的警告和错误。 如果验证成功，则可继续执行以下准备步骤：

        Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName

使用 Azure PowerShell 或 Azure 门户，检查准备就绪的虚拟机的配置。 如果尚未做好迁移准备，因此想要回到旧的状态，请使用以下命令：

        Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName

如果准备好的配置看起来没问题，则可继续进行，使用以下命令提交资源：

        Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName

## <a name="step-63-migrate-a-storage-account"></a>步骤 6.3：迁移存储帐户
完成虚拟机迁移之后，建议迁移存储帐户。

在迁移存储帐户之前，请执行以下先决条件检查：

* **迁移其磁盘存储在存储帐户中的经典虚拟机**

    上述命令返回存储帐户中所有经典 VM 磁盘的 RoleName 和 DiskName 属性。 RoleName 是磁盘附加到的虚拟机的名称。 如果上述命令返回了磁盘，请确保先迁移这些磁盘所附加到的虚拟机，然后再迁移存储帐户。

         $storageAccountName = 'yourStorageAccountName'
          Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Select-Object -ExpandProperty AttachedTo -Property `
          DiskName | Format-List -Property RoleName, DiskName 


* **删除存储帐户中存储的未附加经典 VM 磁盘**

    使用以下命令查找存储帐户中未附加的经典 VM 磁盘： 

            $storageAccountName = 'yourStorageAccountName'
            Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Format-List -Property DiskName  


    如果上述命令返回了磁盘，请使用以下命令删除这些磁盘：

           Remove-AzureDisk -DiskName 'yourDiskName'

* **删除存储帐户中存储的 VM 映像**

    上述命令返回 OS 磁盘存储在该存储帐户中的所有 VM 映像。

            Get-AzureVmImage | Where-Object { $_.OSDiskConfiguration.MediaLink -ne $null -and $_.OSDiskConfiguration.MediaLink.Host.Contains($storageAccountName)`
                                    } | Select-Object -Property ImageName, ImageLabel

     上述命令返回数据磁盘存储在该存储帐户中的所有 VM 映像。

            Get-AzureVmImage | Where-Object {$_.DataDiskConfigurations -ne $null `
                                             -and ($_.DataDiskConfigurations | Where-Object {$_.MediaLink -ne $null -and $_.MediaLink.Host.Contains($storageAccountName)}).Count -gt 0 `
                                            } | Select-Object -Property ImageName, ImageLabel

    使用前面的命令删除上述命令返回的所有 VM 映像：

        Remove-AzureVMImage -ImageName 'yourImageName'

使用以下命令准备要迁移的每个存储帐户。 在此示例中，存储帐户名称为 **myStorageAccount**。 使用自己的存储帐户名称替换示例名称。 

        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName

使用 Azure PowerShell 或 Azure 门户，检查准备就绪的存储帐户的配置。 如果尚未做好迁移准备，因此想要回到旧的状态，请使用以下命令：

        Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName

如果准备好的配置看起来没问题，则可继续进行，使用以下命令提交资源：

        Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName

## <a name="next-steps"></a>后续步骤
* [平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移概述](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
* [有关平台支持的从经典部署模型到 Azure Resource Manager 部署模型的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
* [规划从经典部署模型到 Azure Resource Manager 的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-plan/)
* [使用 CLI 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/)
* [用于帮助将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的社区工具](/documentation/articles/virtual-machines-windows-migration-scripts/)
* [查看最常见的迁移错误](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-errors/)
* [查看有关将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的最常见问题](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-faq/)

<!--Update_Description: wording update-->