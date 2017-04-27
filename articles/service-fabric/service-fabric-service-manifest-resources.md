<properties
    pageTitle="指定 Service Fabric 服务终结点 | Azure"
    description="如何在服务清单中描述终结点资源，包括如何设置 HTTPS 终结点"
    services="service-fabric"
    documentationcenter=".net"
    author="mani-ramaswamy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="da36cbdb-6531-4dae-88e8-a311ab71520d"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/02/2017"
    wacn.date="04/24/2017"
    ms.author="subramar"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d1b81737031afbe09173b0bf682f024f066c391f"
    ms.lasthandoff="04/14/2017" />

# <a name="specify-resources-in-a-service-manifest"></a>在服务清单中指定资源
## <a name="overview"></a>概述
服务清单允许声明/更改服务要使用的资源而无需更改已编译的代码。Azure Service Fabric 支持对服务的终结点资源进行配置。可以通过应用程序清单中的 SecurityGroup 控制对服务清单中指定资源的访问。资源的声明允许在部署时更改这些资源，这意味着服务不需要引入新的配置机制。ServiceManifest.xml 文件的架构定义随 Service Fabric SDK 和工具一起安装到 *C:\\Program Files\\Microsoft SDKs\\Service Fabric\\schemas\\ServiceFabricServiceModel.xsd*。

## <a name="endpoints"></a>终结点
在服务清单中定义了终结点资源时，如果未显式指定端口，则 Service Fabric 从保留的应用程序端口范围中分配端口。 例如，可以查看本段落后面提供的清单代码段中指定的终结点 *ServiceEndpoint1* 。 此外，服务还可以请求在资源中使用特定端口。 在不同群集节点上运行的服务副本可以分配不同的端口号，而运行在同一节点上的服务副本共享同一个端口。 之后服务副本可根据需要将这些端口用于复制和侦听客户端请求。


	<Resources>
	  <Endpoints>
	    <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
	    <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
	    <Endpoint Name="ServiceEndpoint3" Protocol="https"/>
	  </Endpoints>
	</Resources>

请参阅[配置有状态 Reliable Services](/documentation/articles/service-fabric-reliable-services-configuration/)以了解有关从配置包设置文件 (settings.xml) 引用终结点的更多信息。

## <a name="example-specifying-an-http-endpoint-for-your-service"></a>示例：为服务指定 HTTP 终结点
以下服务清单在 &lt;Resources&gt; 元素中定义了 1 个 TCP 终结点资源和 2 个 HTTP 终结点资源。

HTTP 终结点由 Service Fabric 自动建立 ACL。


	<?xml version="1.0" encoding="utf-8"?>
	<ServiceManifest Name="Stateful1Pkg"
	                 Version="1.0.0"
	                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
	                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
	                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	  <ServiceTypes>
	    <!-- This is the name of your ServiceType.
	         This name must match the string used in the RegisterServiceType call in Program.cs. -->
	    <StatefulServiceType ServiceTypeName="Stateful1Type" HasPersistedState="true" />
	  </ServiceTypes>

	  <!-- Code package is your service executable. -->
	  <CodePackage Name="Code" Version="1.0.0">
	    <EntryPoint>
	      <ExeHost>
	        <Program>Stateful1.exe</Program>
	      </ExeHost>
	    </EntryPoint>
	  </CodePackage>

	  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an
	       independently updateable and versioned set of custom configuration settings for your service. -->
	  <ConfigPackage Name="Config" Version="1.0.0" />

	  <Resources>
	    <Endpoints>
	      <!-- This endpoint is used by the communication listener to obtain the port number on which to
	           listen. Note that if your service is partitioned, this port is shared with
	           replicas of different partitions that are placed in your code. -->
	      <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
	      <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
	      <Endpoint Name="ServiceEndpoint3" Protocol="https"/>

	      <!-- This endpoint is used by the replicator for replicating the state of your service.
	           This endpoint is configured through the ReplicatorSettings config section in the Settings.xml
	           file under the ConfigPackage. -->
	      <Endpoint Name="ReplicatorEndpoint" />
	    </Endpoints>
	  </Resources>
	</ServiceManifest>

## <a name="example-specifying-an-https-endpoint-for-your-service"></a>示例：指定用于你的服务的 HTTPS 终结点
HTTPS 协议提供服务器身份验证，用于对客户端-服务器通信进行加密。 若要在 Service Fabric 服务上启用 HTTPS，请在服务清单的“*资源 -> 终结点 -> 终结点*”部分中指定该协议，如前面针对终结点 *ServiceEndpoint3* 的操作所示。

> [AZURE.NOTE]
> 不能在应用程序升级期间更改服务的协议，因为这是一项破坏性更改。
> 
> 

下面是你需要为 HTTPS 设置的一个示例 ApplicationManifest。必须提供证书的指纹。EndpointRef 是对 ServiceManifest 中 EndpointResource 的引用，你为其设置 HTTPS 协议。可以添加多个 EndpointCertificate。


	<?xml version="1.0" encoding="utf-8"?>
	<ApplicationManifest ApplicationTypeName="Application1Type"
	                     ApplicationTypeVersion="1.0.0"
	                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
	                     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
	                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	  <Parameters>
	    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="2" />
	    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
	    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
	  </Parameters>
	  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
	       should match the Name and Version attributes of the ServiceManifest element defined in the
	       ServiceManifest.xml file. -->
	  <ServiceManifestImport>
	    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
	    <ConfigOverrides />
	    <Policies>
	      <EndpointBindingPolicy CertificateRef="TestCert1" EndpointRef="ServiceEndpoint3"/>
	    </Policies>
	  </ServiceManifestImport>
	  <DefaultServices>
	    <!-- The section below creates instances of service types when an instance of this
	         application type is created. You can also create one or more instances of service type by using the
	         Service Fabric PowerShell module.

	         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
	    <Service Name="Stateful1">
	      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_MinReplicaSetSize]">
	        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
	      </StatefulService>
	    </Service>
	  </DefaultServices>
	  <Certificates>
	    <EndpointCertificate Name="TestCert1" X509FindValue="FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0" X509StoreName="MY" />  
	  </Certificates>
	</ApplicationManifest>

<!--Update_Description:wording update;add anchors to sub titles-->