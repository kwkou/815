<properties
   pageTitle="使用 PowerShell 升级 Service Fabric 应用 | Azure"
   description="本文逐步指导你使用 PowerShell 部署 Service Fabric 应用程序、更改代码以及推出升级版本。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/13/2016"
   wacn.date="07/04/2016"/>




# 使用 PowerShell 升级 Service Fabric 应用程序

最常用的推荐升级方法是受监控的滚动升级。Azure Service Fabric 基于一组运行状况策略监视正在升级的应用程序的运行状况。当更新域 (UD) 中的应用程序已升级时，Service Fabric 评估应用程序运行状况，并确定是转到下一个更新域还是基于运行状况策略判定升级失败。

可使用托管或本机 API、PowerShell 或 REST 执行应用程序的受监控升级。有关使用 Visual Studio 执行升级的说明，请参阅[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)。

使用 Service Fabric 监视的滚动升级，应用程序管理员可以配置 Service Fabric 用于确定应用程序运行状况是否正常的运行状况评估策略。此外，管理员还可以配置当运行状况评估失败时要采取的措施（例如，执行自动回滚）。 本部分将演练使用 PowerShell 对其中一个 SDK 示例进行受监视的升级。

## 步骤 1：构建和部署视觉对象示例


在应用程序项目 **VisualObjectsApplication** 上单击右键，然后从 Service Fabric 菜单项中选择“发布”命令以构建和发布应用程序，如以下所示。有关详细信息，请参阅 [Service Fabric 应用程序升级教程](/documentation/articles/service-fabric-application-upgrade-tutorial/)。或者，也可以使用 PowerShell 来部署应用程序。

> [AZURE.NOTE] 要在 PowerShell 中使用任何 Service Fabric 命令，必须先使用 `Connect-ServiceFabricCluster` cmdlet 连接到群集。同样，假设已在本地计算机上设置了群集。请参阅[设置 Service Fabric 部署环境](/documentation/articles/service-fabric-get-started/)上的文章。

在 Visual Studio 中构建项目后，可以使用 PowerShell 命令 **Copy-ServiceFabricApplicationPackage** 将应用程序包复制到 ImageStore。完成此步骤后，接着使用 **Register-ServiceFabricApplicationPackage** cmdlet 将应用程序注册到 Service Fabric 运行时。最后一个步骤是使用 **New-ServiceFabricApplication** cmdlet 来启动应用程序的实例。这三个步骤类似于使用 Visual Studio 中的“部署”菜单项。

现在可以使用 [Service Fabric 资源管理器查看群集和应用程序](/documentation/articles/service-fabric-visualizing-your-cluster/)。应用程序具有一个 Web 服务，可通过在 Internet Explorer 地址栏中键入 [http://localhost:8081/visualobjects](http://localhost:8081/visualobjects) 导航到该 Web 服务。你应在屏幕上看到一些四处移动的浮动视觉对象。此外，可使用 **Get-ServiceFabricApplication** 检查应用程序状态。

## 步骤 2：更新可视对象示例

你可能会注意到，使用步骤 1 中部署的版本，视觉对象不会旋转。让我们将此应用程序升级到可视对象也会旋转的版本。

选择 VisualObjects 解决方案中的 VisualObjects.ActorService 项目，然后打开该项目中的 StatefulVisualObjectActor.cs 文件。在该文件内，导航到 `MoveObject` 方法，注释掉 `this.State.Move()`，然后取消注释 `this.State.Move(true)`。此更改使得对象在服务升级后能够旋转。

我们还需要更新 **VisualObjects.ActorService** 项目的 *ServiceManifest.xml* 文件（位于 PackageRoot 下）。将 *CodePackage* 和服务版本更新到 2.0，以及 *ServiceManifest.xml* 文件中的相应行。可以在右键单击解决方案之后，使用 Visual Studio 中的“编辑清单文件”选项来进行清单文件更改。


完成更改后，清单应该如下所示（突出显示的部分即为所做的更改）：

```xml
<ServiceManifestName="VisualObjects.ActorService" Version="2.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

<CodePackageName="Code" Version="2.0">
```

现在，我们需要将 *ApplicationManifest.xml* 文件（位于 **VisualObjects** 解决方案下的 **VisualObjects** 项目下）更新为使用 **VisualObjects.ActorService** 项目的 2.0 版，并将应用程序从 1.0.0.0 更新到 2.0.0.0。现在，*ApplicationManifest.xml* 中的相应行应如下所示：

```xml
<ApplicationManifestxmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VisualObjects" ApplicationTypeVersion="2.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

 <ServiceManifestRefServiceManifestName="VisualObjects.ActorService" ServiceManifestVersion="2.0" />
```


现在生成项目，方法是选择 **ActorService** 项目，然后单击右键并选择 Visual Studio 中的“生成”。（如果选择“全部重新生成”，可能需要在该项目的 *ServiceManifest.xml* 和 *ApplicationManifest.xml* 中更新其他项目的版本，因为代码已更改）。接下来，让我们将更新的应用程序打包，方法是右键单击 ***VisualObjectsApplication***，选择 Service Fabric 菜单，然后选择“打包”。这将创建可部署的应用程序包。更新的应用程序现在准备就绪，可供部署。


## 步骤 3：确定运行状况策略和升级参数

请熟悉[应用程序升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)和[升级过程](/documentation/articles/service-fabric-application-upgrade/)，以充分了解所应用的各种升级参数、超时和运行状况标准。对于本演练，我们将服务运行状况评估标准保留为默认值（即推荐值），这意味着在升级后所有服务和实例均应为运行状况正常。

但是，我们将 *HealthCheckStableDuration* 增加到 60 秒（这样该服务在进行下一个更新域的升级之前将至少保持 20 秒的运行状况正常）。我们还将 *UpgradeDomainTimeout* 设置为 1200 秒，将 *UpgradeTimeout* 设置为 3000 秒。

最后，我们将 *UpgradeFailureAction* 设置为 rollback，以便在升级期间遇到任何问题时，请求 Service Fabric 将应用程序回滚到前一版本。因此，我们在启动升级（步骤 4 中）时指定的升级参数将如下所示：

FailureAction = Rollback

HealthCheckStableDurationSec = 60

UpgradeDomainTimeoutSec = 1200

UpgradeTimeout = 3000


## 步骤 4：准备应用程序升级

现在，应用程序已生成并准备好进行升级。如果你作为管理员打开 PowerShell 窗口并键入 **Get ServiceFabricApplication**，则你将看到已部署 **VisualObjects** 的 1.0.0.0 应用程序类型。

应用程序包存储在解压 Service Fabric SDK 的位置，其相对路径为 *Samples\\Services\\Stateful\\VisualObjects\\VisualObjects\\obj\\x64\\Debug*。你应在该目录中找到“包”文件夹，这是存储应用程序包的位置。请检查时间戳以确保它是最新版本（可能还需要相应地修改路径）。

现在，让我们将更新的应用程序包复制到 Service Fabric ImageStore（Service Fabric 存储应用程序包的位置）。参数 *ApplicationPackagePathInImageStore* 告知 Service Fabric 可在何处找到应用程序包。我们已使用以下命令将更新的应用程序放入“VisualObjects\_V2”（可能需要再次相应地修改路径）。

```powershell
Copy-ServiceFabricApplicationPackage  -ApplicationPackagePath .\Samples\Services\Stateful\VisualObjects\VisualObjects\obj\x64\Debug\Package
-ImageStoreConnectionString fabric:ImageStore   -ApplicationPackagePathInImageStore "VisualObjects\_V2"
```

下一步是向 Service Fabric 注册此应用程序，可使用以下命令执行此操作：

```powershell
Register-ServiceFabricApplicationType -ApplicationPathInImageStore "VisualObjects\_V2"
```

如果上面的命令未成功，则你可能需要重新生成所有服务。如步骤 2 中所述，你可能还需要更新 WebService 版本。

## 步骤 5：启动应用程序升级

现在，我们将使用以下命令启动应用程序升级：

```powershell
Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/VisualObjects -ApplicationTypeVersion 2.0.0.0 -HealthCheckStableDurationSec 60 -UpgradeDomainTimeoutSec 1200 -UpgradeTimeout 3000   -FailureAction Rollback -Monitored
```


请注意，应用程序名称与 *ApplicationManifest.xml* 文件中所述的相同。Service Fabric 使用此名称来确定升级的应用程序。如果设置的超时太短，则可能遇到一条说明该问题的失败消息。请参阅故障排除部分，或增加超时值。

现在，应用程序升级继续进行，你可以使用 Service Fabric Explorer 或以下 PowerShell 命令对其进行监视：**Get-ServiceFabricApplicationUpgrade fabric:/VisualObjects**。

几分钟后，使用上述 PowerShell 命令获得的状态应说明所有更新域均已升级（已完成）。并且你会发现浏览器窗口中的视觉对象现在已开始旋转！

作为一项联系，你可能想要尝试更改版本，并从版本 2 升级到版本 3，甚至从版本 2 回滚到版本 1（是的，你可以从 v2 升到 v1）。尝试使用超时和运行状况策略，以加深熟悉程度。部署到 Azure 群集时，使用的参数将不同于部署到本地群集时使用的参数，因此，最好保守地设置超时。


## 后续步骤

[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)将逐步指导你使用 Visual Studio 进行应用程序升级。

使用[升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)来控制应用程序的升级方式。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容

参考[高级主题](/documentation/articles/service-fabric-application-upgrade-advanced/)，了解如何在升级应用程序时使用高级功能。

参考 [Troubleshooting Application Upgrades](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)（对应用程序升级进行故障排除）中的步骤来解决应用程序升级时的常见问题。

<!---HONumber=Mooncake_0627_2016-->