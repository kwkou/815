<properties
    pageTitle="使用 Visual Studio 部署虚拟机规模集 | Azure"
    description="使用 Visual Studio 和 Resource Manager 模板部署虚拟机规模集"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gbowerman"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="ed0786b8-34b2-49a8-85b5-2a628128ead6"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/13/2017"
    wacn.date="04/17/2017"
    ms.author="guybo"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="90519886db1be949c2262126fa1238e7dd3d3bc0"
    ms.lasthandoff="04/06/2017" />

# <a name="how-to-create-a-virtual-machine-scale-set-with-visual-studio"></a>如何使用 Visual Studio 创建虚拟机规模集

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本文介绍如何使用 Visual Studio 资源组部署来部署 Azure 虚拟机规模集。

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-visual-studio-login-guide.md)]

[Azure 虚拟机规模集](https://azure.microsoft.com/zh-cn/blog/azure-vm-scale-sets-public-preview/)是一种 Azure 计算资源，可通过负载均衡部署和管理一组类似的虚拟机。 可使用 [Azure Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates)预配和部署虚拟机规模集。 可以使用 Azure CLI、PowerShell、REST 来部署 Azure Resource Manager 模板，也可直接从 Visual Studio 部署。 Visual Studio 提供了一组示例模板，这些模板可以作为 Azure 资源组部署项目的一部分进行部署。

Azure 资源组部署是一种通过单个部署操作将相关的一组 Azure 资源组合并进行发布的方式。

## <a name="pre-requisites"></a>先决条件
若要开始在 Visual Studio 中部署虚拟机规模集，需要以下项：

* Visual Studio 2013 或更高版本
* Azure SDK 2.7、2.8 或 2.9

>[AZURE.NOTE]
>这些说明假定你使用的是包含 [Azure SDK 2.8](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-8-for-net/) 的 Visual Studio。

## <a name="creating-a-project"></a>创建项目
1. 通过依次选择“文件”->“新建”->“项目”，在 Visual Studio 中创建一个新项目。

    ![新建文件][file_new]

2. 在“Visual C#”的“云”下，选择“Azure Resource Manager”，创建用于部署 Azure Resource Manager 模板的项目。

    ![创建项目][create_project]

3. 在模板列表中，选择 Linux 或 Windows 虚拟机规模集模板。

    ![选择模板][select_Template]

4. 创建项目后，将看到 PowerShell 部署脚本、Azure Resource Manager 模板和虚拟机规模集的参数文件。

    ![解决方案资源管理器][solution_explorer]

## <a name="customize-your-project"></a>自定义项目
现在可以编辑模板以根据应用程序的需求自定义它，例如添加 VM 扩展属性或编辑负载均衡规则。 默认情况下，虚拟机规模集模板已配置为部署 AzureDiagnostics 扩展。 它还部署了具有公共 IP 地址且配置有入站 NAT 规则的负载均衡器。

通过负载均衡器，可以使用 SSH (Linux) 或RDP (Windows) 连接到 VM 实例。 前端端口范围从 50000 开始。 对于 Linux，这意味着如果 SSH 连接到端口 50000，将路由到规模集中第一个 VM 的端口 22。 连接到端口 50001 将路由到第二个 VM 的端口 22，依此类推。

 使用 Visual Studio 编辑模板的一种好方法是使用“JSON 概要”来组织参数、变量和资源。 了解架构后，Visual Studio 可以在部署前指出模板中的错误。

![JSON 资源管理器][json_explorer]

## <a name="deploy-the-project"></a>部署项目
1. 部署 Azure Resource Manager 模板来创建虚拟机规模集资源。 右键单击项目节点，然后选择“部署”->“新建部署”。

    ![部署模板][5deploy_Template]

2. 在“部署到资源组”对话框中选择订阅。

    ![部署模板][6deploy_Template]

3. 可以从此处创建要将模板部署到的 Azure 资源组。

    ![新建资源组][new_resource]

4. 接下来，单击“编辑参数”，输入将传递给模板的参数。 提供创建部署所需的操作系统的用户名和密码。 如果未安装用于 Visual Studio 的 PowerShell 工具，建议勾选“保存密码”，以避免隐藏的 PowerShell 命令行提示符，或使用 [Key Vault 支持](https://azure.microsoft.com/zh-cn/blog/keyvault-support-for-arm-templates/)。

    ![编辑参数][edit_parameters]

5. 现在单击“部署”。 “输出”窗口显示部署进度。 请注意，该操作正在执行 **Deploy-AzureResourceGroup.ps1** 脚本。

    ![输出窗口][output_window]

## <a name="exploring-your-virtual-machine-scale-set"></a>探索虚拟机规模集
部署完成后，可在 Visual Studio **云资源管理器**中（刷新列表）查看新的虚拟机规模集。 云资源管理器让你可以在开发应用程序时管理 Visual Studio 中的 Azure 资源。 还可以在 [Azure 门户预览](https://portal.azure.cn)中查看虚拟机规模集。

![云资源管理器][cloud_explorer]

 该门户提供了使用 Web 浏览器直观管理 Azure 基础结构的最佳方式，而 Azure 资源浏览器则通过在“实例视图”中提供窗口，并且还针对要查看的资源显示 PowerShell 命令，提供了方便地浏览和调试 Azure 资源的方式。 虚拟机规模集处于预览状态时，资源浏览器将显示虚拟机规模集的大部分详细信息。

## <a name="next-steps"></a>后续步骤
通过 Visual Studio 成功部署虚拟机规模集后，便可进一步自定义项目以满足应用程序需求。 例如，可以使用自定义脚本扩展来配置将基础结构添加到模板（例如独立 VM）的方法，或者配置部署应用程序的方法。 可以在 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates) GitHub 存储库中（搜索“vmss”）找到很好的示例模板。

[file_new]: ./media/virtual-machine-scale-sets-vs-create/1-FileNew.png
[create_project]: ./media/virtual-machine-scale-sets-vs-create/2-CreateProject.png
[select_Template]: ./media/virtual-machine-scale-sets-vs-create/3b-SelectTemplateLin.png
[solution_explorer]: ./media/virtual-machine-scale-sets-vs-create/4-SolutionExplorer.png
[json_explorer]: ./media/virtual-machine-scale-sets-vs-create/10-JsonExplorer.png
[5deploy_Template]: ./media/virtual-machine-scale-sets-vs-create/5-DeployTemplate.png
[6deploy_Template]: ./media/virtual-machine-scale-sets-vs-create/6-DeployTemplate.png
[new_resource]: ./media/virtual-machine-scale-sets-vs-create/7-NewResourceGroup.png
[edit_parameters]: ./media/virtual-machine-scale-sets-vs-create/8-EditParameter.png
[output_window]: ./media/virtual-machine-scale-sets-vs-create/9-Output.png
[cloud_explorer]: ./media/virtual-machine-scale-sets-vs-create/12-CloudExplorer.png
<!--Update_Description: wording update-->