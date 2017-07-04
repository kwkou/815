<properties
    pageTitle="自动管理 Azure Service Fabric 应用程序 | Azure"
    description="使用 PowerShell 部署、升级、测试和删除 Service Fabric 应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="f03f4294-571d-4262-8a77-cc8b481b959d"
    ms.service="service-fabric"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="03/14/2017"
    wacn.date="04/24/2017"
    ms.author="ryanwi"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="34ac213fc0823c493cf96a7af361f74dd78c845f"
    ms.lasthandoff="04/14/2017" />

# <a name="automate-the-application-lifecycle-using-powershell"></a>使用 PowerShell 自动化应用程序生命周期
可以对 [Service Fabric 应用程序生命周期](/documentation/articles/service-fabric-application-lifecycle/)的许多层面进行自动化。  本文说明如何使用 PowerShell 来自动完成部署、升级、删除和测试 Azure Service Fabric 应用程序的常见任务。  还提供了用于应用管理的托管和 HTTP API。 有关详细信息，请参阅[应用生命周期](/documentation/articles/service-fabric-application-lifecycle/)。  

## <a name="prerequisites"></a>先决条件
在进一步讨论文章中的任务之前，请务必：

* 熟悉 [Service Fabric 技术概述](/documentation/articles/service-fabric-technical-overview/)中所述的 Service Fabric 概念。
* [安装运行时、SDK 和工具](/documentation/articles/service-fabric-get-started/)，同时还将安装 **ServiceFabric** PowerShell 模块。
* [启用 PowerShell 脚本执行](/documentation/articles/service-fabric-get-started/#enable-powershell-script-execution)。
* 启动本地群集。  以管理员身份启动新的 PowerShell 窗口，然后从 SDK 文件夹运行群集设置脚本：`& "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1"`
* 在运行本文中的任何 PowerShell 命令之前，请先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/connect-servicefabriccluster) 连接到本地 Service Fabric 群集：`Connect-ServiceFabricCluster localhost:19000`
* 以下任务需要用于部署的 v1 应用程序包和用于升级的 v2 应用程序包。 下载 [**WordCount** 示例应用程序](http://aka.ms/servicefabricsamples)（位于“入门”示例中）。 在 Visual Studio 中生成和打包应用程序（右键单击解决方案资源管理器中的 **WordCount**，然后选择“**打包**”）。 将 `C:\ServiceFabricSamples\Services\WordCount\WordCount\pkg\Debug` 中的 v1 包复制到 `C:\Temp\WordCount`。 将 `C:\Temp\WordCount` 复制到 `C:\Temp\WordCountV2`，创建用于升级的 v2 应用程序包。 在文本编辑器中打开 `C:\Temp\WordCountV2\ApplicationManifest.xml`。 在 **ApplicationManifest** 元素中，将 **ApplicationTypeVersion** 属性从“1.0.0”更改为“2.0.0”，更新应用版本号。 保存更改后的 ApplicationManifest.xml 文件。

## <a name="task-deploy-a-service-fabric-application"></a>任务：部署 Service Fabric 应用程序
在生成并打包应用程序（或下载应用程序包）之后，可以将应用程序部署到本地 Service Fabric 群集中。 部署过程包括上传应用程序包、注册应用程序类型，以及创建应用程序实例。 使用本部分中的说明将新应用程序部署到群集。

### <a name="step-1-upload-the-application-package"></a>步骤 1：上载应用程序包
将应用程序包上传到映像存储区会将其放在可由内部 Service Fabric 组件访问的位置。  应用程序包包含所需的应用程序清单、服务清单，以及用于创建应用程序和服务实例的代码、配置和数据包。 如果要在本地验证应用包，请使用 [Test-ServiceFabricApplicationPackage](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/test-servicefabricapplicationpackage) cmdlet。  [Copy-ServiceFabricApplicationPackage](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/copy-servicefabricapplicationpackage) 命令用来上载包。 例如：

    Copy-ServiceFabricApplicationPackage C:\Temp\WordCount\ -ImageStoreConnectionString file:C:\SfDevCluster\Data\ImageStoreShare -ApplicationPackagePathInImageStore WordCount

### <a name="step-2-register-the-application-type"></a>步骤 2：注册应用程序类型
注册应用程序包，使应用程序清单中声明的应用程序类型和版本可供使用。系统将读取步骤 1 中上传的包，对该包进行验证（等效于在本地运行 [Test-ServiceFabricApplicationPackage](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/test-servicefabricapplicationpackage)），处理包的内容，并将已处理的包复制到内部系统位置。运行 [Register-ServiceFabricApplicationType](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/register-servicefabricapplicationtype) cmdlet：

    Register-ServiceFabricApplicationType WordCount

若要查看在群集中注册的所有应用程序类型，请运行 [Get-ServiceFabricApplicationType](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/get-servicefabricapplicationtype) cmdlet：

    Get-ServiceFabricApplicationType

### <a name="step-3-create-the-application-instance"></a>步骤 3：创建应用程序实例
可以使用已使用 [New-ServiceFabricApplication](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/new-servicefabricapplication) 命令成功注册的任何应用程序类型版本实例化应用程序。 每个应用程序的名称在部署时声明且必须以 **fabric:** 方案开头，并且对每个应用程序实例是唯一的。 应用程序类型名称和应用程序类型版本在应用程序包的 **ApplicationManifest.xml** 文件中声明。 如果目标应用程序类型的应用程序清单中定义有任何默认服务，则此时将还创建那些服务。

    New-ServiceFabricApplication fabric:/WordCount WordCount 1.0.0

[Get-ServiceFabricApplication](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/get-servicefabricapplication) 命令将列出已成功创建的所有应用程序实例以及其整体状态。 [Get-ServiceFabricService](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/get-servicefabricservice) 命令将列出在给定的应用程序实例中成功创建的所有服务实例。 列出默认服务（如果有）。

    Get-ServiceFabricApplication

    Get-ServiceFabricApplication | Get-ServiceFabricService

## <a name="task-upgrade-a-service-fabric-application"></a>任务：升级 Service Fabric 应用程序
可以使用已更新的应用程序包升级以前部署的 Service Fabric 应用程序。 此任务将升级以前在“任务：部署 Service Fabric 应用程序”中部署的 WordCount 应用程序。 有关详细信息，请参阅 [Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)。

为简单起见，此示例仅在系统必备组件中创建的 WordCountV2 应用程序包中更新了应用程序版本号。 更加实际的方案则包括更新服务代码、配置或数据文件，然后使用已更新的版本号重新生成并打包应用程序。  

### <a name="step-1-upload-the-updated-application-package"></a>步骤 1：上传已更新的应用程序包
WordCount v1 应用程序已准备好升级。 如果以管理员身份打开 PowerShell 窗口并键入 [**Get-ServiceFabricApplication**](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/get-servicefabricapplication)，将看到 1.0.0 版本的 WordCount 应用程序类型已部署。  

现在，将更新的应用程序包复制到 Service Fabric 映像存储（Service Fabric 存储应用程序包的位置）。 参数 **ApplicationPackagePathInImageStore** 告知 Service Fabric 可在何处找到应用程序包。 以下命令将应用程序包复制到映像存储中的 **WordCountV2** ：  

    Copy-ServiceFabricApplicationPackage C:\Temp\WordCountV2\ -ImageStoreConnectionString file:C:\SfDevCluster\Data\ImageStoreShare -ApplicationPackagePathInImageStore WordCountV2


### <a name="step-2-register-the-updated-application-type"></a>步骤 2：注册已更新的应用程序类型
下一步是将新版本的应用程序注册到 Service Fabric，这可以使用 [Register-ServiceFabricApplicationType](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/register-servicefabricapplicationtype) cmdlet 来执行：

    Register-ServiceFabricApplicationType WordCountV2

### <a name="step-3-start-the-upgrade"></a>步骤 3：开始升级
可将各种升级参数、超时和运行状况标准应用到应用程序升级。 若要了解详细信息，请参阅[应用程序升级参数](/documentation/articles/service-fabric-application-upgrade-parameters/)和[升级过程](/documentation/articles/service-fabric-application-upgrade/)文档。 所有服务和实例在升级之后都应该*运行正常*。  将 **HealthCheckStableDuration** 设置为 60 秒（这样该服务在进行下一个升级域的升级之前将至少保持 20 秒的运行状况正常）。  同时，请将 **UpgradeDomainTimeout** 设置为 1200 秒，将 **UpgradeTimeout** 设置为 3000 秒。 最后，将 **UpgradeFailureAction** 设置为 **rollback**，以便在升级期间遇到任何错误时，请求 Service Fabric 将应用程序回滚到前一版本。

现在你可以使用 [Start-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/start-servicefabricapplicationupgrade) cmdlet 开始升级应用程序：

    Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/WordCount -ApplicationTypeVersion 2.0.0 -HealthCheckStableDurationSec 60 -UpgradeDomainTimeoutSec 1200 -UpgradeTimeout 3000  -FailureAction Rollback -Monitored

请注意，应用程序名称与以前部署的 v1.0.0 应用程序名称相同 (fabric:/WordCount)。 Service Fabric 使用此名称来确定升级的应用程序。 如果设置的超时太短，则可能遇到一条说明该问题的超时失败消息。 请参阅[对应用程序升级进行故障排除](/documentation/articles/service-fabric-application-upgrade-troubleshooting/)，或增大超时。

### <a name="step-4-check-upgrade-progress"></a>步骤 4 ：查看升级进度
可以使用 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 或 [Get-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/get-servicefabricapplicationupgrade) cmdlet 来监视应用程序升级的进度：

    Get-ServiceFabricApplicationUpgrade fabric:/WordCount

几分钟后，[Get-ServiceFabricApplicationUpgrade](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/get-servicefabricapplicationupgrade) cmdlet 会显示已升级所有升级域（已完成）。

## <a name="task-test-a-service-fabric-application"></a>任务：测试 Service Fabric 应用程序
若要编写高质量的服务，开发人员需要能够引入不可靠的基础结构故障来测试其服务的稳定性。 Service Fabric 使开发人员能够引入故障操作，并使用混沌和故障转移测试方案来测试故障情况下服务的运行情况。  有关其他信息，请通读[故障分析服务简介](/documentation/articles/service-fabric-testability-overview/)。

### <a name="step-1-run-the-chaos-test-scenario"></a>步骤 1：运行混沌测试方案
混沌测试方案跨整个 Service Fabric 群集生成故障。 一般而言，该方案将几个月或几年经历的故障压缩到几小时。 各种交叉故障的组合并具有高故障率，能够找出很有可能被忽视的极端状况。 以下示例运行了 60 分钟的混沌测试方案。

    $timeToRun = 60
    $maxStabilizationTimeSecs = 180
    $concurrentFaults = 3
    $waitTimeBetweenIterationsSec = 60

    Invoke-ServiceFabricChaosTestScenario -TimeToRunMinute $timeToRun -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -MaxConcurrentFaults $concurrentFaults -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec

### <a name="step-2-run-the-failover-test-scenario"></a>步骤 2：运行故障转移测试方案
故障转移测试方案针对特定的服务分区，而不是整个群集，其他服务不受影响。 该方案将遍历一系列的模块故障和服务验证，同时你的业务逻辑将保持运行。 服务验证失败指出存在需要进一步调查的问题。 故障转移测试一次只引入一个故障，而不像混沌测试方案那样引入多个故障。 以下示例针对 fabric:/WordCount/WordCountService 服务运行故障转移测试 60 分钟。

    $timeToRun = 60
    $maxStabilizationTimeSecs = 180
    $waitTimeBetweenFaultsSec = 10
    $serviceName = "fabric:/WordCount/WordCountService"

    Invoke-ServiceFabricFailoverTestScenario -TimeToRunMinute $timeToRun -MaxServiceStabilizationTimeoutSec $maxStabilizationTimeSecs -WaitTimeBetweenFaultsSec $waitTimeBetweenFaultsSec -ServiceName $serviceName -PartitionKindUniformInt64 -PartitionKey 1

## <a name="task-remove-a-service-fabric-application"></a>任务：删除 Service Fabric 应用程序
你可以删除已部署的应用程序的实例、从群集中删除预配的应用程序类型，以及从 ImageStore 中删除应用程序包。

### <a name="step-1-remove-an-application-instance"></a>步骤 1：删除应用程序实例
当不再需要某应用程序实例时，可以使用 [Remove-ServiceFabricApplication](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/remove-servicefabricapplication) cmdlet 将它永久删除。 这也会将自动删除属于该应用程序的所有服务，永久删除所有服务状态。 此操作无法撤消，并且无法恢复应用程序状态。

    Remove-ServiceFabricApplication fabric:/WordCount

### <a name="step-2-unregister-the-application-type"></a>步骤 2：注销应用程序类型
当不再需要应用程序类型的某个特定版本时，可以使用 [Unregister-ServiceFabricApplicationType](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/unregister-servicefabricapplicationtype) cmdlet 将它注销。 注销未使用的类型将在映像存储中释放该应用程序包使用的存储空间。 只要没有针对其实例化的应用程序或引用它的挂起应用程序升级，就可以注销应用程序类型。

    Unregister-ServiceFabricApplicationType WordCount 1.0.0

### <a name="step-3-remove-the-application-package"></a>步骤 3：删除应用程序包
注销应用程序类型后，可以使用 [Remove-ServiceFabricApplicationPackage](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/remove-servicefabricapplicationpackage) cmdlet 从映像存储中删除应用程序包。

    Remove-ServiceFabricApplicationPackage -ImageStoreConnectionString file:C:\SfDevCluster\Data\ImageStoreShare -ApplicationPackagePathInImageStore WordCount

## <a name="next-steps"></a>后续步骤
[Service Fabric 应用程序生命周期](/documentation/articles/service-fabric-application-lifecycle/)

[部署应用程序](/documentation/articles/service-fabric-deploy-remove-applications/)

[应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

[Azure Service Fabric cmdlet](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/servicefabric)
<!--Update_Description:wording update; add anchors to sub titles-->