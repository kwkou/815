<properties
	pageTitle="资源组准则 | Azure"
	description="了解用于在 Azure 基础结构服务中部署资源组的关键设计和实施准则。"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/30/2016"
	wacn.date="08/08/2016"/>

# Azure 资源组准则

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-intro](../../includes/virtual-machines-windows-infrastructure-guidelines-intro.md)]

本文着重于了解如何以逻辑方式构建环境并将所有组件集合在资源组中。


## 资源组的实施准则

决策：

- 你是要以核心基础结构组件还是以完整的应用程序部署构建资源组？
- 是否需要使用基于角色的访问控制来限制对资源组的访问？

任务：

- 定义所需的核心基础结构组件，以及你是否会使用专用资源组。
- 查看如何实施 Resource Manager 模板以进行一致且可重现的部署。
- 定义控制资源组访问所需的用户访问角色。
- 使用你的命名约定来创建资源组集。你可以使用 Azure PowerShell 或门户。


## 资源组

在 Azure 中，可以采用逻辑方式将相关资源（例如存储帐户、虚拟网络及虚拟机 (VM)）分组在一起，以便将它们作为单个实体进行部署、管理和维护。从管理的角度来看，这可让你在将所有相关资源放在一起的情况下更轻松地部署应用程序，或者授予其他人访问该资源组的权限。若要更全面地了解资源组，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

资源组的一项重要特征是能够使用模板构建环境。模板是简单的 JSON 文件，它声明了存储、网络和计算资源以及要应用的任何相关自定义脚本或配置。通过使用这些模板，可以为应用程序创建一致且可重现的部署。这可让你轻松构建开发环境，然后使用相同的模板创建生产部署，反之亦然。若要更好地了解模板的使用方式，请参阅[模板演练](/documentation/articles/resource-manager-template-walkthrough/)，它将引导你完成构建模板的每个步骤。

在使用资源组设计环境时，可以采用以下两种不同的方法：

- 针对每个应用程序部署使用包含存储帐户、虚拟网络和子网、VM、负载平衡器等等的资源组。
- 使用包含核心虚拟网络和子网或存储帐户的集中式资源组，并让应用程序位于它们自己的资源组中（这些资源组仅包含 VM、负载平衡器、网络接口等等）。

随着规模的扩大，为虚拟网络和子网创建集中式资源组可让你更轻松地为混合连接选项生成跨界网络连接，而不用让每个应用程序都拥有需要单独配置和维护的虚拟网络。[基于角色的访问控制](/documentation/articles/role-based-access-control-what-is/)提供一种精细的方式来控制资源组访问。对于生产应用程序，可以控制能访问那些资源的用户；对于核心基础结构资源，可以限制为只有基础结构工程师可以使用它们。你的应用程序所有者将只能访问其资源组内的应用程序组件，而不能访问环境的核心 Azure 基础结构。在设计环境时，请考虑需要访问资源的用户，并据此设计资源组。


## <a name="next-steps"></a> 后续步骤

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-next-steps](../../includes/virtual-machines-windows-infrastructure-guidelines-next-steps.md)]

<!---HONumber=Mooncake_0801_2016-->