<properties
    pageTitle="Service Fabric 和在 Linux 中部署容器 | Azure"
    description="介绍 Service Fabric，以及如何使用 Docker 容器部署微服务应用程序。本文介绍 Service Fabric 为容器提供的功能，以及如何在群集中部署 Docker 容器映像"
    services="service-fabric"
    documentationcenter=".net"
    author="msfussell"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="4ba99103-6064-429d-ba17-82861b6ddb11"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="2/16/2017"
    wacn.date="03/03/2017"
    ms.author="msfussell" />  


# 将 Docker 容器部署到 Service Fabric


本文介绍在 Linux 上的 Docker 容器中构建容器化服务的步骤。

Service Fabric 提供多种容器功能，可帮助你构建由容器化的微服务组成的应用程序。这些服务称为容器化服务。

功能包括：

* 容器映像部署和激活
* 资源调控
* 存储库身份验证
* 容器端口到主机端口的映射
* 容器到容器的发现和通信
* 能够配置和设置环境变量

## 使用 yeoman 打包 docker 容器
在 Linux 上打包容器时，可以选择使用 yeoman 模板，或者[手动创建应用程序包](#manually)。

Service Fabric 应用程序可以包含一个或多个容器，每个容器都在提供应用程序功能时具有特定角色。适用于 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 生成器，通过此生成器，可以轻松创建应用程序和添加容器映像。让我们使用 Yeoman 创建具有单个 Docker 容器（名为 *SimpleContainerApp*）的新应用程序。可通过编辑生成的清单文件随后添加更多服务。

## 创建应用程序
1. 在终端中键入 **yo azuresfguest**。
2. 对于框架，请选择“容器”。
3. 为应用程序命名 - 例如 SimpleContainerApp
4. 从 DockerHub 存储库提供容器映像的 URL。映像参数采用的格式为 [repo]/[image name]

![适用于容器的 Service Fabric Yeoman 生成器][sf-yeoman]  


## 部署应用程序
构建应用程序后，可以使用 Azure CLI 将它部署到本地群集。

1. 连接到本地 Service Fabric 群集。

    
    	azure servicefabric cluster connect
    
2. 使用模板中提供的安装脚本，将应用程序包复制到群集的映像存储、注册应用程序类型，然后创建应用程序的实例。

    
    	./install.sh
    
3. 打开浏览器并导航到 Service Fabric Explorer (http://localhost:19080/Explorer)（如果在 Mac OS X 上使用 Vagrant，请将 localhost 替换为 VM 的专用 IP）。
4. 展开“应用程序”节点。可以看到，应用程序类型现在有一个条目，另一个条目对应于该类型的第一个实例。
5. 使用模板中提供的卸载脚本删除应用程序实例并取消注册应用程序类型。

    
    	./uninstall.sh
有关示例应用程序，请[查看 GitHub 上的 Service Fabric 容器代码示例](https://github.com/Azure-Samples/service-fabric-dotnet-containers)

## 将更多服务添加到现有应用程序

若要将另一个容器服务添加到已使用 `yo` 创建的应用程序，请执行以下步骤：
1. 将目录更改为现有应用程序的根目录。例如 `cd ~/YeomanSamples/MyApplication`（如果 `MyApplication` 是 Yeoman 创建的应用程序）。
2. 运行 `yo azuresfguest:AddService`

<a id="manually"></a>

## 手动打包和部署容器映像
手动打包容器化服务的过程基于以下步骤进行：

1. 将容器发布到存储库。
2. 创建包目录结构。
3. 编辑服务清单文件。
4. 编辑应用程序清单文件。

## 部署和激活容器映像
在 Service Fabric [应用程序模型](/documentation/articles/service-fabric-application-model/)中，容器代表多个服务副本所在的应用程序主机。若要部署和激活某个容器，请在服务清单的 `ContainerHost` 元素中输入容器映像的名称。

在服务清单中，为入口点添加 `ContainerHost`。然后将 `ImageName` 设置为容器存储库和映像的名称。下面的不完整清单演示如何从名为 `myrepo` 的存储库部署名为 `myimage:v1` 的容器的示例：


	    <CodePackage Name="Code" Version="1.0">
	        <EntryPoint>
	          <ContainerHost>
	            <ImageName>myrepo/myimage:v1</ImageName>
	            <Commands></Commands>
	          </ContainerHost>
	        </EntryPoint>
	    </CodePackage>


可以通过指定可选的 `Commands` 元素以及一组要在容器中运行的命令（以逗号分隔）提供输入命令。

## 了解资源调控
资源调控是一项容器功能，限制容器可在主机上使用的资源。在应用程序清单中指定的 `ResourceGovernancePolicy` 可用于声明服务代码包的资源限制。可以为以下资源设置资源限制：

* 内存
* MemorySwap
* CpuShares（CPU 相对权重）
* MemoryReservationInMB
* BlkioWeight（BlockIO 相对权重）

> [AZURE.NOTE]
>在将来的版本中，支持指定特定的块 IO 限制，例如 IOP、读/写 BPS 和其他限制。
> 
> 


	    <ServiceManifestImport>
	        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	        <Policies>
	            <ResourceGovernancePolicy CodePackageRef="FrontendService.Code" CpuShares="500"
	            MemoryInMB="1024" MemorySwapInMB="4084" MemoryReservationInMB="1024" />
	        </Policies>
	    </ServiceManifestImport>


## 对存储库进行身份验证
若要下载容器，可能必须向容器存储库提供登录凭据。应用程序清单中指定的登录凭据用于指定从映像存储库下载容器映像所需的登录信息或 SSH 密钥。以下示例显示名为 *TestUser* 的帐户以及明文密码（不推荐）：


	    <ServiceManifestImport>
	        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	        <Policies>
	            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
	                <RepositoryCredentials AccountName="TestUser" Password="12345" PasswordEncrypted="false"/>
	            </ContainerHostPolicies>
	        </Policies>
	    </ServiceManifestImport>


建议使用部署到计算机的证书加密密码。

以下示例显示名为 *TestUser* 的帐户，其中密码已使用名为 *MyCert* 的证书加密。可以使用 `Invoke-ServiceFabricEncryptText` PowerShell 命令创建密码的机密加密文本。有关详细信息，请参阅[管理 Service Fabric 应用程序中的机密](/documentation/articles/service-fabric-application-secret-management/)一文。

用于解密密码的证书私钥必须使用带外方法部署到本地计算机。（在 Azure 中，此方法为 Azure Resource Manager。） 然后，当 Service Fabric 将服务包部署到计算机时，其可解密机密。通过使用密钥和帐户名称，它可以对容器存储库进行身份验证。


	    <ServiceManifestImport>
	        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	        <Policies>
	            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
	                <RepositoryCredentials AccountName="TestUser" Password="[Put encrypted password here using MyCert certificate ]" PasswordEncrypted="true"/>
	            </ContainerHostPolicies>
	        </Policies>
	    </ServiceManifestImport>


## 配置容器端口到主机端口的映射
可以在应用程序清单中指定 `PortBinding`，配置用来与容器通信的主机端口。端口绑定将在容器内侦听的服务的端口映射到主机上的端口。


	    <ServiceManifestImport>
	        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	        <Policies>
	            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
	                <PortBinding ContainerPort="8905"/>
	            </ContainerHostPolicies>
	        </Policies>
	    </ServiceManifestImport>


## 配置容器到容器的发现和通信
可以使用 `PortBinding` 策略将容器端口映射到服务清单中的 `Endpoint`，如以下示例中所示。终结点 `Endpoint1` 可指定固定端口（例如，端口 80）。还可不指定任何端口，在此情况下，将从群集的应用程序端口范围内选择一个随机端口。

如果使用来宾容器的服务清单中的 `Endpoint` 标记指定终结点，Service Fabric 可以自动将此终结点发布到命名服务。因此，在群集中运行的其他服务可以通过用于解析的 REST 查询来发现此容器。


	    <ServiceManifestImport>
	        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	        <Policies>
	            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
	                <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
	            </ContainerHostPolicies>
	        </Policies>
	    </ServiceManifestImport>


在命名服务中注册后，可以轻松地在容器中的代码内使用[反向代理](/documentation/articles/service-fabric-reverseproxy/)来执行容器到容器的通信。通过提供反向代理 http 侦听端口和要作为环境变量与其通信的服务的名称来执行通信。有关详细信息，请参阅下一部分。

## 配置和设置环境变量
对于部署在容器中的服务，或者部署为进程/来宾可执行文件的服务，可以在服务清单中为每个代码包指定环境变量。可以在应用程序清单中明确重写这些环境变量的值，或者在部署期间将其指定为应用程序参数。

以下服务清单 XML 代码片段演示如何指定代码包的环境变量：


	    <ServiceManifest Name="FrontendServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	        <Description>a guest executable service in a container</Description>
	        <ServiceTypes>
	            <StatelessServiceType ServiceTypeName="StatelessFrontendService"  UseImplicitHost="true"/>
	        </ServiceTypes>
	        <CodePackage Name="FrontendService.Code" Version="1.0">
	            <EntryPoint>
	            <ContainerHost>
	                <ImageName>myrepo/myimage:v1</ImageName>
	                <Commands></Commands>
	            </ContainerHost>
	            </EntryPoint>
	            <EnvironmentVariables>
	                <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
	                <EnvironmentVariable Name="BackendServiceName" Value=""/>
	            </EnvironmentVariables>
	        </CodePackage>
	    </ServiceManifest>


可在应用程序清单级别重写这些环境变量：


	    <ServiceManifestImport>
	        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	        <EnvironmentOverrides CodePackageRef="FrontendService.Code">
	            <EnvironmentVariable Name="BackendServiceName" Value="[BackendSvc]"/>
	            <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
	        </EnvironmentOverrides>
	    </ServiceManifestImport>


在前面的示例中，为 `HttpGateway` 环境变量指定了显式值 (19000)，通过 `[BackendSvc]` 应用程序参数设置 `BackendServiceName` 参数的值。通过这些设置，可在部署应用程序时指定 `BackendServiceName` 值的值，无需在清单中使用固定值。

## 应用程序清单和服务清单的完整示例

下面是应用程序清单示例：


	    <ApplicationManifest ApplicationTypeName="SimpleContainerApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	        <Description>A simple service container application</Description>
	        <Parameters>
	            <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
	            <Parameter Name="BackEndSvcName" DefaultValue="bkend"></Parameter>
	        </Parameters>
	        <ServiceManifestImport>
	            <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
	            <EnvironmentOverrides CodePackageRef="FrontendService.Code">
	                <EnvironmentVariable Name="BackendServiceName" Value="[BackendSvcName]"/>
	                <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
	            </EnvironmentOverrides>
	            <Policies>
	                <ResourceGovernancePolicy CodePackageRef="Code" CpuShares="500" MemoryInMB="1024" MemorySwapInMB="4084" MemoryReservationInMB="1024" />
	                <ContainerHostPolicies CodePackageRef="FrontendService.Code">
	                    <RepositoryCredentials AccountName="username" Password="****" PasswordEncrypted="true"/>
	                    <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
	                </ContainerHostPolicies>
	            </Policies>
	        </ServiceManifestImport>
	    </ApplicationManifest>


下面是服务清单示例（在前面的应用程序清单中指定）：


	    <ServiceManifest Name="FrontendServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	        <Description> A service that implements a stateless front end in a container</Description>
	        <ServiceTypes>
	            <StatelessServiceType ServiceTypeName="StatelessFrontendService"  UseImplicitHost="true"/>
	        </ServiceTypes>
	        <CodePackage Name="FrontendService.Code" Version="1.0">
	            <EntryPoint>
	            <ContainerHost>
	                <ImageName>myrepo/myimage:v1</ImageName>
	                <Commands></Commands>
	            </ContainerHost>
	            </EntryPoint>
	            <EnvironmentVariables>
	                <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
	                <EnvironmentVariable Name="BackendServiceName" Value=""/>
	            </EnvironmentVariables>
	        </CodePackage>
	        <ConfigPackage Name="FrontendService.Config" Version="1.0" />
	        <DataPackage Name="FrontendService.Data" Version="1.0" />
	        <Resources>
	            <Endpoints>
	                <Endpoint Name="Endpoint1" UriScheme="http" Port="80" Protocol="http"/>
	            </Endpoints>
	        </Resources>
	    </ServiceManifest>


## 后续步骤
现已部署容器化服务，请阅读 [Service Fabric 应用程序生命周期](/documentation/articles/service-fabric-application-lifecycle/)，了解如何管理其生命周期。

* [Service Fabric 和容器概述](/documentation/articles/service-fabric-containers-overview/)
* [使用 Azure CLI 来与 Service Fabric 群集交互](/documentation/articles/service-fabric-azure-cli/)

<!-- Images -->

[sf-yeoman]: ./media/service-fabric-deploy-container-linux/sf-container-yeoman.png

<!---HONumber=Mooncake_0227_2017-->