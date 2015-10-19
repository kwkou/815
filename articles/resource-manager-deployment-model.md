<properties
   pageTitle="了解资源管理器与经典部署模型之间的差异"
   description="介绍资源管理器部署模型与经典（或服务管理）部署模型之间的差异。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="08/20/2015"
   wacn.date="10/3/2015"/>

# 了解资源管理器部署和经典部署

资源管理器部署模型提供了一种新的方式来部署和管理构成应用程序的服务。这一新模型包含与经典部署模型的重要差异，这两个模型彼此之间并不完全兼容。若要简化资源部署和管理，Microsoft 建议您为新资源使用资源管理器，如果可能，请通过资源管理器重新部署现有资源。

您也可能知道经典部署模型就是服务管理模型。

本主题介绍了这两个模型之间的差异，以及从经典模型转换到资源管理器时可能遇到的一些问题。它提供了模型的概述，但并未详细介绍各项服务之间的差异。有关转换计算、存储和网络资源的详细信息，请参阅 [Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)。

许多资源可同时在经典模型和资源管理器中正常运行，不会出现问题。即使是在经典模型中创建的，这些资源也能完全支持资源管理器。您可以在无需任何顾虑或额外努力的情况下转换到资源管理器。

但是，由于模型之间的体系结构差异，一些资源提供程序会提供两个版本的资源（一个用于经典模型，一个用于资源管理器）。区分两个模型的资源提供程序包括：

- 计算
- 存储
- 网络

对于这些资源类型，您必须知道使用的是哪个版本，因为支持的操作会有所不同。

若要了解两个模型之间的体系结构差异，请参阅 [Azure 资源管理器体系结构](/documentation/articles/virtual-machines-azure-resource-manager-architecture)。

## 资源管理器的特性

通过资源管理器创建的资源具备以下共同特性：

- 通过以下方法之一创建：
<!--
  - [预览门户](https://portal.azure.com/)。

        ![preview portal](./media/resource-manager-deployment-model/preview-portal.png)

        For Compute, Storage, and Networking resources, you have the option of using either Resourece Manager or Classic deployment. Select **Resource Manager**.

        ![Resource Manager deployment](./media/resource-manager-deployment-model/select-resource-manager.png)
-->
  - PowerShell 命令在 **AzureResourceManager** 模式下运行。

            PS C:\> Switch-AzureMode -Name AzureResourceManager

  - 适用于 REST 操作的 [Azure 资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)。
  - Azure CLI 命令在 **arm** 模式下运行。

            azure config mode arm

- 资源类型的名称中不包括 **(经典)**。下图显示了**存储帐户**类型。

    ![Web 应用](./media/resource-manager-deployment-model/resource-manager-type.png)

## 经典部署的特性

在经典部署模型中创建的资源具有以下共同特性：

- 通过以下方法之一创建：

  - [Azure 门户](https://manage.windowsazure.com)

        ![Azure portal](./media/resource-manager-deployment-model/azure-portal.png)

        Or, the preview portal and you specify **Classic** deployment (for Compute, Storage, and Networking).

        ![Classic deployment](./media/resource-manager-deployment-model/select-classic.png)

  - PowerShell 命令在 **AzureServiceManagement** 模式下运行（这是默认模式，因此，如果不特意切换到 AzureResourceManager，您会在 AzureServiceManagement 模式下运行）。

            PS C:\> Switch-AzureMode -Name AzureServiceManagement

  - 适用于 REST 操作的[服务管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)。
  - Azure CLI 命令在 **asm** 或默认模式下运行。
- 资源类型的名称中包括 **(经典)**。下图显示了**存储帐户 (经典)** 类型。

    ![经典类型](./media/resource-manager-deployment-model/classic-type.png)

您仍可以使用预览门户来管理通过经典部署创建的资源。

## 使用资源管理器和资源组的好处

资源管理器添加了资源组的概念。通过资源管理器创建的每个资源都存在于资源组内。资源管理器部署模型提供了以下几个好处：

- 您可以以群组形式部署、管理和监视解决方案的所有服务，而不是单独处理这些服务。
- 您可以在整个应用程序生命周期内重复部署应用程序，并确保以一致的状态部署资源。
- 您可以使用声明性模板来定义您的部署。 
- 您可以定义各资源之间的依赖关系，以便按正确的顺序进行部署。
- 您可以将访问控制应用到资源组中的所有服务，因为基于角色的访问控制 (RBAC) 已在本机集成到管理平台。
- 您可以将标记应用到资源，以按照逻辑组织订阅中的所有资源。


在资源管理器之前，通过经典部署创建的每个资源不存在于资源组中。当添加资源管理器时，所有资源都追溯性地添加到默认资源组。如果您现在通过经典部署创建资源，资源会自动在空资源组中创建，即使在部署时未指定该资源组也是如此。但是，仅存在于资源组内并不意味着资源已转换为资源管理器模型。如果资源是通过经典部署创建的，您必须继续通过经典操作对其进行操作。

您可以将资源移到不同的资源组，并将新资源添加到现有的资源组。因此，您的资源组可以包含通过资源管理器和经典部署创建的混合资源。此资源组合会产生意外的结果，因为资源不支持相同的操作。

通过使用声明性模板，您可以简化用于部署的脚本。不要尝试将现有脚本从服务管理转换到资源管理器，而是考虑重新处理您的部署策略以充分利用在模板中定义基础结构和配置的功能。

## 使用标记

标记使您能够按照逻辑组织您的资源。只有通过资源管理器创建的资源才支持标记。您不能将标记应用到经典资源。

有关在资源管理器中使用标记的详细信息，请参阅[使用标记来组织 Azure 资源](/documentation/articles/resource-group-using-tags)。

## 部署模型支持的操作

在经典部署模型中创建的资源不支持资源管理器操作。在某些情况下，资源管理器命令可以检索通过经典部署创建的资源的相关信息，或者可以执行管理任务，例如将经典资源移动到另一个资源组，但这些情况下，并不意味着该类型支持资源管理器操作。例如，假定您有一个资源组包含通过资源管理器和经典模型创建的虚拟机。如果您运行以下 PowerShell 命令，您将看到所有虚拟机：

    PS C:\> Get-AzureResourceGroup -Name ExampleGroup
    ...
    Resources :
     Name                 Type                                          Location
     ================     ============================================  ========
     ExampleClassicVM     Microsoft.ClassicCompute/domainNames          eastus
     ExampleClassicVM     Microsoft.ClassicCompute/virtualMachines      eastus
     ExampleResourceVM    Microsoft.Compute/virtualMachines             eastus
    ...

但是，如果您运行 Get AzureVM 命令，您只能获取通过资源管理器创建的虚拟机。

    PS C:\> Get-AzureVM -ResourceGroupName ExampleGroup
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

有关从经典部署转换到资源管理器时的等效 Azure CLI 命令列表，请参阅<!-- [-->VM 操作的等效资源管理器和服务管理命令<!--](/documentation/articles/xplat-cli-azure-manage-vm-asm-arm)-->。

有关转换计算、存储和网络资源的详细信息，请参阅 [Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)。

若要了解如何从不同部署模型连接虚拟网络，请参阅[将经典 VNet 连接到新 VNet](/documentation/articles/virtual-networks-arm-asm-s2s)。

## 后续步骤

- 若要了解如何创建声明性部署模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。
- 若要查看用于部署模板的命令，请参阅<!--[-->使用 Azure 资源管理器模板部署应用程序<!--](/documentation/articles/resource-group-template-deploy)-->。

<!---HONumber=71-->