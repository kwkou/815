<properties
   pageTitle="在 Service Fabric 中管理多个环境 | Azure"
   description="Service Fabric 应用程序可以在规模为一台计算机到数千台计算机的群集上运行。在某些情况下，你需要以不同的方式针对各种环境配置你的应用程序。本文介绍如何为每个环境定义不同的应用程序参数。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="coreysa"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="04/26/2016"
   wacn.date="07/04/2016"/>

# 管理多个环境的应用程序参数

你可以在任何位置，使用任意数量的计算机（从一台到数千台）来创建 Service Fabric 群集。尽管无需针对各种环境进行修改即可运行应用程序二进制文件，但你通常会根据所要部署的计算机数目，以不同的方式配置应用程序。

举个简单的例子，假设某个无状态服务有 `InstanceCount` 参数。当你在 Azure 中运行应用程序时，通常要将此参数设置为特殊值“-1”。这可确保服务在群集中的每个节点上运行。但是，此配置并不适用于单计算机群集，因为不能有多个进程在单计算机的同一终结点上侦听。在这种情况下，你通常会将 `InstanceCount` 设置为“1”。

## 指定特定于环境的参数

此配置问题的解决方法是使用一组参数化默认服务和应用程序参数文件，其中填充了给定环境的参数值。默认服务和应用程序参数在应用程序和服务清单中进行配置。ServiceManifest.xml 和 ApplicationManifest.xml 文件的架构定义随 Service Fabric SDK 和工具一起安装到 *C:\\Program Files\\Microsoft SDKs\\Service Fabric\\schemas\\ServiceFabricServiceModel.xsd*。

### 默认服务

Service Fabric 应用程序由服务实例的集合组成。尽管你可以先创建一个空应用程序，然后动态创建所有服务实例，但是，大多数应用程序都有一套核心服务，这些服务始终应该在实例化应用程序时创建。这些服务称为“默认服务”。它们在应用程序清单中指定。方括号中包含每个环境配置的占位符：

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

DefaultValue 属性指定当给定的环境缺少更具体的参数时所要使用的值。

>[AZURE.NOTE] 并非所有的服务实例参数都适用于每个环境配置。在上述示例中，已针对服务的所有实例显式定义服务分区方案的 LowKey 和 HighKey 值，因为分区范围与数据域而不是与环境相关。


### 每个环境的服务配置设置

服务可以使用 [Service Fabric 应用程序模型](/documentation/articles/service-fabric-application-model/)加入配置包，其中包含可在运行时读取的自定义键-值对。也可以通过在应用程序清单中指定 `ConfigOverride`，按环境区分这些设置的值。

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

默认情况下，新应用程序包含两个应用程序参数文件，分别名为 Local.xml 和 Cloud.xml：

![解决方案资源管理器中的应用程序参数文件][app-parameters-solution-explorer]

若要创建新的参数文件，只需复制并粘贴现有参数文件并为它指定新名称。

## 在部署期间识别特定于环境的参数

在部署时，需选择要应用于应用程序的适当参数文件。可以通过 Visual Studio 中的“发布”对话框或通过 PowerShell 执行此操作。

### 从 Visual Studio 部署

在 Visual Studio 中发布应用程序时，可以从可用参数文件列表中进行选择。

![在“发布”对话框中选择参数文件][publishdialog]

### 从 PowerShell 部署

应用程序项目模板中包含的 `Deploy-FabricApplication.ps1` PowerShell 脚本可接受发布配置文件作为参数，而 PublishProfile 包含对应用程序参数文件的引用。

  ```PowerShell
    ./Deploy-FabricApplication -ApplicationPackagePath <app_package_path> -PublishProfileFile <publishprofile_path>
  ```

## 后续步骤

若要深入了解本主题中所述的某些核心概念，请参阅 [Service Fabric 技术概述](/documentation/articles/service-fabric-technical-overview/)。有关 Visual Studio 中其他可用应用管理功能的信息，请参阅[在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。

<!-- Image references -->

[publishdialog]: ./media/service-fabric-manage-multiple-environment-app-configuration/publish-dialog-choose-app-config.png
[app-parameters-solution-explorer]: ./media/service-fabric-manage-multiple-environment-app-configuration/app-parameters-in-solution-explorer.png

<!---HONumber=Mooncake_0523_2016-->