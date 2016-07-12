<properties
   pageTitle="在本地群集上部署和升级应用入门 | Azure"
   description="设置本地 Service Fabric 群集，在其中部署现有的应用程序，然后升级该应用程序。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/09/2016"
   wacn.date="07/04/2016"/>

# 在本地群集上部署和升级应用程序入门
Azure Service Fabric SDK 包含完整的本地开发环境，可让你快速地在本地群集上开始部署和管理应用程序。在本文中，你将从 Windows PowerShell 创建本地群集、将现有应用程序部署到该群集，然后将该应用程序升级为新版本。

> [AZURE.NOTE] 本文假设你已[设置开发环境](/documentation/articles/service-fabric-get-started/)。

## 创建本地群集
Service Fabric 群集代表一组可在其中部署应用程序的硬件资源。通常，群集由任意数量的计算机（从 5 台到数千台）组成。不过，Service Fabric SDK 包含可在一台计算机上运行的群集配置。

请务必知道 Service Fabric 本地群集不是模拟器或仿真器。它运行多台计算机群集上使用的相同平台代码。唯一的差别在于它在一台计算机上运行通常会分散于五台计算机的平台进程。

SDK 提供两种方式来设置本地群集：Windows PowerShell 脚本和本地群集管理器系统托盘应用。在本教程中，我们将使用 PowerShell 脚本。

> [AZURE.NOTE] 如果你已通过从 Visual Studio 部署应用程序创建了本地群集，则可以跳过本部分。


1. 以管理员身份启动新的 PowerShell 窗口。

2. 从 SDK 文件夹运行群集设置脚本：

	```powershell
	& "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1"
	```

    群集设置需要一段时间。完成设置后，你应会看到类似于下面的输出：

    ![群集设置输出][cluster-setup-success]

    现已准备好将应用程序部署到群集。

## 部署应用程序
Service Fabric SDK 包含一组丰富的框架以及用于创建应用程序的开发人员工具。如果你有兴趣学习如何在 Visual Studio 中创建应用程序，请参阅[在 Visual Studio 中创建你的第一个 Service Fabric 应用程序](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)。

在本教程中，我们将使用现有的示例应用程序（称为 WordCount），以便我们可以专注于平台的管理层面，包括部署、监视和升级。


1. 以管理员身份启动新的 PowerShell 窗口。

2. 导入 Service Fabric SDK PowerShell 模块。

    ```powershell
    Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
    ```

3. 创建一个目录用于存储将要下载和部署的应用程序，例如 C:\\ServiceFabric。

    ```powershell
    mkdir c:\ServiceFabric\
    cd c:\ServiceFabric\
    ```

4. [下载 WordCount 应用程序](http://aka.ms/servicefabric-wordcountapp)到创建的位置。注意：Microsoft Edge 浏览器将以 *.zip* 扩展名保存文件。你需要将文件扩展名更改为 *.sfpkg*。

5. 连接到本地群集：

    ```powershell
    Connect-ServiceFabricCluster localhost:19000
    ```

6. 调用 SDK 的部署命令来创建新的应用程序，并提供应用程序包的名称和路径。

    ```powershell  
  Publish-NewServiceFabricApplication -ApplicationPackagePath c:\ServiceFabric\WordCountV1.sfpkg -ApplicationName "fabric:/WordCount"
    ```

    如果一切正常，你应会看到如下所示的输出：

    ![将应用程序部署到本地群集][deploy-app-to-local-cluster]

7. 若要查看应用程序的操作情况，请启动浏览器并导航到 [http://localhost:8081/wordcount/index.html](http://localhost:8081/wordcount/index.html)。您应看到与下面类似的内容：

    ![已部署的应用程序 UI][deployed-app-ui]

    WordCount 应用程序非常简单。它包含的客户端 JavaScript 代码可生成五个随机字符的“单词”，然后将这些单词通过 ASP.NET Web API 中继到应用程序。有状态服务将跟踪计算的字数。这些单词根据其第一个字符进行分区。

    我们部署的应用程序包含四个分区。因此，以 A 到 G 开头的单词将存储在第一个分区，以 N 到 H 开头的单词将存储在第二个分区，依此类推。

## 查看应用程序详细信息和状态
我们现已部署应用程序，让我们在 PowerShell 中查看一些应用程序详细信息。

1. 查询群集上所有已部署的应用程序：

    ```powershell
    Get-ServiceFabricApplication
    ```

    假设你只部署了 WordCount 应用程序，你将看到类似于下面的内容：

    ![在 PowerShell 中查询所有已部署的应用程序][ps-getsfapp]

2. 通过查询 WordCount 应用程序中包含的服务集转到下一个级别。

    ```powershell
    Get-ServiceFabricService -ApplicationName 'fabric:/WordCount'
    ```

    ![在 PowerShell 中列出应用程序的服务][ps-getsfsvc]

    请注意，该应用程序由两个服务组成：Web 前端服务和可管理单词的有状态服务。

3. 最后，看看 WordCountService 的分区列表：

    ```powershell
    Get-ServiceFabricPartition 'fabric:/WordCount/WordCountService'
    ```

    ![在 PowerShell 中查看服务分区][ps-getsfpartitions]

    刚使用的命令集（例如所有的 Service Fabric PowerShell 命令）可用于任何你可以连接的群集（本地或远程）。

    若要以更直观的方式来与群集交互，可以在浏览器中导航到 [http://localhost:19080/Explorer](http://localhost:19080/Explorer)，以使用基于 Web 的 Service Fabric Explorer 工具。

    ![在 Service Fabric 资源管理器中查看应用程序详细信息][sfx-service-overview]

    > [AZURE.NOTE] 要了解有关 Service Fabric Explorer 的详细信息，请参阅[使用 Service Fabric Explorer 可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)。

## 升级应用程序
Service Fabric 在应用程序推出于群集时监视其运行状况，从而提供无需停机的升级。让我们对 WordCount 应用程序执行简单的升级。

新版应用程序现在只计算以元音开头的单词。推出升级后，我们将看到应用程序的行为出现两项变化。首先，计数增长的速率应该变慢，因为计算的单词比较少。其次，由于第一个分区有两个元音（A 和 E），而其他每个分区只包含一个元音，因此第一个分区的计数最终会超出其他分区。

1. [下载 WordCount v2 包](http://aka.ms/servicefabric-wordcountappv2)到 v1 包所下载到的同一个位置。

2. 返回到 PowerShell 窗口并使用 SDK 的升级命令在群集中注册新版本。然后开始升级 fabric:/WordCount 应用程序。

    ```powershell
    Publish-UpgradedServiceFabricApplication -ApplicationPackagePath C:\ServiceFabric\WordCountV2.sfpkg -ApplicationName "fabric:/WordCount" -UpgradeParameters @{"FailureAction"="Rollback"; "UpgradeReplicaSetCheckTimeout"=1; "Monitored"=$true; "Force"=$true}
    ```

    开始升级时，你应会在 PowerShell 中看到如下所示的输出。

    ![在 PowerShell 中查看升级进度][ps-appupgradeprogress]

3. 当升级正在进行时，你可能发现从 Service Fabric 资源管理器监视其状态会更加轻松。启动浏览器窗口并导航到 [http://localhost:19080/Explorer](http://localhost:19080/Explorer)。展开左侧树中的“应用程序”，然后选择“WordCount”，最后选择“fabric:/WordCount”。在“基本信息”选项卡中，随着群集升级域的不断升级，你可以看到升级状态。

    ![在 Service Fabric 资源管理器中查看升级进度][sfx-upgradeprogress]

    随着每个域不断升级，系统将执行运行状况检查，以确保应用程序行为正常。

4. 如果对 fabric:/WordCount 应用程序包含的服务集重新运行以前的查询，你会注意到，虽然 WordCountService 的版本已更改，但 WordCountWebService 的版本维持不变：

    ```powershell
    Get-ServiceFabricService -ApplicationName 'fabric:/WordCount'
    ```

    ![升级后查询应用程序服务][ps-getsfsvc-postupgrade]

    这突出显示了 Service Fabric 如何管理应用程序升级。它只处理了已更改的服务集（或这些服务中的代码/配置包），使升级过程变得更快速且更可靠。

5. 最后，请返回到浏览器来观察新应用程序版本的行为。与预期一样，计数进度比较缓慢，第一个分区中的卷数最终稍多一些。

    ![在浏览器中查看应用程序的新版本][deployed-app-ui-v2]

## 清理

结束之前，请务必记住该本地群集非常真实。应用程序将继续在后台运行，直到你删除它们。根据应用的性质，正在运行的应用可能会占用计算机上的大量资源。可通过几个选项对此进行管理：

1. 若要删除单个应用程序及其所有数据，请运行以下命令：

    ```powershell
    Unpublish-ServiceFabricApplication -ApplicationName "fabric:/WordCount"
    ```

    或者在 Service Fabric Explorer 的“操作”菜单或者左窗格中应用程序列表视图的上下文菜单内，使用“删除应用程序”操作。

    ![在 Service Fabric Explorer 中删除应用程序][sfe-delete-application]

2. 从群集中删除应用程序后，可以取消注册 WordCount 应用程序类型的版本 1.0.0 和 2.0.0。这将从群集的映像存储中删除该应用程序包，包括其代码和配置。

    ```powershell
    Remove-ServiceFabricApplicationType -ApplicationTypeName WordCount -ApplicationTypeVersion 2.0.0
    Remove-ServiceFabricApplicationType -ApplicationTypeName WordCount -ApplicationTypeVersion 1.0.0
    ```

    或者，在 Service Fabric Explorer 中选择该应用程序对应的“取消预配类型”。

3. 若要关闭群集，但保留应用程序数据和跟踪，请在系统托盘应用中单击“停止本地群集”。

4. 要完全删除群集，请在系统托盘应用中单击“删除本地群集”。请注意，此选项会导致下次在 Visual Studio 中按 F5 时部署较慢。仅当在一段时间内不想使用本地群集时，或者当需要回收资源时，才使用此选项。

## 后续步骤
- 现在，你已部署并升级某些预先生成的应用程序，接下来可以[尝试在 Visual Studio 中生成你自己的应用程序](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)。
- 也可以对 Azure 群集执行本文中所述的对本地群集执行的所有操作。
- 本文中执行的升级非常简单。若要深入了解 Service Fabric 升级的功能和灵活性，请参阅[升级文档](/documentation/articles/service-fabric-application-upgrade/)。

<!-- Images -->

[cluster-setup-success]: ./media/service-fabric-get-started-with-a-local-cluster/LocalClusterSetup.png
[extracted-app-package]: ./media/service-fabric-get-started-with-a-local-cluster/ExtractedAppPackage.png
[deploy-app-to-local-cluster]: ./media/service-fabric-get-started-with-a-local-cluster/DeployAppToLocalCluster.png
[deployed-app-ui]: ./media/service-fabric-get-started-with-a-local-cluster/DeployedAppUI-v1.png
[deployed-app-ui-v2]: ./media/service-fabric-get-started-with-a-local-cluster/DeployedAppUI-PostUpgrade.png
[sfx-app-instance]: ./media/service-fabric-get-started-with-a-local-cluster/SfxAppInstance.png
[sfx-two-app-instances-different-partitions]: ./media/service-fabric-get-started-with-a-local-cluster/SfxTwoAppInstances-DifferentPartitionCount.png
[ps-getsfapp]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFApp.png
[ps-getsfsvc]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFSvc.png
[ps-getsfpartitions]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFPartitions.png
[ps-appupgradeprogress]: ./media/service-fabric-get-started-with-a-local-cluster/PS-AppUpgradeProgress.png
[ps-getsfsvc-postupgrade]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFSvc-PostUpgrade.png
[sfx-upgradeprogress]: ./media/service-fabric-get-started-with-a-local-cluster/SfxUpgradeOverview.png
[sfx-service-overview]: ./media/service-fabric-get-started-with-a-local-cluster/sfx-service-overview.png
[sfe-delete-application]: ./media/service-fabric-get-started-with-a-local-cluster/sfe-delete-application.png

<!---HONumber=Mooncake_0627_2016-->