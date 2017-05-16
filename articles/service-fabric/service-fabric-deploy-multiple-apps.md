<properties
    pageTitle="部署使用 MongoDB 的 Node.js 应用程序 | Azure"
    description="演练如何打包多个来宾可执行文件以部署到 Azure Service Fabric 群集"
    services="service-fabric"
    documentationcenter=".net"
    author="msfussell"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="b76bb756-c1ba-49f9-9666-e9807cf8f92f"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="04/07/2017"
    wacn.date="05/15/2017"
    ms.author="msfussell;mikhegn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="51ed70036723e996aa6a132149c3fb891c918d93"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="deploy-multiple-guest-executables"></a>部署多个来宾可执行文件
本文介绍如何打包多个来宾可执行文件并部署到 Azure Service Fabric。 若要生成并部署单个 Service Fabric 包，请参阅如何[将来宾可执行文件部署到 Service Fabric](/documentation/articles/service-fabric-deploy-existing-app/)。

虽然本演练演示如何部署将 MongoDB 用作数据存储并具有 Node.js 前端的应用程序，但是你可以将这些步骤套用于任何与另一个应用程序具有依赖关系的应用程序。   

可使用 Visual Studio 生成包含多个来宾可执行文件的应用程序包。 请参阅[使用 Visual Studio 打包现有应用程序](/documentation/articles/service-fabric-deploy-existing-app/#use-visual-studio-to-package-an-existing-executable)。 添加第一个来宾可执行文件后，右键单击应用程序项目，然后依次选择**“添加”->“新建 Service Fabric 服务”**，将第二个来宾可执行文件项目添加到解决方案中。 请注意，如果选择在 Visual Studio 项目中链接源，在生成的 Visual Studio 解决方案中可确保应用程序包与源中的更改保持同步。 

## <a name="samples"></a>示例
* [打包和部署来宾可执行文件的示例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [使用 REST 通过命名服务进行通信的两种来宾可执行文件（C# 和 nodejs）示例](https://github.com/Azure-Samples/service-fabric-dotnet-containers)

## <a name="manually-package-the-multiple-guest-executable-application"></a>手动打包多个来宾可执行文件应用程序
或者可以手动打包来宾可执行文件。 对于手动打包，本文使用 Service Fabric 打包工具，它位于 [http://aka.ms/servicefabricpacktool](http://aka.ms/servicefabricpacktool)。

### <a name="packaging-the-nodejs-application"></a>打包 Node.js 应用程序
本文假设 Service Fabric 群集中的节点上未安装 Node.js。 因此，你需要在打包之前，先将 Node.exe 添加到节点应用程序的根目录中。 Node.js 应用程序（使用 Express Web 框架和 Jade 模板引擎）的目录结构看起来应该与以下类似：

	|-- NodeApplication
		|-- bin
	        |-- www
		|-- node_modules
	        |-- .bin
	        |-- express
	        |-- jade
	        |-- etc.
		|-- public
	        |-- images
	        |-- etc.
		|-- routes
	        |-- index.js
	        |-- users.js
	    |-- views
	        |-- index.jade
	        |-- etc.
	    |-- app.js
	    |-- package.json
	    |-- node.exe


在下一个步骤中，你将为 Node.js 应用程序创建应用程序包。以下代码会创建包含 Node.js 应用程序的 Service Fabric 应用程序包。


	.\ServiceFabricAppPackageUtil.exe /source:'[yourdirectory]\MyNodeApplication' /target:'[yourtargetdirectory] /appname:NodeService /exe:'node.exe' /ma:'bin/www' /AppType:NodeAppType

下面描述了所使用的参数：

* **/source** 指向应打包的应用程序的目录。
* **/target** 定义应在其中创建包的目录。 此目录必须与源目录不同。
* **/appname** 定义现有应用程序的应用程序名称。 请务必了解，这会转换成清单中的服务名称，而不是转换成 Service Fabric 应用程序名称。
* **/exe** 定义 Service Fabric 应启动的可执行文件（在此示例中，为 `node.exe`）。
* **/ma** 定义要用于启动可执行文件的参数。 由于未安装 Node.js，因此 Service Fabric 需要通过执行 `node.exe bin/www` 来启动 Node.js Web 服务器。  `/ma:'bin/www'` 指示打包工具使用 `bin/ma` 作为 node.exe 的参数。
* **/AppType** 定义 Service Fabric 应用程序类型名称。


如果浏览到 /target 参数中指定的目录，则可以看到工具已创建完全正常运行的 Service Fabric 包，如下所示：


	|--[yourtargetdirectory]
	    |-- NodeApplication
	        |-- C
			      |-- bin
	              |-- data
	              |-- node_modules
	              |-- public
	              |-- routes
	              |-- views
	              |-- app.js
	              |-- package.json
	              |-- node.exe
	        |-- config
			      |--Settings.xml
		    |-- ServiceManifest.xml
	    |-- ApplicationManifest.xml

所生成的 ServiceManifest.xml 现在有一个描述应该如何启动 Node.js Web 服务器的部分，如以下代码片段所示：

	<CodePackage Name="C" Version="1.0">
	    <EntryPoint>
	        <ExeHost>
	            <Program>node.exe</Program>
	            <Arguments>'bin/www'</Arguments>
	            <WorkingFolder>CodePackage</WorkingFolder>
	        </ExeHost>
	    </EntryPoint>
	</CodePackage>

在此示例中，Node.js Web 服务器会侦听端口 3000，所以你需要更新 ServiceManifest.xml 文件中的终结点信息，如下所示。


	<Resources>
      	<Endpoints>
     		<Endpoint Name="NodeServiceEndpoint" Protocol="http" Port="3000" Type="Input" />
      	</Endpoints>
	</Resources>

### <a name="packaging-the-mongodb-application"></a>打包 MongoDB 应用程序
既然已打包 Node.js 应用程序，你可以继续打包 MongoDB。 如前文所述，你现在进行的步骤并非特定于 Node.js 和 MongoDB 的步骤。 事实上，它们适用于所有要打包在一起以作为一个 Service Fabric 应用程序的应用程序。  

为了打包 MongoDB，你希望确保打包 Mongod.exe 和 Mongo.exe。这两个二进制文件都位于 MongoDB 安装目录的 `bin` 目录中。目录结构类似于下面的结构。


	|-- MongoDB
		|-- bin
        	|-- mongod.exe
        	|-- mongo.exe
        	|-- anybinary.exe

Service Fabric 需要使用类似于下面的命令来启动 MongoDB，因此打包 MongoDB 时，需要使用 `/ma` 参数。


	mongod.exe --dbpath [path to data]

> [AZURE.NOTE]
> 如果你将 MongoDB 数据目录放在节点的本地目录中，当节点发生故障时，将不会保留数据。 你应该使用持久存储或实现 MongoDB 副本集以防止数据丢失。  
>
>

在 PowerShell 或命令外壳中，我们会使用下列参数运行打包工具：


	.\ServiceFabricAppPackageUtil.exe /source: [yourdirectory]\MongoDB' /target:'[yourtargetdirectory]' /appname:MongoDB /exe:'bin\mongod.exe' /ma:'--dbpath [path to data]' /AppType:NodeAppType


为了将 MongoDB 添加到你的 Service Fabric 应用程序包，需要确保 /target 参数指向已经包含应用程序清单及 Node.js 应用程序的同一个目录。此外，还需要确保你使用的是相同的 ApplicationType 名称。

让我们浏览到该目录并检查已创建的工具。


	|--[yourtargetdirectory]
    	|-- MyNodeApplication
    	|-- MongoDB
        	|-- C
            	|--bin
                	|-- mongod.exe
                	|-- mongo.exe
                	|-- etc.
        	|-- config
		    	|--Settings.xml
	    	|-- ServiceManifest.xml
    	|-- ApplicationManifest.xml

如你所见，工具已将新文件夹“MongoDB”添加到包含 MongoDB 二进制文件的目录中。如果打开 `ApplicationManifest.xml` 文件，可以看到包现在包含 Node.js 应用程序和 MongoDB。以下代码会显示应用程序清单的内容。


	<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyNodeApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   	<ServiceManifestImport>
      	<ServiceManifestRef ServiceManifestName="MongoDB" ServiceManifestVersion="1.0" />
   	</ServiceManifestImport>
   	<ServiceManifestImport>
      	<ServiceManifestRef ServiceManifestName="NodeService" ServiceManifestVersion="1.0" />
   	</ServiceManifestImport>
   	<DefaultServices>
      	<Service Name="MongoDBService">
         	<StatelessService ServiceTypeName="MongoDB">
            	<SingletonPartition />
         	</StatelessService>
      	</Service>
      	<Service Name="NodeServiceService">
         	<StatelessService ServiceTypeName="NodeService">
            	<SingletonPartition />
         	</StatelessService>
      	</Service>
   	</DefaultServices>
	</ApplicationManifest>  

### <a name="publishing-the-application"></a>发布应用程序
最后一个步骤是要使用以下 PowerShell 脚本，将应用程序发布到本地 Service Fabric 群集：


	Connect-ServiceFabricCluster localhost:19000

	Write-Host 'Copying application package...'
	Copy-ServiceFabricApplicationPackage -ApplicationPackagePath '[yourtargetdirectory]' -ImageStoreConnectionString 'file:C:\SfDevCluster\Data\ImageStoreShare' -ApplicationPackagePathInImageStore 'NodeAppType'

	Write-Host 'Registering application type...'
	Register-ServiceFabricApplicationType -ApplicationPathInImageStore 'NodeAppType'

	New-ServiceFabricApplication -ApplicationName 'fabric:/NodeApp' -ApplicationTypeName 'NodeAppType' -ApplicationTypeVersion 1.0  

将应用程序成功发布到本地群集之后，你便可以在我们在 Node.js 应用程序的服务清单中输入的端口（例如 http://localhost:3000）上访问 Node.js 应用程序。

在本教程中，你学习了如何轻松地将两个现有应用程序打包成一个 Service Fabric 应用程序。你也了解了如何将其部署到 Service Fabric，以便让它能够从一些 Service Fabric 功能（例如高可用性和运行状况系统集成）中获益。


## <a name="adding-more-guest-executables-to-an-existing-application-using-yeoman-on-linux"></a>在 Linux 上使用 Yeoman 将更多来宾可执行文件添加到现有应用程序

若要将另一个服务添加到使用 `yo`创建的应用程序，请执行以下步骤： 
1. 将目录更改为现有应用程序的根目录。  例如 `cd ~/YeomanSamples/MyApplication`（如果 `MyApplication` 是 Yeoman 创建的应用程序）。
2. 运行 `yo azuresfguest:AddService` 并提供必要的详细信息。

## <a name="next-steps"></a>后续步骤
* 若要了解如何部署容器，请参阅 [Service Fabric 和容器概述](/documentation/articles/service-fabric-containers-overview/)
* [打包和部署来宾可执行文件的示例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [使用 REST 通过命名服务进行通信的两种来宾可执行文件（C# 和 nodejs）示例](https://github.com/Azure-Samples/service-fabric-dotnet-containers)
<!--Update_Description:linux support for gest executables;add anchors to subtitles-->