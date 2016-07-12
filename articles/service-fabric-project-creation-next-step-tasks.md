<properties
   pageTitle="Service Fabric 项目创建后续步骤 | Azure"
   description="本文包含针对 Service Fabric 执行的一组核心开发任务的链接"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/27/2016"
   wacn.date="07/07/2016"/>

# Service Fabric 应用程序和后续步骤
已创建你的 Azure Service Fabric 应用程序。本文说明项目的构成以及有可能要执行的一些后续步骤。

## 你的应用程序
每个新应用程序都包含一个应用程序项目。根据选择的服务类型，可能有一个或两个附加的项目。

### 应用程序项目
应用程序项目包括：

- 对构成应用程序的服务的引用集。

- 两个发布配置文件（本地和云），可用于维护首选项以适应不同的环境 - 例如，与群集终结点相关的首选项，以及是否按默认执行升级部署。

- 两个应用程序参数文件（本地和云），可用于维护环境特定的应用程序配置，例如，要为服务创建的分区数目。

- 可使用部署脚本从命令行部署应用程序，或者通过自动持续集成和部署管道来部署应用程序。

- 用于描述应用程序的应用程序清单。可以在 ApplicationPackageRoot 文件夹下查找清单。

### 无状态服务
当你添加新的无状态服务时，Visual Studio 将在你的解决方案中添加一个服务项目，其中包含继承自 `StatelessService` 的类型。该服务将递增计数器中的本地变量。

### 有状态服务
当你添加新的有状态服务时，Visual Studio 将在你的解决方案中添加一个服务项目，其中包含继承自 `StatefulService` 的类型。该服务将在其 `RunAsync` 方法中递增计数器，并将结果存储在 `ReliableDictionary` 中。

### 执行组件服务
当你添加新的可靠执行组件时，Visual Studio 会将两个项目添加到你的解决方案：执行组件项目和接口项目。

执行组件项目提供所需的方法用于设置和获取可靠保存在执行组件状态中的计数器的值。接口项目提供其他服务可用来调用执行组件的接口。

### 无状态 Web API
无状态 Web API 项目提供一个基本 Web 服务，可用于向外部客户端打开你的应用程序。有关如何构建该项目的信息，请参阅 [Service Fabric Web API services with OWIN self-hosting（Service Fabric Web API 服务与 OWIN 自托管）](/documentation/articles/service-fabric-reliable-services-communication-webapi/)。

## 后续步骤
### 创建 Azure 群集
Service Fabric SDK 提供一个用于开发和测试的本地群集。若要在 Azure 中创建群集，请参阅 [使用 Azure Resource Manager 模板设置 Service Fabric 群集][create-cluster-in-portal]。
<!--
### 尝试使用合作群集免费部署到 Azure

如果你要尝试在 Azure 中部署和管理应用程序且不设置自己的群集，可以使用免费的[合作群集服务](http://tryazureservicefabric.chinaeast.chinacloudapp.cn/)。-->

### 将应用程序发布到 Azure
可以直接从 Visual Studio 将应用程序发布到 Azure 群集。若要了解操作方法，请参阅 [Publishing your application to Azure（将应用程序发布到 Azure）][publish-app-to-azure]。

### 使用 Service Fabric 资源管理器可视化群集
Service Fabric 资源管理器提供一种用于可视化群集（包括已部署的应用程序和物理布局）的简易方法。有关详细信息，请参阅 [Visualizing your cluster by using Service Fabric Explorer（使用 Service Fabric 资源管理器可视化群集）][visualize-with-sfx]。

### 对服务进行版本控制和升级
Service Fabric 支持单独对应用程序中的独立服务进行版本控制和升级。若要了解详细信息，请参阅 [Versioning and upgrading your services（对服务进行版本控制和升级）][app-upgrade-tutorial]。

### 配置与 Visual Studio Team Services 的持续集成
若要了解如何为 Service Fabric 应用程序设置持续集成过程，请参阅 [Configure continuous integration with Visual Studio Team Services（配置与 Visual Studio Team Services 的持续集成）][ci-with-vso]。



<!-- Links -->
[add-web-frontend]: /documentation/articles/service-fabric-add-a-web-frontend/
[create-cluster-in-portal]: /documentation/articles/service-fabric-cluster-creation-via-arm/
[publish-app-to-azure]: /documentation/articles/service-fabric-publish-app-remote-cluster/
[visualize-with-sfx]: /documentation/articles/service-fabric-visualizing-your-cluster/
[ci-with-vso]: /documentation/articles/service-fabric-set-up-continuous-integration/
[reliable-services-webapi]: /documentation/articles/service-fabric-reliable-services-communication-webapi/
[app-upgrade-tutorial]: /documentation/articles/service-fabric-application-upgrade-tutorial/

<!---HONumber=Mooncake_0425_2016-->