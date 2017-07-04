<properties
    pageTitle="在 Service Fabric 中管理多个环境 | Azure"
    description="Service Fabric 应用程序可以在规模为一台计算机到数千台计算机的群集上运行。在某些情况下，你需要以不同的方式针对各种环境配置你的应用程序。本文介绍如何为每个环境定义不同的应用程序参数。"
    services="service-fabric"
    documentationcenter=".net"
    author="seanmck"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f406eac9-7271-4c37-a0d3-0a2957b60537"
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="2/06/2017"
    wacn.date="03/03/2017"
    ms.author="seanmck" />

# 管理多个环境的应用程序参数

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

可以从任何位置，使用任意数量的计算机（从一台到数千台）来创建 Service Fabric 群集。尽管无需针对各种环境进行修改即可运行应用程序二进制文件，但通常会根据所要部署的计算机数目，以不同的方式配置应用程序。

举个简单的例子，假设某个无状态服务有 `InstanceCount` 参数。在 Azure 中运行应用程序时，通常要将此参数设置为特殊值“-1”。这可确保服务在群集中的每个节点上运行（或在节点类型中的每个节点上运行，如果已设置放置约束）。但是，此配置并不适用于单计算机群集，因为不能有多个进程在单计算机的同一终结点上进行侦听。在这种情况下，你通常会将 `InstanceCount` 设置为“1”。

## 指定特定于环境的参数
此配置问题的解决方法是使用一组参数化默认服务和应用程序参数文件，其中填充了给定环境的参数值。默认服务和应用程序参数在应用程序和服务清单中进行配置。ServiceManifest.xml 和 ApplicationManifest.xml 文件的架构定义随 Service Fabric SDK 和工具一起安装到 *C:\\Program Files\\Microsoft SDKs\\Service Fabric\\schemas\\ServiceFabricServiceModel.xsd*。

### 默认服务
Service Fabric 应用程序由服务实例的集合组成。尽管可以先创建一个空应用程序，然后动态创建所有服务实例，但大多数应用程序都有一套核心服务，这些服务始终应该在实例化应用程序时创建。这些服务称为“默认服务”。它们在应用程序清单中指定。方括号中包含为每个环境配置的占位符：

    <DefaultServices>
        <Service Name="Stateful1">
            <StatefulService
                ServiceTypeName="Stateful1Type"
                TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]"
                MinReplicaSetSize="[Stateful1_MinReplicaSetSize]">

                <UniformInt64Partition
                    PartitionCount="[Stateful1_PartitionCount]"
                    LowKey="-9223372036854775808"
                    HighKey="9223372036854775807"
                />
        </StatefulService>
    </Service>
  </DefaultServices>

必须在应用程序清单的 Parameters 元素中定义每个命名参数：

    <Parameters>
        <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="2" />
        <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
        <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
    </Parameters>

DefaultValue 属性指定给定环境缺少更具体的参数时所要使用的值。

>[AZURE.NOTE] 并非所有的服务实例参数都适用于每个环境配置。在上述示例中，已针对服务的所有实例显式定义服务分区方案的 LowKey 和 HighKey 值，因为分区范围是数据域的函数，而不是环境的函数。


### 每个环境的服务配置设置

服务可以使用 [Service Fabric 应用程序模型](/documentation/articles/service-fabric-application-model/)加入配置包，其中包含可在运行时读取的自定义键值对。也可以通过在应用程序清单中指定 `ConfigOverride`，按环境区分这些设置的值。

假设`Stateful1`服务的 Config\\Settings.xml 文件中存在以下设置：


    <Section Name="MyConfigSection">
      <Parameter Name="MaxQueueSize" Value="25" />
    </Section>

若要重写特定应用程序/环境对的此值，请在应用程序清单中导入服务清单时创建 `ConfigOverride`。

    <ConfigOverrides>
     <ConfigOverride Name="Config">
        <Settings>
           <Section Name="MyConfigSection">
              <Parameter Name="MaxQueueSize" Value="[Stateful1_MaxQueueSize]" />
           </Section>
        </Settings>
     </ConfigOverride>
  </ConfigOverrides>

然后可根据上面所示，按环境配置此参数。为此，可以在应用程序清单的 parameters 节中声明该参数，并在应用程序参数文件中指定特定于环境的值。

>[AZURE.NOTE] 对于服务配置设置，可在三个位置设置键的值：服务配置包、应用程序清单和应用程序参数文件。Service Fabric 始终先从应用程序参数文件（如果已指定）进行选择，再从应用程序清单选择，最后从配置包选择。

### 设置和使用环境变量 
可以先在 ServiceManifest.xml 文件中指定和设置环境变量，然后在 ApplicationManifest.xml 文件中逐个实例地将其重写。以下示例显示了两个环境变量，其中一个的值已设置，另一个将被重写。可以使用应用程序参数设置环境变量值，其方式与使用这些参数进行配置重写相同。


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

若要在 ApplicationManifest.xml 中重写环境变量，请使用 `EnvironmentOverrides` 元素引用 ServiceManifest 中的代码包。


	  <ServiceManifestImport>
	    <ServiceManifestRef ServiceManifestName="FrontEndServicePkg" ServiceManifestVersion="1.0.0" />
	    <EnvironmentOverrides CodePackageRef="MyCode">
	      <EnvironmentVariable Name="MyEnvVariable" Value="mydata"/>
	    </EnvironmentOverrides>
	  </ServiceManifestImport>

 创建命名的服务实例以后，即可从代码访问环境变量。例如，在 C\# 中，可以执行以下命令


    	string EnvVariable = Environment.GetEnvironmentVariable("MyEnvVariable");

### Service Fabric 环境变量
Service Fabric 针对每个服务实例设置内置的环境变量。环境变量的完整列表如下所示，其中以粗体显示的环境变量将用于服务中，其他的则由 Service Fabric 运行时使用。

* Fabric\_ApplicationHostId
* Fabric\_ApplicationHostType
* Fabric\_ApplicationId
* **Fabric\_ApplicationName**
* Fabric\_CodePackageInstanceId
* **Fabric\_CodePackageName**
* **Fabric\_Endpoint\_[YourServiceName]TypeEndpoint**
* **Fabric\_Folder\_App\_Log**
* **Fabric\_Folder\_App\_Temp**
* **Fabric\_Folder\_App\_Work**
* **Fabric\_Folder\_Application**
* Fabric\_NodeId
* **Fabric\_NodeIPOrFQDN**
* **Fabric\_NodeName**
* Fabric\_RuntimeConnectionAddress
* Fabric\_ServicePackageInstanceId
* Fabric\_ServicePackageName
* Fabric\_ServicePackageVersionInstance
* FabricPackageFileName

下列代码说明如何列出 Service Fabric 环境变量

	    foreach (DictionaryEntry de in Environment.GetEnvironmentVariables())
	    {
	        if (de.Key.ToString().StartsWith("Fabric"))
	        {
	            Console.WriteLine(" Environment variable {0} = {1}", de.Key, de.Value);
	        }
	    }

在本地开发计算机上运行时，如果应用程序类型为 `GuestExe.Application`，服务类型为 `FrontEndService`，则会用到以下示例环境变量。

* **Fabric\_ApplicationName = fabric:/GuestExe.Application**
* **Fabric\_CodePackageName = Code**
* **Fabric\_Endpoint\_FrontEndServiceTypeEndpoint = 80**
* **Fabric\_NodeIPOrFQDN = localhost**
* **Fabric\_NodeName = _Node_2**

### 应用程序参数文件

Service Fabric 应用程序项目可以包含一个或多个应用程序参数文件。每个文件为应用程序清单中定义的参数定义特定值：

    <!-- ApplicationParameters\Local.xml -->

    <Application Name="fabric:/Application1" xmlns="http://schemas.microsoft.com/2011/01/fabric">
        <Parameters>
            <Parameter Name ="Stateful1_MinReplicaSetSize" Value="2" />
            <Parameter Name="Stateful1_PartitionCount" Value="1" />
            <Parameter Name="Stateful1_TargetReplicaSetSize" Value="3" />
        </Parameters>
    </Application>

默认情况下，新应用程序包含三个应用程序参数文件，分别名为 Local.1Node.xml、Local.5Node.xml 和 Cloud.xml：

![解决方案资源管理器中的应用程序参数文件][app-parameters-solution-explorer]  


若要创建新的参数文件，只需复制并粘贴现有参数文件并为它指定新名称。

## 在部署期间识别特定于环境的参数
部署时，需选择要应用于应用程序的适当参数文件。可以通过 Visual Studio 中的“发布”对话框或通过 PowerShell 执行此操作。

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-visual-studio-login-guide.md)]

### 从 Visual Studio 部署

在 Visual Studio 中发布应用程序时，可以从可用参数文件列表中进行选择。

![在“发布”对话框中选择参数文件][publishdialog]

### 从 PowerShell 部署

应用程序项目模板中包含的 `Deploy-FabricApplication.ps1` PowerShell 脚本可接受发布配置文件作为参数，而 PublishProfile 包含对应用程序参数文件的引用。

  
    ./Deploy-FabricApplication -ApplicationPackagePath <app_package_path> -PublishProfileFile <publishprofile_path>
  

## 后续步骤

若要深入了解本主题中所述的某些核心概念，请参阅 [Service Fabric 技术概述](/documentation/articles/service-fabric-technical-overview/)。有关 Visual Studio 中其他可用应用管理功能的信息，请参阅[在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。

<!-- Image references -->

[publishdialog]: ./media/service-fabric-manage-multiple-environment-app-configuration/publish-dialog-choose-app-config.png
[app-parameters-solution-explorer]: ./media/service-fabric-manage-multiple-environment-app-configuration/app-parameters-in-solution-explorer.png

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add "Service Fabric 环境变量" section-->