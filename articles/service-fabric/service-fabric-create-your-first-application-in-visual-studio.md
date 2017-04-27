<properties
    pageTitle="创建第一个 Azure 微服务应用程序 | Azure"
    description="使用 Visual Studio 创建、部署和调试 Service Fabric 应用程序"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="c3655b7b-de78-4eac-99eb-012f8e042109"
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="hero-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/07/2017"
    wacn.date="04/24/2017"
    ms.author="ryanwi"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="a9a7e1dfc55f5bd92ae95f2a21a30dd1f7201f43"
    ms.lasthandoff="04/14/2017" />

# <a name="create-your-first-azure-service-fabric-application"></a>创建第一个 Azure Service Fabric 应用程序
> [AZURE.SELECTOR]
- [C# - Windows](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)
- [Java - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
- [C# - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-csharp/)

Service Fabric SDK 包含一个用于 Visual Studio 的外接程序，它可提供用于创建、部署和调试 Service Fabric 应用程序的模板和工具。 本主题介绍在 Visual Studio 2017 或 Visual Studio 2015 中创建用户的第一个应用程序的过程。

## <a name="prerequisites"></a>先决条件
开始之前，请确保已 [设置开发环境](/documentation/articles/service-fabric-get-started/)。



## <a name="create-the-application"></a>创建应用程序
Service Fabric 应用程序可以包含一个或多个服务，每个服务都在提供应用程序功能时具有特定角色。 使用“新建项目”向导创建应用程序项目以及第一个服务项目。 如果需要，稍后还可添加更多服务。

1. 以管理员身份启动 Visual Studio。
2. 单击“文件”>“新建项目”>“云”>“Service Fabric 应用程序”。
3. 命名应用程序，然后单击“确定”。

	![Visual Studio 中的新建项目对话框][1]

4. 在下一页上，选择“有状态”作为要包括在应用程序中的第一种服务类型。命名它，然后单击“确定”。

	![Visual Studio 中的新建服务对话框][2]

	>[AZURE.NOTE] 有关选项的详细信息，请参阅 [Service Fabric 编程模型概述](/documentation/articles/service-fabric-choose-framework/)。

	Visual Studio 会创建应用程序项目和有状态服务项目，并在解决方案资源管理器中显示它们。

	![创建使用有状态服务的应用程序之后的解决方案资源管理器][3]

	应用程序项目不直接包含任何代码。而是引用一组服务项目。此外，它包含三种其他类型的内容：

	* **发布配置文件**：用于为不同环境管理工具首选项。

	* **脚本**：包括用于部署/升级应用程序的 PowerShell 脚本。Visual Studio 在幕后使用脚本。还可以在命令行处直接调用该脚本。

	* **应用程序定义**：包括 *ApplicationPackageRoot* 下的应用程序清单。关联应用程序参数文件位于 *ApplicationParameters* 下，它们定义应用程序并使您可以专门为给定环境对其进行配置。

    有关服务项目的内容概述，请参阅 [Reliable Services 入门](/documentation/articles/service-fabric-reliable-services-quick-start/)。

## <a name="deploy-and-debug-the-application"></a>部署和调试应用程序
现在已具有应用程序，尝试运行它。

1. 按 F5 以部署应用程序以便进行调试。

	>[AZURE.NOTE] 首次部署需要一段时间，因为 Visual Studio 要创建本地群集以用于开发。本地群集只在单台计算机上运行用户在多计算机群集中生成的相同平台代码。群集创建状态显示在 Visual Studio 输出窗口中。

	群集准备就绪时，将从 SDK 随附的本地群集系统托盘管理器应用程序收到通知。

	![本地群集系统托盘通知][4]  


2. 应用程序启动后，Visual Studio 会自动显示诊断事件查看器，在其中可以查看来自服务的跟踪输出。

	![诊断事件查看器][5]

	对于有状态服务模板，消息只显示在 MyStatefulService.cs 的 `RunAsync` 方法中递增的计数器值。

3. 展开事件之一可查看更多详细信息，包括运行代码的节点。 在此例中，它是 _Node_2，不过在你的计算机上可能会有所不同。

	![诊断事件查看器详细信息][6]  


	本地群集包含在单台计算机上托管的五个节点。它会模拟一个五节点群集，其中节点处于不同计算机上。为了模拟丢失计算机的情况，并同时练习使用 Visual Studio 调试器，让我们在本地群集上停止一个节点。

    >[AZURE.NOTE] 项目模板发出的应用程序诊断事件会使用包含的 `ServiceEventSource` 类。有关详细信息，请参阅[如何在本地监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)。

4. 在服务项目中查找派生自 StatefulService 的类（例如 MyStatefulService），然后在 `RunAsync` 方法的第一行上设置断点。

	![有状态服务 RunAsync 方法中的断点][7]


5. 若要启动 Service Fabric Explorer，请右键单击本地群集管理器系统托盘应用并选择“管理本地群集”。

    ![从本地群集管理器启动 Service Fabric 资源管理器][systray-launch-sfx]

    Service Fabric Explorer 提供群集的可视表示形式--包括部署到其中的应用程序集和构成它的物理节点集。要了解有关 Service Fabric Explorer 的详细信息，请参阅[可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)。

6. 在左窗格中，展开“群集”>“节点”，然后查找运行代码的节点。

7. 单击“操作”>“停用（重启）”来模拟计算机重启。 或者，从左窗格中的节点列表视图停用节点。）

	![在 Service Fabric 资源管理器中停止节点][sfx-stop-node]

	随着你在一个节点上进行的计算无缝地故障转移到另一个节点，你应立即在 Visual Studio 中看到命中了断点。

8. 返回到诊断事件查看器并观察消息。计数器在继续递增，即使事件实际上来自不同的节点。

    ![故障转移之后的诊断事件查看器][diagnostic-events-viewer-detail-post-failover]  

## <a name="switch-cluster-mode"></a>切换群集模式
默认情况下，本地开发群集配置为作为五节点群集运行，这对于调试在多个节点中部署的服务很有用。 但是，将应用程序部署到五节点开发群集需要一些时间。 如果想要快速循环访问代码更改，而不需要在 5 个节点上运行应用，可以将开发群集切换到单节点模式。 若要在单节点群集上运行代码，请右键单击系统任务栏中的本地群集管理器，并选择“切换群集模式”->“1 个节点”。  

![切换群集模式][switch-cluster-mode]  

更改群集模式时将重置开发群集，并删除所有在群集上预配或运行的应用程序。

也可使用 PowerShell 更改群集模式：

1. 以管理员身份启动新的 PowerShell 窗口。
2. 从 SDK 文件夹运行群集设置脚本：

        & "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1" -CreateOneNodeCluster

   
    群集设置需要一段时间。 完成设置后，应会看到类似如下的输出：
   
    ![群集设置输出][cluster-setup-success-1-node]

## <a name="cleaning-up"></a>清理
结束之前，请务必记得本地群集是真实的。 停止调试器会删除您的应用程序实例，并注销应用程序类型。 不过，群集将继续在后台运行。 可通过几个选项对群集进行管理：

1. 若要关闭群集，但保留应用程序数据和跟踪，请在系统托盘应用中单击“停止本地群集”  。
2. 要完全删除群集，请在系统托盘应用中单击“删除本地群集”  。 此选项会导致下次在 Visual Studio 中按 F5 时部署较慢。 仅当在一段时间内不想使用本地群集时，或者当需要回收资源时，才删除群集。

## <a name="next-steps"></a>后续步骤
* 了解如何[在 Azure 中创建群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)或[在 Windows 上创建独立群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)。
* 尝试使用 [Reliable Services](/documentation/articles/service-fabric-reliable-services-quick-start/) 或 [Reliable Actors](/documentation/articles/service-fabric-reliable-actors-get-started/) 编程模型创建服务。
* 尝试将现有应用部署为[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)。
* 了解如何使用 [Web 服务前端](/documentation/articles/service-fabric-add-a-web-frontend/)向 Internet 服务公开服务。
* 演练 [动手实验](https://msdnshared.blob.core.windows.net/media/2016/07/SF-Lab-Part-I.docx) 并创建一个无状态服务、配置监视和运行状况报告，然后执行应用程序升级。
* 了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)

<!-- Image References -->

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog.png
[2]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
[3]: ./media/service-fabric-create-your-first-application-in-visual-studio/solution-explorer-stateful-service-template.png
[4]: ./media/service-fabric-create-your-first-application-in-visual-studio/local-cluster-manager-notification.png
[5]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer.png
[6]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail.png
[7]: ./media/service-fabric-create-your-first-application-in-visual-studio/runasync-breakpoint.png
[sfx-stop-node]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-deactivate-node.png
[systray-launch-sfx]: ./media/service-fabric-create-your-first-application-in-visual-studio/launch-sfx.png
[diagnostic-events-viewer-detail-post-failover]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail-post-failover.png
[sfe-delete-application]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-delete-application.png
[switch-cluster-mode]: ./media/service-fabric-create-your-first-application-in-visual-studio/switch-cluster-mode.png
[cluster-setup-success-1-node]: ./media/service-fabric-get-started-with-a-local-cluster/cluster-setup-success-1-node.png
<!--update: add guidance for changing cluster mode;add anchors to sub titles-->