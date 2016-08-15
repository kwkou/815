<properties
   pageTitle="Service Fabric 应用程序部署 | Azure"
   description="如何在 Service Fabric 中部署和删除应用程序"
   services="service-fabric"
   documentationCenter=".net"
   authors="alexwun"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/14/2016"
   wacn.date="07/04/2016"/>

# 部署应用程序

[打包应用程序类型后][10]，就可以部署到 Azure Service Fabric 群集中。部署涉及以下三个步骤：

1. 上载应用程序包
2. 注册应用程序类型
3. 创建应用程序实例

>[AZURE.NOTE] 如果使用 Visual Studio 来部署和调试本地开发群集上的应用程序，则将通过在应用程序项目的 Scripts 文件夹中找到的 PowerShell 脚本自动处理如下所述的全部步骤。本文提供有关这些脚本正在执行什么操作的背景，以便你可以在 Visual Studio 外部执行相同的操作。

## 上载应用程序包

上载应用程序包会将其放在一个可由内部 Service Fabric 组件访问的位置。你可以使用 PowerShell 执行上载。在运行本文中的任何 PowerShell 命令之前，请始终先使用 **Connect-ServiceFabricCluster** 连接到 Service Fabric 群集。

假设你有一个名为 *MyApplicationType* 的文件夹，其中包含必要的应用程序清单、服务清单以及代码/配置/数据包。**Copy-ServiceFabricApplicationPackage** 命令可将包上载到群集映像存储。Service Fabric SDK PowerShell 模块中包含的 **Get-ImageStoreConnectionStringFromClusterManifest** cmdlet 用于获取映像存储连接字符串。若要导入 SDK 模块，请运行 *Import-Module "$ENV:ProgramFiles\\Microsoft SDKs\\Service Fabric\\Tools\\PSModule\\ServiceFabricSDK\\ServiceFabricSDK.psm1"*。

以下示例将上载包：

~~~
PS D:\temp> dir

    Directory: D:\temp

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         3/19/2015   8:11 PM            MyApplicationType

PS D:\temp> tree /f .\MyApplicationType

D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat

PS D:\temp> Copy-ServiceFabricApplicationPackage -ApplicationPackagePath MyApplicationType -ImageStoreConnectionString (Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest))
Copy application package succeeded

PS D:\temp>
~~~

## 注册应用程序包

注册应用程序包，使应用程序清单中声明的应用程序类型和版本可供使用。系统将读取上一步中上载的程序包，验证此包（等效于本地运行 **Test-ServiceFabricApplicationPackage**），处理包的内容，并将已处理的包复制到内部系统位置。

~~~
PS D:\temp> Register-ServiceFabricApplicationType MyApplicationType
Register application type succeeded

PS D:\temp> Get-ServiceFabricApplicationType

ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
DefaultParameters      : {}

PS D:\temp>
~~~

**Register-ServiceFabricApplicationType** 命令将仅在系统成功复制应用程序包后返回。此操作花费的时间取决于应用程序包的内容。如有需要，**-TimeoutSec** 参数可用来提供较长的超时时间。（默认超时值为 60 秒。）

**Get-ServiceFabricApplicationType** 命令将列出已成功注册的所有应用程序类型版本。

## 创建应用程序

可以使用已通过 **New-ServiceFabricApplication** 命令成功注册的任何应用程序类型版本来实例化应用程序。每个应用程序的名称必须以 *fabric:* 方案开头，并且对每个应用程序实例是唯一的。如果目标应用程序类型的应用程序清单中定义有任何默认服务，则此时将还创建那些服务。

~~~
PS D:\temp> New-ServiceFabricApplication fabric:/MyApp MyApplicationType AppManifestVersion1

ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
ApplicationParameters  : {}

PS D:\temp> Get-ServiceFabricApplication  

ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
ApplicationStatus      : Ready
HealthState            : OK
ApplicationParameters  : {}

PS D:\temp> Get-ServiceFabricApplication | Get-ServiceFabricService

ServiceName            : fabric:/MyApp/MyService
ServiceKind            : Stateless
ServiceTypeName        : MyServiceType
IsServiceGroup         : False
ServiceManifestVersion : SvcManifestVersion1
ServiceStatus          : Active
HealthState            : Ok

PS D:\temp>
~~~

**Get-ServiceFabricApplication** 命令将列出已成功创建的所有应用程序实例以及其整体状态。

**Get-ServiceFabricService** 命令将列出在给定的应用程序实例中成功创建的所有服务实例。将在此处列出默认服务（如果有）。

可以为已注册应用程序类型的任何给定版本创建多个应用程序实例。每个应用程序实例都将隔离运行，具有其自己的工作目录和进程。

## 删除应用程序

当不再需要某个应用程序实例时，可以使用 **Remove-ServiceFabricApplication** 命令将其永久删除。此命令也将自动删除属于该应用程序的所有服务，永久删除所有服务状态。此操作无法撤消，并且无法恢复应用程序状态。

~~~
PS D:\temp> Remove-ServiceFabricApplication fabric:/MyApp

Confirm
Continue with this operation?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
Remove application instance succeeded

PS D:\temp> Get-ServiceFabricApplication
PS D:\temp>
~~~

当不再需要应用程序类型的某个特定版本时，应使用 **Unregister-ServiceFabricApplicationType** 命令将其注销。注销未使用的类型将在映像存储中释放该类型的应用程序包内容使用的存储空间。只要没有针对其实例化的应用程序或引用它的挂起应用程序升级，就可以注销应用程序类型。

~~~
PS D:\temp> Get-ServiceFabricApplicationType

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v1
DefaultParameters      : {}

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v2
DefaultParameters      : {}

ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : AppManifestVersion1
DefaultParameters      : {}

PS D:\temp> Unregister-ServiceFabricApplicationType MyApplicationType AppManifestVersion1
Unregister application type succeeded

PS D:\temp> Get-ServiceFabricApplicationType

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v1
DefaultParameters      : {}

ApplicationTypeName    : DemoAppType
ApplicationTypeVersion : v2
DefaultParameters      : {}

PS D:\temp>
~~~

## 故障排除

### Copy-ServiceFabricApplicationPackage 请求 ImageStoreConnectionString

Service Fabric SDK 环境应该已经设置了正确的默认设置。若有需要，所有命令的 ImageStoreConnectionString 都应匹配 Service Fabric 群集正在使用的值。你可以在通过 **Get-ServiceFabricClusterManifest** 命令检索到的群集清单中找到此值：

~~~
PS D:\temp> Copy-ServiceFabricApplicationPackage .\MyApplicationType

cmdlet Copy-ServiceFabricApplicationPackage at command pipeline position 1
Supply values for the following parameters:
ImageStoreConnectionString:

PS D:\temp> Get-ServiceFabricClusterManifest
<ClusterManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Name="Server-Default-SingleNode" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

    [...]

    <Section Name="Management">
      <Parameter Name="ImageStoreConnectionString" Value="file:D:\ServiceFabric\Data\ImageStore" />
    </Section>

    [...]

PS D:\temp> Copy-ServiceFabricApplicationPackage .\MyApplicationType -ImageStoreConnectionString file:D:\ServiceFabric\Data\ImageStore
Copy application package succeeded

PS D:\temp>
~~~

## 后续步骤

[Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

[Service Fabric 运行状况简介](/documentation/articles/service-fabric-health-introduction/)

[对 Service Fabric 进行诊断和故障排除](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)

[在 Service Fabric 中对应用程序建模](/documentation/articles/service-fabric-application-model/)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: /documentation/articles/service-fabric-application-model/
[11]: /documentation/articles/service-fabric-application-upgrade/
 
<!---HONumber=Mooncake_0627_2016-->