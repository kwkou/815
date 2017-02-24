<properties
    pageTitle="在 Linux 上设置开发环境 | Azure"
    description="在 Linux 上安装运行时和 SDK 并创建本地开发群集。完成此设置后，你就可以开始生成应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="seanmck"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="d552c8cd-67d1-45e8-91dc-871853f44fc6"
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="seanmck" />  


# 在 Linux 上准备开发环境


> [AZURE.SELECTOR]
-[ Windows](/documentation/articles/service-fabric-get-started/)
- [Linux](/documentation/articles/service-fabric-get-started-linux/)
- [OSX](/documentation/articles/service-fabric-get-started-mac/)

 若要在 Linux 开发计算机上部署和运行 [Azure Service Fabric 应用程序](/documentation/articles/service-fabric-application-model/)，请安装运行时和通用 SDK。还可以安装适用于 Java 和 .NET Core 的可选 SDK。

## 先决条件

### 支持的操作系统版本
支持使用以下操作系统版本进行开发：

* Ubuntu 16.04（“Xenial Xerus”）

## 更新 apt 源

若要通过 apt-get 安装 SDK 和关联的运行时包，必须先更新 apt 源。

1. 打开终端。
2. 将 Service Fabric 存储库添加到源列表。


        sudo sh -c 'echo "deb [arch=amd64] http://apt-mo.trafficmanager.net/repos/servicefabric/ trusty main" > /etc/apt/sources.list.d/servicefabric.list'


3. 将新的 GPG 密钥添加到 apt keyring。


        sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893


4. 根据新添加的存储库刷新包列表。


        sudo apt-get update


## 安装和设置 SDK

更新源后，可以安装 SDK。

1. 安装 Service Fabric SDK 包。系统会请求用户确认安装并同意许可协议。


        sudo apt-get install servicefabricsdkcommon


2. 运行 SDK 安装脚本。


        sudo /opt/microsoft/sdk/servicefabric/common/sdkcommonsetup.sh


## 设置 Azure 跨平台 CLI

[Azure 跨平台 CLI][azure-xplat-cli-github] 包含用来与 Service Fabric 实体（包括群集和应用程序）交互的命令。它基于 Node.js，因此，请[务必先安装 Node][install-node]，然后继续遵照下面的说明操作。

1. 将 github 存储库克隆到开发计算机。


        git clone https://github.com/Azure/azure-xplat-cli.git


2. 切换到克隆的存储库，然后使用 Node Package Manager \(npm\) 安装 CLI 的依赖项。


        cd azure-xplat-cli
        npm install


3. 创建从所复制存储库的 bin/azure 文件夹到 /usr/bin/azure 的符号链接，以便将它添加到路径并从任何目录使用命令。


        sudo ln -s $(pwd)/bin/azure /usr/bin/azure


4. 最后，启用自动补全 Service Fabric 命令。


        azure --completion >> ~/azure.completion.sh
        echo 'source ~/azure.completion.sh' >> ~/.bash_profile
        source ~/azure.completion.sh


> [AZURE.NOTE]
Service Fabric 命令目前在 Azure CLI 2.0 中不可用。

## 设置本地群集

如果所有组件都已成功安装，应该能够启动本地群集。

1. 运行群集安装脚本。


        sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh


2. 打开 Web 浏览器并导航到 http://localhost:19080/Explorer。如果群集已启动，应会显示 Service Fabric Explorer 仪表板。

    ![Linux 上的 Service Fabric Explorer][sfx-linux]  


现在，可以根据来宾容器或来宾可执行文件，部署预先构建的 Service Fabric 应用程序包或新包。若要使用 Java 或 .NET Core SDK 构建新服务，请按后续部分提供的可选设置步骤操作。


> [AZURE.NOTE]
Linux 不支持独立群集 - 预览版仅支持单机群集和 Azure Linux 多计算机群集。
>
>

##<a name="install-the-java-sdk-and-eclipse-neon-plugin-optional"></a> 安装 Java SDK 和 Eclipse Neon 插件（可选）

Java SDK 提供所需的库和模板用于通过 Java 构建 Service Fabric 服务。

1. 安装 Java SDK 包。


        sudo apt-get install servicefabricsdkjava


2. 运行 SDK 安装脚本。


        sudo /opt/microsoft/sdk/servicefabric/java/sdkjavasetup.sh


可以从 Eclipse Neon IDE 安装适用于 Service Fabric 的 Eclipse 插件。

1. 在 Eclipse 中，请确保已安装 Buildship 1.0.17 或更高版本。可以选择“帮助”\>“安装详细信息”检查已安装的组件版本。可以使用[此处][buildship-update]的说明更新 Buildship。

2. 若要安装 Service Fabric 插件，请选择“帮助”\>“安装新软件...”

3. 在“使用”文本框中，输入：http://dl.windowsazure.com/eclipse/servicefabric

4. 单击“添加”。

    ![Eclipse 插件][sf-eclipse-plugin]  


5. 选择 Service Fabric 插件，然后单击“下一步”。

6. 继续安装，并接受最终用户许可协议。

## 安装 .NET Core SDK（可选）

.NET Core SDK 提供所需的库和模板用于通过跨平台 .NET Core 构建 Service Fabric 服务。

1. 安装 .NET Core SDK 包。


        sudo apt-get install servicefabricsdkcsharp


2. 运行 SDK 安装脚本。


        sudo /opt/microsoft/sdk/servicefabric/csharp/sdkcsharpsetup.sh


## 更新 SDK 和运行时

若要更新到最新版的 SDK 和运行时，请运行以下步骤（从列表中删除不需更新或安装的 SDK）：


	   sudo apt-get update
	   sudo apt-get install servicefabric, servicefabricsdkcommon, servicefabricsdkcsharp, servicefabricsdkjava


若要更新 CLI，请导航到克隆 CLI 的目录，然后运行 `git pull` 进行更新。

## 后续步骤

- [在 Linux 上创建第一个 Java 应用程序](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
- [在 Linux 上创建第一个 CSharp 应用程序](/documentation/articles/service-fabric-create-your-first-linux-application-with-csharp/)
- [在 OSX 上准备开发环境](/documentation/articles/service-fabric-get-started-mac/)
- [使用 Azure CLI 管理 Service Fabric 应用程序](/documentation/articles/service-fabric-azure-cli/)

<!-- Links -->


[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->


[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update; add azure cli 2.0 reference-->