<properties
   pageTitle="使用 PowerShell 升级 Service Fabric 应用升级 | Azure"
   description="本文演示了如何使用 PowerShell 部署 Service Fabric 应用程序、更改代码，以及进行升级。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="11/15/2016"
   wacn.date="12/26/2016"
   ms.author="subramar"/>  





# 使用 PowerShell 升级 Service Fabric 应用程序
>[AZURE.SELECTOR]
- [PowerShell](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)
- [Visual Studio](/documentation/articles/service-fabric-application-upgrade-tutorial/)

最常用的推荐升级方法是受监控的滚动升级。Azure Service Fabric 基于一组运行状况策略监视正在升级的应用程序的运行状况。升级某个更新域 (UD) 之后，Service Fabric 将评估应用程序运行状况，根据运行状况策略继续升级下一个更新域，或者使升级失败。

可使用托管或本机 API、PowerShell 或 REST 执行应用程序的受监控升级。有关使用 Visual Studio 执行升级的说明，请参阅[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)。

使用 Service Fabric 监视的滚动升级，应用程序管理员可以配置 Service Fabric 用于确定应用程序运行状况是否正常的运行状况评估策略。此外，管理员还可以配置当运行状况评估失败时要采取的措施（例如，执行自动回滚）。 本部分将演练使用 PowerShell 对其中一个 SDK 示例进行受监视的升级。

## 步骤 1：构建和部署视觉对象示例
单击右键应用程序项目 **VisualObjectsApplication**，然后选择“发布”命令生成并发布应用程序。有关详细信息，请参阅 [Service Fabric 应用程序升级教程](/documentation/articles/service-fabric-application-upgrade-tutorial/)。或者，也可以使用 PowerShell 来部署应用程序。

> [AZURE.NOTE] 要在 PowerShell 中使用任何 Service Fabric 命令，必须先使用 `Connect-ServiceFabricCluster` cmdlet 连接到群集。同样，假设已在本地计算机上设置了群集。请参阅[设置 Service Fabric 部署环境](/documentation/articles/service-fabric-get-started/)上的文章。

在 Visual Studio 中构建项目后，可以使用 PowerShell 命令 **Copy-ServiceFabricApplicationPackage** 将应用程序包复制到 ImageStore。下一个步骤是使用 **Register-ServiceFabricApplicationPackage** cmdlet 将应用程序注册到 Service Fabric 运行时。最后一个步骤是使用 **New-ServiceFabricApplication** cmdlet 启动应用程序的实例。这三个步骤类似于使用 Visual Studio 中的“部署”菜单项。

现在可以使用 [Service Fabric 资源管理器查看群集和应用程序](/documentation/articles/service-fabric-visualizing-your-cluster/)。应用程序具有一个 Web 服务，可通过在 Internet Explorer 地址栏中键入 [http://localhost:8081/visualobjects](http://localhost:8081/visualobjects) 导航到该 Web 服务。你应在屏幕上看到一些四处移动的浮动视觉对象。此外，可使用 **Get-ServiceFabricApplication** 检查应用程序状态。

## 步骤 2：更新可视对象示例
你可能会注意到，使用步骤 1 中部署的版本，视觉对象不会旋转。让我们将此应用程序升级到可视对象也会旋转的版本。

选择 VisualObjects 解决方案中的 VisualObjects.ActorService 项目，然后打开该项目中的 StatefulVisualObjectActor.cs 文件。在该文件内，导航到 `MoveObject` 方法，注释掉 `this.State.Move()`，然后取消注释 `this.State.Move(true)`。此项更改可在升级服务后旋转对象。

我们还需要更新 **VisualObjects.ActorService** 项目的 *ServiceManifest.xml* 文件（位于 PackageRoot 下）。将 *CodePackage* 和服务版本更新到 2.0，以及 *ServiceManifest.xml* 文件中的相应行。可以在右键单击解决方案之后，使用 Visual Studio 中的“编辑清单文件”选项来进行清单文件更改。

完成更改后，清单应该如下所示（突出显示的部分即为所做的更改）：


	<ServiceManifestName="VisualObjects.ActorService" Version="2.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	
	<CodePackageName="Code" Version="2.0">


现在，*ApplicationManifest.xml* 文件（位于 **VisualObjects** 解决方案下的 **VisualObjects** 项目下）已更新为 **VisualObjects.ActorService** 项目的 2.0 版。此外，应用程序版本已从 1.0.0.0 更新为 2.0.0.0。*ApplicationManifest.xml* 应类似于以下代码片段：


	<ApplicationManifestxmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VisualObjects" ApplicationTypeVersion="2.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
	
	 <ServiceManifestRefServiceManifestName="VisualObjects.ActorService" ServiceManifestVersion="2.0" />



现在请生成项目，方法是选择 **ActorService** 项目，然后单击右键并选择 Visual Studio 中的“生成”选项。如果选择“全部重新生成”，应该更新所有项目的版本，因为代码已更改。接下来，将更新的应用程序打包，方法是右键单击“VisualObjectsApplication”，选择 Service Fabric 菜单，然后选择“打包”。此操作将创建可部署的应用程序包。更新的应用程序已准备就绪，可供部署。

## 步骤 3：确定运行状况策略和升级参数
请熟悉[应用程序升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)和[升级过程](/documentation/articles/service-fabric-application-upgrade/)，充分了解所应用的各种升级参数、超时和运行状况标准。对于本演练，服务运行状况评估标准设置为默认值（即推荐值），这意味着在升级后所有服务和实例均应为*运行状况正常*。

让我们将 *HealthCheckStableDuration* 增加到 60 秒（这样该服务在进行下一个更新域的升级之前将至少保持 20 秒的运行状况正常）。我们还将 *UpgradeDomainTimeout* 设置为 1200 秒，将 *UpgradeTimeout* 设置为 3000 秒。

最后，将 *UpgradeFailureAction* 设置为 rollback。此选项要求 Service Fabric 在升级期间如果遇到任何问题时，将应用程序回滚到旧版。因此在开始升级（步骤 4）时指定了以下参数：

FailureAction = Rollback

HealthCheckStableDurationSec = 60

UpgradeDomainTimeoutSec = 1200

UpgradeTimeout = 3000

## 步骤 4：准备应用程序升级
现在，应用程序已生成并准备好进行升级。如果以管理员身份打开 PowerShell 窗口并键入 **Get ServiceFabricApplication**，将会看到已部署 **VisualObjects** 的 1.0.0.0 应用程序类型。

应用程序包存储在解压 Service Fabric SDK 的位置，其相对路径为 *Samples\\Services\\Stateful\\VisualObjects\\VisualObjects\\obj\\x64\\Debug*。该目录中应会出现一个“Package”文件夹，这是存储应用程序包的位置。请检查时间戳，确保它是最新版本（可能还需要相应地修改路径）。

现在，让我们将更新的应用程序包复制到 Service Fabric ImageStore（Service Fabric 存储应用程序包的位置）。参数 *ApplicationPackagePathInImageStore* 告知 Service Fabric 可在何处找到应用程序包。我们已使用以下命令将更新的应用程序放入“VisualObjects\_V2”（可能需要再次相应地修改路径）。


	Copy-ServiceFabricApplicationPackage  -ApplicationPackagePath .\Samples\Services\Stateful\VisualObjects\VisualObjects\obj\x64\Debug\Package
	-ImageStoreConnectionString fabric:ImageStore   -ApplicationPackagePathInImageStore "VisualObjects\_V2"


下一步是向 Service Fabric 注册此应用程序，可使用以下命令执行此操作：


	Register-ServiceFabricApplicationType -ApplicationPathInImageStore "VisualObjects\_V2"


如果上面的命令未成功，可能需要重新生成所有服务。如步骤 2 中所述，你可能还需要更新 WebService 版本。

## 步骤 5：启动应用程序升级
现在，我们将使用以下命令启动应用程序升级：


	Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/VisualObjects -ApplicationTypeVersion 2.0.0.0 -HealthCheckStableDurationSec 60 -UpgradeDomainTimeoutSec 1200 -UpgradeTimeout 3000   -FailureAction Rollback -Monitored



应用程序名称与 *ApplicationManifest.xml* 文件中所述的相同。Service Fabric 使用此名称来确定升级的应用程序。如果设置的超时太短，则可能遇到一条说明该问题的失败消息。请参阅故障排除部分，或增加超时值。

现在，应用程序升级继续进行，可以使用 Service Fabric Explorer 或以下 PowerShell 命令对其进行监视：**Get-ServiceFabricApplicationUpgrade fabric:/VisualObjects**。

几分钟后，使用上述 PowerShell 命令获得的状态应说明所有更新域均已升级（已完成）。此外，浏览器窗口中的视觉对象已开始旋转！

可以练习从版本 2 升级到版本 3，或者从版本 2 升级到版本 1。从版本 2 过渡到版本 1 也被视为升级。尝试使用超时和运行状况策略，以加深熟悉程度。部署到 Azure 群集时，需要正确设置参数。最好保守地设置超时。

## 后续步骤

[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)逐步讲解了如何使用 Visual Studio 进行应用程序升级。

使用[升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)来控制应用程序的升级方式。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容。

参考[高级主题](/documentation/articles/service-fabric-application-upgrade-advanced/)，了解如何在升级应用程序时使用高级功能。

参考 [Troubleshooting Application Upgrades](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)（对应用程序升级进行故障排除）中的步骤来解决应用程序升级时的常见问题。

<!---HONumber=Mooncake_1219_2016-->