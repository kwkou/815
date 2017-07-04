<properties
    pageTitle="在 Mac OS X 上设置开发环境 | Azure"
    description="安装运行时、SDK 和工具并创建本地开发群集。 完成此设置后，可以在 Mac OS X 上构建应用程序。"
    services="service-fabric"
    documentationcenter="java"
    author="sayantancs"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="bf84458f-4b87-4de1-9844-19909e368deb"
    ms.service="service-fabric"
    ms.devlang="java"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="04/06/2017"
    wacn.date="05/15/2017"
    ms.author="saysa"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="a0c43d26f52ff4d1012f4cebe232de5369918120"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="set-up-your-development-environment-on-mac-os-x"></a>在 Mac OS X 上设置开发环境
> [AZURE.SELECTOR]
- [Windows](/documentation/articles/service-fabric-get-started/)
- [Linux](/documentation/articles/service-fabric-get-started-linux/)
- [OSX](/documentation/articles/service-fabric-get-started-mac/)

你可以使用 Mac OS X 生成在 Linux 群集上运行的 Service Fabric 应用程序。本文介绍了如何设置 Mac 以用于开发。

## <a name="prerequisites"></a>先决条件
Service Fabric 不是在 OS X 上以本机方式运行。为了运行本地 Service Fabric 群集，我们提供了使用 Vagrant 和 VirtualBox 的预配置 Ubuntu 虚拟机。 准备事项：

* [Vagrant（v1.8.4 或更高版本）](http://www.vagrantup.com/downloads.html)
* [VirtualBox](http://www.virtualbox.org/wiki/Downloads)

>[AZURE.NOTE]
> 需使用 Vagrant 和 VirtualBox 的相互支持的版本。 在不受支持的 VirtualBox 版本上，Vagrant 的行为可能不稳定。
>

## <a name="create-the-local-vm"></a>创建本地 VM
若要创建包含 5 节点 Service Fabric 群集的本地 VM，请执行以下步骤：

1. 克隆 `Vagrantfile` 存储库

    	git clone https://github.com/azure/service-fabric-linux-vagrant-onebox.git

    此步骤获取包含 VM 配置和 VM 下载位置的文件 `Vagrantfile`。


2. 导航到本地克隆存储库

    	cd service-fabric-linux-vagrant-onebox

3. （可选）修改默认的 VM 设置

    默认情况下，本地 VM 的配置如下：

   * 3 GB 的分配内存
   * 在 IP 192.168.50.50 配置专用主机网络，以便能够从 Mac 主机传递流量

     可以更改这些设置或向 `Vagrantfile` 中的 VM 添加其他配置。 请参阅 [Vagrant 文档](http://www.vagrantup.com/docs)，获得配置选项的完整列表。
4. 创建 VM


    	vagrant up

    此步骤下载预配置的 VM 映像、在本地将它启动，然后在其中设置一个本地 Service Fabric 群集。 此过程预计需要几分钟时间。 如果成功完成安装，输出中会出现一条消息，指示群集正在启动。

    ![群集安装将在 VM 设置后启动][cluster-setup-script]

    >[AZURE.TIP]
    > 如果 VM 下载耗时过长，可以使用 wget 或 curl 下载，也可通过浏览器下载，只需导航到文件 `Vagrantfile` 中通过 **config.vm.box_url** 指定的链接即可。 将其下载到本地以后，请编辑 `Vagrantfile`，以便指向下载了映射的本地路径。 例如，如果已将映像下载到 /home/users/test/azureservicefabric.tp8.box，则请将 **config.vm.box_url** 设置为该路径。
    >

5. 导航到 http://192.168.50.50:19080/Explorer 的 Service Fabric Explorer（假设保留了默认的专用网络 IP 地址），测试是否已正确安装群集。

    ![从主机 Mac 查看的 Service Fabric Explorer][sfx-mac]

## <a name="install-the-service-fabric-plugin-for-eclipse-neon"></a>为 Eclipse Neon 安装 Service Fabric 插件

Service Fabric 为**适用于 Java IDE 的 Eclipse Neon** 提供了一个插件，可简化创建、生成和部署 Java 服务的过程。 可以按照通用[文档](/documentation/articles/service-fabric-get-started-eclipse/#install-or-update-the-service-fabric-plug-in-in-eclipse-neon)中提及的安装步骤，安装或更新 Service Fabric Eclipse 插件。

## <a name="using-service-fabric-eclipse-plugin-on-mac"></a>在 Mac 上使用 Service Fabric Eclipse 插件

确保已完成 [Service Fabric Eclipse 插件文档](/documentation/articles/service-fabric-get-started-eclipse/)中提及的步骤。 在 Mac 主机上使用 vagrant-guest 容器创建、生成和部署 Service Fabric Java 应用程序的步骤大部分与通用文档所述相同，以下项目除外：

* 由于 Service Fabric Java 应用程序需要 Service Fabric 库，因此需在共享路径中创建 Eclipse 项目。 默认情况下，在 ``Vagrantfile`` 所在主机上的路径中的内容与来宾计算机上的 ``/vagrant`` 路径共享。
* 如果在路径中有 ``Vagrantfile``（例如 ``~/home/john/allprojects/``），则需在位置 ``~/home/john/allprojects/MyActor`` 中创建 Service Fabric 项目 ``MyActor``，而 Eclipse 工作区的路径将是 ``~/home/john/allprojects``。

## <a name="next-steps"></a>后续步骤
<!-- Links -->
* [使用 Yeoman 在 Linux 上创建和部署第一个 Service Fabric Java 应用程序](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
* [使用适用于 Eclipse 的 Service Fabric 插件在 Linux 上创建和部署第一个 Service Fabric Java 应用程序](/documentation/articles/service-fabric-get-started-eclipse/)
* [在 Azure 门户中创建 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)
* [使用 Azure Resource Manager 创建 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-arm/)
* [了解 Service Fabric 应用程序模型](/documentation/articles/service-fabric-application-model/)

<!-- Images -->
[cluster-setup-script]: ./media/service-fabric-get-started-mac/cluster-setup-mac.png
[sfx-mac]: ./media/service-fabric-get-started-mac/sfx-mac.png
[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-mac/sf-eclipse-plugin-install.png
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
<!--Update_Description: simplify content structure;add references to linux and java-->