<properties
   pageTitle="将 VS Code 与 Resource Manager 模板配合使用 | Microsoft Azure"
   description="说明如何设置 Visual Studio Code 以创建 Azure Resource Manager 模板。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="cmatskas"
   manager="timlt"
   editor="tysonn"/>  


<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/26/2016"
   wacn.date="11/21/2016"
   ms.author="chmatsk;tomfitz"/>  


# 在 Visual Studio Code 中使用 Azure Resource Manager 模板

Azure Resource Manager 模板是用于描述资源和相关依赖性的 JSON 文件。这些文件有时可能很大且很复杂，因此工具支持非常重要。Visual Studio Code 是全新的轻量型开源跨平台代码编辑器。它通过[新的扩展](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)支持创建和编辑 Resource Manager 模板。VS Code 可在任何位置运行，除非你还想要部署 Resource Manager 模板，否则它不需要访问 Internet。

如果你没有 VS Code，可以通过 [https://code.visualstudio.com/](https://code.visualstudio.com/) 来安装它。

## 安装 Resource Manager 扩展

若要在 VS Code 中使用 JSON 模板，需要安装一个扩展。以下步骤将下载并安装 Resource Manager JSON 模板的语言支持：

1. 启动 VS Code
2. 运行“快速打开”(Ctrl+P)
3. 运行以下命令：

        ext install azurerm-vscode-tools

4. 在系统提示时重新启动 VS Code 以启用该扩展。

 操作已完成！

## 设置 Resource Manager 代码片段

前面的步骤已安装工具支持，但我们现在需要配置 VS Code 才能使用 JSON 模板代码片段。

1. 将文件的内容从 [azure-xplat-arm-tooling](https://raw.githubusercontent.com/Azure/azure-xplat-arm-tooling/master/VSCode/armsnippets.json) 存储库复制到剪贴板。
2. 启动 VS Code
3. 在 VS Code 中，可以使用以下方式打开 JSON 代码片段文件：导航到“文件”->“首选项”->“用户代码片段”->“JSON”，或选择 F1 并键入**首选项**，直到可以选择“首选项: 代码片段”为止。

    ![首选项代码片段](./media/resource-manager-vs-code/preferences-snippets.png)

    从选项中选择“JSON”。

    ![选择 json](./media/resource-manager-vs-code/select-json.png)

4. 将步骤 1 中所述的文件内容粘贴到用户代码片段文件中最后一个“}”的前面
5. 请确保 JSON 看起来正常，并且任何地方都没有波浪线。
6. 保存并关闭用户代码片段文件。

现在，可以开始使用 Resource Manager 代码片段了。接下来，我们将测试此设置。

## 在 VS Code 中使用模板

开始使用模板的最简单方法是获取 [Github](https://github.com/Azure/azure-quickstart-templates) 上提供的快速启动模板之一，或使用你自己的某个模板。你可以通过门户轻松地针对任何资源组[导出模板](/documentation/articles/resource-manager-export-template/)。

1. 如果从资源组导出了模板，请在 VS Code 中打开提取的文件。

    ![显示文件](./media/resource-manager-vs-code/show-files.png)

2. 打开 template.json 文件，以便对它进行编辑并添加一些附加的资源。在 **"resources": [** 的后面按 Enter，以另起一行。如果键入 **arm**，你将看到选项列表。这些选项是你安装的模板代码片段。它看起来应该如下所示：

    ![显示代码片段](./media/resource-manager-vs-code/type-snippets.png)

3. 选择所需的代码片段。在本文中，我选择了 **arm-ip** 来创建新的公共 IP 地址。在新建资源的右括号“}”后面插入一个逗号，以确保模板语法有效。

     ![添加逗号](./media/resource-manager-vs-code/add-comma.png)

4. VS Code 具有内置的 IntelliSense。当你编辑模板时，VS Code 将建议可用的值。例如，若要将 variables 节添加到模板，请添加 **""**（两个双引号），然后选择这两个引号之间的 **Ctrl+Space**。你将看到包含**变量**的选项。

    ![添加变量](./media/resource-manager-vs-code/add-variables.png)

5. Intellisense 还可以建议可用的值或函数。若要将某个属性设置为参数值，请创建包含 **""** 和 **Ctrl+Space** 的表达式。可以开始键入函数的名称。找到所需的函数后，请选择 **Tab**。

    ![添加参数](./media/resource-manager-vs-code/select-parameters.png)

6. 再次选择函数中的 **Ctrl+Space**，以查看模板中可用参数的列表。

    ![添加参数](./media/resource-manager-vs-code/select-avail-parameters.png)

7. 如果模板中出现任何架构验证问题，编辑器中会出现你所熟悉的波浪线。键入 **Ctrl+Shift+M** 或选择左下角状态栏中的字形，即可查看错误和警告列表。

    ![错误](./media/resource-manager-vs-code/errors.png)

    验证模板可帮助检测语法问题；但是，有些错误是可以忽略的。在某些情况下，编辑器会将模板与不处于最新状态的架构进行比较，因此，所报告的错误其实是正常的。例如，假设你最近将某个函数添加到了 Resource Manager，但架构尚未更新。尽管该函数在部署期间正常运行，但编辑器仍会报告错误。

    ![错误消息](./media/resource-manager-vs-code/unrecognized-function.png)

## 部署新资源

模板准备就绪后，你可以根据以下说明部署新资源：

### Windows

1. 打开 PowerShell 命令提示符
2. 若要登录，请键入：

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

3. 如果有多个订阅，请使用以下命令获取订阅列表：

        Get-AzureRmSubscription

    选择要使用的订阅。
   
        Select-AzureRmSubscription -SubscriptionId <Subscription Id>

4. 更新 parameters.json 文件中的参数
5. 运行 Deploy.ps1 以在 Azure 上部署模板

### OSX/Linux

1. 打开终端窗口
2. 若要登录，请键入：

        azure login -e AzureChinaCloud

3. 如果有多个订阅，请使用以下命令选择适当的订阅：

        azure account set <subscriptionNameOrId> 

4. 更新 parameters.json 文件中的参数。
5. 若要部署模板，请运行：

        azure group deployment create -f <PathToTemplate> 

## 后续步骤

- 若要详细了解模板，请参阅 [Authoring Azure Resource Manager templates](/documentation/articles/resource-group-authoring-templates/)（创作 Azure Resource Manager 模板）。
- 若要了解模板函数，请参阅 [Azure Resource Manager template functions](/documentation/articles/resource-group-template-functions/)（Azure Resource Manager 模板函数）。

<!---HONumber=Mooncake_1114_2016-->