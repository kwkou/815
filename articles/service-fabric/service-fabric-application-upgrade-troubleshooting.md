<properties
    pageTitle="应用程序升级故障排除 | Azure"
    description="本文介绍一些围绕升级 Service Fabric 应用程序的常见问题以及这些问题的解决方法。"
    services="service-fabric"
    documentationcenter=".net"
    author="mani-ramaswamy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="19ad152e-ec50-4327-9f19-065c875c003c"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/02/2017"
    wacn.date="04/24/2017"
    ms.author="subramar"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d2be51d90cea93b02e9429da8652c4621aec46df"
    ms.lasthandoff="04/14/2017" />

# <a name="troubleshoot-application-upgrades"></a>应用程序升级故障排除
本文介绍一些围绕升级 Azure Service Fabric 应用程序的常见问题以及这些问题的解决方法。

## <a name="troubleshoot-a-failed-application-upgrade"></a>失败的应用程序升级故障排除
当升级失败时， **Get-ServiceFabricApplicationUpgrade** 命令的输出将包含用于调试失败的附加信息。  以下列表指定如何使用这些附加信息：

1. 识别失败类型。
2. 识别失败原因。
3. 隔离一个或多个失败组件以进行进一步调查。

当 Service Fabric 检测到失败时，就会提供这些信息，而无论 **FailureAction** 是回滚升级还是挂起升级。

### <a name="identify-the-failure-type"></a>确定失败类型
在 **Get-ServiceFabricApplicationUpgrade** 的输出中，**FailureTimestampUtc** 标识 Service Fabric 检测到升级失败以及触发 **FailureAction** 时的时间戳 (UTC)。 **FailureReason** 识别失败的三个可能的高级别原因之一：

1. UpgradeDomainTimeout - 指示特定的升级域花费了太长时间才完成，并且 **UpgradeDomainTimeout** 过期。
2. OverallUpgradeTimeout - 指示总体升级花费了太长时间才完成，并且 **UpgradeTimeout** 过期。
3. HealthCheck - 指示在升级一个更新域后，根据指定的运行状况策略，应用程序的运行状况仍不正常，并且 **HealthCheckRetryTimeout** 过期。

仅当升级失败并开始回滚时，才会在输出中显示这些项。 将根据失败类型显示进一步的信息。

### <a name="investigate-upgrade-timeouts"></a>调查升级超时
升级超时失败通常由服务可用性问题引起。 当服务副本或实例未能在新代码版本中启动时，此段落后面的输出是升级的典型输出。 **UpgradeDomainProgressAtFailure** 字段捕获失败时所有挂起的升级工作的快照。

    PS D:\temp> Get-ServiceFabricApplicationUpgrade fabric:/DemoApp

    ApplicationName                : fabric:/DemoApp
    ApplicationTypeName            : DemoAppType
    TargetApplicationTypeVersion   : v2
    ApplicationParameters          : {}
    StartTimestampUtc              : 4/14/2015 9:26:38 PM
    FailureTimestampUtc            : 4/14/2015 9:27:05 PM
    FailureReason                  : UpgradeDomainTimeout
    UpgradeDomainProgressAtFailure : MYUD1

                                     NodeName            : Node4
                                     UpgradePhase        : PostUpgradeSafetyCheck
                                     PendingSafetyChecks :
                                         WaitForPrimaryPlacement - PartitionId: 744c8d9f-1d26-417e-a60e-cd48f5c098f0

                                     NodeName            : Node1
                                     UpgradePhase        : PostUpgradeSafetyCheck
                                     PendingSafetyChecks :
                                         WaitForPrimaryPlacement - PartitionId: 4b43f4d8-b26b-424e-9307-7a7a62e79750
    UpgradeState                   : RollingBackCompleted
    UpgradeDuration                : 00:00:46
    CurrentUpgradeDomainDuration   : 00:00:00
    NextUpgradeDomain              :
    UpgradeDomainsStatus           : { "MYUD1" = "Completed";
                                     "MYUD2" = "Completed";
                                     "MYUD3" = "Completed" }
    UpgradeKind                    : Rolling
    RollingUpgradeMode             : UnmonitoredAuto
    ForceRestart                   : False
    UpgradeReplicaSetCheckTimeout  : 00:00:00

在本示例中，升级域 *MYUD1* 的升级失败，两个分区（*744c8d9f-1d26-417e-a60e-cd48f5c098f0* 和 *4b43f4d8-b26b-424e-9307-7a7a62e79750*）已停滞。 分区由于运行时无法将主副本 (*WaitForPrimaryPlacement*) 放在在目标节点 *Node1* 和 *Node4* 上而停滞。

可使用 **Get-ServiceFabricNode** 命令验证这两个节点是否位于升级域 *MYUD1* 中。 *UpgradePhase* 为 *PostUpgradeSafetyCheck*，这意味着这些安全检查在升级域中所有节点完成升级后发生。 所有这些信息表明应用程序代码的新版本可能存在问题。 最常见的问题是打开或升级到主代码路径时的服务错误。

*UpgradePhase* 为 *PreUpgradeSafetyCheck* 意味着在执行升级前，准备升级域时出现了问题。 这种情况下最常见的问题是关闭主代码路径或从该路径降级时的服务错误。

当前 **UpgradeState** 为 *RollingBackCompleted*，因此必须已使用回滚 **FailureAction**（将在失败时自动回滚升级）执行原始升级。 如果已使用手动 **FailureAction** 执行了原始升级，则升级将改为处于挂起状态，以允许对应用程序进行实时调试。

### <a name="investigate-health-check-failures"></a>调查运行状况检查失败
运行状况检查失败可能由各种其他问题触发，这些问题可能发生在升级域中所有节点完成升级、通过所有安全检查之后。 此段落后面的输出是升级因运行状况检查失败而失败时的典型输出。 **UnhealthyEvaluations** 字段根据指定的[运行状况策略](/documentation/articles/service-fabric-health-introduction/)，捕获升级失败时失败的运行状况检查的快照。

    PS D:\temp> Get-ServiceFabricApplicationUpgrade fabric:/DemoApp

    ApplicationName                         : fabric:/DemoApp
    ApplicationTypeName                     : DemoAppType
    TargetApplicationTypeVersion            : v4
    ApplicationParameters                   : {}
    StartTimestampUtc                       : 4/24/2015 2:42:31 AM
    UpgradeState                            : RollingForwardPending
    UpgradeDuration                         : 00:00:27
    CurrentUpgradeDomainDuration            : 00:00:27
    NextUpgradeDomain                       : MYUD2
    UpgradeDomainsStatus                    : { "MYUD1" = "Completed";
                                              "MYUD2" = "Pending";
                                              "MYUD3" = "Pending" }
    UnhealthyEvaluations                    :
                                              Unhealthy services: 50% (2/4), ServiceType='PersistedServiceType', MaxPercentUnhealthyServices=0%.

                                              Unhealthy service: ServiceName='fabric:/DemoApp/Svc3', AggregatedHealthState='Error'.

                                                  Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                                  Unhealthy partition: PartitionId='3a9911f6-a2e5-452d-89a8-09271e7e49a8', AggregatedHealthState='Error'.

                                                      Error event: SourceId='Replica', Property='InjectedFault'.

                                              Unhealthy service: ServiceName='fabric:/DemoApp/Svc2', AggregatedHealthState='Error'.

                                                  Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                                  Unhealthy partition: PartitionId='744c8d9f-1d26-417e-a60e-cd48f5c098f0', AggregatedHealthState='Error'.

                                                      Error event: SourceId='Replica', Property='InjectedFault'.

    UpgradeKind                             : Rolling
    RollingUpgradeMode                      : Monitored
    FailureAction                           : Manual
    ForceRestart                            : False
    UpgradeReplicaSetCheckTimeout           : 49710.06:28:15
    HealthCheckWaitDuration                 : 00:00:00
    HealthCheckStableDuration               : 00:00:10
    HealthCheckRetryTimeout                 : 00:00:10
    UpgradeDomainTimeout                    : 10675199.02:48:05.4775807
    UpgradeTimeout                          : 10675199.02:48:05.4775807
    ConsiderWarningAsError                  :
    MaxPercentUnhealthyPartitionsPerService :
    MaxPercentUnhealthyReplicasPerPartition :
    MaxPercentUnhealthyServices             :
    MaxPercentUnhealthyDeployedApplications :
    ServiceTypeHealthPolicyMap              :

调查运行状况检查失败原因首先需要了解 Service Fabric 运行状况模型。 但即使没有深入理解，我们也可以看到有两个服务是不正常的：*fabric:/DemoApp/Svc3* 和 *fabric:/DemoApp/Svc2*，还可看到错误运行状况报告（本例中为“InjectedFault”）。 在本示例中，四个服务中有两个服务不正常，低于不正常运行状况的默认目标 (*MaxPercentUnhealthyServices*) 0%。

升级因为启动升级时手动指定 **FailureAction** 失败而暂停。 此模式允许我们在采取其他任何措施之前，在失败状态下调查实时系统。

### <a name="recover-from-a-suspended-upgrade"></a>从挂起的升级恢复
使用回滚 **FailureAction**时，无需任何恢复，因为在升级失败时会自动回滚。 使用手动 **FailureAction** 时，有以下几个恢复选项：

1. 触发回滚
2. 手动继续进行升级的其余部分
3. 继续进行受监控的升级

可随时使用 **Start-ServiceFabricApplicationRollback** 命令启动应用程序回滚。 一旦命令成功返回，回滚请求即已在系统中注册，并将立即启动。

**Resume-ServiceFabricApplicationUpgrade** 命令可用于手动继续进行升级的其余部分，一次执行一个升级域。 在此模式下，系统只执行安全检查， 而不会再执行其他运行状况检查。 仅当 *UpgradeState* 显示 *RollingForwardPending* 时才可使用此命令，它表示当前升级域已完成升级但下一个升级域尚未启动（挂起）。

**Update-ServiceFabricApplicationUpgrade** 命令可用于继续进行受监控的升级，同时执行安全检查和运行状况检查。

    PS D:\temp> Update-ServiceFabricApplicationUpgrade fabric:/DemoApp -UpgradeMode Monitored

    UpgradeMode                             : Monitored
    ForceRestart                            :
    UpgradeReplicaSetCheckTimeout           :
    FailureAction                           :
    HealthCheckWaitDuration                 :
    HealthCheckStableDuration               :
    HealthCheckRetryTimeout                 :
    UpgradeTimeout                          :
    UpgradeDomainTimeout                    :
    ConsiderWarningAsError                  :
    MaxPercentUnhealthyPartitionsPerService :
    MaxPercentUnhealthyReplicasPerPartition :
    MaxPercentUnhealthyServices             :
    MaxPercentUnhealthyDeployedApplications :
    ServiceTypeHealthPolicyMap              :

    PS D:\temp>

升级将从上次挂起的升级域继续，并使用与以前相同的升级参数和运行状况策略。 如果需要，在继续进行升级时，可使用同一命令更改上面的输出中显示的任何升级参数和运行状况策略。 在本示例中，升级以监视模式继续，参数和运行状况策略保持不变。

## <a name="further-troubleshooting"></a>进一步的故障排除
### <a name="service-fabric-is-not-following-the-specified-health-policies"></a>Service Fabric 没有遵循指定的运行状况策略
可能的原因 1：

Service Fabric 将所有百分比转换为实际实体（如副本、分区和服务）数，以进行运行状况评估，并且此数目将始终调高到实体整数。 例如，如果最大值 *MaxPercentUnhealthyReplicasPerPartition* 是 21% 且有 5 个副本，则 Service Fabric 允许最多 2 个运行状况不正常的副本（即 `Math.Ceiling (5\*0.21)）。 因此，设置运行状况策略时应考虑到这一点。

可能的原因 2：

运行状况策略以总服务数的百分比指定，而非具体服务实例数的百分比。 例如，如果在升级前应用程序有四个服务实例 A、B、C 和 D，其中服务 D 不正常，但这对应用程序没有明显影响。 我们想要在升级过程中忽略已知的不正常服务 D，并将参数 *MaxPercentUnhealthyServices* 设置为 25%，假设只需 A、B 和 C 处于正常状态。

但在升级期间，D 可能变为正常，而 C 变为不正常。 升级仍会成功，因为只有 25% 的服务运行状况不正常。 但是，这可能导致非预期错误，因为 C 意外地变为不正常，而 D 变为正常。在此情况下，应将 D 建模为不同于 A、B 和 C 的服务类型。由于可根据服务类型指定运行状况策略，因此可将不同的运行状况百分比阈值应用到不同的服务。 

### <a name="i-did-not-specify-a-health-policy-for-application-upgrade-but-the-upgrade-still-fails-for-some-time-outs-that-i-never-specified"></a>我没有为应用程序升级指定运行状况策略，但升级仍因我从未指定的一些超时而失败
当未向升级请求提供运行状况策略时，将使用当前应用程序版本的 *ApplicationManifest.xml* 中的策略。 例如，如果要将应用程序 X 从版本 1.0 升级到版本 2.0，将使用版本 1.0 中指定的应用程序运行状况策略。 如果应对升级使用不同的运行状况策略，则需将该策略指定为应用程序升级 API 调用的一部分。 指定为 API 调用一部分的策略只会在升级期间应用。 升级完成后，将使用 *ApplicationManifest.xml* 中指定的策略。

### <a name="incorrect-time-outs-are-specified"></a>指定了错误的超时值
用户可能要知道当超时设置不一致时会发生什么情况。 例如，*UpgradeTimeout* 小于 *UpgradeDomainTimeout*。 答案是将返回错误。 返回错误的情况包括：*UpgradeDomainTimeout* 小于 *HealthCheckWaitDuration* 和 *HealthCheckRetryTimeout* 的总和，或者 *UpgradeDomainTimeout* 小于 *HealthCheckWaitDuration* 和 *HealthCheckStableDuration* 的总和。

### <a name="my-upgrades-are-taking-too-long"></a>我升级花费的时间过长
完成升级所需的时间取决于运行状况检查和指定的超时。 运行状况检查和超时取决于花费多少时间来复制、部署和稳定应用程序。 超时过短可能意味着会出现更多的失败升级，因此建议在开始时保守地使用较长超时。

让我们快速回顾一下超时如何与升级时间相互作用：

完成升级域升级的时间不会早于 *HealthCheckWaitDuration* + *HealthCheckStableDuration*。

发生升级失败的时间不会早于 *HealthCheckWaitDuration* + *HealthCheckRetryTimeout*。

升级域的升级时间受到 *UpgradeDomainTimeout* 的限制。  如果 *HealthCheckRetryTimeout* 和 *HealthCheckStableDuration* 均不为零，并且应用程序的运行状况保持来回切换，那么升级最终将于 *UpgradeDomainTimeout* 超时。 在当前升级域的升级开始时，*UpgradeDomainTimeout* 就开始倒计时。

## <a name="next-steps"></a>后续步骤
[使用 Visual Studio 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial/)逐步讲解了如何使用 Visual Studio 进行应用程序升级。

[使用 Powershell 升级应用程序](/documentation/articles/service-fabric-application-upgrade-tutorial-powershell/)逐步讲解了如何使用 PowerShell 进行应用程序升级。

使用[升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)来控制应用程序的升级方式。

了解如何使用[数据序列化](/documentation/articles/service-fabric-application-upgrade-data-serialization/)，使应用程序在升级后保持兼容。

参考[高级主题](/documentation/articles/service-fabric-application-upgrade-advanced/)，了解如何在升级应用程序时使用高级功能。

参考[对应用程序升级进行故障排除](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)中的步骤来解决应用程序升级时的常见问题。
<!--Update_Description:add anchors to sub titles-->