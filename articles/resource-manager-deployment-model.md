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
   ms.date="07/29/2016"
   wacn.date="09/19/2016"/>

# Azure Resource Manager 与经典部署：了解部署模型和资源的状态

本主题介绍 Azure Resource Manager 部署模型和经典部署模型、资源的状态，以及使用不同模型来部署资源的原因。Resource Manager 部署模型包含与经典部署模型的重要差异，这两个模型彼此之间并不完全兼容。为了简化资源的部署和管理，Azure 建议对新资源使用 Resource Manager，并尽可能通过 Resource Manager 重新部署现有资源。

如果完全不熟悉 Resource Manager，请先查看 [Azure Resource Manager overview](/documentation/articles/resource-group-overview/)（Azure Resource Manager 概述）中定义的术语。

## 部署模型的历史

Azure 最初只提供经典部署模型。在此模型中，每个资源彼此独立；无法将相关的资源组合在一起。因此，必须手动跟踪哪些资源构成了解决方案或应用程序，并记得以协调的方式管理这些资源。若要部署解决方案，必须通过经典管理门户分别创建每个资源，或创建一个脚本以正确的顺序部署所有资源。若要删除解决方案，必须逐个删除每个资源。无法轻松应用和更新相关资源的访问控制策略。最后，无法将标记应用到资源，因此无法使用有助于监视资源和管理计费的术语来标记资源。

Azure 在 2014 年引入了 Resource Manager，增加了资源组这一概念。资源组是共享同一生命周期的资源的容器。Resource Manager 部署模型具有几大优点：

- 您可以以群组形式部署、管理和监视解决方案的所有服务，而不是单独处理这些服务。
- 可以在解决方案的整个生命周期内重复部署该解决方案，确保以一致的状态部署资源。
- 可以将访问控制应用到资源组中的所有资源。将新资源添加到资源组时，会自动应用这些策略。
- 可以将标记应用到资源，以逻辑方式组织订阅中的所有资源。
- 可以使用 JavaScript 对象表示法 (JSON) 来定义解决方案的基础结构。JSON 文件称为 Resource Manager 模板。
- 您可以定义各资源之间的依赖关系，以便按正确的顺序进行部署。

当添加资源管理器时，所有资源都追溯性地添加到默认资源组。如果你现在通过经典部署创建资源，资源将自动在该服务的默认资源组中创建，即使在部署时未指定该资源组也是如此。但是，仅存在于资源组内并不意味着资源已转换为 Resource Manager 模型。我们将在下一部分说明每个服务如何处理这两种部署模型。

## 了解对模型的支持 

确定用于资源的部署模型时，请注意以下三种方案：

1. 服务支持 Resource Manager 且只提供单个类型。
2. 服务支持 Resource Manager，但提供两个类型 - 一个用于 Resource Manager 部署，一个用于经典部署。这只适用于虚拟机、存储帐户和虚拟网络。
3. 服务不支持 Resource Manager。

若要查明服务是否支持 Resource Manager，请参阅 [Resource Manager supported providers](/documentation/articles/resource-manager-supported-services/)（Resource Manager 支持的提供程序）。

如果要使用的服务不支持 Resource Manager，则必须继续使用经典部署。

如果服务支持 Resource Manager 且**不是**虚拟机、存储帐户或虚拟网络，则可以直接使用 Resource Manager。

对于虚拟机、存储帐户和虚拟网络，如果资源是通过经典部署创建的，则必须继续通过经典操作对其进行操作。如果虚拟机、存储帐户或虚拟网络是通过 Resource Manager 部署创建的，则必须继续使用 Resource Manager 操作。如果订阅包含通过 Resource Manager 部署和经典部署创建的各种资源，则可能特别不容易进行这种区分。此资源组合会产生意外的结果，因为资源不支持相同的操作。

在某些情况下，Resource Manager 命令可以检索通过经典部署创建的资源的相关信息，或者可以执行管理任务（例如将经典资源移到另一个资源组），但这些情况下，并不意味着该类型支持 Resource Manager 操作。例如，假设某个资源组包含使用经典部署创建的虚拟机。如果运行以下 Resource Manager PowerShell 命令：

    Get-AzureRmResource -ResourceGroupName ExampleGroup -ResourceType Microsoft.ClassicCompute/virtualMachines

将返回该虚拟机：
    
    Name              : ExampleClassicVM
    ResourceId        : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.ClassicCompute/virtualMachines/ExampleClassicVM
    ResourceName      : ExampleClassicVM
    ResourceType      : Microsoft.ClassicCompute/virtualMachines
    ResourceGroupName : ExampleGroup
    Location          : chinaeast
    SubscriptionId    : {guid}

但是，Resource Manager cmdlet **Get-AzureRmVM** 仅返回通过 Resource Manager 部署的虚拟机。以下命令不返回通过经典部署创建的虚拟机。

    Get-AzureRmVM -ResourceGroupName ExampleGroup

使用虚拟机时需要注意其他一些重要事项。

- 使用经典部署模型部署的虚拟机不能包含在使用资源管理器部署的虚拟网络中。
- 使用资源管理器部署模型部署的虚拟机必须包含在虚拟网络中。
- 使用经典部署模型部署的虚拟机不一定要包括在虚拟网络中。

若要了解如何从不同部署模型连接虚拟网络，请参阅[将经典 VNet 连接到新 VNet](/documentation/articles/vpn-gateway-connect-different-deployment-models-portal/)。

只有通过资源管理器创建的资源才支持标记。您不能将标记应用到经典资源。有关在资源管理器中使用标记的详细信息，请参阅[使用标记来组织 Azure 资源](/documentation/articles/resource-group-using-tags/)。

## 资源管理器的特征

为了更好地理解这两个模型，让我们看看 Resource Manager 类型的特征：

- 通过 [Azure 门户预览](https://portal.azure.cn/)创建。

     ![Azure 门户预览](./media/resource-manager-deployment-model/portal.png)

     对于计算、存储和网络资源，你可以选择使用资源管理器或经典部署。选择“Resource Manager”。

     ![Resource Manager 部署](./media/resource-manager-deployment-model/select-resource-manager.png)

- 使用 Azure PowerShell cmdlet 的 Resource Manager 版本创建。这些命令采用 *Verb-AzureRmNoun* 格式，如下所示。

        New-AzureRmResourceGroupDeployment

- 通过用于 REST 操作的 [Azure Resource Manager REST API](https://msdn.microsoft.com/library/azure/dn790568.aspx) 创建。

- 通过在 **arm** 模式下运行的 Azure CLI 命令创建。

        azure config mode arm
        azure group deployment create 

- 资源类型的名称中不包括 **(经典)**。下图显示了**存储帐户**类型。

    ![Web 应用](./media/resource-manager-deployment-model/resource-manager-type.png)

下图中所示的应用程序显示了如何在单个资源组中包含通过 Resource Manager 部署的资源。

  ![Resource Manager 体系结构](./media/virtual-machines-azure-resource-manager-architecture/arm_arch3.png)

此外，资源提供程序内各个资源之间还存在相互关系：

- 虚拟机依赖在 SRP 中定义的具体存储帐户，在 Blob 存储中存储其磁盘（必需）。
- 虚拟机引用在 NRP 中定义的具体 NIC（必需）和在 CRP 中定义的可用性集（可选）。
- NIC 引用虚拟机的指定 IP 地址（必需）、虚拟机的虚拟网络的子网（必需）和网络安全组（可选）。
- 虚拟网络内的子网引用网络安全组（可选）。
- 负载平衡器实例引用后端 IP 地址池，包括虚拟机的 NIC（可选），引用负载平衡器的公共或专用 IP 地址（可选）。

## 经典部署的特性

您也可能知道经典部署模型就是服务管理模型。

在经典部署模型中创建的资源具有以下共同特性：

- 通过[经典管理门户](https://manage.windowsazure.cn)创建。

     ![经典管理门户](./media/resource-manager-deployment-model/classic-portal.png)

     或者，通过 Azure 门户预览创建，然后指定“经典”部署（适用于“计算”、“存储”和“网络”资源）。

     ![经典部署](./media/resource-manager-deployment-model/select-classic.png)

- 通过 Azure PowerShell cmdlet 的服务管理版本创建。这些命令名称采用 *Verb-AzureNoun* 格式，如下所示。

        New-AzureVM 

- 通过用于 REST 操作的[服务管理 REST API](https://msdn.microsoft.com/library/azure/ee460799.aspx) 创建。
- 通过在 **asm** 模式下运行的 Azure CLI 命令创建。

        azure config mode asm
        azure vm create 

- 资源类型的名称中包括 **(经典)**。下图显示了**存储帐户(经典)** 类型。

    ![经典类型](./media/resource-manager-deployment-model/classic-type.png)

你仍可以使用 Azure 门户预览来管理通过经典部署创建的资源。

在 Azure 服务管理中，宿主虚拟机的计算、存储或网络资源由以下各项提供：

- 一项必不可少的云服务，用作宿主虚拟机的容器（计算）。虚拟机自动配备一个网络接口卡 (NIC) 并由 Azure 分配的 IP 地址。此外，云服务包含一个外部负载平衡器实例、一个公共 IP 地址以及若干默认终结点，以支持远程桌面、针对 Windows 虚拟机的远程 PowerShell 流量和针对 Linux 虚拟机的 Secure Shell (SSH) 流量。
- 一个必不可少的存储帐户，存储虚拟机的 VHD，包括操作系统、临时文件和附加的数据磁盘（存储）。
- 一个可选的虚拟网络，用作额外的容器，可以在其中创建子网结构并指定虚拟机所在的子网（网络）。

以下是 Azure 服务管理的组件及其关系。

  ![经典体系结构](./media/virtual-machines-azure-resource-manager-architecture/arm_arch1.png)


## 后续步骤

- 若要演练如何创建用于定义虚拟机、存储帐户和虚拟网络的模板，请参阅 [Resource Manager template walkthrough](/documentation/articles/resource-manager-template-walkthrough/)（Resource Manager 模板演练）。
- 若要了解 Resource Manager 模板的结构，请参阅 [Authoring Azure Resource Manager templates](/documentation/articles/resource-group-authoring-templates/)（创作 Azure Resource Manager 模板）。
- 若要查看用于部署模板的命令，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_0912_2016-->