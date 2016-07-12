<!-- Remove vm-migration -->
<properties
   pageTitle="Resource Manager 和经典部署 | Azure"
   description="介绍资源管理器部署模型与经典（或服务管理）部署模型之间的差异。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.date="05/13/2016"
   wacn.date="06/27/2016"/>

# Azure Resource Manager 与经典部署：了解部署模型和资源的状态

在本主题中，你将了解 Azure Resource Manager 和经典部署模型、资源的状态，以及为何要使用不同的模型来部署资源。Resource Manager 部署模型包含与经典部署模型的重要差异，这两个模型彼此之间并不完全兼容。若要简化资源部署和管理，Microsoft 建议您为新资源使用资源管理器，如果可能，请通过资源管理器重新部署现有资源。

对于大多数资源，你可以过渡到 Resource Manager 而不会出现任何问题。但是，由于模型之间的体系结构差异，一些资源提供程序会提供两个版本的资源（一个用于经典模型，一个用于资源管理器）。区分两个模型的资源提供程序包括：

- **计算** - 对虚拟机和可选可用性集的实例提供支持。
- **存储** - 对所需的存储帐户提供支持，存储帐户存储虚拟机的 VHD，包括其操作系统和其附加的数据磁盘。
- **网络** - 对所需的 NIC、虚拟机 IP 地址和虚拟网络内的子网及可选的负载平衡器、负载平衡器 IP 地址和网络安全组提供支持。

对于这些资源类型，您必须知道使用的是哪个版本，因为支持的操作会有所不同。<! -- 如果你已准备好将资源从经典部署迁移到 Resource Manager 部署，请参阅 [Platform supported migration of IaaS resources from Classic to Azure Resource Manager（平台支持从经典部署迁移到 Azure Resource Manager 部署的 IaaS 资源）](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)。 -->

若要了解应使用哪种模型来部署资源，让我们回顾这两个模型。

## 资源管理器的特性

通过资源管理器创建的资源具备以下共同特性：

- 通过以下方法之一创建：
  - 对于低于 1.0 的 Azure PowerShell 版本，命令将在 **AzureResourceManager** 模式下运行。

            PS C:\> Switch-AzureMode -Name AzureResourceManager

  - 对于 Azure PowerShell 1.0，请使用命令的资源管理器版本。这些命令采用 *Verb-AzureRmNoun* 格式，如下所示。

            PS C:\> Get-AzureRmResourceGroupDeployment

  - 适用于 REST 操作的 [Azure 资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)。
  - Azure CLI 命令在 **arm** 模式下运行。

            azure config mode arm

此外，资源提供程序内各个资源之间还存在相互关系：

- 虚拟机依赖在 SRP 中定义的具体存储帐户，在 Blob 存储中存储其磁盘（必需）。
- 虚拟机引用在 NRP 中定义的具体 NIC（必需）和在 CRP 中定义的可用性集（可选）。
- NIC 引用虚拟机的指定 IP 地址（必需）、虚拟机的虚拟网络的子网（必需）和网络安全组（可选）。
- 虚拟网络内的子网引用网络安全组（可选）。
- 负载平衡器实例引用后端 IP 地址池，包括虚拟机的 NIC（可选）。

## 经典部署的特性

在 Azure 服务管理中，宿主虚拟机的计算、存储或网络资源由以下各项提供：

- 一项必不可少的云服务，用作宿主虚拟机的容器（计算）。虚拟机自动配备一个网络接口卡 (NIC) 并由 Azure 分配的 IP 地址。此外，云服务包含一个外部负载平衡器实例、一个公共 IP 地址以及若干默认终结点，以支持远程桌面、针对 Windows 虚拟机的远程 PowerShell 流量和针对 Linux 虚拟机的 Secure Shell (SSH) 流量。
- 一个必不可少的存储帐户，存储虚拟机的 VHD，包括操作系统、临时文件和附加的数据磁盘（存储）。
- 一个可选的虚拟网络，用作额外的容器，可以在其中创建子网结构并指定虚拟机所在的子网（网络）。

在经典部署模型中创建的资源具有以下共同特性：

- 通过以下方法之一创建：

  - [Azure 门户](https://manage.windowsazure.cn)

        ![Azure portal](./media/resource-manager-deployment-model/azure-portal.png)



  - 对于低于 1.0 的 Azure PowerShell 版本，命令在 **AzureServiceManagement** 模式下运行（这是默认模式，因此，如果不特意切换到 AzureResourceManager，则会在 AzureServiceManagement 模式下运行）。

            PS C:\> Switch-AzureMode -Name AzureServiceManagement

  - 对于 Azure PowerShell 1.0，请使用命令的服务管理版本。这些命令采用 *Verb-AzureNoun* 格式，如下所示。

            PS C:\> Get-AzureDeployment

  - 适用于 REST 操作的[服务管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)。
  - Azure CLI 命令在 **asm** 或默认模式下运行。

你仍可以使用 Azure 门户来管理通过经典部署创建的资源。

## 使用资源管理器和资源组的好处

资源管理器添加了资源组的概念。通过资源管理器创建的每个资源都存在于资源组内。资源管理器部署模型提供了以下几个好处：

- 您可以以群组形式部署、管理和监视解决方案的所有服务，而不是单独处理这些服务。
- 您可以在整个应用程序生命周期内重复部署应用程序，并确保以一致的状态部署资源。
- 您可以使用声明性模板来定义您的部署。
- 您可以定义各资源之间的依赖关系，以便按正确的顺序进行部署。
- 你可以向资源组中的所有资源应用访问控制，因为基于角色的访问控制 (RBAC) 已本机集成到管理平台。
- 您可以将标记应用到资源，以按照逻辑组织订阅中的所有资源。


在资源管理器之前，通过经典部署创建的每个资源不存在于资源组中。当添加资源管理器时，所有资源都追溯性地添加到默认资源组。如果你现在通过经典部署创建资源，资源将自动在该服务的默认资源组中创建，即使在部署时未指定该资源组也是如此。但是，仅存在于资源组内并不意味着资源已转换为资源管理器模型。对于虚拟机、存储空间和虚拟网络，如果资源是通过经典部署创建的，则你必须继续通过经典操作对其进行操作。

您可以将资源移到不同的资源组，并将新资源添加到现有的资源组。因此，您的资源组可以包含通过资源管理器和经典部署创建的混合资源。此资源组合会产生意外的结果，因为资源不支持相同的操作。

通过使用声明性模板，您可以简化用于部署的脚本。不要尝试将现有脚本从服务管理转换到资源管理器，而是考虑重新处理您的部署策略以充分利用在模板中定义基础结构和配置的功能。

## 使用标记

标记使您能够按照逻辑组织您的资源。只有通过资源管理器创建的资源才支持标记。您不能将标记应用到经典资源。

有关在资源管理器中使用标记的详细信息，请参阅[使用标记来组织 Azure 资源](/documentation/articles/resource-group-using-tags/)。

## 部署模型支持的操作

在经典部署模型中创建的资源不支持资源管理器操作。在某些情况下，资源管理器命令可以检索通过经典部署创建的资源的相关信息，或者可以执行管理任务，例如将经典资源移动到另一个资源组，但这些情况下，并不意味着该类型支持资源管理器操作。例如，假定您有一个资源组包含通过资源管理器和经典模型创建的虚拟机。如果您运行以下 PowerShell 命令，您将看到所有虚拟机：

    PS C:\> Get-AzureRmResourceGroup -Name ExampleGroup
    ...
    Resources :
     Name                 Type                                          Location
     ================     ============================================  ========
     ExampleClassicVM     Microsoft.ClassicCompute/domainNames          eastus
     ExampleClassicVM     Microsoft.ClassicCompute/virtualMachines      eastus
     ExampleResourceVM    Microsoft.Compute/virtualMachines             eastus
    ...

但是，如果运行 Get-AzureRmVM 命令，则只能获取通过资源管理器创建的虚拟机。

    PS C:\> Get-AzureRmVM -ResourceGroupName ExampleGroup
    ...
    Id       : /subscriptions/xxxx/resourceGroups/ExampleGroup/providers/Microsoft.Compute/virtualMachines/ExampleResourceVM
    Name     : ExampleResourceVM
    ...

通常情况下，不应期望通过经典部署创建的资源使用资源管理器命令。

在使用通过资源管理器创建的资源时，您应使用资源管理器操作，而不是切换回服务管理操作。

## 有关虚拟机的注意事项

使用虚拟机时有一些重要注意事项。

- 使用经典部署模型部署的虚拟机不能包含在使用资源管理器部署的虚拟网络中。
- 使用资源管理器部署模型部署的虚拟机必须包含在虚拟网络中。
- 使用经典部署模型部署的虚拟机不一定要包括在虚拟网络中。

如果你可以承受虚拟机停机带来的损失，则可以使用 [ASM2ARM PowerShell 脚本](https://github.com/fullscale180/asm2arm)将虚拟机从经典部署过渡到资源管理器部署。

有关从经典部署转换到资源管理器时的等效 Azure CLI 命令列表，请参阅 [VM 操作的等效资源管理器和服务管理命令](/documentation/articles/virtual-machines-windows-cli-manage/)。

## 后续步骤

- 若要了解如何创建声明性部署模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。
- 若要查看用于部署模板的命令，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_0620_2016-->