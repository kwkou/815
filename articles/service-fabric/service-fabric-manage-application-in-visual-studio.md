<properties
   pageTitle="在 Visual Studio | Microsoft Azure 中管理应用程序"
   description="使用 Visual Studio 来创建、开发、打包、部署和调试 Service Fabric 应用程序和服务。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/09/2016"
   wacn.date="08/08/2016"
   ms.author="seanmck;mikhegn"/>

# 使用 Visual Studio 简化 Service Fabric 应用程序的编写和管理

你可以通过 Visual Studio 管理 Azure Service Fabric 应用程序和服务。[设置开发环境](/documentation/articles/service-fabric-get-started/)之后，你可以使用 Visual Studio 创建 Service Fabric 应用程序、添加服务，或在本地开发群集中打包、注册和部署应用程序。

## 部署 Service Fabric 应用程序

默认情况下，部署一个应用程序会将以下步骤合并为一个简单的操作：

1. 创建应用程序包
2. 将应用程序包上载到映像存储
3. 注册应用程序类型
4. 删除任何正在运行的应用程序实例
5. 创建新的应用程序实例

在 Visual Studio 中按 **F5** 还可以部署应用程序，并将调试器附加到所有应用程序实例。你可以使用 **Ctrl + F5** 部署应用程序而不进行调试，或者使用发布配置文件将应用程序发布到本地或远程群集。有关详细信息，请参阅[使用 Visual Studio 将应用程序发布到远程群集](/documentation/articles/service-fabric-publish-app-remote-cluster/)。

### 应用程序调试模式

默认情况下，当你停止调试或（如果你在部署应用时未附加调试器）当你重新部署应用程序时，Visual Studio 会删除你的应用程序类型的现有实例。在这种情况下，该应用程序的所有数据都将被删除。进行本地调试时，当你在测试应用程序的新版本时，可能会想要保留已经创建的数据。Visual Studio Service Fabric 工具提供一个名为“应用程序调试模式”的属性，该属性控制在调试会话结束之后，“F5”操作是卸载应用程序还是保留应用程序。

#### 设置“应用程序调试模式”属性

1. 在应用程序项目的快捷菜单上，选择“属性”（或按 **F4** 键）。
2. 在“属性”窗口中，将“应用程序调试模式”属性设置为“删除”或“自动升级”。

    ![设置“应用程序调试模式”属性][debugmodeproperty]

将此属性的值设置为“自动升级”将使应用程序继续在本地群集上运行。再次使用 **F5** 时，该操作会将该部署视为一次升级，使用无监控的自动模式将此应用程序快速升级到附加了日期字符串的较新的版本。此升级过程会保留你在上一个调试会话中输入的任何数据。

![附加了日期 1 的新应用程序版本的示例][preservedate]

将通过利用 Service Fabric 的应用程序升级功能保存数据，但会出于性能优化而非安全性的目的将数据进行优化。有关升级应用程序和在真实环境中如何执行升级的详细信息，请参阅 [Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)。

>[AZURE.NOTE] 1.1 版之前的适用于 Visual Studio 的 Service Fabric 工具版本不具有此属性。对于 1.1 版之前的版本，请使用 “启动时保留数据”属性来达到相同的效果。

## 向 Service Fabric 应用程序添加服务

可以向你的应用程序中添加新的服务来扩展其功能。若要确保你的应用程序包中包含服务，请通过“新 Fabric 服务...”菜单项添加服务。

![将新的 Fabric 服务添加到你的应用程序中][newservice]

选择要添加到你的应用程序的 Service Fabric 项目类型，并指定服务的名称。请参阅[为你的服务选择框架](/documentation/articles/service-fabric-choose-framework/)，以帮助你确定要使用的服务类型。

![选择要添加到你的应用程序的 Fabric 服务项目类型][addserviceproject]

新的服务会添加到你的解决方案和现有应用程序包中。服务引用和默认的服务实例会添加到应用程序清单中。将在下次部署应用程序时创建并启动服务。

![新的服务会添加到应用程序清单中][newserviceapplicationmanifest]

## 打包 Service Fabric 应用程序

要将应用程序及其服务部署到群集中，你需要创建应用程序包。该包会组织应用程序清单、服务清单和特定布局的其他必要文件。Visual Studio 设置和管理“pkg”目录中应用程序项目的文件夹中的包。从“应用程序”上下文菜单中单击“包”以创建或更新应用程序包。如果使用自定义 Powershell 脚本部署应用程序，你可能要执行此操作。

## 使用云资源管理器删除应用程序和应用程序类型

可以使用云资源管理器从 Visual Studio 内部执行基本的群集管理操作，可以从“视图”菜单启动该管理器。例如，可以删除本地或远程群集上的应用程序和取消设置其上的应用程序类型。

![删除应用程序](./media/service-fabric-manage-application-in-visual-studio/removeapplication.png)

>[AZURE.TIP] 有关更丰富的群集管理功能，请参阅[使用 Service Fabric 资源管理器可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)。


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

- [Service Fabric 应用程序模型](/documentation/articles/service-fabric-application-model/)
- [Service Fabric 应用程序部署](/documentation/articles/service-fabric-deploy-remove-applications/)
- [管理多个环境的应用程序参数](/documentation/articles/service-fabric-manage-multiple-environment-app-configuration/)
- [调试 Service Fabric 应用程序](/documentation/articles/service-fabric-debugging-your-application/)
- [使用 Service Fabric 资源管理器可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)

<!--Image references-->
[addserviceproject]: ./media/service-fabric-manage-application-in-visual-studio/addserviceproject.png
[manageservicefabric]: ./media/service-fabric-manage-application-in-visual-studio/manageservicefabric.png
[newservice]: ./media/service-fabric-manage-application-in-visual-studio/newservice.png
[newserviceapplicationmanifest]: ./media/service-fabric-manage-application-in-visual-studio/newserviceapplicationmanifest.png
[preservedata]: ./media/service-fabric-manage-application-in-visual-studio/preservedata.png
[preservedate]: ./media/service-fabric-manage-application-in-visual-studio/preservedate.png
[debugmodeproperty]: ./media/service-fabric-manage-application-in-visual-studio/debugmodeproperty.png

<!---HONumber=Mooncake_0801_2016-->