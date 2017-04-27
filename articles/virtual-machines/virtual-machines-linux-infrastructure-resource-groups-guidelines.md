<properties
	pageTitle="资源组准则 | Azure"
	description="了解用于在 Azure 基础结构服务中部署资源组的关键设计和实施准则。"
	documentationCenter=""
	services="virtual-machines-linux"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/17/2017"
	wacn.date="04/27/2017"
	ms.author="iainfou"/>  


# Azure 资源组准则

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-intro](../../includes/virtual-machines-linux-infrastructure-guidelines-intro.md)]

本文重点介绍如何以逻辑方式构建环境并将所有组件组合到资源组中。


## 资源组的实施准则

决策：

- 是要以核心基础结构组件还是以完整的应用程序部署构建资源组？
- 是否需要使用基于角色的访问控制来限制对资源组的访问？

任务：

- 定义所需的核心基础结构组件和专用资源组。
- 查看如何实施 Resource Manager 模板以进行一致且可重现的部署。
- 定义控制资源组访问所需的用户访问角色。
- 使用你的命名约定来创建资源组集。可以使用 Azure CLI 或门户。


## 资源组

在 Azure 中，以逻辑方式将相关资源（例如存储帐户、虚拟网络及虚拟机 (VM)）组合到一起，将它们作为单个实体进行部署、管理和维护。从管理的角度来看，此方法可以在将所有相关资源放在一起的同时更轻松地部署应用程序，或者授予他人访问该资源组的权限。若要更全面地了解资源组，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

资源组的一项关键功能是，能够使用用于声明存储、网络和计算资源的 JSON 文件来构建环境。你还可以定义要应用的任何相关自定义脚本或配置。通过使用这些 JSON 模板，可以为应用程序创建一致且可重现的部署。此方法可让用户构建开发环境，然后使用相同的模板创建生产部署，反之亦然。

在使用资源组设计环境时，可以采用以下两种不同的方法：

- 针对每个应用程序部署的资源组，包含存储帐户、虚拟网络和子网、VM、负载均衡器等。
- 包含核心虚拟网络和子网或存储帐户的集中式资源组。因此，应用程序所在的资源组只包含 VM、负载均衡器、网络接口等。

随着规模的扩大，为虚拟网络和子网创建集中式资源组可让用户更轻松地为混合连接选项生成跨界网络连接。另一种方法是让每个应用程序都有自己的需要配置和维护的虚拟网络。[基于角色的访问控制](/documentation/articles/role-based-access-control-what-is/)是一种精细的方式，可控制资源组访问。对于生产应用程序，可以控制能访问资源的用户；对于核心基础结构资源，可以限制为只有基础结构工程师才可以使用资源。应用程序所有者只能访问其资源组内的应用程序组件，而不能访问环境的核心 Azure 基础结构。在设计环境时，请考虑需要访问资源的用户，并据此设计资源组。


## <a name="next-steps"></a>后续步骤

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-next-steps](../../includes/virtual-machines-linux-infrastructure-guidelines-next-steps.md)]

<!---HONumber=Mooncake_Quality_Review_1215_2016-->