<properties
   pageTitle="Service Fabric 应用升级教程 | Azure"
   description="本文逐步指导你使用 Visual Studio 部署 Service Fabric 应用程序、更改代码以及推出升级版本。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="samgeo"
   editor=""/>

<tags
    ms.service="service-fabric"
    ms.date="05/18/2016"
    wacn.date="07/04/2016"/>




# 使用 Visual Studio 进行 Service Fabric 应用程序升级的教程

Azure Service Fabric 确保只升级已更改的服务，并在整个升级过程中监视应用程序的运行状况，从而可以简化云应用程序的升级过程。它还能在应用程序发生任何问题时自动回滚到旧版本。Service Fabric 应用程序升级造成的停机时间为零，因为可以在不停机的情况下升级应用程序。本教程介绍如何从 Visual Studio 完成简单的滚动升级。


## 步骤 1：构建和发布可视对象示例

可以通过从 GitHub 下载[可视对象](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/master/Actors/VisualObjects)应用程序来执行这些步骤，然后通过右键单击应用程序项目“VisualObjects”，随后从 Service Fabric 菜单项中选择“发布”命令来生成和发布该应用程序，如以下所示。

![Service Fabric 应用程序的上下文菜单][image1]

这会显示另一个弹出窗口，你可以将“目标配置文件”设置为“PublishProfiles\\Local.xml”。在单击“发布”之前，该窗口应如下所示。

![发布 Service Fabric 应用程序][image2]

现在，可在对话框中单击“发布”。可以使用 [Service Fabric 资源管理器查看群集和应用程序](/documentation/articles/service-fabric-visualizing-your-cluster/)。“可视对象”应用程序有一个 Web 服务，在浏览器的地址栏中输入 [http://localhost:8082/visualobjects/](http://localhost:8082/visualobjects/) 即可转到该服务。你应会在屏幕上看到 10 个四处移动的浮动可视对象。

## 步骤 2：更新可视对象示例

你可能会注意到，使用步骤 1 中部署的版本，可视对象不会旋转。让我们将此应用程序升级到可视对象也会旋转的版本。

选择 VisualObjects 解决方案中的 VisualObjects.ActorService 项目，然后打开 **VisualObjectActor.cs** 文件。在该文件中，转到 `MoveObject` 方法，注释掉 `visualObject.Move(false)`，然后取消注释 `visualObject.Move(true)`。此更改使得对象在服务升级后能够旋转。**现在可以生成（不是重新生成）解决方案**，这将生成修改后的项目。如果选择“全部重新生成”，则必须更新所有项目的版本。

我们还需要设置应用程序的版本。若要在右键单击“VisualObjects”项目之后进行版本更改，可以使用 Visual Studio 的“编辑清单版本”选项。随后将显示用于编辑版本的对话框，如下所示：

![版本控制对话框][image3]

将已修改的项目及其代码包连同应用程序的版本一起更新为 2.0.0。完成更改后，清单应该如下所示（粗体部分即为所做的更改）：

![更新版本][image4]

如果你选择“自动更新应用程序和服务版本”，Visual Studio 工具可以自动滚动更新版本。如果你使用 [SemVer](http://www.semver.org)，则在选择该选项后，需要单独更新代码和/或配置包版本。

保存更改，然后选中“升级应用程序”框。


## 步骤 3：升级应用程序

请熟悉[应用程序升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)和[升级过程](/documentation/articles/service-fabric-application-upgrade/)，以充分了解可应用的各种升级参数、超时和运行状况标准。在本演练中，请将服务运行状况评估标准保留为默认设置（不受监视模式）。若要配置这些设置，可以选择“配置升级设置”，然后视需要修改参数。

现在，我们已经准备好选择“发布”来开始升级应用程序。这会将应用程序升级到对象会旋转的版本 2.0.0。你将发现，Service Fabric 每次会升级一个更新域（先更新一些对象，再更新另一些对象），在此期间你可以通过客户端（浏览器）访问服务。


现在，随着应用程序不断升级，你可以使用 Service Fabric Explorer 来监视应用程序（使用应用程序下的“正在进行升级”选项卡）。

几分钟后，所有更新域应已升级（已完成），Visual Studio 输出窗口应该也会指出升级已完成。此外，你应会发现，浏览器窗口中的*所有* 可视对象现在都在旋转！

你可以尝试通过更改版本来练习本文所述的操作：从版本 2.0.0 升级到版本 3.0.0，或者从版本 2.0.0 降级到版本 1.0.0。尝试使用超时和运行状况策略，以加深熟悉程度。当你部署到 Azure 群集时，所用的参数将不同于部署到本地群集时所用的参数；建议你保守地设置超时。


## 后续步骤

[使用 PowerShell 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)将逐步指导你使用 PowerShell 进行应用程序升级。

使用[升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)来控制应用程序的升级方式。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容。

参考[高级主题](/documentation/articles/service-fabric-application-upgrade-advanced/)，了解如何在升级应用程序时使用高级功能。

参考[对应用程序升级进行故障排除](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)中的步骤来解决应用程序升级时的常见问题。



[image1]: ./media/service-fabric-application-upgrade-tutorial/upgrade7.png
[image2]: ./media/service-fabric-application-upgrade-tutorial/upgrade1.png
[image3]: ./media/service-fabric-application-upgrade-tutorial/upgrade5.png
[image4]: ./media/service-fabric-application-upgrade-tutorial/upgrade6.png

<!---HONumber=Mooncake_0627_2016-->