<properties
   pageTitle="将 Web 角色和辅助角色转换成 Service Fabric 无状态服务的指南 | Azure"
   description="本指南将云服务 Web 角色和辅助角色与 Service Fabric 无状态服务进行比较，以帮助你从云服务迁移到 Service Fabric。"
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="02/29/2016"
   wacn.date="07/04/2016"/>
 
# 将 Web 角色和辅助角色转换成 Service Fabric 无状态服务的指南

本文说明如何将云服务的 Web 角色和辅助角色迁移到 Service Fabric 无状态服务。对于整体体系结构大致保持相同的应用程序来说，这是最简单的云服务到 Service Fabric 迁移路径。

## 云服务项目到 Service Fabric 应用程序项目
 
 云服务项目和 Service Fabric 应用程序项目结构类似，两者都可代表应用程序的部署单位，也就是说，两者各自定义可在部署后运行应用程序的完整包。云服务项目包含一个或多个 Web 角色和辅助角色。同理，Service Fabric 应用程序项目包含一个或多个服务。

两者的差别在于，云服务项目结合应用程序部署与 VM 部署，因此其中包含 VM 配置设置，而 Service Fabric 应用程序项目只定义将要部署到 Service Fabric 群集中一组现有 VM 的应用程序。Service Fabric 群集本身只可通过 ARM 模板或 Azure 门户部署一次，但可在群集中部署多个 Service Fabric 应用程序。

![Service Fabric 与云服务项目的比较][3]
 
## 辅助角色到无状态服务 

从概念上讲，辅助角色代表无状态的工作负荷，这意味着工作负荷的每个实例都是相同的，随时可将请求路由到任何实例。每个实例不需要记住前一个请求。工作负荷的运行状态由外部状态存储（例如 Azure 表存储或 Azure Document DB）管理。在 Service Fabric 中，此类工作负荷以无状态服务来表示。将辅助角色迁移到 Service Fabric 的最简单方法是将辅助角色代码转换成无状态服务。

![辅助角色到无状态服务][4]

## Web 角色到无状态服务

与辅助角色类似，Web 角色也代表无状态的工作负荷，因此在概念上也能映射到 Service Fabric 无状态服务。不过，与 Web 角色不同的是，Service Fabric 不支持 IIS。若要将 Web 应用程序从 Web 角色迁移到无状态服务，需要先移动到可以自我托管且不依赖 IIS 或 System.Web 的 Web 框架（例如 ASP.NET Core 1）。

**应用程序** | **支持** | **迁移路径**
--- | --- | ---
ASP.NET Web 窗体 | 否 | 转换为 ASP.NET Core 1 MVC
ASP.NET MVC | 使用迁移 | 升级到 ASP.NET Core 1
ASP.NET Web API | 使用迁移 | 使用自托管服务器或 ASP.NET Core 1
ASP.NET Core 1 | 是 | 不适用

## 入口点 API 和生命周期

辅助角色和 Service Fabric 服务 API 提供类似的入口点：

**入口点** | **辅助角色** | **Service Fabric 服务**
--- | --- | ---
正在处理 | `Run()` | `RunAsync()`
VM 启动 | `OnStart()` | 不适用
VM 停止 | `OnStop()` | 不适用
为客户端请求打开侦听器 | 不适用 | <ul><li> `CreateServiceInstanceListener()`（适用于无状态）</li><li>`CreateServiceReplicaListener()`（适用于有状态）</li></ul>

### 辅助角色

```C#

using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override void Run()
        {
        }

        public override bool OnStart()
        {
        }

        public override void OnStop()
        {
        }
    }
}

```

### Service Fabric 无状态服务

```C#

using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

namespace Stateless1
{
    public class Stateless1 : StatelessService
    {
        protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
        {
        }

        protected override Task RunAsync(CancellationToken cancelServiceInstance)
        {
        }
    }
}

```

两者都有可从中开始处理的主要“Run”重写。Service Fabric 服务将 `Run`、`Start` 和 `Stop` 合并为单一入口点 `RunAsync`。当 `RunAsync` 启动时，服务应开始工作；发出 `RunAsync` 方法的 CancellationToken 信号时，应停止工作。

辅助角色和 Service Fabric 服务的生命周期与生存期之间有几个主要差异：

 - **生命周期：**最大的差异为辅助角色是 VM，因此其生命周期绑定到 VM，且包含 VM 启动和停止时的事件。Service Fabric 服务的生命周期与 VM 的生命周期不同，因此不包含主机 VM 或计算机启动和停止时的事件，因为它们彼此不相关。

 - **生存期：**如果 `Run` 方法退出，辅助角色实例将回收。但是，Service Fabric 服务中的 `RunAsync` 方法可以运行到完成为止，服务实例将保持运行状态。

Service Fabric 为侦听客户端请求的服务提供可选的通信设置入口点。RunAsync 和通信入口点都是 Service Fabric 服务中的可选重写（服务可选择只侦听客户端请求和/或只运行处理循环），这就是 RunAsync 方法无需重新启动服务实例就可退出的原因，因为它可以继续侦听客户端请求。

## 应用程序 API 和环境

云服务环境 API 提供当前 VM 实例的信息和功能，以及有关其他 VM 角色实例的信息。Service Fabric 提供有关其运行时的信息，以及有关服务当前运行所在的节点的某些信息。

**环境任务** | **云服务** | **Service Fabric**
--- | --- | ---
配置设置和更改通知 | `RoleEnvironment` | `CodePackageActivationContext`
本地存储 | `RoleEnvironment` | `CodePackageActivationContext`
终结点信息 | `RoleInstance` <ul><li>当前实例：`RoleEnvironment.CurrentRoleInstance`</li><li>其他角色和实例：`RoleEnvironment.Roles`</li> | <ul><li>`NodeContext`（适用于当前的节点地址）</li><li>`FabricClient` 和 `ServicePartitionResolver`（适用于服务终结点发现）</li> 
环境模拟 | `RoleEnvironment.IsEmulated` | 不适用
同时更改事件 | `RoleEnvironment` | 不适用

## 配置设置

云服务中的配置设置是针对 VM 角色设置的，将应用到该 VM 角色的所有实例。这些设置是 ServiceConfiguration.*.cscfg 文件中设置的键-值对，可直接通过 RoleEnvironment 进行访问。在 Service Fabric 中，设置单独应用到每个服务和每个应用程序，而不是应用到 VM，因为 VM 可以托管多个服务和应用程序。服务由三个包组成：

 - **代码：**包含服务的可执行文件、二进制文件、DLL 和服务需要运行的任何其他文件。
 - **配置：**服务的所有配置文件和设置。
 - **数据：**与服务关联的静态数据文件。

其中每个包可独立设置版本和进行升级。与云服务类似，可通过 API 以编程方式访问配置包。发生配置包更改时，系统会提供事件来通知服务。Settings.xml 文件可用于键-值配置和编程访问。但是，与云服务不同的是，Service Fabric 配置包可以包含任何格式的任何配置文件，不管是 XML、JSON、YAML 还是自定义的二进制格式。


### 访问配置
#### 云服务

可通过 `RoleEnvironment` 访问 ServiceConfiguration.*.cscfg 中的配置设置。这些设置可全局提供给同一云服务部署中的所有角色实例使用。

```C#

string value = RoleEnvironment.GetConfigurationSettingValue("Key");

```

#### ServiceFabic

每个服务都有自身的独立配置包。可供群集中所有应用程序访问的全局配置设置没有内置机制。使用配置包中的 Service Fabric 特殊配置文件 Settings.xml 时，Settings.xml 中的值可以在应用程序级别覆盖，实现应用程序级别的配置设置。

通过服务的 `CodePackageActivationContext` 可在每个服务实例中访问配置设置。

```C#

ConfigurationPackage configPackage = this.ServiceInitializationParameters.CodePackageActivationContext.GetConfigurationPackageObject("Config");

// Access Settings.xml
KeyedCollection<string, ConfigurationProperty> parameters = configPackage.Settings.Sections["MyConfigSection"].Parameters;

string value = parameters["Key"]?.Value;

// Access custom configuration file:
using (StreamReader reader = new StreamReader(Path.Combine(configPackage.Path, "CustomConfig.json")))
{
    MySettings settings = JsonConvert.DeserializeObject<MySettings>(reader.ReadToEnd());
}

```

### 配置更新事件
#### 云服务

当环境中发生更改（例如配置更改）时，将使用 `RoleEnvironment.Changed` 事件来通知所有角色实例。通过此事件可以使用配置更新，却无需回收角色实例或重新启动辅助角色进程。

```C#

RoleEnvironment.Changed += RoleEnvironmentChanged;

private void RoleEnvironmentChanged(object sender, RoleEnvironmentChangedEventArgs e)
{
   // Get the list of configuration changes
   var settingChanges = e.Changes.OfType<RoleEnvironmentConfigurationSettingChange>();
foreach (var settingChange in settingChanges) 
   {
      Trace.WriteLine("Setting: " + settingChange.ConfigurationSettingName, "Information");
   }
}

```

#### ServiceFabic

在服务中的三个包类型（代码、配置和数据）中，每个类型都会提供可在包更新、添加或删除时通知服务实例的事件。一个服务可以包含每种类型的多个包。例如，一个服务可以有多个配置包，其中每个包可单独设置版本并且可升级。

通过这些事件可以使用服务包中的更改，而无需重新启动服务实例。
 
```C#

this.ServiceInitializationParameters.CodePackageActivationContext.ConfigurationPackageModifiedEvent +=
                    this.CodePackageActivationContext_ConfigurationPackageModifiedEvent;

private void CodePackageActivationContext_ConfigurationPackageModifiedEvent(object sender, PackageModifiedEventArgs<ConfigurationPackage> e)
{
    this.UpdateCustomConfig(e.NewPackage.Path);
    this.UpdateSettings(e.NewPackage.Settings);
}

```

## 启动任务

启动任务是应用程序启动前执行的操作。启动任务通常用于以提升的权限运行设置脚本。云服务和 Service Fabric 均支持启动任务。两者的主要差异是，云服务中的启动任务绑定到 VM，因为 VM 是角色实例的一部分；而 Service Fabric 中的启动任务则绑定到服务，而不绑定到任何特定 VM。

 | 云服务 | Service Fabric
--- | --- | ---
配置位置 | ServiceDefinition.csdef | ServiceManifest.xml
特权 | “受限”或“提升” | 任何用户或计算机帐户
序列 | “简单”、“后台”、“前台” | 在启动服务之前，启动任务的执行必须成功。

### 云服务
云服务中的启动入口点是在 ServiceDefintion.csdef 中针对每个角色配置的。



	<ServiceDefinition>
    	<Startup>
        	<Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" >
            	<Environment>
                	<Variable name="MyVersionNumber" value="1.0.0.0" />
            	</Environment>
        	</Task>
    	</Startup>
    	...
	</ServiceDefinition>


### Service Fabric

Service Fabric 中的启动入口点是在 ServiceManifest.xml 中针对每个服务配置的。



	<ServiceManifest>
  	<CodePackage Name="Code" Version="1.0.0">
    	<SetupEntryPoint>
      	<ExeHost>
        	<Program>Startup.bat</Program>
      	</ExeHost>
    	</SetupEntryPoint>
    	...
	</ServiceManifest>


## 有关开发环境的说明

云服务和 Service Fabric 都使用项目模板来与 Visual Studio 集成，并支持在本地和 Azure 中调试、配置及部署。此外，云服务和 Service Fabric 都提供本地开发运行时环境。差别在于，云服务的开发运行时模拟其运行所在的 Azure 环境，Service Fabric 不使用模拟器，而是使用完整的 Service Fabric 运行时。在本地开发计算机运行的 Service Fabric 环境就是在生产时运行的同一环境。

##后续步骤

阅读有关 Service Fabric Reliable Services 的详细信息以及云服务与 Service Fabric 应用程序体系结构之间的差异，以了解如何利用 Service Fabric 的完整功能集。

 - [Service Fabric Reliable Services 入门](/documentation/articles/service-fabric-reliable-services-quick-start/)

 - [云服务与 Service Fabric 之间差异的概念指南](/documentation/articles/service-fabric-cloud-services-migration-differences/)
 
<!--Image references-->
[3]: ./media/service-fabric-cloud-services-migration-worker-role-stateless-service/service-fabric-cloud-service-projects.png
[4]: ./media/service-fabric-cloud-services-migration-worker-role-stateless-service/worker-role-to-stateless-service.png

<!---HONumber=Mooncake_0418_2016-->