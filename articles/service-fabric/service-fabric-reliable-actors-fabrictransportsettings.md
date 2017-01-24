<properties
    pageTitle="Azure Service Fabric Reliable Actors FabricTransport 配置概述 | Azure"
    description="了解如何配置 Azure Service Fabric 执行组件通信设置。"
    services="Service-Fabric"
    documentationcenter=".net"
    author="suchiagicha"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="dbed72f4-dda5-4287-bd56-da492710cd96"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="11/22/2016"
    wacn.date="01/20/2017"
    ms.author="suchia" />  


# 配置 Reliable Actors 的 FabricTransport 设置

下面是用户可以配置 [FabrictTansportSettings](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.services.communication.fabrictransport.common.fabrictransportsettings) 的设置列表

你可以通过以下方式修改 FabricTransport 的默认配置。

1.  使用程序集属性 - [FabricTransportActorRemotingProvider](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.actors.remoting.fabrictransport.fabrictransportactorremotingproviderattribute?redirectedfrom=MSDN#microsoft_servicefabric_actors_remoting_fabrictransport_fabrictransportactorremotingproviderattribute)。

	需要将此属性应用于执行组件客户端和执行组件服务程序集。下面的示例演示如何更改 FabricTransport OperationTimeout 设置的默认值。
 
		using Microsoft.ServiceFabric.Actors.Remoting.FabricTransport;
		[assembly:FabricTransportActorRemotingProvider(OperationTimeoutInSeconds = 600)]

	第二个示例更改了 FabricTransport MaxMessageSize 和 OperationTimeoutInSeconds 的默认值

	    using Microsoft.ServiceFabric.Actors.Remoting.FabricTransport;
	    [assembly:FabricTransportActorRemotingProvider(OperationTimeoutInSeconds = 600,MaxMessageSize = 134217728)]


2. 使用[配置包](/documentation/articles/service-fabric-application-model/)：

	* 配置执行组件服务的 FabricTransport 设置
	
		在 settings.xml 文件中添加 TransportSettings 部分。

    * SectionName：默认情况下，执行组件代码以“&lt;ActorName&gt;TransportSettings”查找 SectionName。 如果未找到，则以“TransportSettings”查找 SectionName。
			
			<Section Name="MyActorServiceTransportSettings">
	       		<Parameter Name="MaxMessageSize" Value="10000000" />
	       		<Parameter Name="OperationTimeoutInSeconds" Value="300" />
	       		<Parameter Name="SecurityCredentialsType" Value="X509" />
	       		<Parameter Name="CertificateFindType" Value="FindByThumbprint" />
	       		<Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
	       		<Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
	       		<Parameter Name="CertificateStoreName" Value="My" />
	       		<Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
	       		<Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
	   		</Section>  



  * 配置执行组件客户端程序集的 FabricTransport 设置
  
  	如果客户端不是作为服务的一部分运行，可以在客户端 exe 所在的同一位置中创建“&lt;客户端 Exe 名称&gt;.settings.xml”xml 文件。然后，在该文件中添加 TransportSettings 部分。SectionName 应为“TransportSettings”。
	  
		<?xml version="1.0" encoding="utf-8"?>
	  	<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
	    	<Section Name="TransportSettings">
	      		<Parameter Name="SecurityCredentialsType" Value="X509" />
	       		<Parameter Name="OperationTimeoutInSeconds" Value="300" />
	      		<Parameter Name="CertificateFindType" Value="FindByThumbprint" />
	      		<Parameter Name="CertificateFindValue" Value="78 12 20 5a 39 d2 23 76 da a0 37 f0 5a ed e3 60 1a 7e 64 bf" />
	       		<Parameter Name="OperationTimeoutInSeconds" Value="300" />
	      		<Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
	      		<Parameter Name="CertificateStoreName" Value="My" />
	      		<Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
	      		<Parameter Name="CertificateRemoteCommonNames" Value="WinFabric-Test-SAN1-Alice" />
	    	</Section>
	  	</Settings>
 

<!---HONumber=Mooncake_0116_2017-->