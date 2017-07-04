<properties
    pageTitle="应用程序升级：升级参数 | Azure"
    description="介绍与升级 Service Fabric 应用程序相关的参数，包括要执行的运行状况检查，以及用于自动撤消升级的策略。"
    services="service-fabric"
    documentationcenter=".net"
    author="mani-ramaswamy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="a4170ac6-192e-44a8-b93d-7e39c92a347e"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/02/2017"
    wacn.date="04/24/2017"
    ms.author="subramar"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="aa26d72524f035cca92cd39062873ceb15bffac1"
    ms.lasthandoff="04/14/2017" />

# <a name="application-upgrade-parameters"></a>应用程序升级参数
本文介绍 Azure Service Fabric 应用程序升级期间应用的各种参数。 参数包括应用程序的名称和版本。 它们可用于控制升级期间应用的超时和运行状况检查，还可指定在升级失败时必须应用的策略。

<br>

| 参数 | 说明 |
| --- | --- |
| ApplicationName |正在升级的应用程序的名称。 示例：fabric:/VisualObjects、fabric:/ClusterMonitor |
| TargetApplicationTypeVersion |作为升级目标的应用程序类型版本。 |
| FailureAction |升级失败时 Service Fabric 要执行的操作。 可将应用程序回滚到更新前的版本（回滚），或者在当前升级域中停止升级。 对于后者，升级模式还会更改为“手动”。 允许的值为“Rollback”和“Manual”。 |
| HealthCheckWaitDurationSec |完成升级域的升级后，在 Service Fabric 评估应用程序的运行状况之前需要等待的时间（以秒为单位）。 也可将此持续时间视为应用程序应先运行多长时间才可被视为正常运行。 如果运行状况检查通过，升级过程将转到下一个升级域。  如果运行状况检查失败，则在再次重试运行状况检查之前，Service Fabric 会等待一定的时间间隔 (UpgradeHealthCheckInterval)，直到达到 HealthCheckRetryTimeout。 建议的默认值为 0 秒。 |
| HealthCheckRetryTimeoutSec |声明升级失败之前，Service Fabric 继续执行运行状况评估的持续时间（以秒为单位）。 默认为 600 秒。 此持续时间在达到 HealthCheckWaitDuration 后开始。 在此 HealthCheckRetryTimeout 期间，Service Fabric 可能会对应用程序运行状况执行多次运行状况检查。 默认值为 10 分钟，应该针对应用程序相应地自定义该值。 |
| HealthCheckStableDurationSec |在转到下一个升级域或完成升级之前，为了验证应用程序是否稳定而要等待的持续时间（以秒为单位）。 此等待持续时间用于防止在执行了运行状况检查后，未检测到运行状况更改。 默认值为 120 秒，应该针对应用程序相应地自定义该值。 |
| UpgradeDomainTimeoutSec |升级单个升级域的最长时间（以秒为单位）。 如果达到了此超时，升级将会停止，然后根据 UpgradeFailureAction 的设置继续下一步。 默认值为 never（无期限），应该针对应用程序相应地自定义该值。 |
| UpgradeTimeout |应用于整个升级的超时（以秒为单位）。 如果达到了此超时，则升级将停止，并将触发 UpgradeFailureAction。 默认值为 never（无期限），应该针对应用程序相应地自定义该值。 |
| UpgradeHealthCheckInterval |检查运行状况的频率。 在*群集* *清单*的 ClusterManager 节中指定了参数，无法使用 upgrade cmdlet 指定此参数。 默认值为 60 秒。 |

<br>

## <a name="service-health-evaluation-during-application-upgrade"></a>应用程序升级期间的服务运行状况评估
<br>
运行状况评估条件为可选条件。 如果在启动升级时未指定运行状况评估条件，则 Service Fabric 将使用应用程序实例的 ApplicationManifest.xml 中指定的应用程序运行状况策略。

<br>

| 参数 | 说明 |
| --- | --- |
| ConsiderWarningAsError |默认值为 False。 在升级期间评估应用程序的运行状况时，将应用程序的警告运行状况事件视为错误。 默认情况下，Service Fabric 不会将警告运行状况事件评估为失败（错误），因此即使存在警告事件，升级也可以继续。 |
| MaxPercentUnhealthyDeployedApplications |建议的默认值为 0。 指定在将应用程序视为不正常和升级失败之前，可以不正常的最大已部署应用程序数（请参阅[运行状况部分](/documentation/articles/service-fabric-health-introduction/)）。 此参数在节点上定义应用程序运行状况，可帮助检查升级过程中的问题。 通常，应用程序的副本将与另一个节点负载均衡，使应用程序看上去运行正常，从而使升级继续。 通过指定严格的 MaxPercentUnhealthyDeployedApplications 运行状况，Service Fabric 可以快速检测应用程序包的问题，并帮助在升级过程中即时报告错误。 |
| MaxPercentUnhealthyServices |建议的默认值为 0。 指定在将应用程序视为不正常和升级失败之前，应用程序实例中可以不正常的最大服务数。 |
| MaxPercentUnhealthyPartitionsPerService |建议的默认值为 0。 指定在将服务视为不正常之前，服务中可以不正常的最大分区数。 |
| MaxPercentUnhealthyReplicasPerPartition |建议的默认值为 0。 指定在将分区视为不正常之前，分区中不正常的最大副本数。 |
| UpgradeReplicaSetCheckTimeout |**无状态服务**- 在单个升级域内，Service Fabric 尝试确保服务的其他实例可用。 如果有多个目标实例，则 Service Fabric 等待多个实例可用，直到达到最大超时值。 此超时是使用 UpgradeReplicaSetCheckTimeout 属性指定的。 如果超时到期，Service Fabric 将继续进行升级，而无论服务实例的数量。 如果只有一个目标实例，则 Service Fabric 不会等待，而是会立即继续进行升级。 **有状态服务** - 在单个升级域内，Service Fabric 尝试确保副本集具有仲裁。 Service Fabric 将等待一个仲裁可用，直到达到最大超时值（由 UpgradeReplicaSetCheckTimeout 属性指定）。 如果超时到期，Service Fabric 将继续进行升级，而无论是否具有仲裁。 前滚时，此设置设置为 never（无限）；回滚时，设置为 900 秒。 |
| ForceRestart |如果更新配置或数据包而不更新服务代码，则仅当 ForceRestart 属性设置为 true 时，服务才会重新启动。 更新完成后，Service Fabric 将通知服务新的配置包或数据包可用。 该服务负责应用所做的更改。 如有必要，该服务可进行重启。 |



MaxPercentUnhealthyServices、MaxPercentUnhealthyPartitionsPerService 和 MaxPercentUnhealthyReplicasPerPartition 条件可按照应用程序实例的服务类型进行指定。为每个服务设置这些参数可让应用程序包含不同评估策略的不同服务类型。例如，无状态网关服务类型可以有一个不同于特定应用程序实例的有状态引擎服务类型的 MaxPercentUnhealthyPartitionsPerService。

## <a name="next-steps"></a>后续步骤
[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)逐步讲解了如何使用 Visual Studio 进行应用程序升级。

[使用 Powershell 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)逐步讲解了如何使用 PowerShell 进行应用程序升级。

[在 Linux 上使用 Azure CLI 升级应用程序](/documentation/articles/service-fabric-azure-cli/#upgrading-your-application)介绍如何使用 Azure CLI 完成应用程序升级。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容。

参考[高级主题](/documentation/articles/service-fabric-application-upgrade-advanced/)，了解如何在升级应用程序时使用高级功能。

参考[对应用程序升级进行故障排除](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)中的步骤来解决应用程序升级时的常见问题。
<!--Update_Description: add reference to azure cli on linux-->