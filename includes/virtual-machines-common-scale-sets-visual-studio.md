

本文介绍如何使用 Visual Studio 资源组部署来部署 Azure 虚拟机规模集。


[Azure 虚拟机规模集](https://azure.microsoft.com/blog/azure-vm-scale-sets-public-preview/)是一种 Azure 计算资源，可通过轻松集成的自动缩放和负载均衡选项部署和管理一组类似的虚拟机。可以使用 [Azure 资源管理器 (ARM) 模板](https://github.com/Azure/azure-quickstart-templates)预配和部署 VM 规模集。可以使用 Azure CLI、PowerShell、REST 来部署 ARM 模板，也可直接从 Visual Studio 部署。Visual Studio 提供了一组示例模板，这些模板可以作为 Azure 资源组部署项目的一部分进行部署。

Azure 资源组部署是一种通过单个部署操作将相关的一组 Azure 资源组合在一起并进行发布的方式。

## 先决条件

若要开始在 Visual Studio 中部署 VM 规模集，需要以下项：

- Visual Studio 2013 或 2015
- Azure SDK 2.7 或 2.8

注意：这些说明假定你使用的是包含 [Azure SDK 2.8](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/) 的 Visual Studio 2015。

## 创建项目

1. 通过选择 **文件，在 Visual Studio 2015 中创建一个新项目| New | Project**.

	![File New][file_new]

2. **Visual C# 下| Cloud**, choose **Azure Resource Manager** to create a project for deploying an ARM Template.

	![Create Project][create_project]

3.  在模板列表中，选择 Linux 或 Windows 虚拟机规模集模板。

	![选择模板][select_Template]

4. 创建项目后，将看到 PowerShell 部署脚本、Azure Resource Manager 模板和虚拟机规模集的参数文件。

	![解决方案资源管理器][solution_explorer]  


## 自定义项目

现在可以编辑模板以根据应用程序的需求自定义它，例如添加 VM 扩展属性或编辑负载均衡规则。默认情况下 VM 规模集模板已配置为部署 AzureDiagnostics 扩展，这样可通过其轻松添加自动缩放规则。它还使用公共 IP 地址部署了配置有入站 NAT 规则的负载均衡器，可以使用 SSH (Linux) 或 RDP (Windows) 连接到 VM 实例 - 前端端口范围从 50000 开始，这意味着在 Linux 的情况下，如果通过 SSH 连接到公共 IP 地址（或域名）的端口 50000，会路由到规模集中的第一个 VM 的端口 22。连接到端口 50001 将被路由到的第二个 VM 的端口 22，依此类推。

 使用 Visual Studio 编辑模板的良好方法是使用“JSON 概要”来组织参数、变量和资源。了解架构后，Visual Studio 可以在部署前指出模板中的错误。

![JSON 资源管理器][json_explorer]  


## 部署项目

6. 将 ARM 模板部署到 Azure，以创建 VM 规模集资源。右键单击项目节点，选择 **部署| New Deployment**.

	![Deploy Template][5deploy_Template]

7. 在“部署到资源组”对话框中选择订阅。

	![部署模板][6deploy_Template]

8. 还可以从此处创建要将模板部署到的新 Azure 资源组。

	![新建资源组][new_resource]

9. 接下来，选择“编辑参数”按钮以输入参数，这些参数将传递到你的模板，创建部署时需要某些值，例如 OS 的用户名和密码。

	![编辑参数][edit_parameters]  


10. 现在单击“部署”。“输出”窗口将显示部署进度。请注意，该操作正在执行 **Deploy-AzureResourceGroup.ps1** 脚本。

	![输出窗口][output_window]  


## 浏览 VM 规模集

部署完成后，你可以在 Visual Studio **云资源管理器**中（刷新列表）查看新的 VM 规模集。云资源管理器让你可以在开发应用程序时管理 Visual Studio 中的 Azure 资源。此外，还可在 Azure 门户预览和 Azure 资源浏览器中查看 VM 规模集。

![云资源管理器][cloud_explorer]  


 该门户提供了使用 Web 浏览器直观管理 Azure 基础结构的最佳方式，而 Azure 资源浏览器则通过在“实例视图”中提供窗口，并且还针对要查看的资源显示 PowerShell 命令，提供了方便地浏览和调试 Azure 资源的方式。当 VM 规模集处于预览状态时，资源浏览器将显示 VM 规模集的大多数详细信息。

## 后续步骤

通过 Visual Studio 成功部署 VM 规模集后，便可以进一步自定义您的项目以满足应用程序需求。例如，通过添加 Insights 资源设置自动缩放，将独立 VM 等基础结构添加到你的模板，或使用自定义脚本扩展部署应用程序。可以在 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates) GitHub 存储库中（搜索“vmss”）找到很好的示例模板源。

[file_new]: ./media/virtual-machines-common-scale-sets-visual-studio/1-FileNew.png
[create_project]: ./media/virtual-machines-common-scale-sets-visual-studio/2-CreateProject.png
[select_Template]: ./media/virtual-machines-common-scale-sets-visual-studio/3b-SelectTemplateLin.png
[solution_explorer]: ./media/virtual-machines-common-scale-sets-visual-studio/4-SolutionExplorer.png
[json_explorer]: ./media/virtual-machines-common-scale-sets-visual-studio/10-JsonExplorer.png
[5deploy_Template]: ./media/virtual-machines-common-scale-sets-visual-studio/5-DeployTemplate.png
[6deploy_Template]: ./media/virtual-machines-common-scale-sets-visual-studio/6-DeployTemplate.png
[new_resource]: ./media/virtual-machines-common-scale-sets-visual-studio/7-NewResourceGroup.png
[edit_parameters]: ./media/virtual-machines-common-scale-sets-visual-studio/8-EditParameter.png
[output_window]: ./media/virtual-machines-common-scale-sets-visual-studio/9-Output.png
[cloud_explorer]: ./media/virtual-machines-common-scale-sets-visual-studio/12-CloudExplorer.png

<!---HONumber=Mooncake_1114_2016-->