<!-- not suitable for Mooncake -->

<properties
    pageTitle="通过 PowerShell 迁移到 Resource Manager | Azure"
    description="本文详述了如何在支持的平台上使用 Azure PowerShell 命令将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
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
    ms.date="10/19/2016"
    wacn.date=""
    ms.author="cynthn" />

# 使用 Azure PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager
以下步骤演示了如何使用 Azure PowerShell 命令将基础结构即服务 (IaaS) 资源从经典部署模型迁移到 Azure Resource Manager 部署模型。

也可根据需要通过 [Azure 命令行接口 (Azure CLI)](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/) 迁移资源。

* 如需了解受支持的迁移方案的背景信息，请参阅 [Platform-supported migration of IaaS resources from classic to Azure Resource Manager](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)（平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移）。
* 如需详细的指南和迁移演练，请参阅 [Technical deep dive on platform-supported migration from classic to Azure Resource Manager](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)（从技术方面深入探讨如何在支持的平台上完成从经典部署模型到 Azure Resource Manager 部署模型的迁移）。
* [查看最常见的迁移错误](/documentation/articles/virtual-machines-migration-errors/)

## 步骤 1：做好迁移规划
下面是建议你在将 IaaS 资源从经典部署模型迁移到 Resource Manager 部署模型时遵循的一些最佳实践：

* 通读[受支持的和不受支持的功能和配置](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)。如果虚拟机使用不受支持的配置或功能，建议你等到我们宣布支持该配置/功能时再进行迁移。也可根据需要删除该功能或移出该配置，以利迁移进行。
* 如果你通过自动化脚本来部署目前的基础结构和应用程序，则可尝试使用这些脚本进行迁移，以便创建类似的测试性设置。也可以使用 Azure 门户预览设置示例环境。

> [AZURE.IMPORTANT]
目前不支持通过 ExpressRoute 网关和应用程序网关从经典部署模型迁移到 Resource Manager 部署模型。若要使用 ExpressRoute 网关或应用程序网关迁移经典虚拟网络，请在运行提交操作前删除网关，以便移动网络（可以在不删除 ExpressRoute 网关或应用程序网关的情况下运行准备步骤）。完成迁移后，请在 Azure Resource Manager 中重新连接网关。
> 
> 

## 步骤2：安装最新版本的 Azure PowerShell
主要有两个安装 Azure PowerShell 的选项：[PowerShell 库](https://www.powershellgallery.com/profiles/azure-sdk/)和 [Web 平台安装程序 (WebPI)](http://aka.ms/webpi-azps)。WebPI 接收每月的更新。PowerShell 库会持续接收更新。本文基于 Azure PowerShell 2.1.0 版。

如需安装说明，请参阅 [How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）。

<br>  


## 步骤 3：设置订阅并针对迁移进行注册
首先，请启动 PowerShell 提示符。对于迁移，需要针对经典部署模型和 Resource Manager 部署模型设置环境。

登录到 Resource Manager 模型的帐户。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

使用以下命令获取可用订阅：

        Get-AzureRMSubscription | Sort SubscriptionName | Select SubscriptionName

设置当前会话的 Azure 订阅。此示例将默认订阅名称设置为 **My Azure Subscription**。使用自己的订阅名称替换示例名称。

        Select-AzureRmSubscription -SubscriptionName "My Azure Subscription"

> [AZURE.NOTE]
注册是一次性步骤，但必须在尝试迁移之前完成。如果不注册，则会出现以下错误消息：
> 
> *错误请求：未针对迁移注册订阅。*
> 
> 

使用以下命令向迁移资源提供程序注册：

        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

请等五分钟让注册完成。可以使用以下命令来检查审批状态：

        Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

请确保在继续操作之前，RegistrationState 为 `Registered`。

现在，请登录到经典模型的帐户。

        Add-AzureAccount -Environment AzureChinaCloud

使用以下命令获取可用订阅：

        Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName

设置当前会话的 Azure 订阅。此示例将默认订阅设置为 **My Azure Subscription**。使用自己的订阅名称替换示例名称。

        Select-AzureSubscription -SubscriptionName "My Azure Subscription"

<br>  


## 步骤 4：确保在当前部署或 VNET 的 Azure 区域中有足够的 Azure Resource Manager 虚拟机核心
可以使用以下 PowerShell 命令检查 Azure Resource Manager 中目前的核心数量。若要了解有关核心配额的详细信息，请参阅[限制和 Azure Resource Manager](/documentation/articles/azure-subscription-service-limits/#limits-and-the-azure-resource-manager)。

此示例检查**中国北部**区域的可用性。使用自己的区域名称替换示例名称。

    Get-AzureRmVMUsage -Location "China North"

## 步骤 5：运行迁移 IaaS 资源的命令
> [AZURE.NOTE]
此处描述的所有操作都是幂等的。如果你遇到功能不受支持或配置错误以外的问题，建议你重试准备、中止或提交操作。然后，平台会尝试再次操作。
> 
> 

### 迁移云服务中的虚拟机（不在虚拟网络中）
使用以下命令获取云服务列表，然后选取要迁移的云服务。如果云服务中的 VM 在虚拟网络中或者具有 Web 角色或辅助角色，该命令会返回错误消息。

        Get-AzureService | ft Servicename

获取云服务的部署名称。在此示例中，服务名称是 **My Service**。使用自己的服务名称替换示例名称。

        $serviceName = "My Service"
        $deployment = Get-AzureDeployment -ServiceName $serviceName
        $deploymentName = $deployment.DeploymentName

准备迁移云服务中的虚拟机。可以从两个选项中进行选择。

* **选项 1.将 VM 迁移到平台所创建的虚拟网络上**
  
    首先，使用以下命令验证用户是否可以迁移云服务：

        $validate = Move-AzureService -Validate -ServiceName $serviceName `
            -DeploymentName $deploymentName -CreateNewVirtualNetwork
        $validate.ValidationMessages

前一个命令会显示任何阻止迁移的警告和错误。如果验证成功，则可继续执行**准备**步骤：

        Move-AzureService -Prepare -ServiceName $serviceName `
            -DeploymentName $deploymentName -CreateNewVirtualNetwork

* **选项 2.迁移到 Resource Manager 部署模型中的现有虚拟网络**
  
    此示例将资源组名称设置为 **myResourceGroup**，将虚拟网络名称设置为 **myVirtualNetwork**，将子网名称设置为 **mySubNet**。使用自己的资源名称替换示例名称。

        $existingVnetRGName = "myResourceGroup"
        $vnetName = "myVirtualNetwork"
        $subnetName = "mySubNet"

首先，使用以下命令验证用户是否可以迁移云服务：

        $validate = Move-AzureService -Validate -ServiceName $serviceName `
            -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
        $validate.ValidationMessages

前一个命令会显示任何阻止迁移的警告和错误。如果验证成功，则可继续执行以下准备步骤：

        Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
            -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
            -VirtualNetworkName $vnetName -SubnetName $subnetName

使用前述任一选项成功完成准备操作以后，即可查询 VM 的迁移状态。确保 VM 处于“`Prepared`”状态。

此示例将 VM 名称设置为 **myVM**。使用自己的 VM 名称替换示例名称。

        $vmName = "myVM"
        $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
        $vm.VM.MigrationState

使用 PowerShell 或 Azure 门户预览，检查准备就绪的资源的配置。如果尚未做好迁移准备，因此想要回到旧的状态，请使用以下命令：

        Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName

如果准备好的配置看起来没问题，则可继续进行，使用以下命令提交资源：

        Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName

### 迁移虚拟网络中的虚拟机
迁移虚拟网络即可迁移其中的虚拟机。虚拟机随网络自动迁移。选取要迁移的虚拟网络。

此示例将虚拟网络名称设置为 **myVnet**。使用自己的虚拟网络名称替换示例名称。

        $vnetName = "myVnet"

> [AZURE.NOTE]
如果虚拟网络包含的 Web 角色/辅助角色或 VM 的配置不受支持，则会出现验证错误消息。
> 
> 

首先，请使用以下命令验证用户是否可以迁移虚拟网络：

        Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName

前一个命令会显示任何阻止迁移的警告和错误。如果验证成功，则可继续执行以下准备步骤：

        Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName

使用 Azure PowerShell 或 Azure 门户预览，检查准备就绪的虚拟机的配置。如果尚未做好迁移准备，因此想要回到旧的状态，请使用以下命令：

        Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName

如果准备好的配置看起来没问题，则可继续进行，使用以下命令提交资源：

        Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName

### 迁移存储帐户
完成虚拟机迁移之后，建议迁移存储帐户。

使用以下命令准备要迁移的每个存储帐户。在此示例中，存储帐户名称为 **myStorageAccount**。使用自己的存储帐户名称替换示例名称。

        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName

使用 Azure PowerShell 或 Azure 门户预览，检查准备就绪的存储帐户的配置。如果尚未做好迁移准备，因此想要回到旧的状态，请使用以下命令：

        Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName

如果准备好的配置看起来没问题，则可继续进行，使用以下命令提交资源：

        Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName

## 后续步骤
* 有关迁移的详细信息，请参阅 [Platform-supported migration of IaaS resources from classic to Azure Resource Manager](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)（平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移）。
* 若要使用 Azure PowerShell 将其他网络资源迁移到 Resource Manager，请通过 [Move-AzureNetworkSecurityGroup](https://msdn.microsoft.com/zh-cn/library/mt786729.aspx)、[Move-AzureReservedIP](https://msdn.microsoft.com/zh-cn/library/mt786752.aspx) 和 [Move-AzureRouteTable](https://msdn.microsoft.com/zh-cn/library/mt786718.aspx) 执行类似的步骤。
* 若要通过开源脚本将 Azure 资源从经典部署模型迁移到 Resource Manager 部署模型，请参阅[迁移到 Azure Resource Manager 时使用的社区工具](/documentation/articles/virtual-machines-windows-migration-scripts/)

<!---HONumber=Mooncake_1219_2016-->