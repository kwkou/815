<properties
    pageTitle="Service Fabric 应用程序模型 | Azure"
    description="如何在 Service Fabric 中建模和描述应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="mani-ramaswamy" />
<tags
    ms.assetid="17a99380-5ed8-4ed9-b884-e9b827431b02"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="1/05/2017"
    wacn.date="02/20/2017"
    ms.author="ryanwi" />

# 在 Service Fabric 中对应用程序建模
本文提供 Azure Service Fabric 应用程序模型的概述。同时介绍如何通过清单文件定义应用程序和服务，以及如何打包应用程序并准备好进行部署。

## 了解应用程序模型
应用程序是由执行一个或多个特定功能的成分服务组成的集合。服务执行完整且独立的功能（它们可以独立于其他服务启动和运行），并由代码、配置和数据组成。对于每个服务，代码由可执行二进制文件组成，配置由可在运行时加载的服务设置组成，数据则由可供该服务使用的任意静态数据组成。可对此层次应用程序模型中的每个组件进行独立的版本控制和升级。

![Service Fabric 应用程序模型][appmodel-diagram]

应用程序类型是一个应用程序分类，包含大量服务类型。服务类型是一个服务分类。该分类可以具有不同的设置和配置，但核心功能保持相同。服务实例是相同服务类型的不同服务配置变体。

应用程序和服务的类（或“类型”）是通过 XML 文件（应用程序清单和服务清单）进行描述的，这些文件是从群集映像存储实例化应用程序所依据的模板。ServiceManifest.xml 和 ApplicationManifest.xml 文件的架构定义随 Service Fabric SDK 和工具一起安装到 *C:\\Program Files\\Microsoft SDKs\\Service Fabric\\schemas\\ServiceFabricServiceModel.xsd*。

即使在由同一 Service Fabric 节点托管时，不同应用程序实例的代码也将作为单独的进程运行。此外，可以独立管理每个应用程序实例的生命周期（即升级）。下图显示应用程序类型由服务类型组成、服务类型由代码、配置和包组成。为了简化示意图，这里只显示了 `ServiceType4` 的代码/配置/数据包，不过每个服务类型包括其中的部分或全部包类型。

![Service Fabric 应用程序类型和服务类型][cluster-imagestore-apptypes]

使用两种清单文件来描述应用程序和服务，即服务清单和应用程序清单。后续部分将详细介绍这两种清单。

群集中可以有某一服务类型一个或多个实例处于活动状态。例如，有状态服务实例（或称副本）通过在群集中不同节点上的副本之间复制状态实现高可靠性。这种复制本质上是提供冗余，使服务即使在群集中的一个节点出现故障时也可用。[分区服务](/documentation/articles/service-fabric-concepts-partitioning/)进一步跨群集中的节点划分其状态（和该状态的访问模式）。

下图显示应用程序和服务实例、分区与副本之间的关系。

![服务中的分区和副本][cluster-application-instances]


>[AZURE.TIP] 可以使用 http://&lt;yourclusteraddress&gt;:19080/Explorer 上提供的 Service Fabric Explorer 工具查看群集中应用程序的布局。有关详细信息，请参阅[使用 Service Fabric Explorer 可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)。

## 描述服务
服务清单以声明方式定义服务类型和版本。它指定服务元数据，例如服务类型、运行状况属性、负载均衡度量值、服务二进制文件和配置文件。换言之，它描述了组成一个服务包以支持一个或多个服务类型的代码、配置和数据包。下面是服务清单的简单示例：


	<?xml version="1.0" encoding="utf-8" ?>
	<ServiceManifest Name="MyServiceManifest" Version="SvcManifestVersion1" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	  <Description>An example service manifest</Description>
	  <ServiceTypes>
	    <StatelessServiceType ServiceTypeName="MyServiceType" />
	  </ServiceTypes>
	  <CodePackage Name="MyCode" Version="CodeVersion1">
	    <SetupEntryPoint>
	      <ExeHost>
	        <Program>MySetup.bat</Program>
	      </ExeHost>
	    </SetupEntryPoint>
	    <EntryPoint>
	      <ExeHost>
	        <Program>MyServiceHost.exe</Program>
	      </ExeHost>
	    </EntryPoint>
	    <EnvironmentVariables>
	      <EnvironmentVariable Name="MyEnvVariable" Value=""/>
	      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
	    </EnvironmentVariables>
	  </CodePackage>
	  <ConfigPackage Name="MyConfig" Version="ConfigVersion1" />
	  <DataPackage Name="MyData" Version="DataVersion1" />
	</ServiceManifest>


**Version** 特性是未结构化的字符串，并且不由系统进行分析。这些特性用于对每个组件进行版本控制，以进行升级。

**ServiceTypes** 声明此清单中的 **CodePackages** 支持哪些服务类型。当一种服务针对这些服务类型之一进行实例化时，可激活此清单中声明的所有代码包，方法是运行这些代码包的入口点。生成的进程应在运行时注册所支持的服务类型。请注意，在清单级别而不是代码包级别声明服务类型。因此，当存在多个代码包时，每当系统查找任何一种声明的服务类型时，都将激活所有代码包。

**SetupEntryPoint** 是特权入口点，以与 Service Fabric（通常是 *LocalSystem* 帐户）相同的凭据先于任何其他入口点运行。**EntryPoint** 指定的可执行文件通常是长时间运行的服务主机。提供单独的设置入口点可避免长时间使用高特权运行服务主机。由 **EntryPoint** 指定的可执行文件在 **SetupEntryPoint** 成功退出后运行。如果总是终止或出现故障，则将监视并重启所产生的过程（再次从 **SetupEntryPoint** 开始）。

**EnvironmentVariables** 提供为此代码包设置的环境变量的列表。可在 `ApplicationManifest.xml` 中替代这些变量，为不同的服务实例提供不同的值。

**DataPackage** 声明一个由 **Name** 特性命名的文件夹，该文件夹中包含进程将在运行时使用的静态数据。

**ConfigPackage** 声明一个由 **Name** 特性命名的文件夹，该文件夹中包含 *Settings.xml* 文件。此文件包含用户定义的键值对设置部分，进程可在运行时读回这些设置。升级期间，如果仅更改了 **ConfigPackage** **版本**，则不重启正在运行的进程。相反，将通过回调告知进程配置设置已更改，以便动态重新加载。下面是 *Settings.xml* 文件的一个示例：

~~~
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MyConfigurationSecion">
    <Parameter Name="MySettingA" Value="Example1" />
    <Parameter Name="MySettingB" Value="Example2" />
  </Section>
</Settings>
~~~

> [AZURE.NOTE] 服务清单可以包含多段代码、多个配置和数据包。可对它们进行独立的版本控制。

<!--
For more information about other features supported by service manifests, refer to the following articles:

*TODO: LoadMetrics, PlacementConstraints, ServicePlacementPolicies
*TODO: Resources
*TODO: Health properties
*TODO: Trace filters
*TODO: Configuration overrides
-->

##<a name="describe-an-application"></a> 描述应用程序


应用程序清单以声明方式描述应用程序类型和版本。它指定服务组合元数据（如稳定名称、分区方案、实例计数/复制因子、安全/隔离策略、布置约束、配置替代和成分服务类型）。此外还描述用于容纳应用程序的负载均衡域。

因此，应用程序清单在应用程序级别描述元素，并引用一个或多个服务清单，以组成应用程序类型。下面是应用程序清单的简单示例：


	<?xml version="1.0" encoding="utf-8" ?>
	<ApplicationManifest
	      ApplicationTypeName="MyApplicationType"
	      ApplicationTypeVersion="AppManifestVersion1"
	      xmlns="http://schemas.microsoft.com/2011/01/fabric"
	      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	  <Description>An example application manifest</Description>
	  <ServiceManifestImport>
	    <ServiceManifestRef ServiceManifestName="MyServiceManifest" ServiceManifestVersion="SvcManifestVersion1"/>
	    <ConfigOverrides/>
	    <EnvironmentOverrides CodePackageRef="MyCode"/>
	  </ServiceManifestImport>
	  <DefaultServices>
	     <Service Name="MyService">
	         <StatelessService ServiceTypeName="MyServiceType" InstanceCount="1">
	             <SingletonPartition/>
	         </StatelessService>
	     </Service>
	  </DefaultServices>
	</ApplicationManifest>


类似于服务清单，**Version** 特性是未结构化的字符串，并且不由系统进行分析。这些特性也用于对每个组件进行版本控制，以进行升级。

**ServiceManifestImport** 包含对组成此应用程序类型的服务清单的引用。导入的服务清单将确定此应用程序类型中有效的服务类型。在 ServiceManifestImport 内，可替代 Settings.xml 中的配置值和 ServiceManifest.xml 文件中的环境变量。


**DefaultServices** 声明每当一个应用程序依据此应用程序类型进行实例化时自动创建的服务实例。默认服务只是提供便利，创建后，其行为皆如常规服务。它们与应用程序实例中的任何其他服务一起升级，也可将其删除。

> [AZURE.NOTE] 应用程序清单可以包含多个服务清单导入和默认服务。可对每个服务清单导入进行独立的版本控制。

若要了解如何维护不同的应用程序和用于单个环境的服务参数，请参阅[管理多个环境的应用程序参数](/documentation/articles/service-fabric-manage-multiple-environment-app-configuration/)。

<!--
For more information about other features supported by application manifests, refer to the following articles:

*TODO: Application parameters
*TODO: Security, Principals, RunAs, SecurityAccessPolicy
*TODO: Service Templates
-->

##<a name="package-an-application"></a> 打包应用程序

### 包布局
必须以特定布局组织应用程序清单、服务清单和其他必要的包文件，以便将其部署到 Service Fabric 群集中。本文中的示例清单需按以下目录结构进行组织：

~~~
PS D:\temp> tree /f .\MyApplicationType

D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
~~~

文件夹的名称与每个相应元素的 **Name** 特性匹配。例如，如果服务清单包含两个名为 **MyCodeA** 和 **MyCodeB** 的代码包，则两个同名的文件夹将包含用于每个代码包的必要二进制文件。

### 使用 SetupEntryPoint
**SetupEntryPoint** 的典型使用场景是需要在服务启动之前运行可执行文件，或需要使用提升的权限来执行操作时。例如：

- 设置和初始化服务可执行文件所需的环境变量。这并不限于通过 Service Fabric 编程模型编写的可执行文件。例如，npm.exe 需要配置一些环境变量来部署 node.js 应用程序。

- 通过安装安全证书设置访问控制。

有关如何配置 **SetupEntryPoint** 的详细信息，请参阅[配置服务设置入口点的策略](/documentation/articles/service-fabric-application-runas-security/)

### 配置 
### 使用 Visual Studio 生成包
如果使用 Visual Studio 2015 创建应用程序，可以使用 Package 命令自动创建符合上述布局的包。

若要创建包，请在解决方案资源管理器中右键单击应用程序项目，并选择 Package 命令，如下所示：

![使用 Visual Studio 打包应用程序][vs-package-command]

打包完成后，该包的位置将显示在“输出”窗口中。请注意，在 Visual Studio 中部署或调试应用程序时，打包步骤将自动发生。

### 测试包
可以使用 **Test-ServiceFabricApplicationPackage** 命令，通过 PowerShell 在本地验证包结构。此命令将检查是否存在清单分析问题，并验证所有引用。请注意，此命令只验证包中文件与目录结构的正确性。它不验证任何代码或数据包内容，而只检查所有必要的文件是否存在。


	PS D:\temp> Test-ServiceFabricApplicationPackage .\MyApplicationType
	False
	Test-ServiceFabricApplicationPackage : The EntryPoint MySetup.bat is not found.
	FileName: C:\Users\servicefabric\AppData\Local\Temp\TestApplicationPackage_7195781181\nrri205a.e2h\MyApplicationType\MyServiceManifest\ServiceManifest.xml


此错误显示代码包中缺少服务清单 **SetupEntryPoint** 中引用的 *MySetup.bat* 文件。添加缺少的文件后，应用程序验证通过：

~~~
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

PS D:\temp> Test-ServiceFabricApplicationPackage .\MyApplicationType
True
PS D:\temp>
~~~

正确打包应用程序并通过验证后，应用程序即已准备就绪，可供部署。

## 后续步骤
[部署和删除应用程序][10]介绍如何使用 PowerShell 来管理应用程序实例

[管理多个环境的应用程序参数][11]介绍如何为不同的应用程序实例配置参数和环境变量。

[配置应用程序的安全策略][12]介绍如何根据安全策略运行服务，从而限制访问。

<!--Image references-->
[appmodel-diagram]: ./media/service-fabric-application-model/application-model.png
[cluster-imagestore-apptypes]: ./media/service-fabric-application-model/cluster-imagestore-apptypes.png
[cluster-application-instances]: ./media/service-fabric-application-model/cluster-application-instances.png
[vs-package-command]: ./media/service-fabric-application-model/vs-package-command.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: /documentation/articles/service-fabric-deploy-remove-applications/
[11]: /documentation/articles/service-fabric-manage-multiple-environment-app-configuration/
[12]: /documentation/articles/service-fabric-application-runas-security/

<!---HONumber=Mooncake_0213_2017-->
<!--update: update code(add EnvironmentVariables parameter)-->