<properties
   pageTitle="将现有可执行文件部署到 Azure Service Fabric | Microsoft Azure"
   description="有关如何将现有应用程序打包为来宾可执行文件，以便可以在 Azure Service Fabric 群集上部署的演练"
   services="service-fabric"
   documentationCenter=".net"
   authors="clca"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/20/2016"
   wacn.date="07/04/2016"/>

可以在 Azure Service Fabric 中运行任何类型的应用程序，如 Node.js、Java 或本机应用程序。Service Fabric 术语将这些类型的应用程序称为来宾可执行文件。
来宾可执行文件由 Service Fabric 如同无状态服务一样进行处理。因此，它们会基于可用性和其他指标位于群集中的节点上。本文介绍如何使用 Visual Studio 或命令行实用工具打包来宾可执行文件并部署到 Service Fabric 群集。

## 在 Service Fabric 中运行来宾可执行文件的好处

在 Service Fabric 群集中运行来宾可执行文件可带来几个优势：

- 高可用性。在 Service Fabric 中运行的应用程序在初始状态下便高度可用。Service Fabric 可确保应用程序的一个实例始终启动并正在运行。
- 运行状况监视。在初始状态下，Service Fabric 运行状况监视会检测应用程序是否启动并正在运行，并且会在发生故障时提供诊断信息。   
- 应用程序生命周期管理。除了提供无需停机的升级，Service Fabric 还允许你在升级过程中出现问题时回滚到以前的版本。    
- 密度。可以在群集中运行多个应用程序，这样便无需使每个应用程序在自己的硬件上运行。

在本文中，我们会介绍打包来宾可执行文件并将它部署到 Service Fabric 的基本步骤。

## 应用程序和服务清单文件的快速概述

在部署来宾可执行文件过程中，了解 Service Fabric 打包和部署模型会十分有用。Service Fabric 打包部署模型主要依赖两个 XML 文件：应用程序清单和服务清单。ApplicationManifest.xml 和 ServiceManifest.xml 文件的架构定义随 Service Fabric SDK 和工具一起安装到 *C:\\Program Files\\Microsoft SDKs\\Service Fabric\\schemas\\ServiceFabricServiceModel.xsd*。

* **应用程序清单**

  应用程序清单用于描述应用程序。它会列出构成它的服务以及用于定义应如何部署服务（如实例数）的其他参数。

  在 Service Fabric 领域中，应用程序是“可升级单元”。 应用程序可以作为单个单元进行升级，其中的潜在失败（潜在回滚）由平台进行管理。平台可保证升级过程完全成功，或者，如果失败，平台不会使应用程序处于未知/不稳定状态。

* **服务清单**

  服务清单描述服务的组件。它包含数据（如服务的名称和类型（这是 Service Fabric 用于管理服务的信息），及其代码、配置和数据组件。服务清单还包含一些可以用于在部署之后配置服务的其他参数。

  我们不会介绍服务清单中可用的所有不同参数的详细信息。我们会讨论使来宾可执行文件在 Service Fabric 上运行所需的子集。

## 应用程序包文件结构
为了将应用程序部署到 Service Fabric，应用程序需要遵循预定义的目录结构。下面是该结构的一个示例。

```
|-- ApplicationPackage
	|-- code
		|-- existingapp.exe
	|-- config
		|-- Settings.xml
  |-- data    
  |-- ServiceManifest.xml
|-- ApplicationManifest.xml
```

根目录包含定义应用程序的 ApplicationManifest.xml 文件。应用程序中包含的每个服务的子目录用于包含服务需要的所有项目 — ServiceManifest.xml 以及通常以下三个目录：

- *Code*。此目录包含服务代码。
- *Config*。此目录包含一个 Settings.xml 文件（必要时还将包含其他文件），服务可以在运行时访问该文件以检索特定的配置设置。
- *Data*。这是用于存储服务可能需要的其他本地数据的其他目录。注意：Data 只应用于存储临时数据。如果需要对服务进行重新定位（例如在故障转移期间），则 Service Fabric 不会复制对数据目录所做的更改。

注意：如果不需要 `config` 和 `data` 目录，则不必创建它们。

## 打包现有应用的过程

打包来宾可执行文件时，可以选择使用 Visual Studio 项目模板，或者手动创建应用程序包。使用 Visual Studio 时，新的项目向导将为你创建应用程序包结构和清单文件。有关如何使用 Visual Studio 打包来宾可执行文件的分步指导，请参阅下文。

手动打包来宾可执行文件的过程基于以下步骤进行：

1. 创建包目录结构。
2. 添加应用程序的代码和配置文件。
3. 编辑服务清单文件。
4. 编辑应用程序清单文件。

>[AZURE.NOTE] 我们提供一个打包工具，通过它可以自动创建应用程序包。该工具当前提供预览版。可以从[此处](http://aka.ms/servicefabricpacktool)下载它。

### 创建包目录结构
你可以通过按上文所述创建目录结构来开始。

### 添加应用程序的代码和配置文件
创建了目录结构之后，可以在 code 和 config 目录下添加应用程序的代码和配置文件。还可以在 code 或 config 目录下创建其他目录或子目录。

Service Fabric 对应用程序根目录下的内容执行了 xcopy，因此除创建 code 和 settings 这两个顶级目录以外没有其他任何预定义结构可以使用。（如有需要可以选取其他名称。有关详细信息，请参阅下一节内容。）

>[AZURE.NOTE] 确保包含应用程序需要的所有文件/依赖项。Service Fabric 将复制群集中所有节点上的应用程序包的内容，将在群集中部署应用程序的服务。包应包含应用程序运行所需的全部代码。我们建议不要假定已安装依赖项。

### 编辑服务清单文件
下一步是编辑服务清单文件以包含如下信息：

- 服务类型的名称。这是 Service Fabric 用于标识服务的 ID。
- 用于启动应用程序的命令 (ExeHost)。
- 为设置/配置应用程序而需要运行的任何脚本 (SetupEntrypoint)。

下面是 `ServiceManifest.xml` 文件的一个示例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Name="NodeApp" Version="1.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="NodeApp" UseImplicitHost="true"/>
   </ServiceTypes>
   <CodePackage Name="code" Version="1.0.0.0">
      <SetupEntryPoint>
         <ExeHost>
             <Program>scripts\launchConfig.cmd</Program>
         </ExeHost>
      </SetupEntryPoint>
      <EntryPoint>
         <ExeHost>
            <Program>node.exe</Program>
            <Arguments>bin/www</Arguments>
            <WorkingFolder>CodePackage</WorkingFolder>
         </ExeHost>
      </EntryPoint>
   </CodePackage>
   <Resources>
      <Endpoints>
         <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />
      </Endpoints>
   </Resources>
</ServiceManifest>
```

我们先了解一下你需要更新的文件的不同部分：

### ServiceTypes

```xml
<ServiceTypes>
  <StatelessServiceType ServiceTypeName="NodeApp" UseImplicitHost="true" />
</ServiceTypes>
```

- 可以为 `ServiceTypeName` 选取所需的任何名称。该值在 `ApplicationManifest.xml` 文件中用于标识服务。
- 你需要指定 `UseImplicitHost="true"`。此属性告知 Service Fabric 服务基于自包含的应用，因此 Service Fabric 只需要将其作为进程启动并监视其运行状况。

### CodePackage
CodePackage 元素指定服务代码的位置（和版本）。

```xml
<CodePackage Name="Code" Version="1.0.0.0">
```

`Name` 元素用于在应用程序包中指定包含服务代码的目录的名称。`CodePackage` 还具有 `version` 属性。这可以用于指定代码的版本 — 还可能用于通过使用 Service Fabric 应用程序生命周期管理基础结构来升级服务的代码。
### SetupEntrypoint

```xml
<SetupEntryPoint>
   <ExeHost>
       <Program>scripts\launchConfig.cmd</Program>
   </ExeHost>
</SetupEntryPoint>
```
SetupEntrypoint 元素用于指定在启动服务代码之前应执行的任何可执行文件或批处理文件。它是可选元素，因此在无需初始化/设置时无需包含在内。每次重新启动服务时，会执行 SetupEntryPoint。

只有一个 SetupEntrypoint，因此如果应用程序的设置/配置需要多个脚本，则设置/配置脚本需要捆绑在单个批处理文件中。与 SetupEntryPoint 元素一样，SetupEntrypoint 可以执行任何类型的文件 — 可执行文件、批处理文件和 PowerShell cmdlet。在上面的示例中，SetupEntrypoint 基于一个批处理文件 LaunchConfig.cmd，该文件位于代码目录的 `scripts` 子目录中（假定 WorkingFolder 元素设置为 code）。

### 入口点

```xml
<EntryPoint>
  <ExeHost>
    <Program>node.exe</Program>
    <Arguments>bin/www</Arguments>
    <WorkingFolder>CodeBase</WorkingFolder>
  </ExeHost>
</EntryPoint>
```

服务清单文件中的 `Entrypoint` 元素用于指定如何启动该服务。`ExeHost` 元素指定应用于启动该服务的可执行文件（和参数）。

- `Program` 指定为启动服务而应该执行的可执行文件的名称。
- `Arguments` 指定应传递给可执行文件的参数。它可以是带有实参的形参的列表。
- `WorkingFolder` 指定要启动的进程的工作目录。你可以指定两个值：
	- `CodeBase` 指定工作目录将设置为应用程序包中的 code 目录（结构中的 `Code` 目录如下所示）
	- `CodePackage` 指定要设置为应用程序包的根目录的工作目录 (`MyServicePkg`)。
- `WorkingFolder` 用于设置正确的工作目录，以便应用程序或初始化脚本可以使用相对路径。

### 终结点


	<Endpoints>
   		<Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />
	</Endpoints>


`Endpoint` 元素指定应用程序可以侦听的终结点。在此示例中，Node.js 应用程序侦听端口 3000。

## 编辑应用程序清单文件

配置 `Servicemanifest.xml` 文件之后，你需要对 `ApplicationManifest.xml` 文件进行一些更改，以确保使用正确的服务类型和名称。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="NodeAppType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="NodeApp" ServiceManifestVersion="1.0.0.0" />
   </ServiceManifestImport>
</ApplicationManifest>
```

### ServiceManifestImport

在 `ServiceManifestImport` 元素中，你可以指定一个或多个希望包含在应用中的服务。服务使用 `ServiceManifestName` 进行引用，后者指定 `ServiceManifest.xml` 文件所在的目录的名称。

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="NodeApp" ServiceManifestVersion="1.0.0.0" />
</ServiceManifestImport>
```

### 设置日志记录
对于来宾可执行文件，能够查看控制台日志以查明应用程序和配置脚本是否显示了任何错误会非常有用。可以使用 `ConsoleRedirection` 元素在 `ServiceManifest.xml` 文件中配置控制台重定向。

```xml
<EntryPoint>
  <ExeHost>
    <Program>node.exe</Program>
    <Arguments>bin/www</Arguments>
    <WorkingFolder>CodeBase</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="5" FileMaxSizeInKb="2048"/>
  </ExeHost>
</EntryPoint>
```

* `ConsoleRedirection` 可用于将控制台输出（stdout 和 stderr）重定向至工作目录，以便可以将它们用于验证在 Service Fabric 群集中安装或执行应用程序期间没有错误。

	* `FileRetentionCount` 确定保存在工作目录中的文件的数量。例如，值 5 表示前 5 个执行的日志文件存储在工作目录中。
	* `FileMaxSizeInKb` 指定日志文件的最大大小。

日志文件保存在服务的一个工作目录中。要确定文件所在的位置，需要使用 Service Fabric 资源管理器来确定运行服务的节点以及所使用的工作目录。本文中后面部分介绍了此过程。

### 部署
最后一步是部署应用程序。下面的 PowerShell 脚本演示如何将应用程序部署到本地开发群集并启动新的 Service Fabric 服务。

```PowerShell

Connect-ServiceFabricCluster localhost:19000

Write-Host 'Copying application package...'
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath 'C:\Dev\MultipleApplications' -ImageStoreConnectionString 'file:C:\SfDevCluster\Data\ImageStoreShare' -ApplicationPackagePathInImageStore 'nodeapp'

Write-Host 'Registering application type...'
Register-ServiceFabricApplicationType -ApplicationPathInImageStore 'nodeapp'

New-ServiceFabricApplication -ApplicationName 'fabric:/nodeapp' -ApplicationTypeName 'NodeAppType' -ApplicationTypeVersion 1.0

New-ServiceFabricService -ApplicationName 'fabric:/nodeapp' -ServiceName 'fabric:/nodeapp/nodeappservice' -ServiceTypeName 'NodeApp' -Stateless -PartitionSchemeSingleton -InstanceCount 1

```
Service Fabric 服务可以采用各种“配置”进行部署。 例如，可以将其作为单个或多个实例部署，或者可以将其以这样一种方式部署：在 Service Fabric 群集的每个节点上都有一个服务实例。

`New-ServiceFabricService` cmdlet 的 `InstanceCount` 参数用于指定应在 Service Fabric 群集中启动的服务实例的数量。你可以设置 `InstanceCount` 值，具体取决于要部署的应用程序的类型。最常见的两种方案是：

* `InstanceCount = "1"`。在这种情况下，在群集上将仅部署一个服务实例。Service Fabric 的计划程序确定将在哪一个节点上部署服务。

* `InstanceCount ="-1"`。在这种情况下，将在 Service Fabric 群集中的每个节点上部署一个服务实例。最终结果将为群集中的每个节点具有一个（且只有一个）服务实例。

这是前端应用程序（不包括 REST 终结点）的有用配置，因为客户端应用程序只需“连接到”群集中的任何节点以便使用该终结点。例如，当 Service Fabric 群集的所有节点都连接到负载平衡器时，还可以使用此配置，以便客户端流量可以分布在在群集中的所有节点上运行的服务上。

### 检查正在运行的应用程序

在 Service Fabric 资源管理器中，确定服务在其中运行的节点。在此示例中，它在节点 1 上运行：

![运行服务的节点](./media/service-fabric-deploy-existing-app/nodeappinsfx.png)

如果导航到该节点并浏览到应用程序，则会看到基本节点信息（包括在磁盘上的位置）。

![磁盘上的位置](./media/service-fabric-deploy-existing-app/locationondisk2.png)

如果使用服务器资源管理器浏览到目录，则可以按如下所示找到工作目录和服务的日志文件夹。

![日志的位置](./media/service-fabric-deploy-existing-app/loglocation.png)

## 使用 Visual Studio 打包现有应用程序

Visual Studio 提供 Service Fabric 服务模板来帮助你将来宾可执行文件部署到 Service Fabric 群集。需要执行以下过程来完成发布：

>[AZURE.NOTE] 此功能需要 [SDK 版本 2.1.150](https://blogs.msdn.microsoft.com/azureservicefabric/2016/06/13/release-of-service-fabric-sdk-2-1-150-and-runtime-5-1-150/)

1. 选择“文件”->“新建项目”，然后创建新的 Service Fabric 应用程序。
2. 为“服务模板”选择“来宾可执行文件”。
3. 单击“浏览”选择包含你的可执行文件的文件夹，然后填充余下的参数以创建新服务。
  - 可将“代码包行为”设置为将文件夹中的所有内容复制到 Visual Studio 项目，如果可执行文件不会更改，则这样设置会很有用。如果预期可执行文件会更改，并且希望能够动态选择新版本，则可以改为选择文件夹的链接。
  - *Program*：选择为了启动服务而应该执行的可执行文件。
  - *Arguments*：指定应传递给可执行文件的参数。它可以是带有实参的形参的列表。
  - *WorkingFolder*：选择要启动的进程所在的工作目录。你可以指定两个值：
  	- *CodeBase*：指定工作目录将设置为应用程序包中的 code 目录（结构中的 `Code` 目录如下所示）
    - *CodePackage*：指定要设置为应用程序包的根目录的工作目录 (`MyServicePkg`)。
4. 为服务命名，然后单击“确定”。
5. 如果你的服务需要使用终结点进行通信，现在你可以在 ServiceManifest.xml 文件中添加协议、端口和类型，例如：```<Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />```。
6. 现在，可以通过在 Visual Studio 中调试解决方案，尝试针对本地群集执行打包和发布操作。准备就绪后，可以将应用程序发布到远程群集，或者将解决方案签入源代码管理。

>[AZURE.NOTE] 在 Visual Studio 中创建应用程序项目时，可以使用链接的文件夹。这将会从项目内部链接到源位置，使你能够在来宾可执行文件的源目标中对它进行更新，并在生成时使这些更新成为应用程序包的一部分。

## 后续步骤
在本文中，我们了解了如何打包来宾可执行文件并将它部署到 Service Fabric。作为下一步，可以查看本主题的其他内容。

- [在 GitHub 上打包和部署来宾可执行文件的示例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/master/GuestExe/SimpleApplication)，包括打包工具的预发行版本的链接
- [部署多个来宾可执行文件](/documentation/articles/service-fabric-deploy-multiple-apps/)
- [使用 Visual Studio 创建第一个 Service Fabric 应用程序](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)

<!---HONumber=Mooncake_0627_2016-->