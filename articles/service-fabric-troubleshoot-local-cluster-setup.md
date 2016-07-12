<properties
   pageTitle="对本地 Service Fabric 群集设置进行故障排除 | Azure"
   description="本文就本地开发群集的故障介绍一些建议"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="04/06/2016"
   wacn.date="07/04/2016"/>

# 排除本地开发群集安装的故障

如果你在与本地 Azure Service Fabric 开发群集交互时遇到问题，请查看以下建议以获得可能的解决方案。

## 群集安装失败

### 无法清除 Service Fabric 日志

#### 问题

在运行 DevClusterSetup 脚本时，你看到了类似下面的错误：

    Cannot clean up C:\SfDevCluster\Log fully as references are likely being held to items in it. Please remove those and run this script again.
    At line:1 char:1 + .\DevClusterSetup.ps1
    + ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo : NotSpecified: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,DevClusterSetup.ps1


#### 解决方案

关闭当前 PowerShell 窗口，然后以管理员身份打开一个新的 PowerShell 窗口。现在应能成功运行该脚本。

## 群集连接失败

### Azure PowerShell 不能识别 Service Fabric PowerShell cmdlet

#### 问题

如果你尝试运行任何 Service Fabric PowerShell cmdlet（例如，在 Azure PowerShell 窗口中运行 `Connect-ServiceFabricCluster`），该操作将会失败，指出无法识别该 cmdlet。发生这种失败的原因是 Azure PowerShell 使用 32 位版本的 Windows PowerShell（即使在 64 位操作系统版本中），而 Service Fabric cmdlet 只能在 64 位环境中工作。

#### 解决方案

始终直接从 Windows PowerShell 运行 Service Fabric cmdlet。

>[AZURE.NOTE] 最新版本的 Azure PowerShell 不创建特殊的快捷方式，因此不会再出现此问题。

### 类型初始化异常

#### 问题

当你在 PowerShell 中连接到群集时，你将看到针对 System.Fabric.Common.AppTrace 的 TypeInitializationException 错误。

#### 解决方案

在安装期间未正确设置路径变量。请从 Windows 注销并重新登录。这将完全刷新你的路径。

### 群集连接失败，并显示“对象已关闭”

#### 问题

对 Connect-ServiceFabricCluster 的调用失败，并显示类似下面的错误：

    Connect-ServiceFabricCluster : The object is closed.
    At line:1 char:1
    + Connect-ServiceFabricCluster
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo : InvalidOperation: (:) [Connect-ServiceFabricCluster], FabricObjectClosedException
    + FullyQualifiedErrorId : CreateClusterConnectionErrorId,Microsoft.ServiceFabric.Powershell.ConnectCluster

#### 解决方案

关闭当前 PowerShell 窗口，然后以管理员身份打开一个新的 PowerShell 窗口。现在应能成功连接。

### 结构连接被拒绝异常

#### 问题

从 Visual Studio 调试时，系统出现 FabricConnectionDeniedException 错误。

#### 解决方案

此错误通常在你尝试手动启动一个服务主机进程（而不是允许 Service Fabric 运行时为你启动该进程）时出现。

确保在你的解决方案中没有将任何服务项目设置为启动项目。只应将 Service Fabric 应用程序项目设置为启动项目。


## 后续步骤

- [使用系统运行状况报告了解群集并排除故障](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)
- [使用 Service Fabric 资源管理器可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)

<!---HONumber=Mooncake_0503_2016-->