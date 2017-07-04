<properties
    pageTitle="在 Linux 上使用 C# 创建第一个 Azure 微服务应用 | Azure"
    description="使用 C 创建和部署 Service Fabric 应用程序#"
    services="service-fabric"
    documentationcenter="csharp"
    author="mani-ramaswamy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="5a96d21d-fa4a-4dc2-abe8-a830a3482fb1"
    ms.service="service-fabric"
    ms.devlang="csharp"
    ms.topic="hero-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/02/2017"
    wacn.date="04/24/2017"
    ms.author="subramar"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="e593d0d5ba23e7879acabfd8eaad5c4fa84ebf7d"
    ms.lasthandoff="04/14/2017" />

# <a name="create-your-first-azure-service-fabric-application"></a>创建第一个 Azure Service Fabric 应用程序
> [AZURE.SELECTOR]
- [C# - Windows](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)
- [Java - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
- [C# - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-csharp/)

Service Fabric 提供用于在 Linux 上使用 .NET Core 和 Java 构建服务的 SDK。 本教程探讨如何创建适用于 Linux 的应用程序以及使用 C# (.NET Core) 构建服务。

## <a name="prerequisites"></a>先决条件
开始之前，请确保已[设置 Linux 开发环境](/documentation/articles/service-fabric-get-started-linux/)。 如果使用的是 Mac OS X，则可以[使用 Vagrant 在虚拟机中设置 Linux 单机环境](/documentation/articles/service-fabric-get-started-mac/)。

## <a name="create-the-application"></a>创建应用程序
Service Fabric 应用程序可以包含一个或多个服务，每个服务都在提供应用程序功能时具有特定角色。 适用于 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 生成器，使用它可以轻松创建第一个服务并在以后添加更多服务。 让我们使用 Yeoman 创建包含单个服务的应用程序。

1. 在终端中，键入以下命令开始构建基架： `yo azuresfcsharp`
2. 为应用程序命名。
3. 选择第一个服务的类型并为其命名。 对于本教程，我们会选择“可靠角色服务”。
   
  ![适用于 C# 的 Service Fabric Yeoman 生成器][sf-yeoman]  

> [AZURE.NOTE]
> 有关选项的详细信息，请参阅 [Service Fabric 编程模型概述](/documentation/articles/service-fabric-choose-framework/)。
> 
> 

## <a name="build-the-application"></a>构建应用程序
Service Fabric Yeoman 模板包含构建脚本，可用于从终端构建应用程序（在导航到应用程序文件夹后）。

     cd myapp 
     ./build.sh 

## <a name="deploy-the-application"></a>部署应用程序
构建应用程序后，可以使用 Azure CLI 将它部署到本地群集。

1. 连接到本地 Service Fabric 群集。

        azure servicefabric cluster connect

2. 使用模板中提供的安装脚本可将应用程序包复制到群集的映像存储、注册应用程序类型和创建应用程序的实例。

        ./install.sh

3. 打开浏览器并导航到 http://localhost:19080/Explorer 的 Service Fabric Explorer（如果在 Mac OS X 上使用 Vagrant，则使用 VM 的专用 IP 替换 localhost）。
4. 展开应用程序节点，注意现在有一个条目是用于你的应用程序类型，另一个条目用于该类型的第一个实例。

## <a name="start-the-test-client-and-perform-a-failover"></a>启动测试客户端并执行故障转移
执行组件项目没有任何属于自己的项。 它们需要其他服务或客户端发送消息给它们。 执行组件模板包含简单的测试脚本，可用于与执行组件服务交互。

1. 使用监视实用工具运行脚本，查看执行组件服务的输出。

        cd myactorsvcTestClient
        watch -n 1 ./testclient.sh

2. 在 Service Fabric Explorer 中，找到托管执行组件服务主副本的节点。 在以下屏幕截图中，该节点是节点 3。
   
    ![在 Service Fabric Explorer 中查找主副本][sfx-primary]
3. 单击上一步找到的节点，然后在“操作”菜单中选择“停用(重启)”。 此操作在本地群集中重新启动一个节点，从而强制故障转移到在另一个节点上运行的一个辅助副本。 在执行此操作时，请注意来自测试客户端的输出，并注意虽然发生故障转移，但是计数器仍将继续递增。

## <a name="adding-more-services-to-an-existing-application"></a>将更多服务添加到现有应用程序

若要将另一个服务添加到使用 `yo`创建的应用程序，请执行以下步骤： 
1. 将目录更改为现有应用程序的根目录。  例如 `cd ~/YeomanSamples/MyApplication`（如果 `MyApplication` 是 Yeoman 创建的应用程序）。
2. 运行 `yo azuresfcsharp:AddService`

## <a name="next-steps"></a>后续步骤
* [了解有关 Reliable Actors 的详细信息](/documentation/articles/service-fabric-reliable-actors-introduction/)
* [使用 Azure CLI 与 Service Fabric 群集交互](/documentation/articles/service-fabric-azure-cli/)
* 了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-csharp/yeoman-csharp.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-csharp/sfx-primary.png
<!--Update_Description:wording update;add anchors to sub titles-->