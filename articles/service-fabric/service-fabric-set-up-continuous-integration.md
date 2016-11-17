<properties
   pageTitle="Service Fabric 的持续集成 | Azure"
   description="大致了解如何使用 Visual Studio Team Services (VSTS) 为 Service Fabric 应用程序设置持续集成。"
   services="service-fabric"
   documentationCenter="na"
   authors="mthalman-msft"
   manager="timlt"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="08/01/2016"
   wacn.date="08/29/2016" />

# 使用 Visual Studio Team Services 为 Service Fabric 应用程序设置持续集成

本文介绍使用 Visual Studio Team Services (VSTS) 为 Azure Service Fabric 应用程序设置持续集成，从而确保以自动方式生成、打包和部署应用程序的步骤。

本文档反映当前过程，应随时间推移而更改。

## 先决条件

请按照以下步骤开始：

1. 确保你有权访问 Team Services 帐户，否则请自行[创建一个](https://www.visualstudio.com/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services)。

2. 确保你有权访问 Team Services 团队项目，否则请自行[创建一个](https://www.visualstudio.com/docs/setup-admin/create-team-project)。

3. 确保你有一个可以向其部署应用程序的 Service Fabric 群集，或者使用 [Azure Resource Manager 模板](/documentation/articles/service-fabric-cluster-creation-via-arm/)或 [Visual Studio](/documentation/articles/service-fabric-cluster-creation-via-visual-studio/) 创建一个。

4. 确保已创建 Service Fabric 应用程序 (.sfproj) 项目。必须具有使用 Service Fabric SDK 2.1 或更高版本创建或升级的项目（.sfproj 文件应包含 1.1 或更高的 ProjectVersion 属性值）。

>[AZURE.NOTE] 不再需要自定义生成代理。Team Services 托管代理现在预装了 Service Fabric 群集管理软件，因此可以直接从这些代理部署应用程序。

## 配置和共享源文件

你首先要做的事就是准备一个发布配置文件，供将要在 Team Services 中执行的部署进程使用。应将发布配置文件配置为针对此前已准备好的群集：

1.	在应用程序项目中选择一个发布配置文件，以便将其用于持续集成工作流，然后按照[发布说明](/documentation/articles/service-fabric-publish-app-remote-cluster/)将应用程序发布到远程群集。不过，你实际上不需要发布你的应用程序。正确地完成配置操作以后，单击发布对话框中的“保存”超链接即可。
2.	如果你希望每次在 Team Services 中进行部署时都升级应用程序，则需对发布配置文件进行配置，使之允许升级。在步骤 1 所使用的同一个发布对话框中，确保已勾选“升级应用程序”复选框。详细了解如何[配置其他升级设置](/documentation/articles/service-fabric-visualstudio-configure-upgrade/)。单击“保存”超链接，将设置保存到发布配置文件。
3.	确保将更改保存到发布配置文件以后，即可取消发布对话框。
4.	现在可以在 Team Services 中[共享应用程序项目源文件](https://www.visualstudio.com/docs/setup-admin/team-services/connect-to-visual-studio-team-services#vs)。能够在 Team Services 中访问源文件以后，即可转到下一步，即创建生成。

## 创建生成定义

Team Services 生成定义所描述的工作流由一系列按顺序执行的生成步骤组成。之所以要创建生成定义，其目的是生成一个 Service Fabric 应用程序包，并增加一些其他的补充性文件，然后即可使用该程序包将应用程序最终部署到群集。详细了解 Team Services [生成定义](https://www.visualstudio.com/docs/build/define/create)。

### 从生成模板中创建定义

1.	在 Visual Studio Team Services 中打开团队项目。
2.	选择“生成”选项卡。
3.	选择绿色的 **+** 号以创建新的生成定义。
4.	在打开的对话框中，选择“生成”模板类别中的“Azure Service Fabric 应用程序”。
5.	选择“下一步”。
6.	选择与 Service Fabric 应用程序关联的存储库和分支。
7.	选择要使用的代理队列。支持托管代理。
8.	选择“创建”。
9. 保存生成定义并为其命名。
10. 下面是由此模板生成的生成步骤的说明：

| 生成步骤 | 说明 |
| --- | --- |
| Nuget 还原 | 还原解决方案的 NuGet 包。 |
| 生成解决方案 *.sln | 生成整个解决方案。 |
| 生成解决方案 *.sfproj | 生成将要用于部署应用程序的 Service Fabric 应用程序包。请注意，已将应用程序包位置指定为生成的项目目录。 |
| 更新 Service Fabric 应用版本 | 更新应用程序包的清单文件中包含的版本值，使之支持升级。有关详细信息，请参阅[任务文档页](https://go.microsoft.com/fwlink/?LinkId=820529)。 |
| 复制文件 | 将发布配置文件和应用程序参数文件复制到生成的项目中，使之能够用来进行部署。 |
| 发布项目 | 发布生成的项目。这样即可以通过发布定义来使用生成的项目。 |

### 验证任务的默认设置

1.	验证“NuGet 还原”和“生成解决方案”生成步骤的“解决方案”输入字段。默认情况下，会根据关联的存储库中包含的所有解决方案文件执行这些生成步骤。如果你只想在这其中的一个解决方案文件上运行生成定义，则需显式更新该文件的路径。
2.	验证“打包应用程序”生成步骤的“解决方案”输入字段。默认情况下，此生成步骤假定存储库中只存在一个 Service Fabric 应用程序项目 (.sfproj)。如果存储库中有多个这样的文件，而你只想针对其中一个来完成此生成定义，则需显式更新该文件的路径。如果想将多个应用程序项目打包到存储库中，则需在生成定义中创建其他 **Visual Studio**“生成”步骤，使每个步骤都对应一个应用程序项目。然后，还需针对每个这样的生成步骤更新“MSBuild 参数”字段，使包位置对每个这样的生成步骤来说都是唯一的。
3.	验证“更新 Service Fabric 应用版本”生成步骤中定义的版本控制行为。默认情况下，此生成步骤会将生成号追加到应用程序包的清单文件中的所有版本值。有关详细信息，请参阅[任务文档页](https://go.microsoft.com/fwlink/?LinkId=820529)。这对提供应用程序升级支持很有用，因为每个升级部署都需要前一部署中的不同版本值。如果你不打算在工作流中使用应用程序升级，则可考虑禁用此生成步骤。事实上，如果意图生成可用于覆盖现有 Service Fabric 应用程序的版本，则必须禁用此生成步骤，因为如果该生成步骤所生成的应用程序版本与群集中的应用程序不匹配，则部署将失败。
4.	如果您的解决方案包含.NET 核心项目，则必须确保生成定义包含可还原任何 project.json 文件所定义依赖项的生成步骤。为此，请执行以下步骤：
   1. 选择“添加生成步骤...”。
   2. 找到“实用工具”选项卡中的“命令行”任务，单击其“添加”按钮。
   3. 对于任务的输入字段，请使用以下值：
      1. 工具：dotnet
      2. 参数：restore
   4. 拖动该任务，以便它在执行 **NuGet 还原**步骤后立即可用。
5.	保存对生成定义所做的任何更改。

### 试用

选择“为生成排队”以手动启动生成。在推送或签入时也将触发生成。验证生成可以成功执行以后，即可继续操作，以便定义将应用程序部署到群集的发布定义。

## 创建发布定义

Team Services 发布定义所描述的工作流由一系列按顺序执行的任务组成。之所以要创建发布定义，其目的是获取应用程序包并将其部署到群集。一起使用时，生成定义和发布定义可以执行从开始到结束的整个工作流，即一开始只有源文件，而结束时群集中会有一个运行的应用程序。详细了解 Team Services [发布定义](https://www.visualstudio.com/docs/release/author-release-definition/more-release-definition)。

### 从发布模板中创建定义

1.	在 Visual Studio Team Services 中打开你的项目。
2.	选择“发布”选项卡。
3.	选择绿色的 **+** 号以创建新的发布定义，然后在菜单中选择“创建发布定义”。
4.	在打开的对话框中，选择“部署”模板类别中的“Azure Service Fabric 部署”。
5.	选择“下一步”。
6.	选择生成定义，以便将其用作此发布定义的源。发布定义将引用由所选生成定义生成的项目。
7.	如果希望在生成完成时 Team Services 能够自动创建新发布并部署 Service Fabric 应用程序，请选中“连续部署”复选框。
8.	选择要使用的代理队列。支持托管代理。
9.	选择“创建”。
10.	通过单击页面顶部的铅笔图标编辑定义名称。
11.	选择要将您的应用程序从任务的“群集连接”输入字段部署到其中的群集。群集连接提供的必要信息可以让部署任务连接到群集。如果尚未为群集设置群集连接，可选择字段旁的“管理”超链接添加一个。在打开的页面上，执行以下步骤：
    1. 选择“新建服务终结点”，然后从菜单中选择“Azure Service Fabric”。
    2. 选择此终结点针对的群集所要使用的身份验证类型。
    2. 在“连接名称”字段中定义连接的名称。通常情况下，你会使用群集的名称。
    3. 在“群集终结点”字段中定义客户端连接终结点 URL。示例：https://contoso.chinaeast.chinacloudapp.cn:19000。
    4. 对于 Azure Active Directory 凭据，可在“用户名”和“密码”字段中定义需要用来连接到群集的凭据。
    5. 对于基于证书的身份验证，可在“客户端证书”字段中定义客户端证书文件的 Base64 编码。若要了解如何获取该值，请参阅有关该字段的弹出帮助。如果证书受密码保护，可在“密码”字段中定义密码。
    6. 单击“确定”确认所做的更改。导航回发布定义以后，单击“群集连接”字段的刷新图标即可查看刚添加的终结点。
12.	保存发布定义。

创建的定义包含类型 **Service Fabric 应用程序部署**的一个任务。有关此任务的详细信息，请参阅[任务文档页](https://go.microsoft.com/fwlink/?LinkId=820528)。

### 验证模板默认值

1.	验证“部署 Service Fabric 应用程序”任务的“发布配置文件”输入字段。默认情况下，此字段引用包含在生成的项目中的名为 Cloud.xml 的发布配置文件。如果你要引用其他发布配置文件，或者如果生成在其项目中包含多个应用程序包，则需对路径进行相应更新。
2.	验证“部署 Service Fabric 应用程序”任务的“应用程序包”输入字段。默认情况下，此项引用在生成定义模板中使用的默认应用程序包路径。如果已在生成定义中修改默认应用程序包路径，则还需在此处对路径进行相应更新。

### 试用

从“发布”按钮菜单中选择“创建发布”，以便手动创建发布。在打开的对话框中，按需要选择发布所基于的生成，然后单击“创建”。如果已启用连续部署，则当关联的生成定义完成某个生成时，也会自动创建发布。

## 后续步骤

若要了解有关与 Service Fabric 应用程序持续集成的详细信息，请阅读以下文章：

 - [Team Services 文档主页](https://www.visualstudio.com/docs/overview)
 - [Team Services 中的生成管理](https://www.visualstudio.com/docs/build/overview)
 - [Team Services 中的发布管理](https://www.visualstudio.com/docs/release/overview)

<!---HONumber=Mooncake_0822_2016-->