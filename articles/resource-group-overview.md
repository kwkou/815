<!-- Remove best practices -->
<properties
   pageTitle="Azure Resource Manager 概述 | Azure"
   description="介绍如何使用 Azure 资源管理器在 Azure 上部署和管理资源以及对其进行访问控制。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>  


<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/16/2016"
   wacn.date="10/24/2016"/>

# Azure 资源管理器概述

应用程序的体系结构通常由许多组件构成 – 其中可能包括虚拟机、存储帐户、虚拟网络、Web 应用、数据库、数据库服务器和第三方服务。这些组件不会以独立的实体出现，而是以单个实体的相关部件和依赖部件出现。如果你希望以组的方式部署、管理和监视这些这些组件，那么，你可以使用 Azure 资源管理器以组的方式处理解决方案中的资源。可以通过一个协调的操作为解决方案部署、更新或删除所有资源。可以使用一个模板来完成部署，该模板适用于不同的环境，例如测试、过渡和生产。资源管理器提供安全、审核和标记功能，以帮助你在部署后管理资源。

## 术语

如果你不熟悉 Azure Resource Manager，则可能不熟悉某些术语。

- **资源** - 可通过 Azure 获取的可管理项。部分常见资源包括虚拟机、存储帐户、Web 应用、数据库和虚拟网络，但这只是其中一小部分。
- **资源组** - 一个容器，用于保存 Azure 解决方案的相关资源。资源组可以包含解决方案的所有资源，也可以只包含想要作为组来管理的资源。根据对组织有利的原则，决定如何将资源分配到资源组。
- **资源提供程序** — 一种服务，提供可以通过 Resource Manager 进行部署和管理的资源。每个资源提供程序提供用于处理所部署资源的操作。部分常见资源提供程序包括 Microsoft.Compute（提供虚拟机资源）、Microsoft.Storage（提供存储帐户资源）和 Microsoft.Web（提供与 Web 应用相关的资源）。
- **Resource Manager 模板** — 一个 JavaScript 对象表示法 (JSON) 文件，用于定义一个或多个要部署到资源组的资源。它也会定义所部署资源之间的依赖关系。使用模板能够以一致方式反复部署资源。
- **声明性语法** - 一种语法，允许你声明“以下是我想要创建的项目”，而不需要编写一系列编程命令来进行创建。Resource Manager 模板便是声明性语法的其中一个示例。在该文件中，你可以定义要部署到 Azure 的基础结构的属性。

## 使用资源管理器的优势

资源管理器提供多种优势：

- 可以以组的形式部署、管理和监视解决方案的所有资源，而不是单独处理这些资源。
- 你可以在整个开发生命周期内重复部署解决方案，并确保以一致的状态部署资源。
- 你可以通过声明性模板而非脚本来管理基础结构。
- 您可以定义各资源之间的依赖关系，以便按正确的顺序进行部署。
- 您可以将访问控制应用到资源组中的所有服务，因为基于角色的访问控制 (RBAC) 已在本机集成到管理平台。
- 可以将标记应用到资源，以逻辑方式组织订阅中的所有资源。
- 可以通过查看一组共享相同标记的资源的成本来明确组织的帐单。

资源管理器提供了一种新方法来部署和管理您的解决方案。如果你使用早期的部署模型并想要了解这些更改，请参阅[了解资源管理器部署和经典部署](/documentation/articles/resource-manager-deployment-model/)。

## 指南

以下建议可帮助你在使用解决方案时充分利用 Resource Manager。

1. 通过资源管理器模板中的声明性语法而不是强制性的命令来定义和部署基础结构。
2. 在模板中定义所有部署和配置步骤。在设置解决方案时不应执行手动步骤。
3. 运行强制性命令来管理资源，例如启动或停止应用或计算机。
4. 排列资源组中具有相同生命周期的资源。使用标记来组织其他所有资源。

有关详细建议，请参阅 [Best practices for creating Azure Resource Manager templates](/documentation/articles/resource-manager-template-best-practices/)（创建 Azure Resource Manager 模板的最佳实践）。

## <a name="resource-groups"></a> 资源组

定义资源组时，需要考虑以下几个重要因素：

1. 组中的所有资源应该共享相同的生命周期。将这些资源一同部署、更新和删除。如果某个资源（例如数据库服务器）需要采用不同的部署周期，则它应在另一个资源组中。
2. 每个资源只能在一个资源组中。
3. 你随时可以在资源组添加或删除资源。
4. 可以将资源从一个资源组移到另一个组。有关详细信息，请参阅 [Move resources to new resource group or subscription](/documentation/articles/resource-group-move-resources/)（将资源移到新的资源组或订阅）。
4. 资源组可以包含位于不同区域的资源。
5. 资源组可用于划分对管理操作的访问控制。
6. 资源可与其他资源组中的资源进行交互。两个资源相关但并不共享相同生命周期时（例如，连接到数据库的 Web 应用），这种交互会很常见。

创建资源组时，需要为该资源组提供一个位置。你可能会疑惑，“为什么资源组需要一个位置？ 以及，如果资源可以具有与资源组不同的位置，资源组的位置应该不重要啊？” 资源组存储与资源有关的元数据。因此，在指定资源组的位置时，你是在指定元数据的存储位置。出于合规性原因，可能需要确保你的数据存储在某一特定区域。

## 资源提供程序

每个资源提供程序都会提供一组用于技术区域的资源和操作。例如，若要存储密钥和密码，可以使用 **Microsoft.KeyVault** 资源提供程序。此资源提供程序提供名为 **vaults** 的资源类型来创建密钥保管库，提供名为 **vaults/secrets** 的资源类型在密钥保管库中创建密码。它还通过[密钥保管库 REST API 操作](https://msdn.microsoft.com/zh-cn/library/azure/dn903609.aspx)来提供操作。可以直接调用 REST API，也可以使用[密钥保管库 PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/dn868052.aspx) 和[密钥保管库 Azure CLI](/documentation/articles/key-vault-manage-with-cli/) 来管理密钥保管库。还可以使用几种编程语言来处理大多数资源。

若要部署和管理基础结构，需要了解有关资源提供程序的详细信息。需要了解其资源类型、REST API 操作的版本号、支持的操作，以及用于创建资源的架构。若要了解支持的资源提供程序，请参阅 [Resource Manager 提供程序、区域、API 版本和架构](/documentation/articles/resource-manager-supported-services/)。

## <a name="template-deployment"></a> 模板部署

使用 Resource Manager 可以创建一个模板（采用 JSON 格式），用于定义应用程序的部署和配置。使用模板可以在整个应用程序生命周期内反复部署该应用程序，并确保以一致的状态部署资源。Azure Resource Manager 会分析依赖关系，以确保按正确的顺序创建资源。有关详细信息，请参阅[在 Azure 资源管理器模板中定义依赖关系](/documentation/articles/resource-group-define-dependencies/)。

从门户创建解决方案时，该解决方案将自动包含部署模板。你无需从头开始创建模板，因为你可以从解决方案的模板着手，并根据你的特定需求自定义该模板。可以通过导出资源组的当前状态或查看特定部署所用的模板，来检索现有资源组的模板。查看导出的模板是了解模板语法的有用方法。若要了解有关使用导出的模板的详细信息，请参阅[从现有资源导出 Azure Resource Manager 模板](/documentation/articles/resource-manager-export-template/)。

无需在单个模板中定义整个基础结构。通常，合理的做法是将部署要求划分成一组有针对性的模板。可以轻松地将这些模板重复用于不同的解决方案。若要部署特定的解决方案，请创建链接所有所需模板的主模板。有关详细信息，请参阅[将链接的模板与 Azure 资源管理器配合使用](/documentation/articles/resource-group-linked-templates/)。

还可以使用模板对基础结构进行更新。例如，可以将新的资源添加到应用程序，并为已部署的资源添加配置规则。如果模板指定要创建新的资源，但该资源已存在，则 Azure 资源管理器将执行更新而不是创建新资产。Azure 资源管理器会将现有资产更新到相同状态，就如同该资产是新建的一样。或者，你可以指定要让资源管理器删除模板中未指定的所有资源。若要了解部署时的不同选项，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

你可以在模板中指定参数，以进行自定义并提高部署灵活性。例如，可以传递相应的参数值，以便根据测试环境定制部署。通过指定参数，可以使用同一个模板将解决方案部署到不同的环境。

如果你需要其他操作（例如，安装未包含在安装程序中的特定软件）时，资源管理器可提供所需的扩展。如果你已在使用配置管理服务（如 DSC、Chef 或 Puppet），则可以使用扩展来继续处理该服务。

最后，该模板将成为应用程序源代码的一部分。你可以将它签入源代码存储库，并随着应用程序的发展更新该模板。你可以通过 Visual Studio 编辑模板。

有关定义模板的详细信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

有关创建模板的分步说明，请参阅 [Resource Manager Template Walkthrough（Resource Manager 模板演练）](/documentation/articles/resource-manager-template-walkthrough/)。

有关将解决方案部署到不同环境的指南，请参阅 [Azure 中的开发和测试环境](/documentation/articles/solution-dev-test-environments/)。

## 标记

资源管理器提供了标记功能，让你根据管理或计费要求为资源分类。如果你有一系列复杂的资源组和资源，并想要以最有利的方式可视化这些资产，则可以使用标记。例如，你可以标记组织中充当类似角色或者属于同一部门的资源。如果不使用标记，组织中的用户可以创建多个资源，这可能会使将来的标识和管理变得困难。例如，你可能会希望删除特定项目的所有资源。如果这些资源没有针对项目进行标记，则必须手动查找它们。标记是降低不必要的订阅成本的重要方法。

资源不需要驻留在同一个资源组中就能共享一个标记。你可以创建自己的标记分类，以确保组织中的所有用户使用公用的标记，避免用户无意中应用稍有不同的标记（如“dept”而不是“department”）。

有关标记的详细信息，请参阅[使用标记来组织 Azure 资源](/documentation/articles/resource-group-using-tags/)。

## 访问控制

资源管理器可让你控制谁有权访问组织的特定操作。Azure 资源管理器原生地在管理平台中集成了 OAuth 和基于角色的访问控制 (RBAC)，并向资源组中的所有服务应用该访问控制。可以将用户添加到预定义的平台和特定于资源的角色，并将这些角色应用到订阅、资源组或资源以限制访问。例如，你可以利用名为 SQL DB Contributor 的预定义角色，以允许用户管理数据库，但不允许管理数据库服务器或安全策略。为此，可将组织中需要此类访问权限的用户添加到 SQL 数据库“参与者”角色，并将该角色应用到订阅、资源组或资源。

资源管理器会自动记录用户操作以供审核。有关使用审核日志的信息，请参阅 [Audit operations with Resource Manager（使用 Resource Manager 执行审核操作）](/documentation/articles/resource-group-audit/)。

有关基于角色的访问控制的详细信息，请参阅 [Azure Role-Based Access Control（Azure 基于角色的访问控制）](/documentation/articles/role-based-access-control-configure/)。[RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)主题包含内置角色和允许的操作的列表。内置角色包括“所有者”、“读取者”和“参与者”等普通角色，以及“虚拟机参与者”、“虚拟网络参与者”和“SQL 安全管理员”等服务特定角色（这里只是列举了几个可用的角色）。

你可以显式锁定关键资源，以防止用户删除或修改这些资源。有关详细信息，请参阅[使用 Azure Resource Manager 锁定资源](/documentation/articles/resource-group-lock-resources/)。

<!-- 
有关最佳实践，请参阅 [Azure Resource Manager 的安全注意事项](/documentation/articles/best-practices-resource-manager-security/) -->

## 使用自定义策略管理资源

资源管理器可让你创建自定义策略来管理资源。创建的策略类型可包括各种方案。可以在资源上实施命名约定，限制可部署的资源的类型和实例，或限制可托管资源类型的区域。可以要求资源上的标记值按部门组织计费。可以通过创建策略来降低成本并在订阅中保持一致性。有关详细信息，请参阅[使用策略来管理资源和控制访问](/documentation/articles/resource-manager-policy/)。

## 一致的管理层

Resource Manager 通过 Azure PowerShell、适用于 Mac、Linux 和 Windows 的 Azure CLI、Azure 门户或 REST API 提供兼容的操作。你可以使用最适合自己的界面，并快速切换不同的界面，而不会造成任何混淆。

有关 PowerShell 的信息，请参阅[将 Azure PowerShell 用于资源管理器](/documentation/articles/powershell-azure-resource-manager/)和 [Azure Resource Manager Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn757692.aspx)。

有关 Azure CLI 的信息，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager/)。

有关 REST API 的信息，请参阅 [Azure Resource Manager REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)。 <!-- 若要查看已部署的资源的 REST 操作，请参阅 [Use Azure Resource Explorer to view and modify resources（使用 Azure 资源浏览器来查看和修改资源）](/documentation/articles/resource-manager-resource-explorer/)。-->

有关使用门户的信息，请参阅[使用 Azure 门户预览管理 Azure 资源](/documentation/articles/resource-group-portal/)。

Azure Resource Manager 支持跨域资源共享 (CORS)。使用 CORS 时，你可以从驻留在不同域中的 Web 应用程序调用资源管理器 REST API 或 Azure 服务 REST API。如果不支持 CORS，Web 浏览器将阻止一个域中的应用访问另一个域中的资源。资源管理器为所有具有有效身份验证凭据的请求启用 CORS。

## SDK

Azure SDK 适用于多种语言和平台。每种语言实现可通过其生态系统包管理器和 GitHub 来使用。

其中每个 SDK 中的代码是从 Azure RESTful API 规范生成的。
这些规范是开放源代码，基于 Swagger 2.0 规范。
SDK 代码是通过名为 AutoRest 的开源项目生成的。
AutoRest 将这些 RESTful API 规范转换成采用多种语言的客户端库。
如果你想要改进 SDK 中生成的代码的任何方面，可以使用整个工具集来创建 SDK，该工具集属于开放式工具集，免费提供且基于广泛采用的 API 规范格式。

以下是开放源 SDK 存储库。欢迎提供反馈和拉取请求并提出问题。

[.NET](https://github.com/Azure/azure-sdk-for-net) | [Java](https://github.com/Azure/azure-sdk-for-java) | [Node.js](https://github.com/Azure/azure-sdk-for-node) | [PHP](https://github.com/Azure/azure-sdk-for-php) | [Python](https://github.com/Azure/azure-sdk-for-python) | [Ruby](https://github.com/Azure/azure-sdk-ruby)

> [AZURE.NOTE] 如果 SDK 未提供所需的功能，也可以直接调用 [Azure REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)。

## 示例

### .NET

- [管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)
- [使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)

### Java

- [管理 Azure 资源](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource/)
- [管理 Azure 资源组](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group/)
- [使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)

### Node.js

- [管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)
- [使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)

### Python

- [管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)
- [使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)

### Ruby

- [管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)
- [使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)


除了这些示例，还可以通过库模板搜索示例。

[.NET](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=dotnet) | [Java](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=java) | [Node.js](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=nodejs) | [Python](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=python) | [Ruby](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=ruby)

## 后续步骤

- 有关使用模板的简单介绍，请参阅 [Export an Azure Resource Manager template from existing resources（从现有资源导出 Azure Resource Manager 模板）](/documentation/articles/resource-manager-export-template/)。
- 有关创建模板的更全面演练，请参阅 [Resource Manager Template Walkthrough（Resource Manager 模板演练）](/documentation/articles/resource-manager-template-walkthrough/)。
- 若要了解可以在模板中使用的函数，请参阅[模板函数](/documentation/articles/resource-group-template-functions/)

<!--
- 有关将 VS Code 与 Resource Manager 配合使用的信息，请参阅[在 Visual Studio Code 中使用 Azure Resource Manager 模板](/documentation/articles/vs-azure-tools-resource-groups-deployment-projects-create-deploy/) -->

<!---HONumber=Mooncake_1017_2016-->