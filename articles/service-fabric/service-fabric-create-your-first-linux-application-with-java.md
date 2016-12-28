<properties
   pageTitle="使用 Java 在 Linux 上创建第一个 Service Fabric 应用程序 | Azure"
   description="使用 Java 创建并部署 Service Fabric 应用程序"
   services="service-fabric"
   documentationCenter="java"
   authors="seanmck"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="java"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="10/04/2016"
   wacn.date="11/28/2016"
   ms.author="seanmck"/>  



# 创建第一个 Azure Service Fabric 应用程序

> [AZURE.SELECTOR]
- [C# - Windows](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)
- [Java - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
- [C# - Linux](/documentation/articles/service-fabric-create-your-first-linux-application-with-csharp/)

Service Fabric 提供用于在 Linux 上构建服务的 .NET Core 和 Java SDK。本教程探讨如何创建适用于 Linux 的应用程序以及使用 Java 构建服务。

## 先决条件

开始之前，请确保已[设置 Linux 开发环境](/documentation/articles/service-fabric-get-started-linux/)。如果使用的是 Mac OS X，可以[使用 Vagrant 在虚拟机中设置 Linux 一体式环境](/documentation/articles/service-fabric-get-started-mac/)。

## 创建应用程序

Service Fabric 应用程序可以包含一个或多个服务，每个服务都在提供应用程序功能时具有特定角色。适用于 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 生成器，使用它可以轻松创建第一个服务并在以后添加更多服务。让我们使用 Yeoman 创建包含单个服务的新应用程序。

1. 在终端中键入 **yo azuresfjava**。

2. 为应用程序命名。

3. 选择第一个服务的类型并为其命名。本教程将选择 Reliable Actor 服务。

  ![适用于 Java 的 Service Fabric Yeoman 生成器][sf-yeoman]  


>[AZURE.NOTE] 有关选项的详细信息，请参阅 [Service Fabric 编程模型概述](/documentation/articles/service-fabric-choose-framework/)。

## 构建应用程序

Service Fabric Yeoman 模板包含 [Gradle](https://gradle.org/) 的构建脚本，可用于从终端构建应用程序。

  ```bash
  cd myapp
  gradle
  ```

## 部署应用程序

构建应用程序后，可以使用 Azure CLI 将它部署到本地群集。

1. 连接到本地 Service Fabric 群集。

    ```bash
    azure servicefabric cluster connect
    ```

2. 使用模板中提供的安装脚本，将应用程序包复制到群集的映像存储、注册应用程序类型，然后创建应用程序的实例。

    ```bash
    ./install.sh
    ```

3. 打开浏览器并导航到 Service Fabric Explorer (http://localhost:19080/Explorer)（如果在 Mac OS X 上使用 Vagrant，请将 localhost 替换为 VM 的专用 IP）。

4. 展开“应用程序”节点。可以看到，应用程序类型现在有一个条目，另一个条目对应于该类型的第一个实例。

## 启动测试客户端并执行故障转移

执行组件项目没有任何属于自己的项。它们需要其他服务或客户端发送消息给它们。执行组件模板包含简单的测试脚本，可用于与执行组件服务交互。

1. 使用监视实用工具运行脚本，查看执行组件服务的输出。

    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```

2. 在 Service Fabric Explorer 中，找到托管执行组件服务主副本的节点。在以下屏幕截图中，该节点是节点 3。

    ![在 Service Fabric Explorer 中查找主副本][sfx-primary]  


3. 单击在上一步骤中找到的节点，然后从“操作”菜单中选择“禁用(重新启动)”。随后将重新启动本地群集中五个节点中的一个，强制故障转移到另一个节点上运行的某个辅助副本。执行此操作时，请注意测试客户端的输出，可以看到，尽管是故障转移，计数器仍会继续递增。

## 使用 Eclipse Neon 插件构建和部署应用程序

如果已安装适用于 Eclipse Neon 的 Service Fabric 插件，可以使用它来创建、构建和部署使用 Java 构建的 Service Fabric 应用程序。安装 Eclipse 时，请选择“面向 Java 开发人员的 Eclipse IDE”。

### 创建应用程序

可通过 Eclipse 扩展获取 Service Fabric 插件。

1. 在 Eclipse 中，选择“文件”>“其他”>“Service Fabric”。此时将显示一组选项，包括“执行组件”和“容器”。

    ![Eclipse 中的 Service Fabric 模板][sf-eclipse-templates]  


2. 在本例中，请选择“无状态服务”。

3. 系统将要求确认使用 Service Fabric，它可以优化与 Service Fabric 项目配合使用的 Eclipse。选择“是”。

### 部署应用程序

Service Fabric 模板包含一组用于构建和部署应用程序的 Gradle 任务，可以通过 Eclipse 触发这些任务。

1. 选择“运行”>“运行配置”。

2. 展开“Gradle 项目”并选择“ServiceFabricDeployer”。

3. 单击“运行”。

片刻时间内即可构建和部署应用程序。可以从 Service Fabric Explorer 监视其状态。

## 后续步骤

- [详细了解 Reliable Actors](/documentation/articles/service-fabric-reliable-actors-introduction/)
- [使用 Azure CLI 来与 Service Fabric 群集交互](/documentation/articles/service-fabric-azure-cli/)

<!-- Images -->

[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png

<!---HONumber=Mooncake_1121_2016-->