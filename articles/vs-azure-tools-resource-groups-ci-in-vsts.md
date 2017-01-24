<properties
	pageTitle="使用 Azure 资源组项目在 VS Team Services 中连续集成 | Azure"
	description="介绍如何在 Visual Studio 中使用 Azure 资源组部署项目在 Visual Studio Team Services 中连续集成。"
	services="visual-studio-online"
	documentationCenter="na"
	authors="TomArcher"
	manager="douge"
	editor="" />

 <tags
	ms.service="azure-resource-manager"
	ms.date="04/19/2016"
	wacn.date="05/23/2016" />

# 使用 Azure 资源组部署项目在 Visual Studio Team Services 中连续集成

若要部署 Azure 模板，需要在各个阶段执行任务：生成、测试、复制到 Azure（也称为“暂存”）和部署模板。在 Visual Studio Team Services (VS Team Services) 中可通过两种不同的方法执行这些任务。两种方法产生的结果相同，因此请选择最符合工作流的方法。

-	在运行 PowerShell 脚本的生成定义（包含在 Azure 资源组部署项目中，即 Deploy-AzureResourceGroup.ps1）中添加一个步骤。该脚本将复制项目，然后部署模板。
-	添加多个 VS Team Services 生成步骤，每个步骤执行一个阶段任务。

本文演示如何使用第一个选项（使用生成定义来运行 PowerShell 脚本）。此选项的优点之一是开发人员在 Visual Studio 中使用的脚本与 VS Team Services 使用的脚本相同。此过程假设你已经在 VS Team Services 中签入 Visual Studio 部署项目。

## 将项目复制到 Azure 

无论方案如何，如果需要为模板部署使用任何项目，都需要向 Azure Resource Manager 授予这些项目的访问权限。这些项目可以包括如下所述的文件：

-	嵌套模板
-	配置脚本和 DSC 脚本
-	应用程序二进制文件

### 嵌套模板和配置脚本
当你使用 Visual Studio 提供的模板（或以 Visual Studio 代码段生成的模板）时，PowerShell 脚本不但会暂存项目，而且会参数化用于不同部署的资源的 URI。脚本会将项目复制到 Azure 中的安全容器，为该容器创建 SaS 令牌，然后将该信息传递到模板部署。有关嵌套模板的详细信息，请参阅 [创建模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790564.aspx)。

## 在 VS Team Services 中设置连续部署

若要在 VS Team Services 中调用 PowerShell 脚本，请更新生成定义。简而言之，请执行以下步骤：

1.	编辑生成定义。
1.	在 VS Team Services 中设置 Azure 授权。
1.	添加一个引用 Azure 资源组部署项目中 PowerShell 脚本的 Azure PowerShell 生成步骤。
1.	设置 “-ArtifactsStagingDirectory” 参数值，以使用 VS Team Services 中生成的项目。

### 详细演练

以下步骤将引导你完成在 VS Team Services 配置连续部署所要执行的步骤

1.	编辑 VS Team Services 生成定义并添加 Azure PowerShell 生成步骤。在“生成定义”类别下选择生成定义，然后选择“编辑”链接。

    ![][0]

1.	在生成定义中添加“Azure PowerShell”生成步骤，然后选择“添加生成步骤...”按钮。

    ![][1]

1.	选择“部署任务”类别，选择“Azure PowerShell”任务，然后选择其“添加”按钮。

    ![][2]

1.	选择“Azure PowerShell”生成步骤，然后填充其值。

    1.	如果你已将 Azure 服务终结点添加到 VS Team Services，请在“Azure 订阅”下拉列表框中选择订阅，并跳到下一部分。 

        如果 VS Team Services 中没有 Azure 服务终结点，请添加一个。本小节将引导你完成整个过程。如果你的 Azure 帐户使用 Microsoft 帐户（例如 Hotmail），需要执行以下步骤以获取服务主体身份验证。

    1.	选择“Azure 订阅”下拉列表框旁边的“管理”链接。

        ![][3]

    1. 在“新建服务终结点”下拉列表框中选择“Azure”。

        ![][4]

    1.	在“添加 Azure 订阅”对话框中，选择“服务主体”选项。

        ![][5]

    1.	在“添加 Azure 订阅”对话框中添加 Azure 订阅信息。首先需要提供以下项：
        -	订阅 ID
        -	订阅名称
        -	服务主体 ID
        -	服务主体密钥
        -	租户 ID

    1.	在“订阅”名称框中添加所选的名称。此值稍后将显示在 VS Team Services 中的“Azure 订阅”下拉列表内。

    1.	如果你不知道自己的 Azure 订阅 ID，可以使用以下命令之一来获取 ID。
        
        对于 Azure PowerShell 脚本，请使用：

        `Get-AzureRmSubscription`

        对于 Azure CLI，请使用：

        `azure account show`
    

    1.	要获取服务主体 ID、服务主体密钥和租户 ID，请遵循 [使用门户创建 Active Directory 应用程序和服务主体](/documentation/articles/resource-group-create-service-principal-portal/)或 [通过 Azure 资源管理器对服务主体进行身份验证](/documentation/articles/resource-group-authenticate-service-principal/)的过程。

    1.	在“添加 Azure 订阅”对话框中添加服务主体 ID、服务主体密钥和租户 ID 值，然后选择“确定”按钮。

        现在，你已获得了用于运行 Azure PowerShell 脚本的有效服务主体。

1.	编辑生成定义并选择 **Azure PowerShell** 生成步骤。在“Azue 订阅”下拉列表框中选择订阅。（如果订阅未出现，请选择“管理”链接旁边的“刷新”按钮。）

    ![][8]

1.	提供 Deploy-AzureResourceGroup.ps1 PowerShell 脚本的路径。为此，请选择“脚本路径”框旁边的省略号 (…) 按钮，导航到项目的“脚本”文件夹中的 Deploy-AzureResourceGroup.ps1 PowerShell 脚本，选择该脚本，然后选择“确定”按钮。

    ![][9]

1. 选择该脚本后，更新脚本的路径，以便从 Build.StagingDirectory（与 “ArtifactsLocation” 设置到的目录相同）运行脚本。可以通过在脚本路径的开头添加“$(Build.StagingDirectory)/”来执行此操作。

    ![][10]

1.	在“脚本参数”框中输入以下参数（独行输入）。在 Visual Studio 中运行脚本时，可以在“输出”窗口中看到 VS 如何使用参数。在生成步骤中设置参数值时，可使用此设置作为起点。

    | 参数 | 说明|
    |---|---|
    | -ResourceGroupLocation | 资源组所在的地理位置，例如 **China North** 或 **'China East'**。| |
    | -ResourceGroupName | 此部署使用的资源组名称。| |
    | -UploadArtifacts | 如果提供此参数，将指定需要从本地系统将项目上载到 Azure。仅当模板部署需要使用 PowerShell 脚本暂存其他项目（例如配置脚本或嵌套模板）时，才需要设置此开关。 |
    | -StorageAccountName | 用于暂存此部署的项目的存储帐户名称。仅当你要将项目复制到 Azure 时，才需要此参数。部署不会自动创建此存储帐户，因此必须事先提供此存储帐户。| |
    | -StorageAccountResourceGroupName | 与存储帐户关联的资源组名称。仅当你要将项目复制到 Azure 时，才需要此参数。| |
    | -TemplateFile | Azure 资源组部署项目中的模板文件路径。为了提高灵活性，请为此参数使用相对于 PowerShell 脚本位置的路径，而不要使用绝对路径。|
    | -TemplateParametersFile | Azure 资源组部署项目中的参数文件路径。为了提高灵活性，请为此参数使用相对于 PowerShell 脚本位置的路径，而不要使用绝对路径。|
    | -ArtifactStagingDirectory | 此参数使 PowerShell 脚本能够知道应从哪个文件夹中复制项目的二进制文件。此值将覆盖 PowerShell 脚本使用的默认值。为了方便 VS Team Services 使用，请将该值设为：ArtifactStagingDirectory $(Build.StagingDirectory) |

    下面是脚本参数示例（换行是为了便于阅读）：

    ```	
    -ResourceGroupName 'MyGroup' -ResourceGroupLocation 'eastus' -TemplateFile '..\templates\azuredeploy.json' 
    -TemplateParametersFile '..\templates\azuredeploy.parameters.json' -UploadArtifacts -StorageAccountName 'mystorageacct' 
    –StorageAccountResourceGroupName 'Default-Storage-EastUS' -ArtifactStagingDirectory '$(Build.StagingDirectory)'	
    ```

    完成之后，“脚本参数”框应如下所示。

    ![][11]

1.	将所有必需的项添加 Azure PowerShell 生成步骤之后，请选择“队列”生成按钮来生成项目。“生成”屏幕将显示 PowerShell 脚本的输出。

## 后续步骤

有关 Azure Resource Manager 和 Azure 资源组的详细信息，请参阅 [Azure Resource Manager overview（Azure Resource Manager 概述）](/documentation/articles/resource-group-overview/)。


[0]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough1.png
[1]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough2.png
[2]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough3.png
[3]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough4.png
[4]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough5.png
[5]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough6.png
[8]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough9.png
[9]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough10.png
[10]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough11b.png
[11]: ./media/vs-azure-tools-resource-groups-ci-in-vsts/walkthrough12.png
<!---HONumber=Mooncake_0516_2016-->