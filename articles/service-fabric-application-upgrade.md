<properties
   pageTitle="Service Fabric 应用程序升级 | Azure"
   description="本文介绍如何升级 Service Fabric 应用程序，包括选择升级模式和执行运行状况检查。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="samgeo"
   editor=""/>

<tags
    ms.service="service-fabric"
    ms.date="05/18/2016"
    wacn.date="07/04/2016"/>


# Service Fabric 应用程序升级

Azure Service Fabric 应用程序是多个服务的集合。在升级期间，Service Fabric 将新的[应用程序清单](/documentation/articles/service-fabric-application-model/#describe-an-application)与以前的版本进行比较，并确定应用程序中的哪些服务需要升级。Fabric Service 的做法是将服务清单中的版本号与前一版中的版本号进行比较。如果服务未更改，则不升级服务。

## 滚动升级概述

在应用程序滚动升级过程中，分阶段进行升级。在每个阶段，对群集中的部分节点进行升级，这一部分节点称为更新域。因此，应用程序在整个升级过程中保持可用。升级期间，群集中可能混合了新旧版本。

因此，两个版本均必须可向前和向后兼容。如果它们不兼容，则由应用程序管理员负责分阶段进行多阶段升级，以保持可用性。管理员在升级到最终版本之前执行此操作，方法是先使用与先前版本兼容的应用程序中间版本进行升级。

在配置群集时在群集清单中指定更新域。不应假设更新域将按特定顺序接收更新。更新域是应用程序部署的逻辑单元。更新域可让服务在升级过程中保持高可用性。

如果对群集中的所有节点应用升级，即应用程序只有一个更新域，将不可能进行任何滚动升级。由于服务将处于关闭状态，并且在升级时不可用，因此我们不建议这样做。此外，当仅为群集设置了一个更新域时，Azure 不提供任何保证。

## 升级过程中的运行状况检查

必须对升级设置运行状况策略（或者可使用默认值）。当所有更新域均在指定的超时内进行了升级，并且所有更新域均被认为运行正常时，即可称升级为成功升级。运行正常的更新域是指该更新域通过了运行状况策略中指定的所有运行状况检查。例如，运行状况策略可能要求应用程序实例中的所有服务都必须运行正常，因为运行状况是由 Service Fabric 定义的。

升级期间 Service Fabric 执行的运行状况策略和检查与服务和应用程序无关。也就是说，不会执行任何特定于服务的测试。例如，你的服务可能具有最小吞吐量要求。但 Service Fabric 没有对其进行测试的信息，因此将不会检查为应用程序定义的吞吐量。有关要执行的检查，请参阅[运行状况文章](/documentation/articles/service-fabric-health-introduction/)。在升级期间执行的检查包括是否已正确复制应用程序包、是否已启动实例，等等。

应用程序运行状况是应用程序子实体的聚合。简单地说，Service Fabric 通过应用程序报告的运行状况来评估应用程序的运行状况。它还以这种方式评估应用程序的所有服务的运行状况。Service Fabric 通过聚合子实体（如服务副本）的运行状况进一步评估应用程序服务的运行状况。一旦符合应用程序运行状况策略，便可继续升级。如果违反运行状况策略，则应用程序升级将会失败。

## 升级模式

建议的升级模式为受监视模式，这是最常用的模式。受监视模式对一个更新域执行升级，如果所有运行状况检查均通过（按指定的策略），则自动移动到下一个更新域。如果运行状况检查失败和/或达到超时，则更新域的升级将会回滚，或者，如果在升级时选择了不受监视的手动选项，则切换到该模式。

不受监视的手动模式在每次对更新域升级之后都需要人工干预，以开始进行下一个更新域的升级。系统不会执行任何 Service Fabric 运行状况检查。管理员开始在下一个更新域中升级之前，需执行状况或状态检查。

## 应用程序升级流程图

下面的流程图可帮助你理解 Service Fabric 应用程序的升级过程。具体而言，该流程描述当一个更新域的升级被认为成功或失败时，超时（包括 *HealthCheckStableDuration*、*HealthCheckRetryTimeout* 和 *UpgradeHealthCheckInterval*）如何为控制提供帮助。

![Service Fabric 应用程序的升级过程][image]


## 后续步骤

[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)将逐步指导你使用 Visual Studio 进行应用程序升级。

[使用 PowerShell 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)将逐步指导你使用 PowerShell 进行应用程序升级。

使用[升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)来控制应用程序的升级方式。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容。

参考[高级主题](/documentation/articles/service-fabric-application-upgrade-advanced/)，了解如何在升级应用程序时使用高级功能。

参考[对应用程序升级进行故障排除](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)中的步骤来解决应用程序升级时的常见问题。
 


[image]: ./media/service-fabric-application-upgrade/service-fabric-application-upgrade-flowchart.png
 

<!---HONumber=Mooncake_0627_2016-->