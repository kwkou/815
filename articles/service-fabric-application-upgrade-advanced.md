<properties
   pageTitle="应用程序升级：高级主题 | Azure"
   description="本文介绍了与升级 Service Fabric 应用程序有关的一些高级主题。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="samgeo"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/18/2016"
   wacn.date="07/04/2016"/>

# Service Fabric 应用程序升级：高级主题

## 在升级应用程序期间添加或删除服务

如果将新服务添加到已部署的应用程序，并将其作为升级项发布，则该新服务将添加到部署的应用程序（同时，升级不会影响到已属于应用程序一部分的任何服务）。但是，若要激活新服务，必须启动已添加的服务的实例（使用 `New-ServiceFabricService` cmdlet）。

还可以在升级期间从应用程序中删除服务，但是，在继续升级之前，用户需要确保服务（要在升级期间删除的服务）的所有当前实例已停止（使用 `Remove-ServiceFabricService` cmdlet）。

## 手动升级模式

> [AZURE.NOTE]  只应针对已失败或暂停的升级考虑使用不受监视的手动模式。受监视模式是建议用于 Service Fabric 应用程序的升级模式。

Azure Service Fabric 提供了多个升级模式，可支持开发和生产群集。每个部署选项都非常适合不同的环境。

受监视应用程序滚动升级是生产中使用的最典型的升级。当指定了升级策略时，Service Fabric 可确保在升级继续进行之前，应用程序处于正常状态。

 应用程序管理员可以使用手动滚动应用程序升级模式，通过各种升级域获得升级过程的完全控制权。对于某些需要自定义程度更高或更复杂的运行状况评估策略的情况，或者如果存在非常规升级（例如，应用程序已有数据丢失），此模式非常有用。

最后，自动应用程序滚动升级对开发或测试环境很有用，可在服务开发期间提供快速迭代周期。

## 切换到手动升级模式
**手动** - 在当前 UD 停止应用程序升级，并将升级模式更改为不受监视的手动模式。管理员需要手动调用 **MoveNextApplicationUpgradeDomainAsync**，以继续进行升级或通过启动新升级触发回滚。升级进入到“手动”模式后，就会保持为“手动”模式，直到启动新的升级。**GetApplicationUpgradeProgressAsync** 命令将返回 FABRIC\\APPLICATION\\UPGRADE\\STATE\\ROLLING\\FORWARD\\PENDING。

## 使用差异包升级

Service Fabric 应用程序可以通过预配一个完整且独立的应用程序包进行升级。此外，还可以通过使用一个仅包含已更新应用程序文件、已更新应用程序清单和服务清单文件的差异包对应用程序进行升级。

完整的应用程序包中包含启动和运行 Service Fabric 应用程序所需的所有文件。差异包仅包含在最后一次设置与当前升级之间更改的文件，以及完整的应用程序清单和服务清单文件。如果应用程序清单或服务清单中存在任何 Service Fabric 无法在生成布局中找到的引用，则 Service Fabric 将在映像存储中搜索。

向群集首次安装应用程序时，需要完整的应用程序包。后续更新可以是完整的应用程序包或差异包。

在以下情况下，使用差异包将是一个不错的选择：

* 当你拥有一个引用了多个服务清单文件和/或多个代码包、配置包或数据包的大型应用程序包时，首选差异包。

* 当你的部署系统直接从应用程序生成过程产生生成布局时，首选差异包。在这种情况下，即使未对代码进行任何更改，新生成的程序集也将具有不同的校验和。使用完整的应用程序包需要你更新所有代码包上的版本。使用差异包时，你只提供更改的文件和其中的版本已更改的清单文件。

如果应用程序是使用 Visual Studio 升级的，将自动发布差异包。如果你想要手动创建差异包（例如，为了使用 PowerShell 升级），则应更新应用程序和服务清单，而只包含最终应用程序包中更改的包。

例如，让我们从以下应用程序开始（为便于理解，这里提供了版本号）：

```text
app1       	1.0.0
  service1 	1.0.0
    code   	1.0.0
    config 	1.0.0
  service2 	1.0.0
    code   	1.0.0
    config 	1.0.0
```

现在，假设你只想要使用 PowerShell 和差异包来更新 service1 的代码包。现在，更新的应用程序将如下所示：

```text
app1       	2.0.0      <-- new version
  service1 	2.0.0      <-- new version
    code   	2.0.0      <-- new version
    config 	1.0.0
  service2 	1.0.0
    code   	1.0.0
    config 	1.0.0
```

在本例中，你已将应用程序清单更新为 2.0.0，并更新了 service1 的服务清单以反映代码包更新。应用程序包的文件夹结构将如下所示：

```text
app1/
  service1/
    code/
```

## 后续步骤

[Uprading your Application Using Visual Studio](/documentation/articles/service-fabric-application-upgrade-tutorial/)（使用 Visual Studio 升级应用程序）将逐步指导你使用 Visual Studio 进行应用程序升级。

[Uprading your Application Using Powershell](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)（使用 PowerShell 升级应用程序）将逐步指导你使用 PowerShell 进行应用程序升级。使用[升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)来控制应用程序的升级方式。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容

参考 [Troubleshooting Application Upgrades](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)（对应用程序升级进行故障排除）中的步骤来解决应用程序升级时的常见问题。
 

<!---HONumber=Mooncake_0627_2016-->