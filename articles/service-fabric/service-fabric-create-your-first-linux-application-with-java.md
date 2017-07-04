<properties
    pageTitle="使用 Java 在 Linux 上创建第一个 Service Fabric 应用程序 | Azure"
    description="使用 Java 创建并部署 Service Fabric 应用程序"
    services="service-fabric"
    documentationcenter="java"
    author="seanmck"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="02b51f11-5d78-4c54-bb68-8e128677783e"
    ms.service="service-fabric"
    ms.devlang="java"
    ms.topic="hero-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="seanmck" />  


# 创建你的第一个 Azure Service Fabric 应用程序

> [AZURE.SELECTOR]
- [C# - Windows](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)
- [Java - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
- [C# - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-csharp/)

Service Fabric 提供用于在 Linux 上构建服务的 .NET Core 和 Java SDK。在本教程中，我们创建适用于 Linux 的应用程序并使用 Java 构建服务。

> [AZURE.NOTE]
仅在 Linux 预览版中支持使用 Java 这种一流的内置编程语言（Windows 支持已在计划中）。但是，不管是在 Windows 上还是 Linux 上，任何应用程序（包括 Java 应用程序）都可作为来宾可执行文件运行，或者在容器中运行。有关详细信息，请参阅[将现有的可执行文件部署到 Azure Service Fabric](/documentation/articles/service-fabric-deploy-existing-app/) 。
>

## 视频教程

以下 Microsoft 虚拟大学视频逐步讲解了在 Linux 上创建 Java 应用的过程：
<center><a target="\_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=DOX8K86yC_206218965">  
<img src="./media/service-fabric-create-your-first-linux-application-with-java/LinuxVid.png" WIDTH="360" HEIGHT="244">  
</a></center>


## 先决条件

开始之前，请确保已[设置 Linux 开发环境](/documentation/articles/service-fabric-get-started-linux/)。如果使用的是 Mac OS X，可以[使用 Vagrant 在虚拟机中设置 Linux 一体式环境](/documentation/articles/service-fabric-get-started-mac/)。

## 创建应用程序
Service Fabric 应用程序可以包含一个或多个服务，每个服务都在提供应用程序功能时具有特定角色。适用于 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 生成器，使用它可以轻松创建第一个服务并在以后添加更多服务。让我们使用 Yeoman 创建包含单个服务的应用程序。

1. 在终端中，键入 ``yo azuresfjava``。

2. 为应用程序命名。

3. 选择第一个服务的类型并为其命名。本教程选择了 Reliable Actor 服务。

  ![适用于 Java 的 Service Fabric Yeoman 生成器][sf-yeoman]  


>[AZURE.NOTE] 有关选项的详细信息，请参阅 [Service Fabric 编程模型概述](/documentation/articles/service-fabric-choose-framework/)。

## 构建应用程序
Service Fabric Yeoman 模板包含 [Gradle](https://gradle.org/) 的构建脚本，可用于从终端构建应用程序。


    cd myapp
    gradle


## 部署应用程序
构建应用程序后，可以使用 Azure CLI 将它部署到本地群集。

1. 连接到本地 Service Fabric 群集。


        azure servicefabric cluster connect


2. 使用模板中提供的安装脚本，将应用程序包复制到群集的映像存储、注册应用程序类型，然后创建应用程序的实例。


        ./install.sh


3. 打开浏览器并导航到 Service Fabric Explorer \(http://localhost:19080/Explorer\)（如果在 Mac OS X 上使用 Vagrant，请将 localhost 替换为 VM 的专用 IP）。

4. 展开“应用程序”节点。可以看到，应用程序类型现在有一个条目，另一个条目对应于该类型的第一个实例。

## 启动测试客户端并执行故障转移
执行组件项目没有任何属于自己的项。它们需要其他服务或客户端发送消息给它们。执行组件模板包含简单的测试脚本，可用于与执行组件服务交互。

1. 使用监视实用工具运行脚本，查看执行组件服务的输出。


        cd myactorsvcTestClient
        watch -n 1 ./testclient.sh


2. 在 Service Fabric Explorer 中，找到托管执行组件服务主副本的节点。在以下屏幕截图中，该节点是节点 3。

    ![在 Service Fabric Explorer 中查找主副本][sfx-primary]  


3. 单击在上一步骤中找到的节点，然后从“操作”菜单中选择“禁用\(重新启动\)”。此操作重新启动本地群集中五个节点中的一个，强制故障转移到另一个节点上运行的某个辅助副本。执行此操作时，请注意测试客户端的输出，可以看到，尽管是故障转移，计数器仍继续递增。

## 使用 Eclipse Neon 插件构建和部署应用程序

如果已安装适用于 Eclipse Neon 的 [Service Fabric 插件](/documentation/articles/service-fabric-get-started-linux/#install-the-java-sdk-and-eclipse-neon-plugin-optional)，可以使用它来创建、构建和部署使用 Java 构建的 Service Fabric 应用程序。安装 Eclipse 时，请选择“面向 Java 开发人员的 Eclipse IDE”。

### 创建应用程序

可通过 Eclipse 扩展获取 Service Fabric 插件。

1. 在 Eclipse 中，选择“文件”\>“其他”\>“Service Fabric”。此时将显示一组选项，包括“执行组件”和“容器”。

    ![Eclipse 中的 Service Fabric 模板][sf-eclipse-templates]  


2. 在本例中，请选择“无状态服务”。

3. 系统将要求确认使用 Service Fabric，它可以优化与 Service Fabric 项目配合使用的 Eclipse。选择“是”。

### 部署应用程序
Service Fabric 模板包含一组用于构建和部署应用程序的 Gradle 任务，可以通过 Eclipse 触发这些任务。

1. 选择“运行”\>“运行配置”。

2. 指定“本地”或“云”。默认安装为“本地”。若要部署到远程群集，请选择“云”。

3. 根据需要编辑 `local.json` 或 `cloud.json`，确保在发布配置文件中填充适当的信息。

4. 单击“运行”。

片刻时间内即可构建和部署应用。可以从 Service Fabric Explorer 监视其状态。

## 将更多服务添加到现有应用程序

若要将另一个服务添加到使用 `yo` 创建的应用程序，请执行以下步骤：
1. 将目录更改为现有应用程序的根目录。例如 `cd ~/YeomanSamples/MyApplication`（如果 `MyApplication` 是 Yeoman 创建的应用程序）。
2. 运行 `yo azuresfjava:AddService`


## 后续步骤
* [详细了解 Reliable Actors](/documentation/articles/service-fabric-reliable-actors-introduction/)
* [使用 Azure CLI 来与 Service Fabric 群集交互](/documentation/articles/service-fabric-azure-cli/)
* [排查部署问题](/documentation/articles/service-fabric-azure-cli/#troubleshooting)
* 了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)

<!-- Images -->

[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png

<!---HONumber=Mooncake_0213_2017-->